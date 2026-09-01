# Technical Documents | 기술 문서

## Overview | 개요

This repository contains **code-based technical documentation derived from actual source-code implementations** in a subset of GitHub repositories.

이 저장소는 일부의 GitHub Repository에 실제로 구현된 소스 코드를 기반으로 작성한 **Code-Based Technical Documentation(코드 기반 기술 문서)**을 통합 관리합니다.

The documents provide technical evidence of implemented systems by describing their architecture, component and module structures, processing flows, implementation decisions, technical requirements, limitations, and improvement opportunities.

각 문서는 구현된 시스템의 **아키텍처, 컴포넌트 및 모듈 구조, 처리 흐름, 구현 및 설계 결정, 기술 요구사항, 한계 및 개선 사항**을 분석하여 실제 구현에 대한 기술적 근거를 제공합니다.

The documentation covers multiple engineering domains, including **Software Architecture, Backend Engineering, AI, Deep Learning, Machine Learning, LLM, RAG, and related technologies**.

문서의 기술 범위는 특정 프레임워크에 한정되지 않으며 **Software Architecture, Backend Engineering, AI, Deep Learning, Machine Learning, LLM, RAG 및 관련 엔지니어링 기술**을 포함합니다.

---

## Purpose | 목적

The purpose of this repository is to provide **technical evidence and architectural documentation for a subset of implemented software projects**, including projects whose complete source code is not publicly disclosed.

이 저장소의 목적은 일부 실제 구현된 소프트웨어 프로젝트에 대한 **기술적 구현 근거와 아키텍처 문서**를 제공하는 것입니다. 전체 소스 코드가 공개되지 않은 프로젝트도 문서화 대상에 포함됩니다.

Each document is intended to demonstrate not only **what was implemented**, but also **how the implementation is structured, how components interact, which technical decisions were made, and how the implementation aligns with official technical references**.

각 기술 문서는 단순히 **무엇을 구현했는지**만 나열하지 않고, **어떤 구조로 구현했는지, 컴포넌트가 어떻게 상호작용하는지, 어떤 기술적 설계 결정을 적용했는지, 구현 내용이 공식 기술 Reference와 어떻게 대응되는지**를 설명합니다.

---

## Documentation Principles | 문서 작성 원칙

The documentation follows four primary principles.

기술 문서는 다음 네 가지 원칙을 기준으로 작성합니다.

### 1. Implementation-Based | 실제 구현 기반

Technical descriptions are derived from actual source code, project configuration, dependency definitions, and related implementation artifacts.

기술적 설명은 실제 **소스 코드, 프로젝트 설정, 의존성 정의 및 관련 구현 산출물**을 근거로 작성합니다.

Architecture descriptions are not treated as implemented functionality unless corresponding implementation evidence can be identified.

구현 근거를 확인할 수 없는 아키텍처 또는 기능은 실제 구현된 것으로 간주하지 않습니다.

### 2. Traceable | 추적 가능성

Technical findings should be traceable to identifiable implementation elements such as modules, packages, classes, configuration, APIs, pipelines, or dependencies.

기술 분석 결과는 **모듈, 패키지, 클래스, 설정, API, Pipeline 또는 Dependency** 등 실제 구현 요소와 연결하여 추적할 수 있도록 구성합니다.

### 3. Reference-Based | 공식 Reference 기반

Where applicable, implementations are analyzed against official framework documentation, API references, architectural concepts, and established engineering practices.

필요한 경우 구현 내용을 **공식 Framework Documentation, API Reference, Architecture Concept 및 확립된 Engineering Practice**와 비교하여 분석합니다.

### 4. Verification-Aware | 검증 범위 구분

Static analysis and runtime verification are treated separately.

**Static Analysis(정적 분석)**과 **Runtime Verification(실행 기반 검증)**의 범위를 구분합니다.

Behavior that requires application execution, integration testing, performance measurement, monitoring, or external infrastructure verification is not considered verified solely through static source-code analysis.

애플리케이션 실행, 통합 테스트, 성능 측정, 모니터링 또는 외부 인프라 확인이 필요한 항목은 소스 코드 정적 분석만으로 검증된 것으로 판단하지 않습니다.

