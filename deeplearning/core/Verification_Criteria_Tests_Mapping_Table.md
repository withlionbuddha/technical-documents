# Verification Criteria — Tests Mapping Table

Repository(저장소): `withlionbuddha/deep-learning-core` · Branch(브랜치): `ground`

## Testing Roles(테스트의 역할)

- **Unit Tests(단위 테스트):** 개별 함수·클래스의 입력 계약, 반환 타입, 수치 계산, 예외 및 경계 조건을 독립적으로 검증합니다.
- **Notebook Execution(노트북 실행):** 패키지 API를 연결하여 모델 구성·학습·검증·진단·저장 흐름이 실행되는지 확인합니다.
- **Role Separation(역할 구분):** Unit Test는 기능별 정확성에, Notebook은 통합 실행 흐름에 중점을 둡니다. Notebook 실행만으로 모든 단위 테스트 조건을 검증했다고 판단하지 않습니다.
- **Repository Paths(저장소 경로):** 아래 경로는 저장소 루트 기준이며, 외부에서 제공된 Notebook은 별도로 구분합니다.
- **Failure Detection(실패 시 발견 가능한 문제):** FAIL 발생 시 의심하거나 확인할 수 있는 오류를 설명합니다.
- **Pass Assurance(통과 시 확인되는 사항):** PASS 발생 시 정의된 입력과 실행 조건에서 검증된 기능을 설명합니다.
- **Coverage Limitations(검증 범위의 한계):** 테스트에서 확인하지 않은 조건을 명시합니다. PASS가 모든 입력·환경에서의 무결함을 보장하지는 않습니다. 실행 결과와 PASS 건수는 별도 검증 없이 근거로 사용하지 않습니다.

## Verification Criteria — Vertical Mapping(검증 기준별 세로형 매핑)

### 01. Input Validation / Return Type Validation / Nested Type Checking

| **Title(제목)** | **Content(내용)** |
| --- | --- |
| **Verification Criteria(검증 기준)** | **Input Validation / Return Type Validation / Nested Type Checking**<br>**입력 유효성 검사 / 반환 타입 유효성 검사 / 중첩 타입 검사** |
| **Detailed Verification(상세 검증)** | **함수 인자·반환값·중첩 자료형의 타입 계약 및 오류 거부** |
| **Unit Tests(단위 테스트)** | `tests/units/test_function_type_contracts.py`, `tests/units/device/test_profiler_path_and_type_contracts.py` |
| **Notebooks(노트북)** | `notebooks/01_builders_and_calculators.ipynb` |
| **Failure Detection(실패 시 발견 가능한 문제)** | **잘못된 입력·반환값이 후속 계산으로 전달되는 문제를 검사 조건에서 발견** |
| **Pass Assurance(통과 시 확인되는 사항)** | 해당 단위 테스트가 PASS이면 명시된 입력·경계 조건에서 기대 결과가 충족되었음을 확인합니다. Notebook의 통합 실행 성공 여부는 별도로 확인해야 합니다. |
| **Coverage Limitations(검증 범위의 한계)** | 열거되지 않은 입력·장치·환경과 장시간 학습 동작은 이 매핑만으로 검증되지 않습니다. |

### 02. Data Splitting / Batch Integrity / Masked Accuracy

| **Title(제목)** | **Content(내용)** |
| --- | --- |
| **Verification Criteria(검증 기준)** | **Data Splitting / Batch Integrity / Masked Accuracy**<br>**데이터 분할 / 배치 무결성 / 마스킹 정확도** |
| **Detailed Verification(상세 검증)** | **분할 개수·중복·재현성·배치 보존·마스킹 정확도** |
| **Unit Tests(단위 테스트)** | `tests/units/data/test_data_split_contract_matrix.py`, `tests/units/metrics/test_accuracy_contract_matrix.py` |
| **Notebooks(노트북)** | `notebooks/01_builders_and_calculators.ipynb`, `notebooks/04_transformer_training.ipynb` |
| **Failure Detection(실패 시 발견 가능한 문제)** | **데이터 누락·중복과 패딩을 정확도에 포함하는 오류를 발견** |
| **Pass Assurance(통과 시 확인되는 사항)** | 해당 단위 테스트가 PASS이면 명시된 입력·경계 조건에서 기대 결과가 충족되었음을 확인합니다. Notebook의 통합 실행 성공 여부는 별도로 확인해야 합니다. |
| **Coverage Limitations(검증 범위의 한계)** | 열거되지 않은 입력·장치·환경과 장시간 학습 동작은 이 매핑만으로 검증되지 않습니다. |

