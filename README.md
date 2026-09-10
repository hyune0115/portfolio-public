# infra-portfolio-summary

인프라/SRE 엔지니어링 프로젝트를 기획서 형태로 간단히 정리한 요약 포트폴리오입니다.
각 문서는 배경 → 기대효과/AS-IS-TO-BE → 구성도 → 성과 순으로 정리되어 있습니다.
(상세 기술 문서는 비공개로 별도 관리 중이며, 필요 시 요청하시면 공유드립니다.)

## Summary

### 핵심 성과

- **ArgoCD-GitLab 연쇄 장애 대응** — 일일 장애 발생 고객사 75-80% 감소
  ([자세히](cicd/02-argocd-gitlab-cascading-failure.md))
- **CSAP 인증심사 대응 및 유지보수** — kube-bench 기반 141개 항목 전수 진단, 즉시조치 30건 +
  Kyverno 기반 중기조치 27건으로 CSAP 인증 요건 충족, 이후 유지·갱신 심사에 필요한 반복 증적을
  자동 생성하는 체계까지 구축해 현재도 운영 중
  ([진단·조치 자세히](security/01-k8s-cis-benchmark-remediation.md) ·
  [유지보수 자동화 자세히](ops/05-csap-maintenance-automation.md))
- **Kubernetes 클러스터 구성 자동화 4-Tier 리팩토링** — 설치 소요시간 약 20% 단축, 전체 신규
  구축 표준으로 적용 ([자세히](iac/01-k8s-install-automation-refactor.md))
- **Prometheus + Grafana 커스텀 모니터링 대시보드 구축** — 별도 모니터링 서버 없이 기존 자산만으로
  K8s+서버 통합 관측성 체계를 구축해 전체 배포 표준으로 반영
  ([자세히](monitoring/01-grafana-dashboard-custom-build.md))
- **대규모 고객사 솔루션 신규 구축 프로젝트 리딩** — 6개 이상 클러스터 규모의 구축 프로젝트를
  설치계획부터 오픈 지원까지 전 주기 단독 리딩, 정상 서비스 오픈으로 완료
  ([자세히](project/01-solution-installation-project-lead.md))

### 기술적 문제 해결 사례

**k8s-ops**

- **네트워크** — 고객사 NAC 장비의 주기적 ARP 갱신과 Ingress의 노드별 개별 MAC 광고가
  충돌해 발생한 서비스 접속 장애를, 패킷 덤프 기반으로 여러 원인 후보를 배제해가며 근본 원인을
  특정하고 MetalLB 도입으로 해결 ([자세히](k8s-ops/01-nac-arp-mac-floating-conflict.md))
- **컨트롤플레인/HA** — kube-apiserver `--advertise-address` 오설정이 엔드포인트·kubelet
  probe·kubeconfig 5곳에 파생되어 발생한 상관 장애를 kubeadm 소스 코드 수준까지 추적해 규명하고,
  HA 클러스터 업그레이드를 완주시킴 ([자세히](k8s-ops/07-kube-apiserver-ha-advertise-address-misconfig.md))
- **OS/프로세스** — TCP probe가 MySQL 핸드셰이크 없이 연결을 열고 닫아 발생하던 Aborted
  connect 로그 폭증의 원인을 규명하고, exec 기반 probe로 전환해 근본적으로 해결
  ([자세히](k8s-ops/06-mariadb-healthcheck-probe-tcp-abort.md))
- **런타임/커널** — RHEL9 전환(cgroup v1→v2) 이후 JVM이 컨테이너 limit이 아닌 호스트
  전체 메모리 기준으로 힙을 계산해 발생한 OOM을, 커널 파라미터만 바꿔 재현하는 교차 검증으로
  원인을 cgroup 버전에 정확히 특정하고 JDK 업그레이드로 근본 해결
  ([자세히](k8s-ops/03-redhat9-cgroupv2-jdk-oom.md))
- **네트워크** — 멀티 NIC 환경에서 Calico가 "먼저 발견된 인터페이스"로 BGP Node IP를
  암묵적으로 선택하다 재부팅마다 통신이 끊기던 구조적 결함을, 명시적 대역 지정 방식으로 전환해
  재발 가능성 자체를 제거 ([자세히](k8s-ops/09-calico-multi-nic-bgp-node-ip-misdetection.md))

**cicd**

- **인증/보안** — 전체 고객사 프로젝트가 root 계정의 공용 Access Token을 공유하던 구조를
  분석해, 레거시 GitLab 버전 제약까지 고려한 프로젝트/용도별 분리 발급 체계로 재설계
  ([자세히](cicd/01-gitlab-pat-redesign.md))

**ops**

- **스토리지/컨테이너 런타임** — OpenEBS NDM이 백업 솔루션의 가상 디바이스 메타데이터를
  가져오지 못해 스캔이 행(hang)에 걸리며 메모리가 누적되는 근본 원인을 로그 분석과 공식 문서로
  규명하고, 설정 변경만으로 다운타임 없이 해결
  ([자세히](ops/07-openebs-ndm-memory-leak.md))
