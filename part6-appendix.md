# VMware Private AI Foundation 가이드 - Part 6
## FAQ, 버전 호환성 매트릭스, 참고 자료 (10장 + 부록)

> **문서 버전:** 3.1 (v3.1 업데이트)  
> **기반 버전:** VCF 9.0.1, PAIF 9.0, PAIS 2.0.89, DLVM 9.0.1

---

## 10. FAQ (자주 묻는 질문)

### 10.1 PAIF/PAIS 기본 개념

#### Q1: PAIF와 PAIS의 관계는 무엇인가요?

**A:** PAIS(Private AI Services)는 PAIF(Private AI Foundation)에 **포함된 모듈**입니다.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           VMware Cloud Foundation                           │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                    PAIF (Private AI Foundation)                     │  │
│   │                                                                     │  │
│   │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │  │
│   │   │    PAIS     │  │    DLVM     │  │   기타      │                 │  │
│   │   │  (서비스)    │  │   (개발VM)  │  │  구성요소   │                 │  │
│   │   │             │  │             │  │             │                 │  │
│   │   │ • Model EP  │  │ • JupyterLab│  │ • Harbor    │                 │  │
│   │   │ • Agent     │  │ • vLLM      │  │ • DSM       │                 │  │
│   │   │ • KB        │  │ • PyTorch   │  │ • VCF Auto  │                 │  │
│   │   └─────────────┘  └─────────────┘  └─────────────┘                 │  │
│   │                                                                     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│   💡 PAIF 라이선스 = PAIS 포함 (별도 라이선스 불필요. PAIF 도 VCF 9 에 포함)   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

#### Q2: PAIS 없이 DLVM만으로 AI 서비스를 구축할 수 있나요?

**A:** 기술적으로는 가능하지만, **권장하지 않습니다.**

| 항목 | DLVM Only | PAIS 활용 |
|------|-----------|-----------|
| 모델 서빙 | vLLM 직접 설치/운영 | Model Endpoint (자동) |
| RAG 구성 | LangChain 직접 코딩 | Agent Builder GUI |
| 벡터 DB | pgvector 직접 설정 | Knowledge Base (자동) |
| API Gateway | 직접 구현 | PAIS 제공 (OpenAI 호환) |
| 스케일링 | 수동 | 자동 스케일링 |
| 운영 부담 | 높음 | 낮음 |

**권장:** PAIF 라이선스에 PAIS가 포함되어 있으므로, PAIS를 활용하는 것이 기본입니다. DLVM은 모델 평가, 실험, 커스텀 개발 등의 목적으로 사용합니다.

---

#### Q3: "AI 플레이그라운드"는 어떤 제품인가요?

**A:** "AI 플레이그라운드"는 별도 제품이 아니라 **개념적 영역**입니다.

- PAIF/PAIS 환경이 구축되면 자동으로 형성되는 **AI 개발/운영 공간**
- 물리적 구성요소가 아닌 **논리적/정책적 경계**
- 개발/QA/스테이징/프로덕션 모두 이 영역 안에서 Supervisor Namespace로 분리

**⚠️ 용어 혼동 주의:**

| 용어 | 성격 | 설명 |
|------|------|------|
| **AI 플레이그라운드** | 개념적 영역 | PAIF/PAIS가 제공하는 전체 AI 개발/운영 공간 (추상적 개념) |
| **PAIS Playground** | 실제 UI 기능 | Agent Builder 내 테스트/프리뷰 화면 (구체적 기능) |

- "AI 플레이그라운드에서 개발한다" = PAIF 환경 전체를 활용한다는 의미
- "Playground에서 테스트한다" = Agent Builder UI의 테스트 기능 사용

> 📌 상세 용어 구분은 **Part 1의 1.3절** 참조

---

#### Q4: Model Endpoint와 Agent의 차이는 무엇인가요?

**A:** 

| 구분 | Model Endpoint | Agent |
|------|---------------|-------|
| **역할** | 단일 모델 API 노출 | RAG 파이프라인 전체 캡슐화 |
| **입력** | 프롬프트 + 대화 이력 | 사용자 질문만 |
| **출력** | LLM 응답만 | 응답 + 출처 + 세션 관리 |
| **RAG** | ❌ 없음 (직접 구현 필요) | ✅ 내장 |
| **Knowledge Base** | ❌ 연결 불가 | ✅ 연결 가능 |
| **세션 관리** | ❌ 없음 | ✅ 대화 히스토리 자동 유지 |
| **API** | `/v1/chat/completions` | `/v1/agents/{name}/chat` |
| **사용 시나리오** | 커스텀 RAG 구현, 단순 챗봇 | **일반적인 RAG 앱 (권장)** |

