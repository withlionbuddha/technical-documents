# deep-learning-core — Model & Neuron Diagnostics Technical Document

> **Source:** [withlionbuddha/deep-learning-core — ground](https://github.com/withlionbuddha/deep-learning-core/tree/ground/modules/diagnostics)  
> **Analysis date:** 2026-10-28 · **Evidence level:** Source inspection (not runtime benchmark)

## 1. Purpose — 구현 분석 대상

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

**구현 대상:** `nn.Sequential` 내부의 구조적 순회.

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

| Key | Decision Rule |
|---|---|
| `zero_weights` | 모든 원소가 유한하고 `zero_ratio == 1.0` |
| `constant_weights` | 모두 유한하며 `std <= tolerance` (기본 `1e-12`) |
| `invalid_weights` | `nan_count > 0 or inf_count > 0` |
| `weight_imbalance` | 양수/음수 비율 및 `abs(positive-negative)` |

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

**구현된 판정 조건:** `all_zero_weights`.

### 7.2. Similarity & Input Contribution

`OutputWeightInspector.inspect_neuron_similarity()`는 두 뉴런 Weight의 Cosine Similarity와 Euclidean Distance를 반환한다. 계산 결과는 Weight 벡터 간 유사도 및 거리다.

`inspect_input_contribution()`의 실제 실행 분기는 다음과 같다.

- **Linear:** 입력 요소와 Weight의 곱, Weighted Sum, Bias, Pre-activation, 최대 절댓값 기여 연결 인덱스
- **Bilinear:** 두 입력과 Weight의 항별 곱 및 합산

`inspect_connection_ablation()`은 `nn.Linear`에서 특정 입력 연결을 0으로 가정한 Pre-activation 차이를 계산한다.

## 8. TransformerModelAnalyzer — Static Estimates

`TransformerModelAnalyzer`는 `ModelAnalyzer`의 일반 구조 통계에 Transformer별 계산을 추가한다. 구현은 모델에 다음 속성이 있다고 가정한다.

- `model.fc_out.in_features/out_features`
- `model.layers[0].attention.heads/head_dim`
- `model.embedding.embedding.embedding_dim`
- `model.context_length` 또는 `sequence_length` 또는 `max_sequence_length`

분석기는 위 모델 속성을 직접 참조한다.

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

**구현된 계산 범위:** 위 표의 Tensor 및 연산 항을 사용한 정적 추정.

### 8.2. Bandwidth Estimate

`memory_bandwidth_usage()`는 추정 Traffic × Iterations/s를 GiB/s로 변환한다. `estimate_bandwidth_saturation_batch_size()`는 1부터 `maximum_batch_size`까지 순회하여 사용률 임계값에 최초 도달한 Batch Size를 반환한다.

`bandwidth_utilization()`은 계산된 사용률을 `utilization_threshold`로 상한 제한한다.

## 9. Source Traceability

- [SequentialLayerResolver](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/resolvers/SequentialLayerResolver.py)
- [SequentialLayerInspectorFactory](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/factory/SequentailLayerFactory.py)
- [LayerInspectorFactory](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/factory/LayerInspectorFactory.py)
- [SequentialLayerInspector](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/inspectors/SequentialLayerInspector.py)
- [OutputWeightInspector](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/inspectors/weights/OutputWeightInspector.py)
- [TensorStatistics](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/common/TensorStatistics.py)
- [WeightDiagnostic](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/diagnostics/WeightDiagnostic.py)
- [TransformerModelAnalyzer](https://github.com/withlionbuddha/deep-learning-core/blob/ground/modules/diagnostics/analyzer/TransformerModelAnalyzer.py)

