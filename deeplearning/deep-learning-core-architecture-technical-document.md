# deep-learning-core — Architecture & Implementation Technical Document

> **Source repository:** [withlionbuddha/deep-learning-core](https://github.com/withlionbuddha/deep-learning-core/tree/ground)  
> **Analyzed branch:** `ground` · **Analysis date:** 2026-10-28  
> **Method:** Source-Code-Driven Static Analysis  
> **Classification:** Implemented / Recommended / Runtime Verification Required

## 1. Introduction — 분석 목적과 경계

**핵심 포인트:** 특정 모델 프로젝트에서 학습·평가·체크포인트·진단 기능을 분리하여 재사용하는 PyTorch 라이브러리의 실제 구현 구조를 기술한다.

프로젝트의 `README.md`는 모델 고유 기능(Tokenizer, Vocabulary, Masking, Generation)을 모델 프로젝트에 남기고, 공통 Training/Evaluation/Diagnostics를 `deep-learning-core`에 배치하는 것을 설계 원칙으로 제시한다. 이 문서는 해당 의도를 구현 파일과 대응시키며, 코드의 존재와 실제 실행 검증을 구분한다.

## 2. System Overview — 모듈 책임

| Package | 주요 책임 | Source evidence | Status |
|---|---|---|---|
| `modules/model` | ModelBuilder, SequentialModel 등 모델 래핑 | `modules/model/ModelBuilder.py` | Implemented |
| `modules/training/runners` | Batch 단위 학습/검증 및 Epoch 실행 | `BatchRunner.py`, `TrainEpochRunner.py`, `ValidEpochRunner.py` | Implemented |
| `modules/training/trainer` | Train-only, validation 포함, validation-only 실행 조합 | `TrainWithValidationTrainer.py` 등 | Implemented |
| `modules/training` | Optimizer/Criterion 구성 | `CompileConfigs.py` | Implemented |
| `modules/checkpoint` | Model/Optimizer 상태 저장 및 복원 | `CheckpointSaver.py`, `CheckpointRestorer.py` | Implemented |
| `modules/storage` | History/Metadata 저장·복원 | `HistorySaver.py`, `MetadataConverter.py` 등 | Implemented |
| `modules/metrics` | 정확도 계산 | `AccuracyCalculator.py`, `TransformerAccuracy.py` | Implemented |
| `modules/device` | Forward-hook 실행 기록 | `DeviceProfiler.py` | Implemented |
| `modules/diagnostics` | 모델 구조·Weight·뉴런·Transformer 분석 | `analyzer/`, `inspectors/`, `resolvers/` | Implemented |
| `modules/callbacks` | Early stopping | `EarlyStopping.py` | Implemented |

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

**설계 결정:** 공통 학습 실행은 모델의 Forward 호출과 Loss/Accuracy 계산을 조합하고, 도메인별 토큰화나 생성 규칙을 직접 구현하지 않는다. 단, 완전한 범용 `nn.Module` 호환을 보장하지는 않는다. `BatchRunner._move_batch_to_device()`는 `self.model.get_device()`를 호출하므로, 단순한 `nn.Module`에는 해당 메서드가 없을 수 있다. **추가 어댑터 또는 Model Wrapper가 필요한 계약**이다.

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

**Limitation:** 저장 Payload에 RNG 상태나 DataLoader/Sampler 진행 상태가 포함된다는 근거는 없다. 따라서 이 구현만으로 중단 시점과 완전히 동일한 학습 재현성을 보장한다고 표현할 수 없다.

## 6. Model Analysis vs Runtime Profiling

### 6.1. ModelAnalyzer — 구조적 분석

`modules/diagnostics/analyzer/ModelAnalyzer.py`는 다음 값을 산출한다.

- 전체 / 학습 가능 / 학습 불가능 Parameter Count
- Parameter Memory = Σ(`numel × element_size`)
- 현재 존재하는 Gradient Memory (`grad is not None`만 합산)
- Root Module을 제외한 Submodule Count 및 Type별 Count
- Epoch·데이터 수·Batch Size·경과 시간에 기반한 평균 Iterations/s 계산

**주의:** Gradient Memory는 현재 할당된 Gradient Tensor의 바이트 수이지 학습 과정의 Peak Memory가 아니다.

### 6.2. DeviceProfiler — Forward Hook 기반 실행 관찰

`modules/device/DeviceProfiler.py`는 `named_modules()`를 순회하며 Forward Pre/Post Hook을 등록한다. 실행 전후의 시각, 입력/출력 Shape, Allocated/Reserved Memory를 기록한다. CUDA/XPU에서는 사용 가능한 경우 `synchronize()`로 비동기 실행을 동기화한다.

**제약:** CPU의 메모리 함수는 0.0을 반환하므로 CPU RAM 프로파일러로 해석할 수 없다. Hook 기반 시간에는 동기화·Python Hook 오버헤드가 포함될 수 있다. 각 Module 기록은 중첩 호출의 Inclusive Time 성격을 가지므로 단순 합계가 모델 전체 지연 시간과 일치하지 않을 수 있다.

## 7. Technical Requirements

- Python 환경에서 PyTorch 설치 필요. CPU/CUDA/XPU 빌드는 소비 프로젝트가 선택한다.
- 개발·테스트 의존성: Repository `requirements.txt`.
- Editable installation: `python -m pip install -e .`.
- 모델 프로젝트에서의 설치: `python -m pip install -e ../deep-learning-core`.
- 테스트 실행: `python -m pytest tests/units -v`.

**Source:** `README.md` 및 `modules/` 구현.

## 8. Technical Findings & Limitations

| Finding | Classification | Evidence / Reason |
|---|---|---|
| 모델 고유 로직과 공통 학습 로직 분리 | Implemented | `README.md`, `modules/training/` |
| Train/Validation 별도 Gradient 동작 | Implemented | `BatchRunner.py` |
| 중첩 입력 Tensor의 Device 이동 | Implemented | `BatchRunner._move_to_device()` |
| 순수 `nn.Module`의 무조건적 Plug-and-Play 지원 | Runtime Verification Required | `model.get_device()` 의존 |
| CPU RAM 실측 메모리 기록 | Not Implemented in DeviceProfiler | CPU에서 메모리 조회 결과 0.0 |
| Full deterministic resume | Runtime Verification Required | RNG/Sampler 상태 저장 근거 미확인 |
| 모든 Accelerator에서 성능 동일 | Runtime Verification Required | Device/Driver/Kernel 별 검증 필요 |
| End-to-end 서비스 환경 성능 | Runtime Verification Required | 소스 정적 분석만 수행 |

## 9. Improvement Recommendations

1. **Model Contract 명문화:** `get_device()` 요구 또는 `next(model.parameters()).device` 기반 대안의 지원 범위를 명시한다.
2. **Checkpoint 재현성:** RNG·Sampler·Scheduler 상태의 선택적 저장과 Restore 통합 테스트를 추가한다.
3. **Profiler 측정 정의:** Inclusive/Exclusive Time, Synchronization Overhead, CPU Memory 미지원 범위를 문서화한다.
4. **Integration Test:** 실제 모델 프로젝트에서 Train → Checkpoint → Restore → Validate 시나리오를 검증한다.
5. **Public API 안정화:** README의 구식 Analyzer Import 예시를 현재 경로인 `modules.diagnostics.analyzer.ModelAnalyzer`로 갱신한다.

## 10. Architecture Summary

이 프로젝트의 확인된 강점은 모델별 프로젝트에 중복되기 쉬운 학습 실행·저장·진단 기능을 모듈로 분리했다는 점이다. 다만 **소스 구현의 존재, 단위 테스트 통과, 실제 Accelerator 성능, 운영환경 안정성은 서로 다른 증거 수준**이다. 문서는 현재 확인한 Source-Level Evidence만으로 구현 범위를 판정한다.

## Source Traceability

- [Source README](https://github.com/withlionbuddha/deep-learning-core/blob/ground/README.md)
- [BatchRunner](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/training/runners/BatchRunner.py)
- [TrainWithValidationTrainer](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/training/trainer/TrainWithValidationTrainer.py)
- [CheckpointSaver](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/checkpoint/CheckpointSaver.py)
- [ModelAnalyzer](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/analyzer/ModelAnalyzer.py)
- [DeviceProfiler](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/device/DeviceProfiler.py)

> **Verification note:** 본 문서는 GitHub `ground`의 소스 코드를 정적으로 검토한 결과다. 이 문서 작성 과정에서 pytest, GPU/XPU 실행, End-to-End Benchmark는 수행하지 않았다.
