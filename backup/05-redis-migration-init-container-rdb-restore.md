# Kubernetes 클러스터 간 Redis 마이그레이션 — Init Container 기반 RDB 복원 자동화

## 1. 배경

### 가. 기존 문제

1) 구 클러스터(A) → 신규 클러스터(B) 인프라 이전 작업에서, 트래픽 차단 직후 생성된 최종
   RDB 스냅샷을 신규 클러스터에 신속히 복원해야 했으나, Redis는 기동 시점에만 `dump.rdb`를
   읽어 로드하는 엔진 특성상 볼륨에 스냅샷을 사전 주입해둬야 하는 제약이 있었음
2) StatefulSet을 Scale-down(0)한 뒤 볼륨에 직접 파일을 주입하려 했으나, 블록 스토리지
   기반 StorageClass의 ReadWriteOnce(RWO) 제약으로 Pod 미기동 상태에서는 외부 접근이나
   임시 Pod 마운트 자체가 불가능해 교착 상태 발생

## 2. 기대효과

### 가. 컷오버 절차 단순화

1) 수동 볼륨 마운트나 임시 Pod를 통한 CLI 데이터 주입 과정 없이, StatefulSet 배포
   즉시 데이터가 자동으로 복원되도록 구성

### 나. RWO 제약의 K8s 네이티브 해결

1) Pod가 기동되며 RWO 볼륨이 마운트되는 시점을 그대로 활용 — Redis 메인 컨테이너가
   시작되기 직전(Init Container 단계)에 스냅샷을 `/data`에 배치해, 별도 스토리지
   마이그레이션 도구 없이 K8s Pod 생명주기만으로 병목을 해소

### 다. AS-IS / TO-BE 비교

| 항목 | AS-IS | TO-BE |
|---|---|---|
| 스냅샷 복원 방식 | Scale-down 후 볼륨 직접 주입 시도 | Init Container가 기동 시점에 자동 배치 |
| RWO 볼륨 접근 | Pod 미기동 상태에서 접근 불가 → 교착 | Pod 기동과 동시에 마운트되는 시점을 활용 |
| 복원 대상 | 전체 노드에 개별 주입 검토 | Master 우선 복원 → Replica는 복제로 동기화 |
| 검증 | 해당 없음 | 복제 로그 + Redis key 개수 비교로 정합성 확인 |

## 3. 구성도 (복원 흐름)

```
STEP 1  A 클러스터 트래픽 차단 직후 SAVE/BGSAVE로 최종 RDB 스냅샷 확보
  │
  ▼
STEP 2  kubectl cp(로컬 추출) → scp(B 클러스터 대상 노드 hostPath 전송)
  │
  ▼
STEP 3  B 클러스터 Master Pod 기동 — Init Container가 hostPath를 마운트해
        RDB를 /data로 복사, 메인 컨테이너가 기동하며 자동 로드
  │
  ▼
STEP 4  Master 정상 기동 확인 후 Replica 순차 Scale-out
        (Replica는 RDB 직접 복원 없이 Master로부터 정상 복제로 동기화)
  │
  ▼
STEP 5  복제 로그 + Redis key 개수 비교로 데이터 정합성 검증
```

## 4. 성과

1) 수동 볼륨 마운트나 임시 Pod를 통한 CLI 데이터 주입 과정 없이, StatefulSet 배포만으로
   데이터가 자동 복원되도록 컷오버 절차를 단순화
2) RWO 블록 스토리지 환경에서 발생한 볼륨 접근 병목을 별도 도구 없이 K8s Pod
   생명주기(Init Container)만으로 해결
3) Sentinel/Replica 구조에서 Master 우선 복원 → Replica 순차 Scale-out 방식을 적용해,
   전체 노드에 개별적으로 파일을 주입할 필요 없이 복제 메커니즘을 그대로 활용

---

## 상세 문서

최종 스냅샷 확보·전달 절차, Init Container 구성, Master/Replica 순차 적용 및 검증 절차 등
세부 내용을 포함한 상세 문서는 비공개 리포지토리에 별도 보관 중입니다. 필요 시 요청해주시면
공유드리겠습니다.