**권장:** 대부분의 경우 **Agent API** 사용. RAG를 직접 구현해야 할 특별한 이유가 없다면 Agent가 훨씬 간편합니다.

---

### 10.2 역할 및 워크플로우

#### Q5: "AI 앱 개발자"는 어떤 역할인가요?

**A:** "AI 앱 개발자"는 별도의 새로운 직군이 아닙니다.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    AI 앱 개발자 = 전통적인 개발자                            │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                                                                     │  │
│   │   AI 앱 개발자 = Frontend/Backend 개발자 + AI API 호출              │  │
│   │                                                                     │  │
│   │   하는 일:                                                          │  │
│   │   • React/Vue로 채팅 UI 개발                                        │  │
│   │   • FastAPI/Node.js로 Backend 개발                                  │  │
│   │   • PAIS Agent API 호출 (REST API)                                  │  │
│   │   • 비즈니스 로직 구현                                              │  │
│   │                                                                     │  │
│   │   하지 않는 일:                                                     │  │
│   │   • 모델 학습/튜닝 (Data Scientist 역할)                            │  │
│   │   • 모델 서빙 인프라 운영 (MLOps 역할)                              │  │
│   │   • RAG 파이프라인 설계 (Data Scientist 역할)                       │  │
│   │                                                                     │  │
│   │   필요한 것:                                                        │  │
│   │   • PAIS Agent API URL                                              │  │
│   │   • 인증 정보 (Token)                                               │  │
│   │   • 로컬 PC (DLVM 불필요!)                                          │  │
│   │                                                                     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

#### Q6: DLVM은 누가, 언제 사용하나요?

**A:** 

| 역할 | DLVM 사용 여부 | 용도 |
|------|---------------|------|
| **Data Scientist** | ✅ 필수 | 모델 평가, 프롬프트 튜닝, 실험 |
| **MLOps Engineer** | ✅ 필요 | 모델 다운로드, 검증, Harbor Push |
| **App Developer** | ❌ 불필요 | 로컬 PC에서 API 호출로 개발 |
| **Platform Engineer** | ❌ 불필요 | vSphere/VCF 관리 도구 사용 |

**핵심:** DLVM은 "GPU가 필요한 AI 작업"에만 사용합니다. API 호출만 하는 앱 개발에는 불필요합니다.

---

#### Q7: Data Scientist와 MLOps Engineer의 역할 차이는?

**A:**

| 구분 | Data Scientist | MLOps Engineer |
|------|----------------|----------------|
| **주 관심사** | "어떤 모델이 좋은가?" | "모델을 어떻게 서빙할까?" |
| **주요 업무** | 모델 선택/평가, 프롬프트 엔지니어링, RAG 설계 | 모델 배포, Endpoint 생성, 운영 |
| **산출물** | 최적 모델+설정+프롬프트 가이드 | API Endpoint URL + 인증 정보 |
| **사용 도구** | JupyterLab, Python, LangChain | Harbor, PAIS UI, CLI |
| **DLVM 사용** | 실험/평가용 | 모델 Push용 |

**협업 흐름:**
```
Data Scientist → "이 모델과 프롬프트가 최적입니다"
       ↓
MLOps Engineer → Harbor Push → Model Endpoint 생성 → API URL 전달
       ↓
App Developer → API URL로 앱 개발
```

---

### 10.3 기술 및 구성

#### Q8: Knowledge Base 인덱싱은 얼마나 걸리나요?

**A:** 문서 규모와 설정에 따라 다릅니다.

| 문서 규모 | 예상 시간 | 비고 |
|----------|----------|------|
| 100개 문서 (10MB) | 수 분 | 소규모 테스트 |
| 1,000개 문서 (100MB) | 10~30분 | 일반적인 부서 문서 |
| 10,000개 문서 (1GB) | 1~2시간 | 대규모 지식 베이스 |

**영향 요소:**
- 문서 형식 (PDF 파싱이 가장 오래 걸림)
- Chunk 크기 (작을수록 청크 수 증가)
- Embedding 모델 (대형 모델일수록 느림)
- Embedding Endpoint Replicas 수

