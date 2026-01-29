# VCF 9 Private AI Foundation 가이드

VCF(VMware Cloud Foundation) 기반 Private AI 인프라 구축 및 운영을 위한 실무 가이드입니다.

## 📋 개요

**대상 독자:** VMware 솔루션 아키텍트, 파트너 엔지니어, 인프라/플랫폼 담당자, AI/ML 엔지니어, 데이터 사이언티스트, 개발자

**기반 버전:** VCF 9.0.1, PAIF 9.0, PAIS 2.0.89, DLVM 9.0.1

**문서 버전:** 3.1 (2026년 1월)

## 📖 목차

| 파트 | 제목 | 주요 내용 |
|------|------|----------|
| [Part 1](part1-introduction.md) | 개념 정리 및 AI 앱 개발 워크플로우 | PAIF/PAIS/DLVM 개념, 용어 정의, 전체 흐름 이해 |
| [Part 2](part2-architecture.md) | 아키텍처 개요 및 구축 순서 | VCF 기반 AI 인프라 아키텍처, 구성요소별 역할, 구축 순서 |
| [Part 3](part3-playground.md) | AI 플레이그라운드 및 역할별 워크플로우 | PAIS Playground, 페르소나별(Data Scientist, MLOps, AI App Dev) 워크플로우 |
| [Part 4-1](part4-1-dev-scenarios.md) | 개발 환경별 시나리오 | DLVM, VKS, PAIS 환경별 개발 시나리오 및 선택 가이드 |
| [Part 4-2](part4-2-ai-app-dev.md) | AI 앱 개발 이해 | RAG 파이프라인, API 연동, 실제 개발 패턴 |
| [Part 5](part5-production.md) | 프로덕션 아키텍처 | HA/DR, 스케일링, 모니터링, 운영 베스트 프랙티스 |
| [Part 6](part6-appendix.md) | FAQ 및 참고 자료 | 자주 묻는 질문, 버전 호환성 매트릭스, 참고 링크 |

## 🚀 빠른 시작

- **"PAIF가 뭔가요?"** → [Part 1](part1-introduction.md)부터 시작
- **"아키텍처 구성이 궁금해요"** → [Part 2](part2-architecture.md)
- **"개발자로서 뭘 할 수 있나요?"** → [Part 3](part3-playground.md) + [Part 4-1](part4-1-dev-scenarios.md)
- **"프로덕션 운영은?"** → [Part 5](part5-production.md)
- **"특정 질문이 있어요"** → [Part 6](part6-appendix.md) FAQ 확인

## 📌 주요 용어

| 용어 | 설명 |
|------|------|
| **VCF** | VMware Cloud Foundation - 통합 SDDC 플랫폼 |
| **PAIF** | Private AI Foundation - VCF 기반 AI 인프라 솔루션 |
| **PAIS** | Private AI Services - AI 서비스 레이어 (Model Endpoint, RAG, Agent Builder) |
| **DLVM** | Deep Learning VM - GPU 워크로드용 가상머신 |
| **VKS** | vSphere Kubernetes Service - vSphere 네이티브 K8s |

## 📜 라이선스

이 문서는 자유롭게 활용하실 수 있습니다. **출처 표기**를 부탁드립니다.

```
출처: https://github.com/JaeHoYun/vcf-private-ai-guide
```

## 🔄 업데이트 이력

- **v3.1 (2026-01):** 전체 구조 재정비, HA/DR 섹션 강화, 코드 예제 간소화
- **v3.0 (2025-12):** 초기 공개 버전

## 💬 피드백

오류 발견, 개선 제안, 질문은 [Issues](../../issues)에 남겨주세요.

---

⚠️ 면책 조항 (Disclaimer)

비공식 문서
이 가이드는 VCF 공식 기술 문서를 기반으로 작성된 비공식 실무 가이드입니다. Broadcom, NVIDIA 또는 기타 벤더의 공식 입장을 대변하지 않습니다.
정확성 및 최신성

본 문서의 내용은 작성 시점(2026년 1월) 기준이며, 제품 업데이트에 따라 내용이 달라질 수 있습니다.
정확성을 위해 노력하였으나, 오류나 누락이 있을 수 있습니다.
프로덕션 환경 적용 전 반드시 공식 문서를 확인하시기 바랍니다.

책임 한계

본 문서를 참고하여 발생한 직접적, 간접적 손해에 대해 작성자는 책임을 지지 않습니다.
실제 구축 및 운영은 각 조직의 요구사항과 환경에 맞게 검토 후 진행하시기 바랍니다.
기술 지원이 필요한 경우 브로드컴 공식 지원 채널을 이용하시기 바랍니다.

상표권 고지

VMware, VMware Cloud Foundation, vSphere, vSAN, NSX, VCF Automation, VCF Operation, Private AI Foundation 등은 Broadcom 의 등록 상표입니다.
NVIDIA, CUDA, NIM 등은 NVIDIA Corporation의 등록 상표입니다.
기타 언급된 제품명 및 회사명은 각 소유자의 상표 또는 등록 상표입니다.
