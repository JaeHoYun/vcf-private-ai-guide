# VMware Private AI Foundation 가이드 - Part 4-2
## AI 앱 개발 이해 (8장)

> **문서 버전:** 3.1 (v3.1 업데이트)  
> **기반 버전:** VCF 9.0.1, PAIF 9.0, PAIS 2.0.89, DLVM 9.0.1

---

## 8. AI 앱 개발 이해

이 장에서는 PAIF 환경에서 AI 애플리케이션을 개발할 때 알아야 할 핵심 개념과 아키텍처를 설명합니다. 인프라 담당자와 솔루션 아키텍트가 개발팀과 협업하거나, 고객에게 전체 그림을 설명할 때 필요한 관점을 제공합니다.

> **💡 핵심 전제:** PAIS는 별도 솔루션이 아니라 **PAIF에 포함된 모듈**입니다. PAIF 라이선스에 PAIS가 포함되어 있으므로, AI 플레이그라운드에서 PAIS를 활용한 개발이 **기본 권장 방식**입니다.

### 8.1 일반 웹앱 vs AI 앱 구조 비교

#### 8.1.1 전통적인 웹 애플리케이션 구조

전통적인 웹 애플리케이션은 Frontend, Backend, Database로 구성된 **3-Tier 아키텍처**를 따릅니다.

<img width="789" height="440" alt="image" src="https://github.com/user-attachments/assets/0dd45283-7e59-49fa-8fcc-3035831bcd1c" />

#### 8.1.2 AI 애플리케이션 구조

AI 애플리케이션은 전통적인 3-Tier에 **AI 서비스 계층**이 추가된 구조입니다.

<img width="792" height="627" alt="image" src="https://github.com/user-attachments/assets/f8923ee9-208b-4d8d-a1be-1700b49065c6" />

#### 8.1.3 핵심 차이점

| 관점 | 전통적 웹앱 | AI 앱 |
|------|-----------|-------|
| **응답 특성** | 결정적 (Deterministic) | 확률적 (Probabilistic) |
| **로직 구현** | 코드로 명시적 작성 | 모델이 학습된 패턴으로 추론 |
| **응답 시간** | 빠름, 예측 가능 (ms) | 느림, 가변적 (초) |
| **리소스** | CPU 중심 | GPU 필수 (LLM 추론) |
| **데이터 흐름** | 정형 데이터 CRUD | 비정형 데이터 → 벡터 → 추론 |
| **에러 처리** | 명확한 예외 처리 | 환각(Hallucination) 관리 필요 |
| **테스트** | 단위/통합 테스트 | 품질 평가 (정확도, 관련성) |

> **💡 인프라 관점:** AI 앱은 GPU 리소스가 필수이며, 응답 시간이 초 단위로 길어지므로 타임아웃 설정, 비동기 처리, 스트리밍 응답 등을 고려해야 합니다.

---

### 8.2 AI 앱의 계층별 역할

#### 8.2.1 4-Tier 아키텍처 개념

AI 애플리케이션은 **Frontend → Backend → AI Service → Infrastructure** 4개 계층으로 구성됩니다.

<img width="396" height="894" alt="image" src="https://github.com/user-attachments/assets/1f74ce85-7716-4c69-8a72-f176de6a41c6" />

#### 8.2.2 계층별 책임 분리

| 계층 | 책임 | PAIS가 담당 | 개발팀이 담당 |
|------|------|------------|--------------|
| **Frontend** | UI/UX, 사용자 경험 | ❌ | ✅ 전체 |
| **Backend** | 비즈니스 로직, 인증 | ❌ | ✅ 전체 |
| **AI Service** | LLM 추론, RAG | ✅ Agent/Endpoint | 프롬프트, KB 구성 |
| **Infrastructure** | GPU, 컴퓨팅 | ✅ 자동 프로비저닝 | ❌ |

> **💡 핵심 이해:** PAIS는 **Tier 3 (AI Service)**를 담당합니다. 개발팀은 Tier 1-2에 집중하고, AI 로직은 PAIS Agent API를 호출하여 처리합니다.

#### 8.2.3 "PAIS Agent = AI 두뇌" 개념

PAIS Agent가 제공하는 것과 개발팀이 구현해야 하는 것의 경계를 명확히 이해해야 합니다.

