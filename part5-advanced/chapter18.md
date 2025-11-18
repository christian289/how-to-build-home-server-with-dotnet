# 제18장: 쿠버네티스 입문

## 18.1 K3s 설치와 설정

[K3s](https://k3s.io/)는 경량 Kubernetes 배포판입니다.

### K3s 설치

```bash
# K3s 서버 설치
curl -sfL https://get.k3s.io | sh -

# 설치 확인
sudo k3s kubectl get nodes

# kubeconfig 설정
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config

# kubectl 별칭
echo "alias k='kubectl'" >> ~/.bashrc
source ~/.bashrc
```

## 18.2 Helm으로 애플리케이션 배포

[Helm](https://helm.sh/)은 Kubernetes 패키지 매니저입니다.

```bash
# Helm 설치
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# 저장소 추가
helm repo add stable https://charts.helm.sh/stable
helm repo update

# 앱 설치 예제: PostgreSQL
helm install my-postgres bitnami/postgresql --set auth.postgresPassword=secretpassword

# 릴리스 목록
helm list
```

## 18.3 Longhorn으로 스토리지 관리

[Longhorn](https://longhorn.io/)은 K3s용 분산 스토리지입니다.

```bash
# Longhorn 설치
helm repo add longhorn https://charts.longhorn.io
helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace

# UI 접근
kubectl -n longhorn-system port-forward svc/longhorn-frontend 8090:80
```

## 18.4 ArgoCD로 GitOps 구현

[ArgoCD](https://argo-cd.readthedocs.io/)로 GitOps 워크플로우 구축:

```bash
# ArgoCD 설치
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 초기 비밀번호
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# UI 접근
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

---

**다음 장에서는**: Minecraft, Valheim 등 게임 서버 호스팅을 알아보겠습니다.