### 03. Loss Calculation / Gradient Consistency / Parameter Updates

| **Title(제목)** | **Content(내용)** |
| --- | --- |
| **Verification Criteria(검증 기준)** | **Loss Calculation / Gradient Consistency / Parameter Updates**<br>**손실 계산 / 기울기 일관성 / 파라미터 갱신** |
| **Detailed Verification(상세 검증)** | **손실 수식 대조·역전파 기울기·최적화 후 가중치 변화** |
| **Unit Tests(단위 테스트)** | `tests/units/loss/test_cross_entropy_contract_matrix.py`, `tests/units/training/runners/test_batch_runner.py`, `tests/units/device/test_xpu_execution.py` |
| **Notebooks(노트북)** | `notebooks/02_training_and_validation.ipynb`, `notebooks/08_deep_dnn_training.ipynb`, `notebooks/09_xpu_complex_dnn_training.ipynb` |
| **Failure Detection(실패 시 발견 가능한 문제)** | **손실 불일치·기울기 누락·가중치 미갱신을 조기 발견** |
| **Pass Assurance(통과 시 확인되는 사항)** | 해당 단위 테스트가 PASS이면 명시된 입력·경계 조건에서 기대 결과가 충족되었음을 확인합니다. Notebook의 통합 실행 성공 여부는 별도로 확인해야 합니다. |
| **Coverage Limitations(검증 범위의 한계)** | 열거되지 않은 입력·장치·환경과 장시간 학습 동작은 이 매핑만으로 검증되지 않습니다. |

### 04. Parameter Counting / Memory Estimation / Gradient Tracking

| **Title(제목)** | **Content(내용)** |
| --- | --- |
| **Verification Criteria(검증 기준)** | **Parameter Counting / Memory Estimation / Gradient Tracking**<br>**파라미터 개수 계산 / 메모리 추정 / 기울기 추적** |
| **Detailed Verification(상세 검증)** | **파라미터 수·dtype별 메모리·공유/고정 파라미터·기울기 상태** |
| **Unit Tests(단위 테스트)** | `tests/units/diagnostics/test_model_analyzer.py`, `tests/units/diagnostics/test_analyzer_boundary_regressions.py`, `tests/units/diagnostics/test_transformer_model_analyzer.py` |
| **Notebooks(노트북)** | `notebooks/06_model_analysis_and_profiling.ipynb`, `notebooks/09_xpu_dnn_transformer_analysis.ipynb` |
| **Failure Detection(실패 시 발견 가능한 문제)** | **중복 집계·메모리 산식 오류·기울기 누락을 발견. 실제 장치 메모리 보장은 아님** |
| **Pass Assurance(통과 시 확인되는 사항)** | 해당 단위 테스트가 PASS이면 명시된 입력·경계 조건에서 기대 결과가 충족되었음을 확인합니다. Notebook의 통합 실행 성공 여부는 별도로 확인해야 합니다. |
| **Coverage Limitations(검증 범위의 한계)** | 열거되지 않은 입력·장치·환경과 장시간 학습 동작은 이 매핑만으로 검증되지 않습니다. |

### 05. Layer Resolution / Inspector Selection / Registry Deduplication

