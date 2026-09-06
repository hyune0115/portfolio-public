# 네이버클라우드(NCP) 공공클라우드 SaaS PoC

## 1. 배경

### 가. PoC 개요

1) 온프레미스 구축형으로만 제공되던 Amaranth10을 네이버클라우드(NCP) 공공클라우드 환경에서
   SaaS로 제공하기 위한 CSAP 인증 목표의 PoC (2023.06 ~ 12)
2) 관리형 Kubernetes(NKS), 관리형 DB/캐시/검색엔진 등을 우선 활용하는 구성으로 검증하고,
   호환성 문제 시 그룹사와 동일한 구축형 Kubernetes 환경으로 전환하는 것을 목표로 진행

### 나. 본인 역할

1) 팀 내 역할은 인프라 구성과 SaaS 배포로 분담되어 있었고, 본인은 NKS 구축, 관리형 서비스
   (Redis/Elasticsearch/Kafka) 연동, 네트워크·로드밸런서·SSL/TLS 구성을 담당
2) 애플리케이션 배포와 DB 마이그레이션은 협업 동료가 별도로 담당

## 2. 기대효과

### 가. 관리형 서비스 이식 가능성 실전 검증

1) 자체 구축 대신 NCP 관리형 Kubernetes/DB/캐시/검색엔진을 활용해, 온프레미스 구축형
   아키텍처가 관리형 서비스 환경에서도 동작 가능한지를 실제 환경에서 검증

### 나. 관리형 서비스 특유 제약에 대한 우회 방안 확보

1) 관리형 Redis의 인증 방식 제약, LB당 SSL 인증서 개수 제한, mqtt 트래픽 타겟 설정 미지원 등
   자체 구축 환경에서는 없던 관리형 서비스 특유의 제약을 실제로 겪고 각각의 우회 방안을 도출

### 다. AS-IS / TO-BE 비교

| 항목 | AS-IS (온프레미스 구축형) | TO-BE (NCP 공공클라우드 SaaS PoC) |
|---|---|---|
| Kubernetes | 자체 구축(native) | 관리형 Kubernetes(NKS) |
| Redis/ES/Kafka | 자체 구축 | NCP 관리형 서비스로 전환 |
| Redis 인증 | 기본(root) 계정 인증 | ACL만 지원 → NKS 내 Pod로 별도 기동해 우회 |
| SSL 인증서 처리 | 제한 없음 | LB당 등록 개수 제한(25개) → 별도 Nginx VM으로 분리 |
| mqtt 트래픽 | LB에서 직접 처리 | Public LB 타겟 설정 미지원 → 전용 NLB 별도 구성 |

## 3. 조치 흐름

```
STEP 1  NKS 구축 및 인프라 성격 컴포넌트(Mqtt/Nginx/Auth/Daemon/Mail) 배포
  │
  ▼
STEP 2  Elasticsearch/Redis/Kafka를 NCP 관리형 서비스로 전환, Private LB로 연동
  │
  ▼
STEP 3  관리형 Redis 인증 제약 확인 → NKS 내 Redis를 Pod로 별도 기동해 우회
  │
  ▼
STEP 4  LB당 SSL 인증서 개수 제한 확인 → 별도 Nginx VM으로 SSL 처리 분리
  │
  ▼
STEP 5  Public LB의 mqtt 타겟 설정 미지원 확인 → mqtt 전용 NLB 구성, TLS 오프로딩 처리
  │
  ▼
STEP 6  Ingress Controller(ALB) 설정 변경 시 IP/CNAME 변경 이슈 → 이미지 패치로 해결
```

## 4. 성과

1) 이 PoC는 기술적 문제가 아니라 회사의 SaaS 인증 우선순위가 타 제품의 IaaS 기반 CSAP 인증으로
   전환되며 종료됨
2) 관리형 Kubernetes/DB/캐시/검색엔진을 온프레미스 구축형 아키텍처와 호환되도록 이식하며,
   관리형 서비스 특유의 제약을 실제로 겪고 우회 방안을 도출한 실전 경험 확보

---

## 상세 문서

PoC 진행 일정, 담당자별 세부 업무 분담 등 세부 내용을 포함한 상세 문서는 비공개 리포지토리에
별도 보관 중입니다. 필요 시 요청해주시면 공유드리겠습니다.
