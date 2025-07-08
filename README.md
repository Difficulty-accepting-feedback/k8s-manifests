# k8s-manifests

> ArgoCD를 활용한 쿠버네티스 매니페스트 통합 관리 레포지토리

## 📖 개요

이 레포지토리는 쿠버네티스 클러스터에 배포되는 모든 애플리케이션의 매니페스트 파일을 **GitOps 방식**으로 관리합니다. ArgoCD를 통해 선언적 배포 자동화를 구현하며, 개발, 스테이징, 운영 환경에서 일관된 배포 프로세스를 제공합니다.

## 🏗️ 레포지토리 구조

```text
k8s-manifests/
├── 📁 argocd/ # ArgoCD 설정 파일
│ ├── projects/ # AppProject 정의
│ └── applications/ # Application 정의
├── 📁 manifests/ # 순수 Kubernetes YAML 매니페스트
│ ├── common/ # 공통 리소스 (네임스페이스, RBAC 등)
│ ├── development/ # 개발 환경
│ └── production/ # 운영 환경
├── 📁 helm/ # Helm 차트 및 values 파일
│ ├── charts/ # 애플리케이션별 Helm 차트
│ └── values/ # 환경별 values 파일
└── 📁 kustomize/ # Kustomize 구성
├── base/ # 기본 매니페스트
└── overlays/ # 환경별 오버레이
```

## 🚀 시작하기

### 전제 조건

- Kubernetes 클러스터 (v1.19+)
- ArgoCD 설치 및 구성
- kubectl CLI 도구
- 기본적인 Kubernetes 및 GitOps 개념 이해

---

**⚡ 빠른 시작을 위한 체크리스트**

- [ ] ArgoCD 설치 및 구성 완료
- [ ] 레포지토리 ArgoCD에 등록
- [ ] 첫 번째 애플리케이션 배포 테스트
- [ ] 환경별 동기화 정책 설정
- [ ] 팀 구성원 권한 설정
- [ ] 모니터링 및 알림 구성

---

### ArgoCD 설치

ArgoCD 네임스페이스 생성

```
$ kubectl create namespace argocd
```

ArgoCD 설치
```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

ArgoCD 서버 접근
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
### 레포지토리 등록

ArgoCD CLI로 레포지토리 등록

```text
argocd repo add https://github.com/your-org/k8s-manifests.git --username your-username --password your-token
```

---

## 📦 애플리케이션 관리

### 새로운 애플리케이션 추가

1. **매니페스트 파일 생성**

환경별 디렉토리에 매니페스트 파일 추가
```text
mkdir -p manifests/{development,staging,production}/new-service
```

2. **ArgoCD Application 정의**
   argocd/applications/production/new-service.yaml

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application

metadata: 
  name: new-service-prod
  namespace: argocd

spec: 
  project: default

source:
  repoURL: https://github.com/Difficulty-accepting-feedback/k8s-manifests.git
  targetRevision: HEAD
  path: manifests/production/new-service
  destination:
    server: https://kubernetes.default.svc
    namespace: production
    
syncPolicy:
  automated:
    prune: true
    selfHeal: true

```


### 배포 프로세스

1. **개발 환경 배포**
- `manifests/development/` 디렉토리에 변경사항 커밋
- ArgoCD가 자동으로 변경사항을 감지하고 배포

2. **스테이징 환경 승격**
- 개발 환경에서 검증된 변경사항을 스테이징으로 복사
- Pull Request를 통한 코드 리뷰 진행

3. **운영 환경 배포**
- 승인된 변경사항을 운영 환경으로 승격
- 배포 전 추가 검증 및 승인 프로세스

## 🔧 환경별 구성

### Development 환경
- **목적**: 개발 및 기능 테스트
- **동기화**: 자동 (Auto-sync)
- **네임스페이스**: `development`

### Staging 환경
- **목적**: 통합 테스트 및 사전 검증
- **동기화**: 수동 승인 후 자동
- **네임스페이스**: `staging`

### Production 환경
- **목적**: 운영 서비스
- **동기화**: 수동 승인 필수
- **네임스페이스**: `production`

## 📝 매니페스트 작성 가이드

### 네이밍 컨벤션

리소스 이름 형식: <service-name>-<environment>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-service-prod
  namespace: production
  labels:
    app: backend-service
    environment: production
    version: v1.0.0
```
---

### 배포 상태 확인

모든 애플리케이션 상태 확인
```text
argocd app list
```
특정 애플리케이션 상태 확인

```
argocd app get <app-name>
```

동기화 상태 확인
```
argocd app sync <app-name>
```

## 📚 참고 자료

- [ArgoCD 공식 문서](https://argo-cd.readthedocs.io/)
- [GitOps 모범 사례](https://www.gitops.tech/)
- [Kubernetes 매니페스트 작성 가이드](https://kubernetes.io/docs/concepts/overview/working-with-objects/)
- [Kustomize 문서](https://kustomize.io/)
- [Helm 차트 개발 가이드](https://helm.sh/docs/chart_best_practices/)

---

## 🤝 기여 방법

1. **이슈 생성**: 버그 리포트나 기능 요청
2. **Pull Request**: 변경사항 제안
3. **코드 리뷰**: 팀원들과 함께 검토
4. **문서 업데이트**: README 및 가이드 개선