---

## Analysis Methodology | 분석 방법론

Technical documents are produced using a **source-code-driven analysis process**.

기술 문서는 **Source-Code-Driven Analysis(소스 코드 중심 분석)** 방식으로 작성합니다.

### Analysis Flow | 분석 흐름

**Source Code
→ Static Analysis
→ Architecture & Implementation Analysis
→ Reference Validation
→ Technical Documentation**

**소스 코드
→ 정적 분석
→ 아키텍처 및 구현 분석
→ 공식 Reference 기반 검증
→ 기술 문서**

The analysis may examine:

분석 대상에는 다음 항목이 포함될 수 있습니다.

* **Repository / Project Structure** — Repository 및 프로젝트 구조
* **Package / Module Structure** — 패키지 및 모듈 구조
* **Component / Class Relationships** — 컴포넌트 및 클래스 관계
* **Inheritance / Composition / Dependencies** — 상속, 구성 및 의존 관계
* **Application Configuration** — 애플리케이션 설정
* **Framework Configuration** — 프레임워크 설정
* **Build / Dependency Configuration** — 빌드 및 의존성 설정
* **API / Application Entry Points** — API 및 애플리케이션 진입점
* **Request / Response Flow** — 요청 및 응답 흐름
* **Data Processing Flow** — 데이터 처리 흐름
* **Security Architecture** — 보안 아키텍처
* **Persistence Architecture** — 영속성 아키텍처
* **Training / Validation / Inference Pipeline** — 학습, 검증 및 추론 Pipeline
* **AI / ML Model Integration** — AI/ML 모델 통합
* **Infrastructure / Deployment Configuration** — 인프라 및 배포 설정

---

## AI-Assisted Analysis | AI 보조 분석

AI-assisted analysis may be used to support **source-code inspection, structural analysis, architecture identification, reference comparison, and technical-document preparation**.

AI 기반 분석은 **소스 코드 검토, 구조 분석, 아키텍처 식별, 공식 Reference 비교 및 기술 문서 작성**을 보조하는 수단으로 활용할 수 있습니다.

AI-generated analysis is not considered implementation evidence by itself. Technical conclusions should be based on identifiable source-code or project artifacts and, where applicable, validated against official technical references.

AI 분석 결과 자체를 구현의 근거로 간주하지 않습니다. 기술적 결론은 식별 가능한 **소스 코드 또는 프로젝트 산출물**을 근거로 하며, 필요한 경우 공식 기술 Reference를 통해 검증합니다.

---

## Technical Analysis Scope | 기술 분석 범위

| Engineering Area / 기술 영역  | Analysis Scope / 분석 범위                                                                                              |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Software Architecture** | Application Architecture, Layered Architecture, Component Design / 애플리케이션 아키텍처, 계층 구조, 컴포넌트 설계                      |
| **Backend Engineering**   | Spring Boot, Web Architecture, Security, Persistence / Spring Boot, Web 아키텍처, 보안, 영속성                               |
| **Deep Learning**         | PyTorch, Trainer Architecture, Training Pipeline / PyTorch, Trainer 아키텍처, 학습 Pipeline                               |
| **Machine Learning**      | ML Pipeline, Data Processing, Model Evaluation / ML Pipeline, 데이터 처리, 모델 평가                                         |
| **LLM Engineering**       | Model Integration, Fine-Tuning, PEFT, LoRA / 모델 통합, Fine-Tuning, PEFT, LoRA                                         |
| **RAG**                   | Chunking, Embedding, Vector DB, Hybrid Search, Reranking / Chunking, Embedding, Vector DB, Hybrid Search, Reranking |
| **AI Systems**            | AI Application Architecture, Inference Pipeline, Agent Architecture / AI 애플리케이션 아키텍처, 추론 Pipeline, Agent 아키텍처       |
| **Infrastructure**        | Containerization, Deployment Architecture, Runtime Configuration / Container, 배포 아키텍처, Runtime 설정                   |

---

## Technical Finding Classification | 기술 분석 결과 분류

