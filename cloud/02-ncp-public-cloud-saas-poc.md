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

1) 자체 구축 대신 NCP 관리형 Kubernetes/DB/검색엔진 등을 활용해, 온프레미스 구축형
   아키텍처가 관리형 서비스 환경에서도 동작 가능한지를 실제 환경에서 검증

### 나. 관리형 서비스 특유 제약에 대한 우회 방안 확보

1) 관리형 Redis의 인증 방식 제약(결국 관리형 채택 포기), LB당 SSL 인증서 개수 제한, LB가 다른
   LB를 타겟으로 잡지 못하는 제약 등 자체 구축 환경에서는 없던 관리형 서비스 특유의 제약을
   실제로 겪고 각각의 우회 방안을 도출

### 다. AS-IS / TO-BE 비교

| 항목 | AS-IS (온프레미스 구축형) | TO-BE (NCP 공공클라우드 SaaS PoC) |
|---|---|---|
| Kubernetes | 자체 구축(native) | 관리형 Kubernetes(NKS) |
| Elasticsearch/Kafka | 자체 구축 | NCP 관리형 서비스로 전환 |
| Redis | 자체 구축 | 관리형 전환 시도 → 인증 방식 제약(ACL만 지원)으로 채택 포기, NKS 내 Pod로 자체 구축 유지 |
| SSL 인증서 처리 | 제한 없음 | LB당 등록 개수 제한(25개) → 별도 Nginx VM으로 분리 |
| mqtt 트래픽 | LB에서 직접 처리 | 최상단 LB가 mqtt 전용 LB를 타겟으로 등록하는 LB-to-LB 구성 미지원 → mqtt LB를 최상단으로 재배치, 서버를 직접 타겟으로 |

## 3. 조치 흐름

```
STEP 1  NKS 구축 및 인프라 성격 컴포넌트(Mqtt/Nginx/Auth/Daemon/Mail) 배포
  │
  ▼
STEP 2  Elasticsearch/Kafka를 NCP 관리형 서비스로 전환, Private LB로 연동
  │
  ▼
STEP 3  Redis도 관리형 전환 시도 → 인증 방식 제약(ACL만 지원) 확인 → 채택 포기, NKS 내
        Pod로 자체 구축 유지
  │
  ▼
STEP 4  LB당 SSL 인증서 개수 제한 확인 → 별도 Nginx VM으로 SSL 처리 분리
  │
  ▼
STEP 5  최상단 LB가 mqtt 전용 LB를 타겟으로 등록하는 LB-to-LB 구성 미지원 확인 → mqtt LB를
        최상단으로 재배치, 서버를 직접 타겟으로, TLS 오프로딩 처리
  │
  ▼
STEP 6  Ingress Controller(ALB) 설정 변경 시 IP/CNAME 변경 이슈 → 이미지 패치로 해결
```

## 4. 성과

1) mqtt LB-to-LB 타겟팅 제약은 [`온프레미스 솔루션의 퍼블릭 클라우드 공통 아키텍처
   설계`](01-public-cloud-nlb-alb-architecture.md)에서 다룬 "LB가 다른 LB를 타겟으로 잡는 기능은
   클라우드마다 다르다(AWS의 NLB→ALB target type은 AWS 고유 기능)"는 제약을 NCP에서도 실제로
   재확인한 사례
2) 이 PoC는 기술적 문제가 아니라 회사의 SaaS 인증 우선순위가 타 제품의 IaaS 기반 CSAP 인증으로
   전환되며 종료됨
3) 관리형 Kubernetes/DB/검색엔진을 온프레미스 구축형 아키텍처와 호환되도록 이식하며, 관리형
   서비스 특유의 제약을 실제로 겪고 우회 방안을 도출한 실전 경험 확보

---

## 상세 문서

PoC 진행 일정, 담당자별 세부 업무 분담 등 세부 내용을 포함한 상세 문서는 비공개 리포지토리에
별도 보관 중입니다. 필요 시 요청해주시면 공유드리겠습니다.
