# deep-learning-core — Model & Neuron Diagnostics Technical Document

> **Source:** [withlionbuddha/deep-learning-core — ground](https://github.com/withlionbuddha/deep-learning-core/tree/ground/modules/diagnostics)  
> **Analysis date:** 2026-10-28 · **Evidence level:** Source inspection (not runtime benchmark)

## 1. Purpose — 핵심 기술 포인트

이 문서는 `deep-learning-core`의 **모델 구조 탐색 → Inspector 등록 → 뉴런 Weight 선택 → 통계 계산 → 조건 기반 진단**을 구현 파일 단위로 추적한다. 모델의 최종 Accuracy가 아닌 **Weight Tensor와 뉴런별 연결 구조**를 분석하는 정적 진단 아키텍처가 대상이다.

## 2. Component Architecture

```text
nn.Sequential
  │
  ▼
SequentialLayerInspectorFactory.create(model)
  ├─ SequentialLayerResolver.resolve(model)
  │    └─ LayerDefinition + ModuleStage
  ├─ LayerInspectorFactory.create_for_module(primary_module)
  │    └─ OutputWeightInspector
  │         ├─ TensorNeuronSelector
  │         └─ WeightInspector
  │              ├─ TensorStatistics
  │              └─ WeightDiagnostic
  └─ LayerInspectorRegistry
       │
       ▼
SequentialLayerInspector
  ├─ inspect_neuron(layer_index, neuron_index)
  ├─ inspect_layer(layer_index)
  ├─ inspect_model()
  ├─ inspect_neuron_similarity(...)
  └─ get_problem_neurons()
```

**Evidence:** `factory/SequentailLayerFactory.py`, `factory/LayerInspectorFactory.py`, `resolvers/SequentialLayerResolver.py`, `inspectors/SequentialLayerInspector.py`.

## 3. Layer Resolution — Recursive Structure

`SequentialLayerResolver`는 입력이 `nn.Sequential`인지 검사한다. `_iter_modules()`는 중첩 `nn.Sequential`을 재귀적으로 순회하고 `parent.child` 형태의 Qualified Name을 생성한다.

- **Primary Layer:** `SequentialLayerDefinition.PRIMARY_LAYER_MODULES`에 해당하는 Module
- **ModuleStage:** Primary Layer 이후 다음 Primary Layer가 나오기 전까지의 Module
- **Pre-primary Stage:** 첫 Primary Layer 이전에 나타난 Module
- **LayerDefinition:** `layer_index`, `layer_name`, `layer_type`, `module`, `stages`

**한계:** 모든 PyTorch Graph 또는 임의의 `nn.Module`을 분석하는 Graph Tracer가 아니다. `nn.Sequential` 내부의 구조적 순회에 한정한다.

## 4. Inspector Registration — Supported Modules

`SequentialLayerInspectorFactory.create()`는 Resolver 결과를 받아 Layer별 `LayerInspectorFactory.create_for_module()`을 호출한다. 현재 Factory가 자동 등록하는 Inspector는 `OutputWeightInspector`이며, 지원 Module은 다음과 같다.

- `nn.Linear`
- `nn.Conv1d`, `nn.Conv2d`, `nn.Conv3d`
- `nn.Bilinear`

**구분:** `WeightDiagnostic.SUPPORT_MODULES`에는 Embedding/RNN/Normalization 등 더 많은 타입이 선언되어 있지만, 이것이 곧 Factory에서 해당 Module의 **뉴런별 Inspector 자동 등록을 지원한다는 뜻은 아니다.** 지원 범위는 `OutputWeightInspector.SUPPORTED_MODULES` 및 Factory 로직으로 판단해야 한다.

## 5. Tensor Statistics — Calculation Contract

`TensorStatistics.calculate(tensor)`는 비어 있지 않은 Real Tensor를 요구하고, `detach().to(float64)`로 변환한다.

| Metric | Calculation / Interpretation |
|---|---|
| `count` | 전체 원소 수 |
| `finite_count` | NaN/Inf가 아닌 원소 수 |
| `nan_count`, `inf_count` | 비유한값 개수 |
| `mean`, `std` | 유한값 평균 및 모집단 표준편차 (`unbiased=False`) |
| `min`, `max` | 유한값 범위 |
| `zero_ratio` | 유한값 중 0인 값의 비율 |
| `l1_norm`, `l2_norm` | 유한값 벡터 Norm |
| `mean_abs`, `max_abs` | 절댓값 크기 통계 |
| `positive_ratio`, `negative_ratio` | 유한값 중 부호 비율 |

**중요:** `zero_ratio`와 부호 비율의 분모는 `finite_count`에 해당한다. NaN/Inf가 섞인 경우 전체 Tensor 원소에 대한 비율로 오해해서는 안 된다. 모든 값이 비유한값이면 관련 통계는 `None`이다.

## 6. WeightDiagnostic — Decision Rules

`WeightDiagnostic.analyze(statistics)`는 네 가지 진단 결과를 반환한다.

| Key | Decision Rule | Limitation |
|---|---|---|
| `zero_weights` | 모든 원소가 유한하고 `zero_ratio == 1.0` | Weight가 0이어도 Bias/Activation 결과가 0이라고 단정할 수 없음 |
| `constant_weights` | 모두 유한하며 `std <= tolerance` (기본 `1e-12`) | 상수 Weight가 반드시 결함은 아님 |
| `invalid_weights` | `nan_count > 0 or inf_count > 0` | 이상치의 원인을 설명하지는 않음 |
| `weight_imbalance` | 양수/음수 비율 및 `abs(positive-negative)` | 부호 비율 차이 자체는 오류 판정이 아님 |

`small_weights` 및 `large_weights`는 `analyze()`의 기본 결과가 아니라 **호출자가 명시한 임계값**으로 별도 진단한다. 크기 임계값은 데이터·초기화·Layer 구조에 따라 달라지므로 절대적인 정상/비정상 기준이 아니다.

## 7. Neuron-Level Processing Flow

```text
inspect_neuron(layer_index, neuron_index)
  → get_layer(layer_index)
  → registry.get(layer.module)
  → OutputWeightInspector.inspect_neuron(...)
  → TensorNeuronSelector.select(weight, neuron_index, axis=0)
  → WeightInspector.inspect_weights(...)
  → TensorStatistics.calculate(...)
  → WeightDiagnostic.analyze(...)
  → {layer_index, layer_name, layer_type, neuron_index, inspections}
```

`inspect_layer()`는 한 Layer의 뉴런 전체를 순회하고, `inspect_model()`은 Resolver가 만든 Layer 전체를 순회한다.

### 7.1. get_problem_neurons() — 현재 구현 범위

`SequentialLayerInspector.get_problem_neurons()`는 등록된 `OutputWeightInspector`가 있는 Layer를 순회하며, 각 뉴런의 `diagnostics["zero_weights"]["all_zero"]`가 `True`인 경우에만 결과에 포함한다.

예시 반환 형식:

```python
[
    {
        "layer_index": 0,
        "neuron_index": 1,
        "reasons": ["all_zero_weights"],
    }
]
```

**제약:** 이 메서드는 현재 **All-Zero Output Weights만 문제 뉴런으로 집계**한다. NaN/Inf, Gradient 이상, Activation 포화 등을 통합 판정하는 범용 문제 뉴런 탐지기로 기술하면 부정확하다.

### 7.2. Similarity & Input Contribution

`OutputWeightInspector.inspect_neuron_similarity()`는 두 뉴런 Weight의 Cosine Similarity와 Euclidean Distance를 반환한다. 이는 **Weight의 기하학적 유사도**이며, 기능적 중복성이나 Pruning 가능성을 확정하지 않는다.

`inspect_input_contribution()`의 실제 실행 분기는 다음과 같다.

- **Linear:** 입력 요소와 Weight의 곱, Weighted Sum, Bias, Pre-activation, 최대 절댓값 기여 연결 인덱스
- **Bilinear:** 두 입력과 Weight의 항별 곱 및 합산
- **Conv:** 현재 위치별 Unfold/Patch 기반 분석을 구현하지 않았고 `NotImplementedError`를 발생시킨다.

또한 `inspect_connection_ablation()`은 **Linear에 한해** 지정 연결을 0으로 가정했을 때의 Pre-activation 변화를 계산한다. 모델 재학습 또는 전체 출력의 인과적 영향 평가와는 다르다.

## 8. TransformerModelAnalyzer — Static Estimates

`TransformerModelAnalyzer`는 `ModelAnalyzer`의 일반 구조 통계에 Transformer별 계산을 추가한다. 구현은 모델에 다음 속성이 있다고 가정한다.

- `model.fc_out.in_features/out_features`
- `model.layers[0].attention.heads/head_dim`
- `model.embedding.embedding.embedding_dim`
- `model.context_length` 또는 `sequence_length` 또는 `max_sequence_length`

이는 범용 Hugging Face Transformer Reflection API가 아니라 **특정 속성 계약을 요구하는 분석기**다.

### 8.1. Core Formulas

| Estimate | Formula | Meaning |
|---|---|---|
| Embedding memory | `B × S × E × bytes_per_element` | Embedding 출력 Tensor 크기 |
| Attention score memory | `B × H × S² × bytes_per_element` | Attention Score Tensor 크기 |
| Logits memory | `B × S × V × bytes_per_element` | Vocabulary Logits 크기 |
| Attention FLOPs | `2 × B × H × S² × head_dim` | 구현이 계산하는 Attention 연산 항 |
| Output projection FLOPs | `2 × B × S × E × V` | 출력 투영 연산 항 |
| Arithmetic intensity | `(Attention FLOPs + Projection FLOPs) / (Embedding + Score + Logits bytes)` | 선택한 항목에 한정된 추정 비율 |

여기서 `B`는 Batch Size, `S`는 Sequence Length, `E`는 Embedding Size, `H`는 Head Count, `V`는 Vocabulary Size다.

**범위 주의:** Attention FLOPs는 구현된 수식에 한정되며 Q/K/V Projection, Softmax, Feed-forward Network 등의 전체 Transformer 연산량을 모두 포함한다고 주장할 수 없다. Memory Traffic도 주요 Tensor 크기의 합에 접근 배수를 곱한 **모형 기반 추정치**이지 장치의 실측 Memory Transaction이 아니다.

### 8.2. Bandwidth Estimate

`memory_bandwidth_usage()`는 추정 Traffic × Iterations/s를 GiB/s로 변환한다. `estimate_bandwidth_saturation_batch_size()`는 1부터 `maximum_batch_size`까지 순회하여 사용률 임계값에 최초 도달한 Batch Size를 반환한다.

`bandwidth_utilization()`은 사용률을 `utilization_threshold`로 상한 제한하므로, 반환되는 값은 실제 장치 사용률의 독립적인 측정치가 아니다. 추천 Batch Size도 **지정된 가정 하의 모델 계산 결과**다.

## 9. Findings — Verification Classification

| Finding | Classification |
|---|---|
| Recursive `nn.Sequential` Layer Resolution | Implemented |
| Supported Module 기반 OutputWeightInspector 자동 등록 | Implemented |
| Neuron Weight Statistics 및 All-Zero 검출 | Implemented |
| Weight Cosine Similarity / Euclidean Distance | Implemented |
| Linear/Bilinear 입력 기여도 | Implemented |
| Conv 위치별 입력 기여도 | Not Implemented |
| `inspect_potential_redundancy()` | Not Implemented (removed API) |
| `build_diagnostic_summary()` | Not Implemented (removed API) |
| End-to-end 모델 정확도 개선 입증 | Runtime Verification Required |
| GPU/XPU 메모리 대역폭 실측 입증 | Runtime Verification Required |

## 10. Improvement Recommendations

1. **Layer Contract:** 지원 Primary Module과 Inspector 지원 Module의 차이를 공개 API 문서에 명시한다.
2. **Numerical Edge Cases:** NaN/Inf 포함 시 `finite_count` 기준 비율의 의미를 API 응답에 명시한다.
3. **Neuron Diagnostics:** `get_problem_neurons()`에 향후 진단 유형을 확장할 때는 Rule별 Evidence를 독립적으로 기록한다.
4. **Conv Attribution:** Patch/Unfold 기반 위치별 계산을 별도 구현·테스트한 후 지원한다고 선언한다.
5. **Transformer Validation:** 실제 Allocated Memory 및 Profiling 결과와 정적 추정의 오차를 실험으로 비교한다.
6. **Regression Tests:** Nested Sequential, Unsupported Module, All-Zero Neuron, NaN/Inf, Shape Mismatch, Batch/Sequence 경계값을 유지한다.

## 11. Source Traceability

- [SequentialLayerResolver](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/resolvers/SequentialLayerResolver.py)
- [SequentialLayerInspectorFactory](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/factory/SequentailLayerFactory.py)
- [LayerInspectorFactory](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/factory/LayerInspectorFactory.py)
- [SequentialLayerInspector](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/inspectors/SequentialLayerInspector.py)
- [OutputWeightInspector](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/inspectors/weights/OutputWeightInspector.py)
- [TensorStatistics](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/common/TensorStatistics.py)
- [WeightDiagnostic](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/diagnostics/WeightDiagnostic.py)
- [TransformerModelAnalyzer](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/analyzer/TransformerModelAnalyzer.py)

> **Verification note:** 이 문서는 Source-Level Implementation Evidence에 기반한다. 문서 작성 시점에 테스트 실행, Accelerator 실측, 실제 서비스 운영 검증은 수행하지 않았다.