| **Title(제목)** | **Content(내용)** |
| --- | --- |
| **Verification Criteria(검증 기준)** | **Layer Resolution / Inspector Selection / Registry Deduplication**<br>**계층 해석 / 검사기 선택 / 레지스트리 중복 제거** |
| **Detailed Verification(상세 검증)** | **중첩 계층·stage 해석·검사기 선택·중복 등록 방지** |
| **Unit Tests(단위 테스트)** | `tests/units/diagnostics/test_layer_inspector_factory_registration.py`, `tests/units/diagnostics/resolvers/test_stage_resolver.py`, `tests/units/diagnostics/common/test_diagnostic_registry.py` |
| **Notebooks(노트북)** | `notebooks/05_resolvers_hooks_and_neurons.ipynb`, `notebooks/05_diagnostics.ipynb` |
| **Failure Detection(실패 시 발견 가능한 문제)** | **계층 오지정·검사기 오선택·중복 등록 문제를 발견** |
| **Pass Assurance(통과 시 확인되는 사항)** | 해당 단위 테스트가 PASS이면 명시된 입력·경계 조건에서 기대 결과가 충족되었음을 확인합니다. Notebook의 통합 실행 성공 여부는 별도로 확인해야 합니다. |
| **Coverage Limitations(검증 범위의 한계)** | 열거되지 않은 입력·장치·환경과 장시간 학습 동작은 이 매핑만으로 검증되지 않습니다. |

### 06. Weight Statistics / Neuron Contribution / Connection Ablation

| **Title(제목)** | **Content(내용)** |
| --- | --- |
| **Verification Criteria(검증 기준)** | **Weight Statistics / Neuron Contribution / Connection Ablation**<br>**가중치 통계 / 뉴런 기여도 / 연결 제거 분석** |
| **Detailed Verification(상세 검증)** | **가중치 통계·뉴런 입력 기여도·연결 제거 전후 순전파 비교** |
| **Unit Tests(단위 테스트)** | `tests/units/diagnostics/test_inspection_numeric_regressions.py`, `tests/units/diagnostics/test_output_weight_inspector.py`, `tests/units/diagnostics/test_neuron_selector_contract_matrix.py` |
| **Notebooks(노트북)** | `notebooks/04_weight_and_gradient_diagnostics.ipynb`, `notebooks/05_diagnostics.ipynb` |
| **Failure Detection(실패 시 발견 가능한 문제)** | **통계·기여도·연결 제거 영향의 계산 오류를 발견** |
| **Pass Assurance(통과 시 확인되는 사항)** | 해당 단위 테스트가 PASS이면 명시된 입력·경계 조건에서 기대 결과가 충족되었음을 확인합니다. Notebook의 통합 실행 성공 여부는 별도로 확인해야 합니다. |
| **Coverage Limitations(검증 범위의 한계)** | 열거되지 않은 입력·장치·환경과 장시간 학습 동작은 이 매핑만으로 검증되지 않습니다. |

### 07. Hook Registration / Execution Recording / Cleanup

| **Title(제목)** | **Content(내용)** |
| --- | --- |
| **Verification Criteria(검증 기준)** | **Hook Registration / Execution Recording / Cleanup**<br>**Hook 등록 / 실행 기록 / 정리** |
| **Detailed Verification(상세 검증)** | **Hook 등록·호출 기록·재연결·예외 이후 해제** |
| **Unit Tests(단위 테스트)** | `tests/units/diagnostics/test_profiler_lifecycle_regressions.py`, `tests/units/diagnostics/test_profiler_execution_contract_matrix.py` |
| **Notebooks(노트북)** | `notebooks/05_resolvers_hooks_and_neurons.ipynb`, `notebooks/06_model_analysis_and_profiling.ipynb` |
| **Failure Detection(실패 시 발견 가능한 문제)** | **중복 기록·누락·잔여 Hook으로 인한 오동작을 발견** |
| **Pass Assurance(통과 시 확인되는 사항)** | 해당 단위 테스트가 PASS이면 명시된 입력·경계 조건에서 기대 결과가 충족되었음을 확인합니다. Notebook의 통합 실행 성공 여부는 별도로 확인해야 합니다. |
| **Coverage Limitations(검증 범위의 한계)** | 열거되지 않은 입력·장치·환경과 장시간 학습 동작은 이 매핑만으로 검증되지 않습니다. |

### 08. Checkpoint Restoration / Optimizer State / Training Continuity

