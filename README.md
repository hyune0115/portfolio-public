# infra-portfolio-summary

인프라/SRE 엔지니어링 프로젝트를 기획서 형태로 간단히 정리한 요약 포트폴리오입니다.
각 문서는 배경 → 기대효과/AS-IS-TO-BE → 구성도 → 성과 순으로 정리되어 있습니다.
(상세 기술 문서는 비공개로 별도 관리 중이며, 필요 시 요청하시면 공유드립니다.)

## Summary

### 🚀 Key Achievements (핵심 성과)

- **GitOps(ArgoCD-GitLab) 배포 파이프라인 안정화** — 배포 연쇄 장애 원인 규명 및 파이프라인 안정화로
  일일 장애 발생 고객사 75-80% 감소 ([자세히](cicd/02-argocd-gitlab-cascading-failure.md))
- **SaaS K8s 컨트롤플레인 HA 재설계 및 심층 트러블슈팅** — all-master 구조를 마스터/워커로 분리하고
  HAProxy+Keepalived 기반 HA를 구축해 다운타임을 95.5% 단축(2분대 → 5초)했으며, 구축 중 발생한
  advertise-address 다중 파생 장애를 kubeadm 소스코드 분석으로 규명해 무중단 업그레이드 완수
  *(공동 진행)* ([HA 구축 자세히](k8s-ops/07-saas-k8s-ha-master-worker-separation.md) ·
  [장애 규명 자세히](k8s-ops/04-kube-apiserver-ha-advertise-address-misconfig.md))
- **RHEL9 cgroup v2 전환 대응 및 JVM 메모리 미인식 OOM 해결** — OS 업그레이드 후 발생한 컨테이너
  OOM의 근본 원인을 커널 파라미터 교차 검증으로 특정(cgroup v2 미인식), 런타임 최적화 및 호환
  JDK 업그레이드로 무중단 서비스 안정화 ([자세히](k8s-ops/01-redhat9-cgroupv2-jdk-oom.md))
- **Kubeadm 기반 클러스터 라이프사이클 툴 전환** — Kubespray 대체 경량화 엔진 도입으로 버전
  업그레이드 소요 시간 75.6% 단축(18분 23초 → 4분 29초)
  ([자세히](iac/02-kubespray-to-kubeadm-migration.md))
- **CSAP SaaS 보안 인증 획득 및 감사 증적 자동화** — kube-bench 141개 항목 전수 조치(즉시 30건 +
  Kyverno 27건)로 인증 충족 및 정기 감사 증적 자동 생성 체계 구축
  ([진단·조치 자세히](security/01-k8s-cis-benchmark-remediation.md) ·
  [유지보수 자동화 자세히](security/02-csap-maintenance-automation.md))
- **대규모 멀티 클러스터(6+ Clusters) 신규 구축 프로젝트 리딩** — 6개 이상 엔터프라이즈 클러스터
  구축 프로젝트를 설치 계획부터 프로덕션 오픈까지 전 주기 단독 완수
  ([자세히](project/01-solution-installation-project-lead.md))

---

## 🛠 Technical Case Studies & Archives

> 클러스터 설계, 커널/네트워크 심층 장애 분석, 컴플라이언스 및 자동화에 대한 상세 엔지니어링 기록입니다.

### 🌟 Featured Deep Dives (핵심 트러블슈팅 & 아키텍처)

- **[NAC ARP 갱신과 Ingress IP 충돌 분석 및 패킷 레벨 장애 해결](network/01-nac-arp-mac-floating-conflict.md)** — tcpdump 패킷 덤프 기반 원인 후보 배제로 L2/L3 충돌 규명, MetalLB 도입으로 근본 해결
- **[멀티 NIC 환경 Calico BGP 라우팅 결함 분석 및 명시적 인터페이스 격리](network/04-calico-multi-nic-bgp-node-ip-misdetection.md)** — CNI의 암묵적 인터페이스 선택 결함을 분석하고 명시적 CIDR 바인딩으로 라우팅 경로 정상화
- **[커널–애플리케이션 IPv6 불일치에 따른 소켓 바인딩 장애 분석 및 IPv4 단일 스택 표준화](network/05-ipv6-disable-app-bind-failure.md)** — 커널 비활성화와 앱 설정 불일치로 인한 크래시 규명, 배포 스펙 IPv4 단일 스택 통일로 재발 차단
- **[클러스터 프로비저닝 자동화 프레임워크 4-Tier 계층화 및 멱등성 개선](iac/01-k8s-install-automation-refactor.md)** — 계층별 의존성 분리 및 실행 상태 추적을 통해 실패 시 불필요한 반복 실행 문제 해결 및 멱등성 보장
- **[Loki 멀티라인 대용량 로그 유실(Drop) 구조적 원인 분석 및 Ingestion Throttling 최적화](monitoring/04-loki-log-drop-rate-limit-tuning.md)** — 멀티라인 스택 트레이스의 Ingestion 쿼터 초과 구조 규명 및 버퍼/Rate Limit 최적화로 유실 차단
- **[K8s MariaDB TCP Probe로 인한 Aborted Connection 병목 해결 및 최소권한 보안 개선](k8s-ops/03-mariadb-healthcheck-probe-tcp-abort.md)** — 불완전 핸드셰이크를 exec probe로 전환해 소켓 병목을 해결하고, root 사용 관행을 최소권한 전용 계정으로 개선
- **[글로벌 Root CA 전환에 따른 PKI 인증 체인 결함 해결 (Cross-Signing 적용)](security/04-ssl-ca-root-transition-pki-error.md)** — 신·구 Root CA 간 교차 서명(Cross-Signing) 체인 구성으로 레거시 호환성 보장 및 TLS 통신 단절 방지

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
