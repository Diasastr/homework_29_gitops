Налаштування середовища для Kubernetes і ArgoCD на Windows
==========================================================

Цей репозиторій містить інструкції та конфігураційні файли для налаштування середовища розробки з Kubernetes і ArgoCD на Windows, використовуючи Minikube та kubectl для розгортання веб-додатку.

Передумови
----------

Переконайтеся, що у вас встановлено Chocolatey на вашій машині з Windows.

Інсталяція необхідного ПЗ
-------------------------

### Встановлення Minikube і kubectl

Відкрийте PowerShell як адміністратор та виконайте наступні команди:

```powershell

# Встановлення Minikube і встановлення kubectl
choco install minikube
```

### Встановлення ArgoCD CLI

```powershell
# Встановлення Argo-CLI
choco install argo-cli
```
### Запуск Minikube

```powershell
minikube start
```

### Встановлення ArgoCD у Minikube

```powershell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Налаштування доступу до ArgoCD UI

```powershell
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
