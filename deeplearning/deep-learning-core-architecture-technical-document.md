# deep-learning-core — Architecture & Implementation Technical Document

> **Source repository:** [withlionbuddha/deep-learning-core](https://github.com/withlionbuddha/deep-learning-core/tree/ground)  
> **Analyzed branch:** `ground` · **Analysis date:** 2026-10-28  
> **Method:** Source-Code-Driven Static Analysis  
> **Scope:** GitHub에 커밋된 구현 코드만 기술

## 1. Introduction — 분석 목적과 범위

**핵심 포인트:** 특정 모델 프로젝트에서 학습·평가·체크포인트·진단 기능을 분리하여 재사용하는 PyTorch 라이브러리의 실제 구현 구조를 기술한다.

프로젝트의 `README.md`는 모델 고유 기능(Tokenizer, Vocabulary, Masking, Generation)을 모델 프로젝트에 남기고, 공통 Training/Evaluation/Diagnostics를 `deep-learning-core`에 배치하는 것을 설계 원칙으로 제시한다. 이 문서는 해당 의도를 구현 파일과 대응시키며, 코드의 존재와 실제 실행 검증을 구분한다.

## 2. System Overview — 모듈 책임

| Package | 주요 책임 | Source evidence |
|---|---|---|
| `modules/model` | ModelBuilder, SequentialModel 등 모델 래핑 | `modules/model/ModelBuilder.py` |
| `modules/training/runners` | Batch 단위 학습/검증 및 Epoch 실행 | `BatchRunner.py`, `TrainEpochRunner.py`, `ValidEpochRunner.py` |
| `modules/training/trainer` | Train-only, validation 포함, validation-only 실행 조합 | `TrainWithValidationTrainer.py` 등 |
| `modules/training` | Optimizer/Criterion 구성 | `CompileConfigs.py` |
| `modules/checkpoint` | Model/Optimizer 상태 저장 및 복원 | `CheckpointSaver.py`, `CheckpointRestorer.py` |
| `modules/storage` | History/Metadata 저장·복원 | `HistorySaver.py`, `MetadataConverter.py` 등 |
| `modules/metrics` | 정확도 계산 | `AccuracyCalculator.py`, `TransformerAccuracy.py` |
| `modules/device` | Forward-hook 실행 기록 | `DeviceProfiler.py` |
| `modules/diagnostics` | 모델 구조·Weight·뉴런·Transformer 분석 | `analyzer/`, `inspectors/`, `resolvers/` |
| `modules/callbacks` | Early stopping | `EarlyStopping.py` |

## 3. Architecture Overview — 의존성 경계

```text
Model-specific repository
  ├─ architecture / forward / inference
  ├─ tokenizer / vocabulary / masking
  └─ data-specific preprocessing
                │ imports
                ▼
deep-learning-core
  ├─ ModelBuilder / model wrappers
  ├─ Trainer → EpochRunner → BatchRunner
  │                             ├─ LossCalculator
  │                             └─ AccuracyCalculator
  ├─ Checkpoint / Storage / Callbacks
  └─ Analyzer / Inspector / DeviceProfiler
```

**구현된 호출 계약:** `BatchRunner`는 `self.model.get_device()`로 Device를 조회하고, `model(x_batch)`를 호출하여 Forward를 수행한다.

## 4. Training Processing Flow — BatchRunner

**핵심 포인트:** 학습 단계와 검증 단계에서 Gradient 업데이트 경로를 분리한다.

```text
Train batch:
  (inputs, targets) → recursive device transfer
  → optimizer.zero_grad()
  → model(inputs) → loss_calculator(predictions, targets)
  → loss.backward() → optimizer.step()
  → accuracy_calculator(...) → {loss, correct, total, accuracy}

Validation batch:
  (inputs, targets) → recursive device transfer
  → torch.no_grad() → model(inputs) → loss / accuracy
  → {loss, correct, total, accuracy}
```

**Evidence:** `modules/training/runners/BatchRunner.py`.

`_move_to_device()`는 Tensor, dict, tuple, list를 재귀적으로 순회하며, 그 외 객체는 그대로 전달한다. 따라서 입력 컨테이너 이동은 지원하지만 임의의 커스텀 객체나 데이터 클래스에 대한 자동 이동은 보장하지 않는다.

`TrainWithValidationTrainer`는 Validation DataLoader 유무를 검사하고 `TrainValidEpochRunner(TrainEpochRunner, ValidEpochRunner)`를 생성한다. Optimizer는 `CompileConfigs.get_optimizer(model)`에서 필요 시 생성한다.

## 5. Checkpoint Architecture — Best-Loss Policy

**핵심 포인트:** Checkpoint 저장 여부는 현재 Loss가 이전 최솟값보다 작은지로 결정된다.

`CheckpointSaver.save(model, optimizer, epoch, current_loss)`는 `current_loss < best_loss`일 때만 저장한다. 저장 Payload는 다음을 포함한다.

- `epoch`, `loss`
- `model_class`, `model_config`, `model_state_dict`
- `optimizer_class`, `optimizer_config`, `optimizer_state_dict`

**Evidence:** `modules/checkpoint/CheckpointSaver.py`. 파일 저장은 `torch.save`를 사용한다. `model_config`는 모델이 `get_config()`를 제공할 때만 기록된다. Optimizer가 없으면 관련 항목은 `None`이다.



## 6. Model Analysis vs Runtime Profiling

### 6.1. ModelAnalyzer — 구조적 분석

`modules/diagnostics/analyzer/ModelAnalyzer.py`는 다음 값을 산출한다.

- 전체 / 학습 가능 / 학습 불가능 Parameter Count
- Parameter Memory = Σ(`numel × element_size`)
- 현재 존재하는 Gradient Memory (`grad is not None`만 합산)
- Root Module을 제외한 Submodule Count 및 Type별 Count
- Epoch·데이터 수·Batch Size·경과 시간에 기반한 평균 Iterations/s 계산

**구현된 계산:** `grad is not None`인 Tensor의 바이트 수를 합산한다.

### 6.2. DeviceProfiler — Forward Hook 기반 실행 관찰

`modules/device/DeviceProfiler.py`는 `named_modules()`를 순회하며 Forward Pre/Post Hook을 등록한다. 실행 전후의 시각, 입력/출력 Shape, Allocated/Reserved Memory를 기록한다. CUDA/XPU에서는 사용 가능한 경우 `synchronize()`로 비동기 실행을 동기화한다.

**구현된 반환값:** CPU 분기에서 메모리 조회 메서드는 `0.0`을 반환한다.

## 7. Installation & Execution Commands

- Python 환경에서 PyTorch 설치 필요. CPU/CUDA/XPU 빌드는 소비 프로젝트가 선택한다.
- 개발·테스트 의존성: Repository `requirements.txt`.
- Editable installation: `python -m pip install -e .`.
- 모델 프로젝트에서의 설치: `python -m pip install -e ../deep-learning-core`.
- 테스트 실행 명령(README): `python -m pytest tests`.

**Source:** `README.md` 및 `modules/` 구현.

## 8. Architecture Summary

`deep-learning-core`에는 Batch/Epoch 학습 실행, Checkpoint 저장, Model 구조 분석, Forward Hook 기반 기록, Weight/Neuron 분석을 수행하는 Python 클래스가 구현되어 있다. 각 기능의 세부 계산과 호출 관계는 위의 Source Evidence에 대응한다.

## Source Traceability

- [Source README](https://github.com/withlionbuddha/deep-learning-core/blob/ground/README.md)
- [BatchRunner](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/training/runners/BatchRunner.py)
- [TrainWithValidationTrainer](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/training/trainer/TrainWithValidationTrainer.py)
- [CheckpointSaver](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/checkpoint/CheckpointSaver.py)
- [ModelAnalyzer](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/analyzer/ModelAnalyzer.py)
- [DeviceProfiler](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/device/DeviceProfiler.py)

