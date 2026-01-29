# VMware Private AI Foundation 완벽 가이드
## PAIF/PAIS/DLVM 개념 정리 및 AI 앱 개발 워크플로우

> **문서 버전:** 3.1  
> **작성일:** 2026년 1월 29일 (v3.1 업데이트)  
> **대상 독자:** VMware 솔루션 아키텍트, AI/ML 엔지니어, 플랫폼 엔지니어, 앱 개발자  
> **기반 버전:** VCF 9.0.1, PAIF 9.0, PAIS 2.0.89, DLVM 9.0.1

---

## 버전 히스토리

| 버전 | 날짜 | 주요 변경 사항 |
|------|------|--------------|
| 1.0 | 2026.01 | 초안 작성 |
| 2.0 | 2026.01 | 페르소나별 워크플로우 추가, RAG 파이프라인 상세화 |
| 2.1 | 2026.01 | 코드 예시 및 UI 목업 추가 |
| 2.2 | 2026.01.28 | FAQ 확장, 인증 흐름 상세화 |
| 3.0 | 2026.01.29 | 최신 릴리즈 반영 (DLVM 9.0.1), Deprecated 항목 명시, 버전 호환성 매트릭스 추가, DirectPath I/O 옵션 추가 |
| **3.1** | **2026.01.29** | **용어 혼동 주의 섹션 추가 ("AI 플레이그라운드" vs "PAIS Playground" 명확화)** |

---

## 목차