- **네트워크/OS** — 고객사가 커널 레벨에서 비활성화한 IPv6와 일부 애플리케이션의 IPv6 소켓
  바인딩이 충돌해 발생한 기동 오류를 규명하고, IPv4 단일스택으로 통일해 재발 자체를 차단
  ([자세히](ops/08-ipv6-disable-app-bind-failure.md))

## 목차

### [k8s-ops/](k8s-ops/) - Kubernetes 인프라 운영, 장애대응

- [고객사 NAC 장비 ARP 갱신 × Ingress 개별 IP 광고 충돌 대응](k8s-ops/01-nac-arp-mac-floating-conflict.md)
- [Traefik 서비스 설정 누락으로 인한 클라이언트 IP 미보존](k8s-ops/02-traefik-service-externaltrafficpolicy-missing.md)
- [RHEL9 전환에 따른 cgroup v2 미인식으로 인한 JVM 애플리케이션 OOM](k8s-ops/03-redhat9-cgroupv2-jdk-oom.md)
- [kube-proxy 모드 IPVS Deprecated 대응 — IPVS에서 iptables로 재전환](k8s-ops/04-kubeproxy-mode-ipvs-to-iptables.md)
- [Go 애플리케이션 재기동 시 좀비 프로세스/CPU 부하 대응](k8s-ops/05-go-zombie-process-graceful-shutdown.md)
- [MariaDB Pod Healthcheck Probe 개선 — TCP → Exec 전환](k8s-ops/06-mariadb-healthcheck-probe-tcp-abort.md)
- [kube-apiserver HA 오설정으로 인한 kubeadm 업그레이드 실패 대응](k8s-ops/07-kube-apiserver-ha-advertise-address-misconfig.md)
- [Traefik 전역 TLS 정책과 레거시 클라이언트 호환성 충돌 대응](k8s-ops/08-traefik-tls-minversion-scoped-override.md)
- [멀티 NIC 환경에서 Calico BGP Node IP 오선택 장애 대응](k8s-ops/09-calico-multi-nic-bgp-node-ip-misdetection.md)
- [ConfigMap/Secret 변경 자동 반영 검토 및 부분 구현](k8s-ops/10-config-reloader-review.md)

### [security/](security/) - 보안, 컴플라이언스

- [CSAP 인증 대응 Kubernetes 보안 취약점 전수 진단 및 조치](security/01-k8s-cis-benchmark-remediation.md)

### [cicd/](cicd/) - CI/CD, GitOps

- [GitLab Project Access Token 기반 인증 재설계](cicd/01-gitlab-pat-redesign.md)
- [ArgoCD-GitLab 연쇄 장애 대응](cicd/02-argocd-gitlab-cascading-failure.md)

### [iac/](iac/) - IaC, 구축 자동화

- [Kubernetes 클러스터 구성 자동화 4-Tier 리팩토링](iac/01-k8s-install-automation-refactor.md)

### [ops/](ops/) - 운영, 장애대응

- [컨테이너 레지스트리 CDN 매니페스트 캐싱 이슈 대응](ops/02-registry-cdn-manifest-cache-bypass.md)
- [Loki 로그 추출/백업 자동화 파이프라인 구축](ops/03-loki-log-backup-automation.md)
- [CSAP 인증 유지·갱신 대응 유지보수 자동화](ops/05-csap-maintenance-automation.md)
- [Redis RDB Replica 기반 Diskless 원격 백업 구현](ops/06-redis-rdb-remote-backup.md)
- [OpenEBS NDM 메모리 누수로 인한 노드 OOM 대응](ops/07-openebs-ndm-memory-leak.md)
- [커널 IPv6 비활성화로 인한 애플리케이션 기동 오류 대응](ops/08-ipv6-disable-app-bind-failure.md)

### [monitoring/](monitoring/) - 모니터링, 관측성

- [Prometheus Stack + Grafana 커스텀 모니터링 대시보드 구축](monitoring/01-grafana-dashboard-custom-build.md)
- [Loki 중앙집중형 로깅 S3 오브젝트 스토리지 전환 PoC](monitoring/02-loki-s3-storage-poc.md)
- [로그 수집 에이전트 Promtail → Alloy 마이그레이션](monitoring/03-promtail-to-alloy-migration.md)

### [cloud/](cloud/) - 퍼블릭 클라우드, 아키텍처 설계

- [온프레미스 솔루션의 퍼블릭 클라우드 공통 아키텍처 설계](cloud/01-public-cloud-nlb-alb-architecture.md)
- [네이버클라우드(NCP) 공공클라우드 SaaS PoC](cloud/02-ncp-public-cloud-saas-poc.md)

### [project/](project/) - 프로젝트 리딩, PM

- [대규모 고객사 솔루션 신규 구축 프로젝트 리딩](project/01-solution-installation-project-lead.md)
