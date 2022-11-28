# ⚓️ k8s.s6n.jp

Kubernetes resources which are continiously deployed to MicroK8s cluster at home.

## 🏗️ Requirements

- Kubernetes v1.25+ (MicroK8s is recommended)
- Helm v3.9+ (Can be installed together by MicroK8s)

## 📦 Getting Started

### 1. Import unmanaged secrets

#### CA key pair

This stack requires a key pair of intermediate or root CA (Certificate Authority).
To avoid sharing the pair other than this stack, we recommend to use an intermediate one.

```shell
kubectl create namespace cert-manager
kubectl create secret tls -n cert-manager --cert ./cert.pem --key ./key.pem
```

#### GitHub PAT for actions-runner-controller

actions-runner-controller requires a GitHub PAT (Personal Access Token) to initiate self-hosted runners.

```shell
kubectl create namespace actions-runner
kubectl create secret generic -n actions-runner controller-manager --from-literal='github_token=<YOUR_PAT_HERE>'
```

### 2. Install ArgoCD

```shell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

For details, refer the official getting started guide:
https://argo-cd.readthedocs.io/en/stable/getting_started/ .

### 3. Add this repository as an application

Here is an application spec of this repository for ArgoCD:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: k8s
  namespace: argocd
spec:
  project: default
  destination:
    server: https://kubernetes.default.svc
  source:
    repoURL: https://github.com/s6n-jp/k8s.git
    path: .
    targetRevision: main
    directory:
      recurse: true
  syncPolicy:
    automated: {}
    syncOptions:
    - CreateNamespace=true
```

```shell
kubectl apply -f <path_to_spec>.yaml
```

You are now ready to go!
