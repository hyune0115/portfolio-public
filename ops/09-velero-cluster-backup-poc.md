# Velero 기반 Kubernetes 클러스터 백업/마이그레이션 PoC

## 1. 배경

### 가. 목적

1) 그룹사 SaaS 서비스 환경의 Kubernetes object 및 PV 백업 체계 마련을 위해 Velero 기반 백업 PoC 진행
2) 백업 스토리지는 그룹사가 이미 보유한 NetApp ONTAP(S3 호환 API) 활용을 계획했으나, PoC 시점 펌웨어가
   필요 API를 지원하지 않아 MinIO로 대체 검증 — 이후 펌웨어 업그레이드가 완료되어 ONTAP 대상 재PoC 예정

## 2. 기대효과

### 가. 실사용 가능 여부를 명확히 규명

1) 되는 것과 안 되는 것을 구체적인 제약사항까지 포함해 PoC로 검증 — hostPath 볼륨 등 Kubernetes 표준 볼륨이 아닌 영역은 백업이 불가하다는 것을 명확히 확인해, 도입 여부를 근거 있게 판단할 수 있는 자료 확보

### 나. 정량적 데이터 정합성 검증

1) Redis/MariaDB/ElasticSearch/Kafka 등 모듈별로 마이그레이션 전후 데이터를 1:1 비교해 백업/복원 신뢰성을 수치로 검증

### 다. AS-IS / TO-BE 비교

| 항목 | AS-IS | TO-BE(PoC 결과) |
|---|---|---|
| 백업 체계 | 미구축 | MinIO(오브젝트 스토리지, NetApp ONTAP 펌웨어 제약으로 대체 검증) + Velero 기반 백업 체계 검증 |
| 백업 범위 | 미검증 | K8s object·PV는 가능, hostPath 볼륨은 불가로 명확화 |
| 클러스터 마이그레이션 | 미검증 | 1-node/3-node 환경에서 백업→복원 시나리오 검증 완료 |
| 데이터 정합성 | 미검증 | 모듈별(Redis/MariaDB/ES/Kafka) 정량 비교로 검증 |

## 3. 구성도 (백업/마이그레이션 흐름)

![오픈소스·상용 백업 PoC 교차 검증 흐름](images/09-10-backup-poc-cross-validation.svg)

```
STEP 1  MinIO(오브젝트 스토리지) 설치 — Velero 백업 대상 스토리지
  │
  ▼
STEP 2  Velero 설치 및 MinIO 연동 (File-system 백업 활성화)
  │
  ▼
STEP 3  네임스페이스 단위 백업/복원 테스트
  │
  ▼
STEP 4  클러스터 마이그레이션 테스트 (1-node / 3-node)
  │
  ▼
STEP 5  모듈별 데이터 정합성 정량 비교 (Redis/MariaDB/ES/Kafka)
  │
  ▼
STEP 6  제약사항 정리 및 PoC 결론 도출
```

## 4. 성과

1) Velero+MinIO 기반 K8s 클러스터 백업/마이그레이션 체계의 실사용 가능 여부를 구체적인 제약사항까지 포함해 명확히 규명
2) 데이터 정합성을 모듈별로 정량 비교해 백업/복원 신뢰성을 검증
3) 복원 시 Velero 네임스페이스 exclude 누락으로 인한 미완료 이슈 등 실전 운영 노하우를 확보·문서화

## 5. 향후 계획

1) NetApp ONTAP 펌웨어 업그레이드 완료 — 실제 운영 스토리지인 ONTAP 대상 재PoC 예정

---

## 상세 문서

설치 설정값, 제약사항 상세, 데이터 정합성 비교 수치 등을 포함한 상세 문서는 비공개 리포지토리에 별도 보관 중입니다. 필요 시 요청해주시면 공유드리겠습니다.
