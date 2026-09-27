# 애플리케이션 배포 매니페스트 표준화 PoC — Helm 공통 템플릿 전환 검증

## 1. 배경

### 가. 현행 배포 구조
1) WAS 업무 모듈마다 raw Deployment YAML을 하나씩 두고, ArgoCD가 디렉터리 전체를 동기화하는 구조 (모듈별 파일 34개)

### 나. 한계점
1) 모듈이 추가될 때마다 템플릿 구성과 필드 순서가 제각각이라 공통 설정의 일괄 수정·배포가 어려움
2) 이미지 레지스트리·태그가 34개 파일에 하드코딩되어, 버전을 올릴 때마다 모든 파일 수정
3) 기존 파일을 복사해 신규 모듈을 만드는 방식이라 롤링 업데이트 전략·서비스 계정·필드 순서 등 설정 drift 누적
4) 리소스·JVM 힙 설정이 모듈마다 제각각이고 기본값·산정 근거 부재

## 2. 기대효과

### 가. 운영 효율
1) **신규 모듈 추가 단순화**: 파일 복사 후 수정 → values 목록에 항목 추가
2) **일괄 변경**: 이미지 버전·공통 정책을 values/템플릿 한 곳에서 변경

### 나. 설정 일관성
1) **drift 차단**: 프로브·롤링 업데이트·노드 분산 정책을 템플릿에 고정
2) **리소스 기준 수립**: 기본값 fallback + 모듈별 산정 근거를 values에 기록

### 다. AS-IS / TO-BE 비교

| 항목 | AS-IS | TO-BE |
|---|---|---|
| 매니페스트 구성 | 모듈별 raw YAML 34개 | 공통 템플릿 1개 + 모듈 목록(values) |
| 신규 모듈 추가 | 기존 파일 복사 후 수정 | values에 항목 추가 |
| 이미지 버전 교체 | 34개 파일 수정 | values 한 줄 수정 |
| 공통 정책(프로브·전략) | 파일마다 상이 | 템플릿에 고정 |
| 리소스 설정 | 모듈별 임의 값, 기본값 없음 | 기본값 fallback + 산정 기준 |

## 3. 변경 구성도 (Topology)

### 가. AS-IS

```
webapps/deploy/
├── deploy-<module-a>.yml   (115 lines, image/tag hardcoded)
├── deploy-<module-b>.yml   (170 lines, different field order)
└── ... x34
        │
        ▼
ArgoCD Application (directory.recurse) ──▶ Cluster
```

### 나. TO-BE

```
values.yaml (defaults + module list)
        │
        ▼
_pods.tpl  "deploy.webapp"  ── range modules ──▶ Deployment x N
_svc.tpl / _log4j.tpl       ── range modules ──▶ Service / ConfigMap
        │
        ▼
ArgoCD Application (Helm) ──▶ Cluster
```

## 4. PoC 결과

1) 파일럿 모듈로 렌더링·동작을 검증하고, 기존 매니페스트와 대조해 동일성 확인
2) 이미 raw 매니페스트 구조로 운영 중인 고객사들의 업데이트 로직까지 바꿔야 하는 부담으로 운영 적용은 보류, PoC로 마무리

## 5. 역할

1) PoC 기획·차트 설계·템플릿 구현·검증 단독 수행, 모듈별 리소스 산정만 팀원과 공동 진행
