# Kubernetes 클러스터 부트스트랩 툴 고도화 — Kubespray에서 kubeadm 기반으로 전환

## 1. 배경

### 가. 현행 구조
1) Kubernetes 클러스터 설치 및 버전 업그레이드를 Kubespray 프로젝트 실행 방식으로 운영

### 나. 한계점
1) Kubespray가 Ansible/Python 런타임, OS 패키지·커널 파라미터, CNI/CSI/Ingress 버전까지 하나의
   프로젝트 버전에 묶어 관리하는 구조라, K8s 버전 업그레이드 시마다 Kubespray 프로젝트 자체의 변경점을
   전부 새로 조사해야 함
2) 실제 K8s 버전 업그레이드 난이도보다, Kubespray 프로젝트 변경사항 추적 비용이 더 커지는 구조

## 2. 기대효과

### 가. 유지보수 비용 절감
1) **조사 범위 축소**: 신규 버전 배포 시 검토 대상이 "해당 K8s 버전 릴리즈 노트의 deprecated
   API·신규 기능"으로 한정, Kubespray 프로젝트 자체 변경점 추적 불필요
2) **속도 개선**: kubeadm 직접 제어 방식으로 전환하며 업그레이드 소요시간 단축

### 나. 구성 유연성
1) **개별 버저닝**: CNI/CSI/Ingress를 Kubespray의 통합 버전 관리에서 분리해 컴포넌트별 독립 업그레이드 가능

### 다. AS-IS / TO-BE 비교

| 항목 | AS-IS (Kubespray) | TO-BE (kubeadm 기반 자체 엔진) |
|---|---|---|
| 설치·업그레이드 실행 주체 | Kubespray 프로젝트 전체 실행 | Ansible이 `kubeadm init`/`join`을 직접 호출 |
| 버전 업그레이드 검토 범위 | Kubespray 프로젝트 자체의 변경점 전수 조사 | K8s 릴리즈 노트의 deprecated API·신규 기능만 확인 |
| CNI/CSI/Ingress 버전 관리 | Kubespray 프로젝트 버전에 종속 | 컴포넌트별 개별 버전 지정·업그레이드 |
| 버전 업그레이드 소요시간 (v1.26.5→v1.27.10) | 18분 23초 | 4분 29초 (**75.6% 단축**) |

## 3. 구성 변경 (Topology)

### 가. AS-IS

```
K8s 버전 업그레이드 요청
        │
        ▼
Kubespray 프로젝트 실행 (Ansible + Python 런타임, OS 패키지, 커널 파라미터 포함)
        │
        ├── Kubespray 버전에 종속된 CNI 버전
        ├── Kubespray 버전에 종속된 CSI 버전
        └── Kubespray 버전에 종속된 Ingress 버전
        │
        ▼
버전 업그레이드마다 Kubespray 프로젝트 변경점 전수 재조사 필요
소요시간: 18분 23초
```

### 나. TO-BE

```
K8s 버전 업그레이드 요청
        │
        ▼
Ansible Playbook
        │
        ├── 노드별 사전 준비 (OS 패키지 / 커널 파라미터 / 컨테이너 런타임)
        │
        ▼
kubeadm init / kubeadm join 직접 호출
        │
        ├── CNI  ── 개별 버전 지정
        ├── CSI  ── 개별 버전 지정
        └── Ingress ── 개별 버전 지정
        │
        ▼
검토 범위: K8s 릴리즈 노트 기준 deprecated API·신규 기능만 확인
소요시간: 4분 29초 (75.6% 단축)
```

## 4. 성과

1) K8s 버전 업그레이드(v1.26.5 → v1.27.10) 소요시간 **75.6% 단축** (18분 23초 → 4분 29초)
2) 신규 버전 배포 시 검토 범위 단순화로 유지보수 비용 감소
3) CNI/CSI/Ingress 개별 버전 관리 체계 확보
4) 본 kubeadm 기반 설치 엔진은 이후 [Kubernetes 클러스터 구성 자동화 4-Tier
   리팩토링](01-k8s-install-automation-refactor.md)의 토대가 됨
5) 설계 단계부터 팀원과 함께 진행한 협업 프로젝트 (2024년 3~4월)

---

## 상세 문서

이 기획서 요약본의 상세 설계(Kubespray 프로젝트 변경점 조사 사례, Ansible-kubeadm 래핑 구조 등)는
비공개 리포지토리에 기술 문서로 별도 보관 중입니다. 필요 시 요청해주시면 공유드리겠습니다.