**권장:** 대규모 인덱싱은 업무 시간 외에 수행하고, 이후 **증분 인덱싱** 활용

---

#### Q9: 어떤 LLM 모델을 선택해야 하나요?

**A:** 용도와 리소스에 따라 선택합니다.

| 모델 | 파라미터 | GPU 요구 | 적합한 용도 |
|------|---------|---------|------------|
| **Llama 3.1 8B** | 8B | A100 40GB x1 | 일반 Q&A, 문서 요약 (권장 시작점) |
| **Llama 3.1 70B** | 70B | A100 80GB x2+ | 복잡한 추론, 고품질 응답 |
| **Mistral 7B** | 7B | A100 40GB x1 | 빠른 응답, 코드 생성 |
| **Qwen 2.5** | 7B/72B | 다양 | 다국어, 아시아 언어 |

**선택 기준:**
1. **정확도 우선:** 큰 모델 (70B+)
2. **속도/비용 우선:** 작은 모델 (7B~8B)
3. **한국어:** Qwen 또는 한국어 파인튜닝 모델 고려

---

#### Q10: Embedding 모델은 GPU가 필수인가요?

**A:** 아니요, **CPU만으로도 가능**합니다.

| 실행 환경 | 지원 여부 | 성능 | 권장 상황 |
|----------|----------|------|----------|
| **GPU** | ✅ 지원 | 빠름 | 대규모 인덱싱, 실시간 임베딩 |
| **CPU** | ✅ 지원 (Infinity) | 중간 | 소규모, 비용 절감 |

PAIS의 Embedding Endpoint는 Infinity 엔진을 사용하며, CPU 전용 배포가 가능합니다. 이는 GPU 리소스가 제한된 환경에서 유용합니다.

---

#### Q11: Harbor에 모델을 Push하는 이유는 무엇인가요?

**A:** 여러 가지 이점이 있습니다.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Harbor 모델 레지스트리의 장점                              │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │   1. 버전 관리                                                      │  │
│   │      • 모델 버전별 태그 (v1.0, v1.1, latest)                        │  │
│   │      • 롤백 용이                                                    │  │
│   │                                                                     │  │
│   │   2. 접근 제어                                                      │  │
│   │      • 프로젝트별 권한 분리                                         │  │
│   │      • 누가 어떤 모델을 Push/Pull 했는지 감사                       │  │
│   │                                                                     │  │
│   │   3. 취약점 스캔                                                    │  │
│   │      • 컨테이너 이미지 보안 검사                                    │  │
│   │      • 정책 기반 배포 승인                                          │  │
│   │                                                                     │  │
│   │   4. 내부망 배포                                                    │  │
│   │      • 외부 NGC 의존 제거                                           │  │
│   │      • Air-Gap 환경 지원                                            │  │
│   │                                                                     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│   💡 Model Endpoint 생성 시 Harbor 경로만 지정하면 PAIS가 자동 Pull         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### 10.4 트러블슈팅

#### Q12: Model Endpoint 생성이 실패합니다. 어떻게 해야 하나요?

**A:** 다음 체크리스트를 확인하세요.

| 증상 | 가능한 원인 | 해결 방안 |
|------|------------|----------|
| Pending 상태 지속 | GPU 리소스 부족 | Supervisor Namespace GPU 쿼터 확인 |
| ImagePullBackOff | Harbor 인증 실패 | Harbor 프로젝트 권한, Secret 확인 |
| CrashLoopBackOff | 모델 로딩 실패 | GPU 메모리 vs 모델 크기 확인 |
| OOMKilled | 메모리 부족 | 더 작은 모델 또는 양자화 버전 사용 |

**디버깅 명령어:**
```bash
# Pod 상태 확인
kubectl get pods -n <namespace> -l app=model-endpoint

# Pod 이벤트 확인
kubectl describe pod <pod-name> -n <namespace>

# 로그 확인
kubectl logs <pod-name> -n <namespace>
```

---

#### Q13: Agent 응답에서 "문서를 찾을 수 없습니다"가 반복됩니다.

**A:** Knowledge Base 설정을 점검하세요.