Technical findings are classified according to their implementation and verification status.

기술 분석 결과는 구현 및 검증 상태에 따라 다음과 같이 구분합니다.

| Classification                    | 구분            | Definition / 정의                                                                                                                        |
| --------------------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| **Implemented**                   | 구현            | Implementation confirmed in source code or project artifacts / 소스 코드 또는 프로젝트 산출물에서 구현 확인                                               |
| **Required**                      | 필수            | Required by the architecture, framework, or defined implementation requirements / 아키텍처, 프레임워크 또는 정의된 구현 요구사항에 필요한 항목                   |
| **Recommended**                   | 권장            | Improvement recommended based on technical references or engineering practices / 기술 Reference 또는 Engineering Practice를 근거로 개선을 권장하는 항목 |
| **Not Implemented**               | 미구현           | Implementation not identified in the analyzed source code / 분석된 소스 코드에서 구현을 확인하지 못한 항목                                                 |
| **Runtime Verification Required** | Runtime 검증 필요 | Requires execution, testing, monitoring, or environment verification / 실행, 테스트, 모니터링 또는 환경 검증이 필요한 항목                                  |

---

## Relationship to Source Repositories | Source Repository와의 관계

Source code is maintained independently in its corresponding GitHub repository.

실제 소스 코드는 각각의 GitHub Source Repository에서 독립적으로 관리합니다.

This repository provides the corresponding **architecture and implementation documentation**, allowing technical analysis to be maintained separately from source-code repositories.

이 저장소에서는 해당 프로젝트의 **아키텍처 및 구현 기술 문서**를 별도로 관리하여 소스 코드와 기술 분석 문서를 분리합니다.

For projects whose source code is not publicly disclosed, the technical documentation provides an architectural and implementation-level description while preserving the confidentiality of the complete source code.

전체 소스 코드를 공개하지 않는 프로젝트의 경우, 전체 코드의 비공개 상태를 유지하면서 **Architecture 및 Implementation Level의 기술적 구조와 구현 내용을 설명하는 자료**로 활용합니다.

---

## Document Structure | 문서 구성

A technical document may contain the following sections depending on the characteristics of the project.

프로젝트와 기술의 특성에 따라 다음과 같은 구조로 기술 문서를 구성할 수 있습니다.

1. **Introduction** — 소개
2. **System Overview** — 시스템 개요
3. **Architecture Overview** — 아키텍처 개요
4. **Component / Module Architecture** — 컴포넌트 및 모듈 아키텍처
5. **Processing / Data Flow** — 처리 및 데이터 흐름
6. **Implementation Analysis** — 구현 분석
7. **Reference-Based Evaluation** — 공식 Reference 기반 평가
8. **Technical Requirements** — 기술 요구사항
9. **Issues / Limitations** — 문제점 및 한계
10. **Improvement Recommendations** — 개선 권고사항
11. **Architecture Summary** — 아키텍처 요약

The structure may be extended with technology-specific analysis for **Spring Boot, PyTorch, DL/ML systems, LLM applications, RAG architectures, and other engineering technologies**.

문서 구조는 **Spring Boot, PyTorch, DL/ML System, LLM Application, RAG Architecture 및 기타 엔지니어링 기술**의 특성에 따라 확장할 수 있습니다.

---

## Disclaimer | 분석 범위 및 제한

The documents represent technical analysis based on the source code and project artifacts available at the time of analysis.

기술 문서는 분석 시점에 확인 가능한 **소스 코드 및 프로젝트 산출물**을 기반으로 작성한 기술 분석 결과입니다.

Static analysis cannot fully verify runtime behavior, production configuration, performance characteristics, external system behavior, or operational conditions.

정적 분석만으로는 **Runtime 동작, Production 환경 설정, 성능 특성, 외부 시스템 동작 및 운영 환경**을 완전히 검증할 수 없습니다.

Such areas require separate runtime testing, integration testing, performance evaluation, or operational verification.

해당 영역은 별도의 **Runtime Test, Integration Test, Performance Evaluation 또는 Operational Verification**을 통해 검증해야 합니다.
