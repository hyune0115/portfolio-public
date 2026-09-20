# infra-portfolio-summary

인프라/SRE 엔지니어링 프로젝트를 기획서 형태로 간단히 정리한 요약 포트폴리오입니다.
각 문서는 배경 → 기대효과/AS-IS-TO-BE → 구성도 → 성과 순으로 정리되어 있습니다.
(상세 기술 문서는 비공개로 별도 관리 중이며, 필요 시 요청하시면 공유드립니다.)

## Summary

### 🚀 Key Achievements (핵심 성과)

- **ArgoCD-GitLab 연쇄 장애 대응** — 일일 장애 발생 고객사 75-80% 감소
  ([자세히](cicd/02-argocd-gitlab-cascading-failure.md))
- **사내 SaaS K8s HA 구축 (마스터/워커 분리)** — all-master 구성을 마스터/워커로 분리하고
  HAProxy+Keepalived 기반 API 서버 HA를 구축해, 컨트롤플레인 장애 시 다운타임을 2분대에서
  최대 5초(95.45% 감소) 수준까지 단축 (동료 엔지니어와 공동 진행)
  ([자세히](k8s-ops/07-saas-k8s-ha-master-worker-separation.md))
- **kube-apiserver HA 오설정 근본원인 규명** — advertise-address 오설정이 엔드포인트·kubelet
  probe·kubeconfig 5곳에 파생되어 발생한 상관 장애를 kubeadm 소스 코드 레벨까지 추적해 규명하고
  HA 클러스터 업그레이드를 완주시킴 ([자세히](k8s-ops/04-kube-apiserver-ha-advertise-address-misconfig.md))
- **Kubernetes 클러스터 부트스트랩 툴 고도화** — Kubespray에서 kubeadm 기반으로 전환해 버전
  업그레이드 소요시간을 75.6% 단축(18분 23초 → 4분 29초)
  ([자세히](iac/02-kubespray-to-kubeadm-migration.md))
- **CSAP 인증심사 대응 및 유지보수** — kube-bench 기반 141개 항목 전수 진단, 즉시조치 30건 +
  Kyverno 기반 중기조치 27건으로 CSAP 인증 요건 충족, 이후 유지·갱신 심사에 필요한 반복 증적을
  자동 생성하는 체계까지 구축해 현재도 운영 중
  ([진단·조치 자세히](security/01-k8s-cis-benchmark-remediation.md) ·
  [유지보수 자동화 자세히](security/02-csap-maintenance-automation.md))
- **대규모 고객사 솔루션 신규 구축 프로젝트 리딩** — 6개 이상 클러스터 규모의 구축 프로젝트를
  설치계획부터 오픈 지원까지 전 주기 단독 리딩, 정상 서비스 오픈으로 완료
  ([자세히](project/01-solution-installation-project-lead.md))

## 🛠 Technical Case Studies & Archives

> 클러스터 설계, 커널/네트워크 심층 장애 분석, 컴플라이언스 및 자동화에 대한 상세 엔지니어링 기록입니다.

### 🌟 Featured Deep Dives (핵심 트러블슈팅 & 아키텍처)

- **[kube-apiserver HA 오설정 근본원인 규명 및 해결](k8s-ops/04-kube-apiserver-ha-advertise-address-misconfig.md)** — `kubeadm` 소스코드 분석을 통한 다중 파생 장애 해결
- **[cgroup v2 전환에 따른 JVM OOM 근본원인 규명](k8s-ops/01-redhat9-cgroupv2-jdk-oom.md)** — RHEL9 마이그레이션 시 커널 파라미터 교차 검증으로 원인 특정 및 해결
- **[멀티 NIC 환경 Calico BGP 라우팅 오선택 원인 규명 및 해결](network/04-calico-multi-nic-bgp-node-ip-misdetection.md)** — 암묵적 인터페이스 선택 구조 결함을 명시적 대역 지정으로 근본 해결
- **[CSAP SaaS 보안 인증 대응 및 증적 자동화 체계 구축](security/01-k8s-cis-benchmark-remediation.md)** — 141개 항목 전수 진단·조치 및 Kyverno 기반 감사 자동화
- **[온프레미스 → 퍼블릭 클라우드 공통 아키텍처 설계](cloud/01-public-cloud-nlb-alb-architecture.md)** — 하이브리드 환경을 고려한 클라우드 SaaS 확장 모델

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
