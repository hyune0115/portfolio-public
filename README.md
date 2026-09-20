# infra-portfolio-summary

인프라/SRE 엔지니어링 프로젝트를 기획서 형태로 간단히 정리한 요약 포트폴리오입니다.
각 문서는 배경 → 기대효과/AS-IS-TO-BE → 구성도 → 성과 순으로 정리되어 있습니다.
(상세 기술 문서는 비공개로 별도 관리 중이며, 필요 시 요청하시면 공유드립니다.)

## Summary

### 🚀 Key Achievements (핵심 성과)

- **GitOps(ArgoCD-GitLab) 배포 파이프라인 안정화** — 일일 장애 발생 고객사 75-80% 감소
  ([자세히](cicd/02-argocd-gitlab-cascading-failure.md))
- **SaaS Kubernetes 고가용성(HA) 아키텍처 재설계** — all-master 구성을 마스터/워커로 분리하고
  HAProxy+Keepalived 기반 API 서버 HA를 구축해, 컨트롤플레인 장애 시 다운타임을 2분대에서
  최대 5초(95.45% 감소) 수준까지 단축 (동료 엔지니어와 공동 진행)
  ([자세히](k8s-ops/07-saas-k8s-ha-master-worker-separation.md))
- **Kubernetes HA 구성 심층 트러블슈팅** — advertise-address 오설정이 엔드포인트·kubelet
  probe·kubeconfig 5곳에 파생되어 발생한 상관 장애를 kubeadm 소스 코드 레벨까지 추적해 규명하고
  HA 클러스터 업그레이드를 완주시킴 ([자세히](k8s-ops/04-kube-apiserver-ha-advertise-address-misconfig.md))
- **Kubespray → Kubeadm 기반 클러스터 라이프사이클 툴 전환** — Kubespray에서 kubeadm 기반으로 전환해 버전
  업그레이드 소요시간을 75.6% 단축(18분 23초 → 4분 29초)
  ([자세히](iac/02-kubespray-to-kubeadm-migration.md))
- **CSAP SaaS 보안 인증 획득 및 증적 자동화 체계 구축** — kube-bench 기반 141개 항목 전수 진단, 즉시조치 30건 +
  Kyverno 기반 중기조치 27건으로 CSAP 인증 요건 충족, 이후 유지·갱신 심사에 필요한 반복 증적을
  자동 생성하는 체계까지 구축해 현재도 운영 중
  ([진단·조치 자세히](security/01-k8s-cis-benchmark-remediation.md) ·
  [유지보수 자동화 자세히](security/02-csap-maintenance-automation.md))
- **대규모 멀티 클러스터(6+ Nodes/Clusters) 신규 구축 프로젝트 리딩** — 6개 이상 클러스터 규모의 구축 프로젝트를
  설치계획부터 오픈 지원까지 전 주기 단독 리딩, 정상 서비스 오픈으로 완료
  ([자세히](project/01-solution-installation-project-lead.md))

## 🛠 Technical Case Studies & Archives

> 클러스터 설계, 커널/네트워크 심층 장애 분석, 컴플라이언스 및 자동화에 대한 상세 엔지니어링 기록입니다.

### 🌟 Featured Deep Dives (핵심 트러블슈팅 & 아키텍처)

