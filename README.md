# ⚓️ k8s.s6n.jp

Kubernetes resources which are continiously deployed to MicroK8s cluster at home.

## 📦 Getting Started

### 1. Install ArgoCD

```shell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

For details, refer the official getting started guide:
https://argo-cd.readthedocs.io/en/stable/getting_started/ .

### 2. Add this repository as an application

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