| 체크 항목 | 확인 방법 |
|----------|----------|
| **인덱싱 완료 여부** | KB 상태가 "Ready"인지 확인 |
| **Embedding Endpoint 동작** | Embedding Endpoint 상태 확인 |
| **Chunk 설정** | Chunk 크기가 너무 크면 검색 정확도 저하 |
| **Top-K 설정** | 너무 낮으면 관련 문서 누락 (기본 5~10 권장) |
| **Similarity Threshold** | 너무 높으면 검색 결과 없음 (0.7~0.8 권장) |

**팁:** Agent Builder의 Playground에서 직접 질문하고, 검색된 청크를 확인하면 디버깅이 쉽습니다.

---

#### Q14: DLVM에서 NGC 모델 다운로드가 실패합니다.

**A:** NGC API Key 설정을 확인하세요.

| 문제 | 해결 방안 |
|------|----------|
| **401 Unauthorized** | NGC_API_KEY 환경 변수 설정 확인 |
| **Rate Limit** | 잠시 후 재시도 또는 Personal Key 사용 |
| **네트워크 오류** | Proxy 설정, 방화벽 확인 |

**NGC API Key 설정:**
```bash
# ~/.bashrc에 추가
export NGC_API_KEY="your-api-key"

# 또는 실행 시 지정
NGC_API_KEY="your-api-key" huggingface-cli download ...
```

**DLVM 9.0.1 주의사항:** vGPU 드라이버 다운로드가 packages.vmware.com에서 **NVIDIA NGC**로 변경되었습니다. Personal 또는 Service API Key가 필요합니다.

---

#### Q15: DLVM에서 vLLM 실행 시 GPU 메모리 부족 오류?

**A:** GPU 메모리 관리가 필요합니다.

| 해결 방안 | 설명 |
|----------|------|
| **gpu-memory-utilization 조정** | `--gpu-memory-utilization 0.8` (기본 0.9) |
| **더 작은 모델 사용** | 70B → 8B 모델로 변경 |
| **양자화 모델 사용** | GPTQ, AWQ 양자화 버전 |
| **tensor-parallel 사용** | 다중 GPU에 모델 분산 |
| **max-model-len 축소** | 컨텍스트 길이 제한 |

**GPU 메모리 요구량 참고:**
- 8B 모델 (FP16): ~16GB
- 70B 모델 (FP16): ~140GB (A100 80GB x2 필요)

---

### 10.5 라이선스 및 비용

#### Q16: PAIF 라이선스 구성은 어떻게 되나요?

**A:** PAIF는 VCF의 Add-on으로 제공됩니다.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    라이선스 구성                                             │
│                                                                             │
│   VCF (VMware Cloud Foundation)                                             │
│       │                                                                     │
│       └── PAIF Add-on (Private AI Foundation with NVIDIA)                   │
│               │                                                             │
│               ├── PAIS (Private AI Services) ← 포함                         │
│               ├── DLVM 이미지 ← 포함                                        │
│                                                                             │
│   💡 PAIF 라이선스에 PAIS가 포함되어 있으므로 별도 구매 불필요                 │
│                                                                             │
│   추가로 필요한 것:                                                          │
│   • NVIDIA vGPU 라이선스 & NVAIE 소프트웨어                                  │
│   • NGC API Key (vGPU 드라이버 다운로드용)                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

#### Q17: GPU 비용을 최적화하려면?

**A:** 다음 전략을 고려하세요.

| 전략 | 설명 | 절감 효과 |
|------|------|----------|
| **적정 모델 선택** | 8B로 충분하면 70B 사용 금지 | 높음 |
| **자동 스케일링** | 부하에 따라 Replicas 조정 | 중간 |
| **개발/테스트 분리** | 개발 환경은 최소 리소스 | 중간 |
| **Embedding CPU 사용** | GPU 대신 CPU로 Embedding | 낮음 |
| **스케줄링** | 야간/주말 Replicas 축소 | 낮음 |

**가장 큰 영향:** 모델 크기 선택 (70B vs 8B는 GPU 요구량 10배 이상 차이)

---

## 부록 A: 버전 호환성 매트릭스

### A.1 VCF / PAIF / PAIS 버전 매트릭스

| VCF 버전 | PAIF 버전 | PAIS 버전 | DLVM 버전 | 비고 |
|---------|----------|----------|----------|------|
| VCF 9.0 | PAIF 9.0 | PAIS 2.0.89 | DLVM 9.0.0 | 초기 릴리즈 |
| **VCF 9.0.1** | **PAIF 9.0** | **PAIS 2.0.89** | **DLVM 9.0.1** | **현재 권장** |

