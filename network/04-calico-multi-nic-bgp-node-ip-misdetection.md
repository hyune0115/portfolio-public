# 멀티 NIC 환경에서 Calico BGP Node IP 오선택 장애 대응

## 1. 배경

### 가. 증상

1) 멀티 NIC(다중 네트워크 인터페이스) 환경에서 서버 재부팅 또는 Calico 재기동 시 BGP
   피어링이 정상적으로 형성되지 않아 Pod 간 통신이 실패하는 장애 발생 — 원인 진단부터
   조치까지 단독 수행, 실제 다운타임 발생

### 나. 원인

1) Calico의 `nodeAddressAutodetectionV4` 설정이 `firstFound: true`로 되어 있어, "가장
   먼저 발견된 인터페이스"의 IP를 BGP Node IP로 자동 선택하는 방식으로 동작
2) 인터페이스 초기화 순서는 노드마다 달라질 수 있어, 멀티 NIC 환경에서는 일부 노드가 BGP
   통신용이 아닌 관리망 인터페이스의 IP를 선택 — 노드 간 BGP Node IP 대역이 어긋나며 피어
   정보 불일치 발생

## 2. 기대효과

### 가. 결정적(Deterministic) 동작으로 전환

1) "먼저 발견되는 순서"라는 비결정적 조건에 의존하던 방식을, 실제 BGP 통신에 쓸 네트워크
   대역을 명시적으로 지정하는 방식으로 전환해 재부팅/재기동 시점과 무관하게 항상 동일하게
   동작하도록 개선

### 나. 클러스터 구성 방식별 대응 기준 확보

1) 직접 설치형(`k8s,bgp`)과 Tigera Operator 관리형(`k8s,Operating,bgp`) 각각에 맞는 조치
   방법을 구분해 문서화, 향후 유사 환경에서도 바로 적용 가능

### 다. AS-IS / TO-BE 비교

| 항목 | AS-IS | TO-BE |
|---|---|---|
| BGP Node IP 감지 방식 | `firstFound`(가장 먼저 발견된 인터페이스) | 대역 명시(`cidr`/`cidrs`)로 결정적 지정 |
| 멀티 NIC 환경 안정성 | 재부팅/재기동 시 대역이 달라질 수 있음 | 항상 동일한 대역의 IP가 선택됨 |
| 노드 간 BGP 피어 정보 | 대역 불일치로 피어링 실패 가능 | 일관된 대역으로 피어링 안정화 |

## 3. 구성도 (조치 흐름)

```
STEP 1  멀티 NIC 환경에서 BGP 피어링 실패로 Pod 통신 장애 발생 확인
  │
  ▼
STEP 2  calicoctl node status로 노드 간 PEER ADDRESS 대역 불일치 확인
  │
  ▼
STEP 3  CLUSTER_TYPE 확인 — 직접 설치형 vs Tigera Operator 관리형 분기
  │
  ▼
STEP 4  nodeAddressAutodetectionV4를 firstFound → cidrs(또는 DaemonSet env)로 명시 지정
  │
  ▼
STEP 5  재기동 후 피어링 정상화 확인
```

## 4. 성과

1) "가장 먼저 발견된 인터페이스"라는 암묵적 가정에 의존하던 설정을 명시적 CIDR 지정으로
   전환해, 멀티 NIC 환경에서도 결정적으로 BGP 통신 인터페이스가 선택되도록 근본 해결
2) 클러스터 구성 방식에 따라 조치 방법을 구분해 향후 유사 환경에서도 재사용 가능한 대응
   기준을 확보

---

## 상세 문서

`calicoctl node status` 원본 출력, DaemonSet/Installation 리소스 수정 전문을 포함한 상세
문서는 비공개 리포지토리에 별도 보관 중입니다. 필요 시 요청해주시면 공유드리겠습니다.