<img width="433" height="660" alt="image" src="https://github.com/user-attachments/assets/12463438-7939-4490-b564-ed6e5c0d8572" />

---

### 8.3 PAIS API 연동 개념

#### 8.3.1 PAIS API 유형

PAIS는 두 가지 주요 API를 제공합니다:

| API 유형 | 용도 | 엔드포인트 | 사용 시나리오 |
|---------|------|-----------|--------------|
| **Model Endpoint API** | 단순 LLM 추론 | \`/v1/chat/completions\` | RAG 없이 직접 LLM 호출 |
| **Agent API** | RAG 통합 추론 | \`/v1/agents/{name}/chat\` | KB 기반 Q&A (권장) |

<img width="439" height="601" alt="image" src="https://github.com/user-attachments/assets/e7be80aa-bfcf-4615-99ad-135b1fe7b2b1" />

#### 8.3.2 인증 흐름

PAIS API 호출에는 인증이 필요합니다. VCF의 OIDC 기반 인증을 사용합니다.

<img width="514" height="511" alt="image" src="https://github.com/user-attachments/assets/9285e4b0-2bbe-4006-afcc-9ff612f65010" />

#### 8.3.3 API 호출 패턴

**Agent API 호출 예시 (curl):**

\`\`\`bash
# Agent API 호출
curl -X POST 'https://pais.company.com/v1/agents/hr-assistant/chat' \
  -H 'Authorization: Bearer <access_token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "message": "연차 휴가는 며칠인가요?",
    "session_id": "user-123-session-456"
  }'
\`\`\`

**응답 구조:**

\`\`\`json
{
  "response": "회사 휴가 정책에 따르면, 정규직 직원은 연간 15일의 연차 휴가가 부여됩니다...",
  "references": [
    {
      "source": "HR_Policy_2026.pdf",
      "page": 12,
      "content": "정규직 직원의 연차 휴가는 연간 15일이며..."
    }
  ],
  "session_id": "user-123-session-456"
}
\`\`\`

> **💡 개발팀 전달 정보:** MLOps/Data Scientist가 Agent 생성 후 다음 정보를 App Developer에게 전달합니다:
> - PAIS Base URL
> - Agent Name
> - 인증 정보 (Client ID/Secret 또는 Token Endpoint)
> - 사용 가이드 (세션 관리, 타임아웃 권장값 등)

---

### 8.4 배포 아키텍처

#### 8.4.1 AI 앱 배포 위치

AI 애플리케이션은 **VKS(VMware Kubernetes Service) 클러스터**에 배포하는 것이 권장됩니다.

<img width="442" height="626" alt="image" src="https://github.com/user-attachments/assets/09d5dde7-3c50-42f0-b99b-7885d788e69c" />

#### 8.4.2 배포 방식 비교

| 배포 방식 | 설명 | 장점 | 단점 | 권장 상황 |
|----------|------|------|------|----------|
| **VKS 배포** | Kubernetes에 컨테이너로 배포 | 스케일링, 롤링 업데이트, 고가용성 | 초기 설정 복잡 | 프로덕션 |
| **DLVM 직접 실행** | DLVM 내에서 직접 서비스 실행 | 빠른 테스트 가능 | 스케일링 어려움, 단일 장애점 | 개발/PoC |
| **외부 배포** | 플레이그라운드 외부에 배포 | 기존 인프라 활용 | 네트워크 설정 복잡 | 특수한 경우 |

> **💡 권장:** 개발/테스트는 DLVM에서 진행하고, 프로덕션은 VKS에 배포합니다.

#### 8.4.3 컨테이너화 개념

프로덕션 배포를 위해 앱을 컨테이너 이미지로 패키징합니다.

<img width="300" height="837" alt="image" src="https://github.com/user-attachments/assets/98ddb1ce-a704-4a3c-98e8-d6bc4bc4e847" />

#### 8.4.4 VKS 배포 시 필요한 리소스

| Kubernetes 리소스 | 용도 | 설명 |
|------------------|------|------|
| **Deployment** | 앱 Pod 관리 | Replicas, 롤링 업데이트 정의 |
| **Service** | 내부 네트워킹 | Pod 간 통신, ClusterIP |
| **Ingress** | 외부 노출 | L7 라우팅, TLS, 도메인 매핑 |
| **ConfigMap** | 설정 관리 | 환경 변수, 설정 파일 |
| **Secret** | 민감 정보 | API 키, 인증 정보 |
| **PVC** | 영구 스토리지 | 파일 업로드 저장 (필요시) |

---

### 8.5 운영 고려사항

#### 8.5.1 모니터링

AI 앱 운영 시 모니터링해야 할 핵심 지표:

<img width="298" height="631" alt="image" src="https://github.com/user-attachments/assets/be772391-42d4-4a11-90b4-8ddd13adbdda" />

#### 8.5.2 보안 고려사항

| 영역 | 고려사항 | 권장 방안 |
|------|---------|----------|
| **인증/인가** | API 접근 제어 | OIDC 토큰 기반, RBAC 적용 |
| **데이터 보호** | 민감 정보 노출 방지 | 입력/출력 필터링, PII 마스킹 |
| **네트워크** | 통신 암호화 | TLS 필수, 내부 통신도 암호화 권장 |
| **프롬프트 인젝션** | 악의적 입력 방어 | 입력 검증, 샌드박싱 |
| **모델 보안** | 모델 탈취 방지 | Harbor 접근 제어, 네트워크 격리 |
| **감사 로깅** | 규정 준수 | 모든 API 호출 로깅, 보관 정책 |

> **⚠️ 프롬프트 인젝션 주의:** 사용자 입력이 그대로 LLM에 전달되면, 악의적 사용자가 시스템 프롬프트를 우회하거나 의도치 않은 동작을 유발할 수 있습니다. 입력 검증과 샌드박싱이 필요합니다.

#### 8.5.3 비용 고려사항

<img width="310" height="639" alt="image" src="https://github.com/user-attachments/assets/57a27a4c-a915-4ad0-b4d6-ae993ef343df" />

#### 8.5.4 성능 최적화 팁

| 영역 | 문제 | 최적화 방안 |
|------|------|------------|
| **응답 시간** | LLM 추론이 느림 (수 초) | 스트리밍 응답 활용, 사용자에게 진행 상태 표시 |
| **동시 처리** | 많은 요청 시 병목 | Model Endpoint Replicas 증가, 자동 스케일링 |
| **검색 품질** | RAG 결과가 부정확 | Chunk 크기 조정, Similarity Cutoff 튜닝 |
| **대화 맥락** | 긴 대화 시 맥락 손실 | Chat History Length 조정, 요약 기법 적용 |
| **첫 응답** | Cold Start 지연 | 최소 Replicas 1 이상 유지, 워밍업 요청 (Cold Start의 상세 원인과 완화 전략은 Part 5 9.5.3절 참조) |

---

### 8.6 개발 환경 vs 프로덕션 환경

#### 8.6.1 환경별 차이

| 항목 | 개발/테스트 | 프로덕션 |
|------|------------|----------|
| **배포 위치** | DLVM 직접 실행 | VKS 클러스터 |
| **모델 크기** | 소형 (8B) | 요구사항에 맞게 선택 |
| **Replicas** | 1 (최소) | 2+ (고가용성) |
| **스케일링** | 수동 | 자동 스케일링 |
| **모니터링** | 기본 로그 | 전체 스택 모니터링 |
| **보안** | 내부망 | TLS, 인증 강화, 감사 로깅 |
| **데이터** | 테스트 데이터 | 실제 데이터 (백업 필수) |

#### 8.6.2 환경 전환 체크리스트

<img width="283" height="794" alt="image" src="https://github.com/user-attachments/assets/a0b3744a-7017-4045-b87c-a494220e75b8" />

---

### 8.7 정리: AI 앱 개발의 핵심 포인트

<img width="383" height="635" alt="image" src="https://github.com/user-attachments/assets/1462dcd8-8b77-46bf-8f54-bcfe6a534dea" />

---

**[Part 4-2 완료]**

다음 Part에서 다루는 추가 주제:
- **Part 5 (9장):** 프로덕션 아키텍처 패턴 (HA, DR, 멀티사이트, Cold Start 최적화)
- 10장: FAQ 및 트러블슈팅
- 부록: 버전 호환성 매트릭스, 용어집
