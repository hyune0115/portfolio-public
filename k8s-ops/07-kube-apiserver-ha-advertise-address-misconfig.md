# kube-apiserver HA 오설정으로 인한 kubeadm 업그레이드 실패 대응

## 1. 배경

### 가. 증상

1) 3대 컨트롤플레인 HA 클러스터에서 kubeadm 패치 버전 업그레이드 도중 업그레이드가 완주되지
   않음
2) 업그레이드 창 동안 3대 컨트롤플레인의 kubelet이 동시에 apiserver probe 실패를 기록 —
   원인 진단부터 조치까지 단독 수행

### 나. 원인

1) 3대 컨트롤플레인 모두 kube-apiserver의 `--advertise-address`가 각 노드의 실제 IP가 아니라
   LB VIP로 동일하게 오설정
2) 이 값이 엔드포인트 등록, kubelet probe host, controller-manager/scheduler kubeconfig까지
   총 5곳에 그대로 파생되어, 노드별로 독립이어야 할 상태 판정이 LB라는 단일 지점에 묶임
3) 업그레이드 중 LB의 backend health-check가 일시적으로 에러를 내자 3대 kubelet의 probe가
   동시에 실패 — 롤링 업그레이드가 "이 노드만의 상태"를 판별하지 못해 완주 실패

## 2. 기대효과

### 가. 구조적 원인의 소스 레벨 규명

1) 증상 대응에 그치지 않고 kubeadm 내부 동작(소스 코드 수준)까지 근거를 추적해, 하나의
   설정값 오류가 왜 5곳에 동시에 영향을 미치는지 구조적으로 설명

### 나. 상관 장애(Correlated Failure) 위험의 사전 식별

1) HA 구성에서 "노드별로 달라야 하는 값"과 "클러스터 공통 값"이 섞이면, 평시에는 드러나지
   않다가 롤링 업그레이드 같은 특정 시점에만 노드 3대가 동시에 실패하는 구조적 위험을 도출

### 다. AS-IS / TO-BE 비교

| 항목 | AS-IS (오설정) | TO-BE (조치 후) |
|---|---|---|
| `--advertise-address` | 3대 모두 LB VIP로 동일 설정 | 각 노드의 실제 인터페이스 IP |
| `kubectl get endpoints kubernetes` | VIP 1개만 등록 | 노드 실 IP 3개 모두 등록 |
| kubelet probe 대상 | 3대 모두 LB VIP 감시 (상관 장애 위험) | 각자 자기 노드의 apiserver 감시 |
| controller-manager/scheduler kubeconfig | LB VIP 경유 | 같은 노드 apiserver에 직통 |
| 롤링 업그레이드 | 노드별 상태 판별 불가로 완주 실패 | 노드별 독립 판정 가능 |

## 3. 구성도 (원인 → 조치 흐름)

```
STEP 1  kubeadm 패치 업그레이드 중 3대 kubelet 동시 probe 실패 확인
  │
  ▼
STEP 2  진단 — endpoints가 VIP 1개뿐임을 확인, 노드별 advertise-address가 VIP로 오설정된 것을 특정
  │
  ▼
STEP 3  근본 원인 규명 — advertise-address가 엔드포인트/probe host/kubeconfig 5곳에 파생되는
        구조를 kubeadm 소스 기준으로 확인
  │
  ▼
STEP 4  조치 — kube-apiserver 매니페스트 5곳을 노드 실 IP로 수정, controller-manager/scheduler
        kubeconfig 재생성, 한 대씩 순차 적용·검증
  │
  ▼
STEP 5  재발 방지 — "노드별 값 vs 클러스터 공통 값" 구분 원칙을 체크리스트로 정리
```

## 4. 성과

1) 단순 증상 대응이 아니라 kubeadm 소스 코드 수준까지 근거를 추적해 근본 원인을 구조적으로
   규명
2) 컨트롤플레인 추가/재설치 시 상시 확인 가능한 체크 커맨드를 재발 방지 대책으로 확보
3) HA 구성에서 발생할 수 있는 상관 장애 위험을 사전 식별 가능한 진단 기준을 도출

---

## 상세 문서

진단·조치에 사용한 전체 커맨드, kubeadm 소스 코드 근거, 노드별 값 표까지 포함한 상세 문서는
비공개 리포지토리에 별도 보관 중입니다. 필요 시 요청해주시면 공유드리겠습니다.
