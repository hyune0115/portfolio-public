# Traefik 전역 TLS 정책과 레거시 클라이언트 호환성 충돌 대응

## 1. 배경

### 가. 증상

1) 특정 외부 연동 시스템이 TLS 1.0으로만 handshake를 요청하는데, Traefik(v2.10.0)의 기본
   `minTLSVersion`이 1.2로 설정되어 있어 해당 시스템과의 TLS 통신이 handshake 단계에서
   실패하는 오류가 발생 — 원인 진단부터 조치까지 단독 수행

### 나. 원인

1) 해당 서비스의 Ingress에 Traefik `TLSOption`이 별도로 정의되어 있지 않아, Traefik 전역
   기본 TLS 정책(최소 버전 1.2)이 그대로 적용되고 있었음
2) 외부 연동 시스템 쪽이 레거시 TLS 1.0까지만 지원하는 구버전 클라이언트여서 최신 보안
   기본값과 충돌

## 2. 기대효과

### 가. 전역 보안 저하 없는 호환성 확보

1) Traefik 전역 `minTLSVersion`을 낮추는 대신, 해당 서비스에만 적용되는 전용 `TLSOption`을
   정의해 완화된 TLS 범위를 그 서비스로만 한정

### 나. 재사용 가능한 조치 패턴 확보

1) `TLSOption` CRD + `allowCrossNamespace` 조합으로 서비스 단위 TLS 정책을 격리하는 패턴을
   확보해, 유사한 레거시 연동 요구사항 발생 시 동일하게 적용 가능

### 다. AS-IS / TO-BE 비교

| 항목 | AS-IS | TO-BE |
|---|---|---|
| TLS 정책 적용 범위 | Traefik 전역 기본값(minVersion 1.2) 단일 적용 | 해당 서비스만 전용 TLSOption(1.0~1.3)으로 스코프 분리 |
| 레거시 클라이언트 호환성 | Handshake 실패 | 정상 통신 |
| 나머지 서비스 보안 수준 | - | 전역 기본값(1.2 이상) 그대로 유지 |
| TLSOption 참조 범위 | 기본은 동일 네임스페이스만 | `allowCrossNamespace`로 다른 네임스페이스 IngressRoute에서도 참조 |

## 3. 구성도 (조치 흐름)

```
STEP 1  특정 외부 연동 시스템과의 TLS handshake 오류 발생 확인
  │
  ▼
STEP 2  openssl s_client -tls1로 TLS 1.0 handshake 재현해 원인 확정
  │
  ▼
STEP 3  전용 TLSOption 정의 (minVersion TLS1.0 ~ maxVersion TLS1.3)
  │
  ▼
STEP 4  allowCrossNamespace 활성화 + 해당 서비스 IngressRoute에서 TLSOption 참조
  │
  ▼
STEP 5  helm upgrade 반영, 계획된 다운타임(약 5분) 내 정상화 확인
```

## 4. 성과

1) 레거시 TLS 1.0만 지원하는 외부 연동 시스템과의 호환성 문제를, 클러스터 전체의 TLS 보안
   수준을 낮추지 않고 해당 서비스 하나로 스코프를 좁혀 해결
2) Traefik `TLSOption` CRD와 `allowCrossNamespace` 조합을 활용한 서비스 단위 TLS 정책
   격리 패턴을 확보

---

## 상세 문서

TLSOption/IngressRoute 매니페스트 전문과 진단 커맨드 전체를 포함한 상세 문서는 비공개
리포지토리에 별도 보관 중입니다. 필요 시 요청해주시면 공유드리겠습니다.