1. [핵심 개념 정의](#1-핵심-개념-정의)
2. [페르소나 및 역할 정의](#2-페르소나-및-역할-정의)
3. [아키텍처 개요](#3-아키텍처-개요)
4. [구축 순서 및 의존성](#4-구축-순서-및-의존성)
5. [AI 플레이그라운드 개념](#5-ai-플레이그라운드-개념)
6. [역할별 워크플로우](#6-역할별-워크플로우)
7. [개발 환경별 시나리오](#7-개발-환경별-시나리오)
8. [AI 앱 개발 상세](#8-ai-앱-개발-상세)
9. [프로덕션 아키텍처](#9-프로덕션-아키텍처)
10. [FAQ](#10-faq)
11. [부록: 버전 호환성 매트릭스](#부록-버전-호환성-매트릭스)

---

## 1. 핵심 개념 정의

### 1.1 용어 정의

| 용어 | 정식 명칭 | 역할 | 비유 |
|------|----------|------|------|
| **PAIF** | VMware Private AI Foundation with NVIDIA | GPU 인프라 + AI 서비스 플랫폼 전체 | 건물 + 모든 시설 |
| **PAIS** | Private AI Services | 관리형 AI 서비스 레이어 (추론, RAG, Agent) | 건물 내 비즈니스 서비스 |
| **DLVM** | Deep Learning VM | GPU 장착 개발자 워크스테이션 VM | 개발자 책상 + 장비 |
| **VKS** | vSphere Kubernetes Service | 프로덕션용 GPU 가속 K8s 클러스터 | 프로덕션 서버팜 |
| **DSM** | VMware Data Services Manager | 데이터베이스 서비스 (pgvector 포함) | 데이터베이스 관리 시스템 |

### 1.2 DLVM과 PAIS의 관계

```
❌ 잘못된 이해:
   PAIS = 개발 플레이그라운드 → 그 안에서 DLVM 배포

✅ 올바른 이해:
   DLVM = 개발자 개인용 GPU VM (워크스테이션) - 모델 검증/실험용
   PAIS = 모델을 프로덕션 API로 서빙하는 관리형 플랫폼
   
   둘 다 "AI 플레이그라운드"라는 개념적 영역 안에서 함께 동작
```

**핵심 포인트:**
- DLVM은 PAIS와 **독립적**으로 배포 가능 (vSphere 하이퍼바이저 위에 직접)
- DLVM은 **모든 개발자가 사용하는 것이 아님** - 주로 Data Scientist와 MLOps Engineer가 사용
- **App Developer는 DLVM 접속 없이** PAIS API URL만으로 앱 개발 가능

### 1.3 용어 혼동 주의: "AI 플레이그라운드" vs "Playground"

본 문서에서 사용하는 용어들의 정확한 의미:

| 용어 | 의미 | 범위 | 해당 제품/기능 |
|------|------|------|----------------|
| **AI 플레이그라운드** | PAIF/PAIS 기반 AI 개발/운영 전체 공간 | 개념적 영역 (DEV/QA/STAGING/PROD 포함) | 없음 (개념) |
| **PAIS Playground** | Agent Builder 내 대화형 테스트 UI | PAIS UI의 특정 기능 | PAIS Agent Builder |
| **LLM Playground** | 프롬프트 테스트 환경 (업계 일반 용어) | PAIF 용어 아님 | OpenAI, HuggingFace 등 |

> **💡 핵심:** 본 문서의 "AI 플레이그라운드"는 특정 UI 화면이 아닙니다.
> 인프라팀이 PAIF/PAIS 환경을 구축하면 자동으로 형성되는 **개념적 영역**으로,
> 개발자들이 DLVM, Model Endpoint, Agent 등을 자유롭게 실험하고 배포하는 공간입니다.

### 1.4 PAIS가 제공하는 서비스

| 모듈 | 역할 | 대상 사용자 | 버전 정보 (PAIS 2.0.89) |
|------|------|------------|------------------------|
| **Model Gallery** | Harbor 기반 ML 모델 저장소 (OCI 아티팩트) | MLOps Engineer | Harbor Registry |
| **Model Runtime** | LLM/Embedding 모델을 API Endpoint로 자동 배포 | MLOps Engineer | vLLM 0.6.5, Infinity 0.0.43 |
| **Data Indexing & Retrieval** | Knowledge Base 관리, 벡터 인덱싱 자동화 | Data Scientist | pgvector 0.8.0 (PostgreSQL 16.8) |
| **Agent Builder** | RAG 에이전트 GUI 구성 도구 + Playground | Data Scientist, MLOps | Built-in UI |

### 1.5 핵심 컴포넌트 버전 매트릭스 (v3.0 신규)

| 컴포넌트 | PAIS 2.0.89 버전 | 비고 |
|----------|-----------------|------|
| vLLM | 0.6.5 | Completion 모델용 추론 엔진 |
| Infinity | 0.0.43 | Embedding 모델용 추론 엔진 |
| PostgreSQL | 16.8 | DSM 제공 |
| pgvector | 0.8.0 | 벡터 DB 확장 |
| NVIDIA GPU Operator | 24.9.0 | 기본값 (오버라이드 가능) |
| VKr (Kubernetes Release) | 1.32 | ClusterClass builtin-generic-v3.2.0 |

### 1.6 GPU 지원 옵션 (v3.0 신규)

PAIF에서 GPU를 사용하는 두 가지 방식:

| 방식 | 설명 | 요구 사항 | 장점 | 단점 |
|------|------|----------|------|------|
| **vGPU** | GPU를 여러 VM에 분할 공유 | NVIDIA AI Enterprise (NVAIE) 라이선스 | 유연한 리소스 할당, vMotion 지원 | 라이선스 비용 |
| **DirectPath I/O** | GPU를 단일 VM에 패스스루 | NVAIE 라이선스 불필요 | 라이선스 비용 절감, 전체 GPU 성능 | VM당 1GPU, vMotion 제한 |

> **참고:** DLVM 및 VKS 모두 DirectPath I/O 모드로 PAIS 사용 가능 (2026.01 검증됨)

---

## 2. 페르소나 및 역할 정의

### 2.1 AI 프로젝트 핵심 역할

| 역할 | 실제 하는 일 | 사용 도구 | 결과물 |
|------|-------------|----------|--------|
| **Data Scientist** | 모델 선택/평가, 프롬프트 엔지니어링, RAG 파이프라인 설계 | DLVM JupyterLab | "최적의 모델 + 설정 + 프롬프트" 가이드 |
| **MLOps Engineer** | 모델 배포, 버전 관리, Endpoint 생성, 인프라 운영 | DLVM CLI, PAIS UI, Harbor | API Endpoint URL + 인증 정보 |
| **App Developer** | API 호출 코드, 비즈니스 로직, UI/UX 개발 | **로컬 PC** (VS Code 등) | 실제 서비스 앱 |
| **Platform Engineer** | VCF/PAIF 인프라 구축 및 운영 | vSphere Client, VCF Automation | AI 플레이그라운드 환경 |
| **VI Admin** | GPU 호스트 관리, vGPU 드라이버, 네트워킹 | ESXi, vCenter | GPU 가속 Workload Domain |

### 2.2 역할별 DLVM 필요 여부

| 역할 | DLVM 필요 여부 | 이유 |
|------|---------------|------|
| **Data Scientist** | ✅ 필수 | GPU로 모델 테스트, 프롬프트 튜닝 필요 |
| **MLOps Engineer** | ✅ 필요 | 모델 Push용 CLI, PAIS UI는 브라우저 |
| **App Developer** | **❌ 불필요** | **API URL만 있으면 로컬 PC에서 개발 가능** |
| **Platform Engineer** | ❌ 불필요 | vSphere/VCF 관리 도구 사용 |

### 2.3 Data Scientist (데이터 사이언티스트) 상세

#### 2.3.1 RAG 파이프라인이란?

> **RAG = Retrieval-Augmented Generation**  
> "검색 → 증강 → 생성" 의 **데이터 흐름**을 의미합니다. (CI/CD 파이프라인과는 다른 개념!)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         RAG 파이프라인 (데이터 흐름)                         │
│                                                                             │
│  [사용자 질문]                                                               │
│       │                                                                     │
│       │ "회사 휴가 정책이 뭐야?"                                             │
│       │                                                                     │
│       ▼                                                                     │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐                  │
│  │ 1. Embed    │────▶│ 2. Search   │────▶│ 3. Retrieve │                  │
│  │ 질문을 벡터로│     │ 벡터 DB에서 │     │ 관련 문서   │                  │
│  │ 변환        │     │ 유사 문서   │     │ 조각 가져옴 │                  │
│  │             │     │ 검색        │     │             │                  │
│  └─────────────┘     └─────────────┘     └──────┬──────┘                  │
│                                                  │                         │
│                                                  │ "HR_Policy.pdf 12페이지: │
│                                                  │  연차는 15일이며..."     │
│                                                  │                         │
│                                                  ▼                         │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │                        4. Generate (LLM)                              │ │
│  │                                                                       │ │
│  │  [시스템 프롬프트] + [검색된 문서] + [사용자 질문]                     │ │
│  │                          ↓                                            │ │
│  │  "회사 휴가 정책에 따르면, 정규직 직원은 연차 15일이 부여됩니다..."    │ │
│  │                                                                       │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### 2.3.2 PAIS가 제공하는 것 vs Data Scientist가 결정하는 것

| 구분 | PAIS가 제공 (인프라/도구) | Data Scientist가 결정 (설계/최적화) |
|------|--------------------------|-----------------------------------|
| **문서 청킹** | 청킹 기능 자체 | chunk_size=500? 1000? overlap=50? 100? |
| **임베딩** | 임베딩 엔진 (Infinity) | 어떤 모델? bge-small? bge-large? multilingual? |
| **벡터 검색** | pgvector DB, 검색 기능 | top_k=3? 5? 10? similarity_threshold=0.7? 0.8? |
| **LLM 추론** | vLLM 엔진 | 어떤 모델? Llama3? Mistral? temperature=0.7? |
| **프롬프트** | Agent Builder GUI | 시스템 프롬프트 내용 (품질에 가장 큰 영향!) |

**비유:**
```
PAIS = 주방 (오븐, 냄비, 재료)
Data Scientist = 요리사 (레시피, 온도 설정, 조리 시간 결정)

주방이 있다고 요리가 자동으로 되는 게 아님!
```

#### 2.3.3 JupyterLab에서 하는 구체적인 일

**1. 청킹 전략 실험**

```python
# DLVM JupyterLab에서 실험

from langchain.text_splitter import RecursiveCharacterTextSplitter

# 청킹 전략 A: 작은 청크
splitter_a = RecursiveCharacterTextSplitter(
    chunk_size=300,
    chunk_overlap=30
)

# 청킹 전략 B: 큰 청크
splitter_b = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=100
)

# 둘 다 테스트해서 어떤 게 더 좋은 답변을 만드는지 비교
# → "우리 문서는 chunk_size=500, overlap=50이 최적" 결론 도출
```

> **왜 이게 중요한가?**
> - 청크가 너무 작으면 → 문맥이 끊김
> - 청크가 너무 크면 → 관련 없는 내용도 포함됨

**2. 임베딩 모델 선택**

```python
# 여러 임베딩 모델 비교

models = [
    "BAAI/bge-small-en-v1.5",    # 작고 빠름, 영어 전용
    "BAAI/bge-base-en-v1.5",     # 중간 크기
    "BAAI/bge-m3",               # 다국어 지원 (한국어 포함)
    "intfloat/multilingual-e5-large"  # 다국어, 대형
]

# 각 모델로 테스트 쿼리 실행
# → "우리 데이터는 한국어가 많으니 bge-m3가 최적" 결론 도출
```

**3. 프롬프트 엔지니어링**

```python
# 시스템 프롬프트 최적화

prompt_v1 = "당신은 도움이 되는 AI입니다."
# → 너무 일반적, 응답이 장황함

prompt_v2 = """당신은 HR 정책 전문 어시스턴트입니다.
규칙:
1. 제공된 문서 내용만 기반으로 답변하세요
2. 확실하지 않으면 "확인이 필요합니다"라고 말하세요
3. 답변은 3문장 이내로 간결하게 하세요
4. 출처 문서명을 항상 언급하세요"""
# → 훨씬 정확하고 일관된 응답

# 반복 테스트 후 최적 프롬프트 확정
```

### 2.4 MLOps Engineer 상세

#### 2.4.1 주요 업무

```
┌────────────────────────────────────────────────────────────────────────────┐
│                         MLOps Engineer 워크플로우                          │
│                                                                            │
│  1. 모델 소싱                                                               │
│     └─▶ Hugging Face, NGC에서 모델 다운로드                                 │
│                                                                            │
│  2. 모델 검증                                                               │
│     └─▶ 보안 스캔, 라이선스 확인, 성능 벤치마크                             │
│                                                                            │
│  3. 모델 등록                                                               │
│     └─▶ Harbor Model Gallery에 Push (vcf pais CLI 사용)                    │
│                                                                            │
│  4. Endpoint 생성                                                           │
│     └─▶ PAIS UI/API로 Model Endpoint 배포                                  │
│                                                                            │
│  5. 운영 및 모니터링                                                        │
│     └─▶ GPU 사용률, 응답 시간, 오류율 모니터링                              │
│                                                                            │
│  [산출물] API Endpoint URL + 인증 정보 → App Developer에게 전달             │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

#### 2.4.2 vcf pais CLI 사용 (⚠️ Deprecated 주의)

> **⚠️ 주의:** `pais` CLI 1.0.0은 DLVM 9.0.1에서 Deprecated 되었으며, 다음 릴리즈에서 제거될 예정입니다.  
> 향후 VCF Consumption CLI의 새로운 방식으로 대체될 예정입니다.

```bash
# 현재 방식 (Deprecated 예정)
vcf pais models push \
  --modelName baai/bge-small-en-v1.5 \
  --modelStore harbor.company.com/models \
  -t v1

# 주의사항:
# 1. 모델 폴더 내에서 실행해야 함 (cd 필수)
# 2. 모델명은 DNS 명명 규칙 준수 (소문자, 공백 없음)
# 3. Harbor TLS 인증서를 시스템 신뢰 저장소에 추가 필요
```

### 2.5 App Developer 상세

#### 2.5.1 App Developer의 핵심 이해

> **App Developer는 AI 전문가가 아닙니다.**  
> REST API를 호출하는 일반 개발자입니다.

| 구분 | Data Scientist | App Developer |
|------|----------------|---------------|
| **주 업무** | 모델 선택, 프롬프트 최적화, RAG 설계 | UI/UX 개발, 비즈니스 로직 구현 |
| **사용 도구** | JupyterLab, Python, ML 프레임워크 | VS Code, React/Vue, FastAPI |
| **필요 지식** | ML/DL, NLP, 통계 | 웹 개발, REST API, 소프트웨어 공학 |
| **산출물** | "최적의 모델+설정" 가이드 | 실제 서비스 앱 |
| **DLVM 필요** | ✅ 필수 | ❌ 불필요 |

#### 2.5.2 App Developer 워크플로우

```
[App Developer 작업 환경: 로컬 PC]

1. MLOps로부터 수령
   • PAIS Agent API URL: https://pais.company.com/v1/agents/hr-assistant/chat
   • 인증 정보: API Token 또는 OAuth 설정

2. 로컬 개발 환경에서 작업
   • VS Code, WebStorm 등 IDE
   • React, Vue, Next.js 등 프론트엔드
   • FastAPI, Node.js 등 백엔드

3. API 연동 코드 작성
   • 일반적인 REST API 호출과 동일
   • 회사 VPN으로 내부망 접근 (필요시)

4. 테스트 및 배포
   • 로컬 테스트 → CI/CD → VKS 배포
```

---

**[Part 1 완료]**

다음 Part 2에서는 다음 내용을 다룹니다:
- 3장: 아키텍처 개요 (계층 구조, 구성 요소 간 관계)
- 4장: 구축 순서 및 의존성 (Phase별 설치 순서, 의존성 다이어그램)