- **[RHEL9 cgroup v2 전환에 따른 JVM 메모리 미인식 OOM 규명 및 런타임 최적화](k8s-ops/01-redhat9-cgroupv2-jdk-oom.md)** — 커널 파라미터 교차 검증으로 원인 특정, JDK 업그레이드로 해결
- **[NAC ARP 갱신과 Ingress IP 충돌 분석 및 패킷 레벨 장애 해결](network/01-nac-arp-mac-floating-conflict.md)** — 패킷 덤프 기반 원인 후보 배제로 근본 원인 특정, MetalLB 도입으로 해결
- **[멀티 NIC 환경 Calico BGP 라우팅 결함 분석 및 명시적 인터페이스 격리](network/04-calico-multi-nic-bgp-node-ip-misdetection.md)** — 암묵적 인터페이스 선택 구조 결함을 명시적 대역 지정으로 근본 해결
- **[커널–애플리케이션 IPv6 불일치에 따른 소켓 바인딩 장애 분석 및 IPv4 단일 스택 표준화](network/05-ipv6-disable-app-bind-failure.md)** — 커널·앱 설정 불일치 규명, IPv4 단일스택 통일로 재발 차단
- **[클러스터 프로비저닝 자동화 프레임워크 4-Tier 계층화 및 멱등성 개선](iac/01-k8s-install-automation-refactor.md)** — 완료 작업 반복 실행 문제 해결 포함
- **[Loki 멀티라인 대용량 로그 유실(Drop) 구조적 원인 분석 및 Ingestion Throttling 최적화](monitoring/04-loki-log-drop-rate-limit-tuning.md)** — 특정 모듈 대용량 로그가 ingestion 제한을 초과시키는 구조적 원인 규명
- **[K8s MariaDB TCP Probe로 인한 Aborted Connection 병목 해결 및 최소권한 보안 개선](k8s-ops/03-mariadb-healthcheck-probe-tcp-abort.md)** — exec probe 전환으로 근본 해결, root 계정 상시 사용 관행을 최소권한 계정으로 개선
- **[글로벌 Root CA 전환에 따른 PKI 인증 체인 결함 해결 (Cross-Signing 적용)](security/04-ssl-ca-root-transition-pki-error.md)**

---

### 📂 Directory Index (전체 기술 아카이브)

| 도메인 | 주요 기술 스택 | 핵심 주제 및 트러블슈팅 사례 | 디렉터리 |
| :--- | :--- | :--- | :---: |
| **Kubernetes & OS** | K8s, Linux Kernel, cgroup v2, kubeadm | • cgroup v2 JVM OOM 근본원인 규명<br>• kube-apiserver HA 다중 파생 장애 규명 및 해결<br>• MariaDB Exec Probe 전환 및 NDM 메모리 누수 해결 | [`k8s-ops/`](k8s-ops/) |
| **Networking & Ingress** | Calico(BGP), Traefik, IPVS/iptables, ARP | • NAC ARP 충돌 원인 규명 및 MetalLB 전환<br>• 멀티 NIC BGP Node IP 오선택 구조 결함 해결<br>• kube-proxy IPVS Deprecated 대응 iptables 전환 | [`network/`](network/) |
| **Security & Compliance** | CSAP, Kyverno, kube-bench, PKI | • CSAP 보안 취약점 전수 진단·조치 및 자동화<br>• 글로벌 Root CA 전환 Cross-Signing 대응 | [`security/`](security/) |
| **IaC & CI/CD** | Ansible, ArgoCD, GitLab, Harbor | • 클러스터 구성 자동화 4-Tier 리팩토링<br>• 부트스트랩 도구 전환(Kubespray → kubeadm)으로 업그레이드 시간 75.6% 단축<br>• ArgoCD-GitLab 연쇄 장애 근본원인 규명 및 해결 | [`iac/`](iac/)<br>[`cicd/`](cicd/) |
| **Observability & DR** | Prometheus, Loki, Alloy, Velero | • Promtail → Alloy 에이전트 마이그레이션<br>• Loki Ingestion Rate Limit 튜닝 및 S3 전환<br>• Redis Replica 기반 Diskless 원격 백업 구현 | [`monitoring/`](monitoring/)<br>[`backup/`](backup/) |
| **Cloud & Leadership** | NCP, Hybrid Cloud, PM | • 온프레미스 솔루션 클라우드 공통 아키텍처 설계<br>• 6개 이상 클러스터 규모 구축 프로젝트 전 주기 리딩 | [`cloud/`](cloud/)<br>[`project/`](project/) |