### A.2 주요 컴포넌트 버전

| 컴포넌트 | 버전 | 비고 |
|---------|------|------|
| vLLM | 0.6.5 | LLM 서빙 엔진 |
| Infinity | 0.0.43 | Embedding 서빙 엔진 |
| pgvector | 0.8.0 | 벡터 DB 확장 |
| GPU Operator | 24.9.0 | Kubernetes GPU 관리 |
| VKr (Kubernetes) | 1.32 | VKS 기반 버전 |
| NVIDIA vGPU | 17.x | vGPU 드라이버 |
| CUDA | 12.4 | DLVM 기본 CUDA |

### A.3 DLVM 9.0.1 변경사항

| 항목 | 9.0.0 | 9.0.1 |
|------|-------|-------|
| vGPU 드라이버 다운로드 | packages.vmware.com | **NVIDIA NGC** |
| NGC API Key | Legacy만 지원 | **Personal/Service Key 지원** |
| Container Toolkit | v1.16.x | **v1.17.1** |
| pais CLI | 지원 | **Deprecated 예정** |

> **⚠️ 주의:** `vcf pais` CLI는 DLVM 9.0.1에서 Deprecated 되었으며, 향후 제거 예정입니다.

---

## 부록 B: 용어집

| 용어 | 정의 |
|------|------|
| **Active-Active** | 양 사이트 동시 운영, 자동 Failover 지원하는 고가용성 DR 패턴 |
| **Agent** | Model Endpoint + Knowledge Base + Prompt를 결합한 RAG 캡슐화 |
| **Chunk** | 문서를 분할한 텍스트 조각 (RAG용) |
| **Cold Start** | 새 AI 서비스 인스턴스 시작 시 모델 로딩으로 인한 초기 지연. vLLM + GPU 메모리 적재 시간 포함 |
| **DirectPath I/O** | GPU 패스스루 (VM이 GPU 직접 액세스) |
| **DLVM** | Deep Learning VM - GPU 가속 개발 환경 가상머신 |
| **DSM** | Data Services Manager - 관리형 데이터베이스 서비스 |
| **Embedding** | 텍스트를 벡터로 변환한 수치 표현 |
| **Envoy Proxy** | PAIS ML API Gateway 내부에서 사용하는 고성능 L7 프록시 |
| **GPU Workload Domain** | GPU 호스트로 구성된 VCF Workload Domain |
| **GSLB** | Global Server Load Balancing - 지리적 분산 사이트 간 트래픽 분배. NSX ALB 또는 외부 LB로 구현 |
| **Hallucination** | LLM이 사실이 아닌 내용을 생성하는 현상 |
| **Harbor** | 컨테이너 이미지/모델 레지스트리 |
| **Infinity** | Embedding 모델 서빙 엔진 (CPU 지원) |
| **Knowledge Base** | 문서를 벡터화하여 저장하는 PAIS 관리형 저장소 |
| **Model Endpoint** | PAIS가 제공하는 관리형 LLM/Embedding 서빙 API |
| **NGC** | NVIDIA GPU Cloud - 모델/컨테이너 레지스트리 |
| **PAIF** | Private AI Foundation - VCF 기반 온프레미스 AI 플랫폼 |
| **PAIS** | Private AI Services - PAIF에 포함된 관리형 AI 서비스 모듈 |
| **pgvector** | PostgreSQL 벡터 검색 확장 |
| **Pilot Light** | 핵심 구성요소만 DR 사이트에 유지하는 최소 비용 DR 패턴 |
| **Prompt Engineering** | LLM 입력 최적화 기법 |
| **RAG** | Retrieval-Augmented Generation - 검색 기반 응답 생성 기법 |
| **RPO** | Recovery Point Objective - 데이터 손실 허용 최대 시점 |
| **RTO** | Recovery Time Objective - 장애 후 서비스 복구까지 허용 최대 시간 |
| **Similarity** | 벡터 간 유사도 (Cosine Similarity 등) |
| **Supervisor** | vSphere with Tanzu의 Kubernetes 컨트롤 플레인 |
| **Supervisor Namespace** | Supervisor 내 격리된 워크로드 영역 |
| **Token** | LLM이 처리하는 텍스트 단위 |
| **vGPU** | NVIDIA Virtual GPU - GPU 가상화 기술 |
| **VKS** | VMware Kubernetes Service - Supervisor 기반 K8s |
| **vLLM** | LLM 추론 최적화 엔진 (PagedAttention) |
| **vSphere HA** | ESXi 호스트 장애 시 VM을 다른 호스트에서 자동 재시작하는 VCF 고가용성 기능 |
| **Warm Standby** | 축소 규모로 DR 사이트 가동, 장애 시 스케일업하는 DR 패턴 |