| **Title(제목)** | **Content(내용)** |
| --- | --- |
| **Verification Criteria(검증 기준)** | **Checkpoint Restoration / Optimizer State / Training Continuity**<br>**체크포인트 복원 / 최적화 상태 / 학습 연속성** |
| **Detailed Verification(상세 검증)** | **모델·optimizer 상태 복원 후 다음 학습 단계 일치** |
| **Unit Tests(단위 테스트)** | `tests/units/checkpoint/test_optimizer_resume_equivalence.py`, `tests/units/checkpoint/test_param_group_resume_contract_matrix.py` |
| **Notebooks(노트북)** | `notebooks/03_checkpointing_and_history.ipynb`, `notebooks/08_deep_dnn_training.ipynb` |
| **Failure Detection(실패 시 발견 가능한 문제)** | **재개 학습 시 상태 손실·갱신 불일치를 발견** |
| **Pass Assurance(통과 시 확인되는 사항)** | 해당 단위 테스트가 PASS이면 명시된 입력·경계 조건에서 기대 결과가 충족되었음을 확인합니다. Notebook의 통합 실행 성공 여부는 별도로 확인해야 합니다. |
| **Coverage Limitations(검증 범위의 한계)** | 열거되지 않은 입력·장치·환경과 장시간 학습 동작은 이 매핑만으로 검증되지 않습니다. |

### 09. Path Creation / Collision Handling / Write Failure / JSON Preservation

| **Title(제목)** | **Content(내용)** |
| --- | --- |
| **Verification Criteria(검증 기준)** | **Path Creation / Collision Handling / Write Failure / JSON Preservation**<br>**경로 생성 / 충돌 처리 / 쓰기 실패 / JSON 보존** |
| **Detailed Verification(상세 검증)** | **고유 실행 경로·이름 충돌·쓰기 실패·JSON 보존** |
| **Unit Tests(단위 테스트)** | `tests/units/storage/test_log_path_manager.py`, `tests/units/device/test_profiler_path_and_type_contracts.py` |
| **Notebooks(노트북)** | `notebooks/03_checkpointing_and_history.ipynb` |
| **Failure Detection(실패 시 발견 가능한 문제)** | **경로 충돌·저장 실패·기록 손상을 발견** |
| **Pass Assurance(통과 시 확인되는 사항)** | 해당 단위 테스트가 PASS이면 명시된 입력·경계 조건에서 기대 결과가 충족되었음을 확인합니다. Notebook의 통합 실행 성공 여부는 별도로 확인해야 합니다. |
| **Coverage Limitations(검증 범위의 한계)** | 열거되지 않은 입력·장치·환경과 장시간 학습 동작은 이 매핑만으로 검증되지 않습니다. |

### 10. Conditional Execution / Frozen Parameters / Detached Gradients

| **Title(제목)** | **Content(내용)** |
| --- | --- |
| **Verification Criteria(검증 기준)** | **Conditional Execution / Frozen Parameters / Detached Gradients**<br>**조건부 실행 / 고정 파라미터 / 분리된 기울기** |
| **Detailed Verification(상세 검증)** | **분기 실행·고정 파라미터·계산 그래프 분리 시 관측** |
| **Unit Tests(단위 테스트)** | `tests/units/diagnostics/test_shared_layer_edge_cases.py`, `tests/units/diagnostics/inspectors/test_gradient_inspector.py` |
| **Notebooks(노트북)** | `notebooks/04_weight_and_gradient_diagnostics.ipynb` |
| **Failure Detection(실패 시 발견 가능한 문제)** | **실행하지 않은 계층과 0 기울기 혼동·고정 파라미터 오판을 발견** |
| **Pass Assurance(통과 시 확인되는 사항)** | 해당 단위 테스트가 PASS이면 명시된 입력·경계 조건에서 기대 결과가 충족되었음을 확인합니다. Notebook의 통합 실행 성공 여부는 별도로 확인해야 합니다. |
| **Coverage Limitations(검증 범위의 한계)** | 열거되지 않은 입력·장치·환경과 장시간 학습 동작은 이 매핑만으로 검증되지 않습니다. |

### 11. Shared Layers / Tied Parameters / Reentrant Hooks