---

## 부록 C: 참고 자료

### C.1 공식 문서

| 문서 | URL | 설명 |
|------|-----|------|
| VMware PAIF 공식 문서 | docs.vmware.com | 설치/구성 가이드 |
| VCF 9.0 Release Notes | docs.vmware.com | VCF 릴리즈 노트 |
| DLVM Release Notes | KB 문서 | DLVM 버전별 변경사항 |
| NVIDIA NGC Catalog | ngc.nvidia.com | 모델/컨테이너 카탈로그 |
| vLLM Documentation | docs.vllm.ai | vLLM 공식 문서 |
| LangChain Documentation | python.langchain.com | LangChain 프레임워크 |

### C.2 Broadcom 지원 리소스

| 리소스 | 용도 |
|--------|------|
| Broadcom Support Portal | 기술 지원, KB 문서 |
| Broadcom Community | 커뮤니티 포럼, 토론 |
| VMware Validated Designs | 검증된 아키텍처 설계 |

### C.3 NVIDIA 리소스

| 리소스 | 용도 |
|--------|------|
| NGC Catalog | AI 모델, 컨테이너, Helm 차트 |
| NVIDIA AI Enterprise | 엔터프라이즈 AI 소프트웨어 |
| NVIDIA Developer | 개발자 문서, 샘플 코드 |

---

## 부록 D: 문서 버전 히스토리

| 버전 | 날짜 | 주요 변경 사항 |
|------|------|--------------|
| 1.0 | 2025.01 | 초안 작성 |
| 2.0 | 2025.01 | 페르소나별 워크플로우 추가, RAG 파이프라인 상세화 |
| 2.1 | 2025.01 | 코드 예시 및 UI 목업 추가 |
| 2.2 | 2025.01.28 | FAQ 확장, 인증 흐름 상세화 |
| 3.0 | 2025.01.29 | 6개 파트 구조로 재편, DLVM 9.0.1 반영, 프로덕션 아키텍처 추가 |
| **3.1** | **2025.01.29** | **DR/HA/부하분산 개념 VCF 기반 설명 보강, 용어 구분 명확화 (AI 플레이그라운드 vs Playground), 코드 샘플 최적화** |

---

## 부록 E: 문서 구조 요약

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PAIF 완벽 가이드 v3.1 전체 구조                            │
│                                                                             │
│   Part 1: 1-2장                                                             │
│   └── 핵심 개념 정의, 페르소나 및 역할 정의                                    │
│                                                                             │
│   Part 2: 3-4장                                                             │
│   └── 아키텍처 개요, 구축 순서 및 의존성                                       │
│                                                                             │
│   Part 3: 5-6장                                                             │
│   └── AI 플레이그라운드 개념, 역할별 워크플로우                                │
│                                                                             │
│   Part 4: 7-8장                                                             │
│   └── 개발 환경별 시나리오, AI 앱 개발 이해                                   │
│                                                                             │
│   Part 5: 9장                                                               │
│   └── 프로덕션 아키텍처 (HA, DR, 멀티테넌트, 보안)                             │
│                                                                             │
│   Part 6: 10장 + 부록                                                       │
│   └── FAQ, 버전 호환성 매트릭스, 용어집, 참고 자료                             │
│                                                                             │
│   ═══════════════════════════════════════════════════════════════════════   │
│                                                                             │
│   대상 독자:                                                                │
│   • VMware 솔루션 아키텍트                                                   │
│   • 파트너 엔지니어                                                          │
│   • 고객사 인프라 담당자                                                     │
│   • AI/ML 엔지니어                                                          │
│   • 플랫폼 엔지니어                                                          │
│                                                                             │
│   핵심 메시지:                                                               │
│   • PAIS는 PAIF에 포함된 모듈 (별도 라이선스 불필요)                           │
│   • PAIS 활용이 기본 권장 방식                                               │
│   • AI 플레이그라운드에서 실험 → 프로덕션 전환이 자연스러움                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

**[Part 6 완료]**

**[PAIF 가이드 v3.1 전체 초안 완료]**