| **Title(제목)** | **Content(내용)** |
| --- | --- |
| **Verification Criteria(검증 기준)** | **Shared Layers / Tied Parameters / Reentrant Hooks**<br>**공유 계층 / 결합 파라미터 / 재진입 Hook** |
| **Detailed Verification(상세 검증)** | **공유 객체 중복 집계·기울기 누적·재진입 기록** |
| **Unit Tests(단위 테스트)** | `tests/units/diagnostics/test_shared_layer_edge_cases.py`, `tests/units/checkpoint/test_shared_parameter_checkpoint_edge_cases.py`, `tests/units/diagnostics/test_reentrant_hook_edge_cases.py` |
| **Notebooks(노트북)** | `notebooks/05_resolvers_hooks_and_neurons.ipynb`, `notebooks/06_model_analysis_and_profiling.ipynb` |
| **Failure Detection(실패 시 발견 가능한 문제)** | **공유 파라미터 중복 집계·재진입 기록 누락을 발견** |
| **Pass Assurance(통과 시 확인되는 사항)** | 해당 단위 테스트가 PASS이면 명시된 입력·경계 조건에서 기대 결과가 충족되었음을 확인합니다. Notebook의 통합 실행 성공 여부는 별도로 확인해야 합니다. |
| **Coverage Limitations(검증 범위의 한계)** | 열거되지 않은 입력·장치·환경과 장시간 학습 동작은 이 매핑만으로 검증되지 않습니다. |

### 12. XPU Availability / Device Execution / Forward & Backward / Parameter Updates / Profiling

| **Title(제목)** | **Content(내용)** |
| --- | --- |
| **Verification Criteria(검증 기준)** | **XPU Availability / Device Execution / Forward & Backward / Parameter Updates / Profiling**<br>**XPU 사용 가능 여부 / 장치 실행 / 순전파·역전파 / 파라미터 갱신 / 프로파일링** |
| **Detailed Verification(상세 검증)** | **장치 감지·실제 연산·학습·갱신·프로파일링** |
| **Unit Tests(단위 테스트)** | `tests/units/device/test_xpu_execution.py` |
| **Notebooks(노트북)** | `notebooks/09_xpu_complex_dnn_training.ipynb`, `notebooks/09_xpu_dnn_transformer_analysis.ipynb` |
| **Failure Detection(실패 시 발견 가능한 문제)** | **지원 장치에서 실행 실패·기울기 누락·갱신 오류를 발견. CPU 검증으로 대체 불가** |
| **Pass Assurance(통과 시 확인되는 사항)** | 해당 단위 테스트가 PASS이면 명시된 입력·경계 조건에서 기대 결과가 충족되었음을 확인합니다. Notebook의 통합 실행 성공 여부는 별도로 확인해야 합니다. |
| **Coverage Limitations(검증 범위의 한계)** | 열거되지 않은 입력·장치·환경과 장시간 학습 동작은 이 매핑만으로 검증되지 않습니다. |

### 13. Gradient Vanishing Diagnostics

| **Title(제목)** | **Content(내용)** |
| --- | --- |
| **Verification Criteria(검증 기준)** | **Gradient Vanishing Diagnostics**<br>**기울기 소실 진단** |
| **Detailed Verification(상세 검증)** | **기울기 RMS·L2·0 비율·유한값 비율·초기/후반 가중치 RMS 비율** |
| **Unit Tests(단위 테스트)** | `tests/units/diagnostics/test_gradient_vanishing_metrics.py` |
| **Notebooks(노트북)** | `첨부 sgd_adam_backprop_diagnostic(1).ipynb (저장소 외부)` |
| **Failure Detection(실패 시 발견 가능한 문제)** | **미생성·0·비유한 기울기와 초기/후반 기울기 불균형을 발견. 원인 확정은 불가** |
| **Pass Assurance(통과 시 확인되는 사항)** | 해당 단위 테스트가 PASS이면 명시된 입력·경계 조건에서 기대 결과가 충족되었음을 확인합니다. Notebook의 통합 실행 성공 여부는 별도로 확인해야 합니다. |
| **Coverage Limitations(검증 범위의 한계)** | 열거되지 않은 입력·장치·환경과 장시간 학습 동작은 이 매핑만으로 검증되지 않습니다. |

## Limitations / 한계

테스트 파일과 노트북의 매칭은 검증 대상의 대응 관계이며, 현재 실행 성공을 의미하지 않습니다. 기울기 소실 지표의 정의·구현 상태는 `docs/GRADIENT_VANISHING_DIAGNOSTIC_METRICS.md`를 참조합니다.
