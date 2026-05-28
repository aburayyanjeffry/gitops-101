# gitops-101

Introduction to GitOps for Kubernetes using Argo CD and NGINX on a local Kubernetes cluster running in Docker Desktop on a MacBook Pro M1.

This tutorial will cover:

- Installing Kubernetes locally with Docker Desktop
- Installing Helm
- Installing Argo CD
- Creating a GitOps repository
- Deploying an NGINX web server using Helm
- Syncing deployments automatically with Argo CD
- Updating index.html through Git and letting Argo CD redeploy automatically

---

# Architecture

Git Repository
       │
       ▼
    Argo CD
       │
       ▼
 Kubernetes Cluster
       │
       ▼
    NGINX Pod

Whenever changes are pushed to Git, Argo CD detects the changes and synchronizes the Kubernetes cluster automatically.

---

# Prerequisites

- MacBook Pro M1/M2
- Homebrew installed
- Docker Desktop installed
- Internet connection

---

# 1. Install Docker Desktop

Follow the official Docker Desktop installation guide for macOS:

https://docs.docker.com/desktop/setup/install/mac-install/

After installation, start Docker Desktop.

---

# 2. Enable Kubernetes

1. Open Docker Desktop
2. Go to Settings → Kubernetes
3. Enable Kubernetes
4. Click Apply & Restart

Docker Desktop will request permission to install Kubernetes components.

Select Allow or Yes to continue.

Verify Kubernetes is running:

```bash
kubectl cluster-info
```

---

# 3. Install Homebrew (Optional)

If Homebrew is not installed:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Verify installation:

```bash
brew --version
```

---

# 4. Install Helm

Install Helm using Homebrew:

```bash
brew install helm
```

Verify installation:

```bash
helm version
```

---

# 5. Install kubectl

Docker Desktop usually installs kubectl automatically.

Verify:

```bash
kubectl version --client
```

If missing:

```bash
brew install kubectl
```

---

# 6. Create Namespace for Argo CD

```bash
kubectl create namespace argocd
```

---

# 7. Install Argo CD

Install Argo CD using the official manifests:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Verify pods are running:

```bash
kubectl get pods -n argocd
```

Wait until all pods show STATUS = Running.

---

# 8. Access Argo CD UI

Port forward the Argo CD server:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open browser:

https://localhost:8080

---

# 9. Get Argo CD Admin Password

Retrieve the initial admin password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d
```

Login credentials:

Username: admin
Password: <output from command above>

---

# 10. Create GitOps Repository Structure

Example repository structure:

```text
gitops-101/
├── charts/
│   └── nginx/
├── applications/
│   └── nginx.yaml
└── README.md
```

---

# 11. Create NGINX Helm Chart

Generate a Helm chart:

```bash
mkdir charts
cd charts

helm create nginx
```

---

# 12. Modify NGINX index.html

Create a ConfigMap for the custom HTML page.

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-html
data:
  index.html: |
    <html>
    <body>
      <h1>Hello from GitOps 101</h1>
    </body>
    </html>
```

Mount the ConfigMap into the NGINX container.

---

# 13. Install NGINX Chart Locally

Test deployment:

```bash
helm install nginx ./charts/nginx
```

Verify:

```bash
kubectl get pods
kubectl get svc
```

---

# 14. Expose NGINX

Port forward the service:

```bash
kubectl port-forward svc/nginx 8081:80
```

Open:

http://localhost:8081

---

# 15. Push Repository to GitHub

Repository:

https://github.com/aburayyanjeffry/gitops-101/

Push code:

```bash
git init

git add .

git commit -m "Initial GitOps setup"

git branch -M main

git remote add origin https://github.com/aburayyanjeffry/gitops-101.git

git push -u origin main
```

---

# 16. Create Argo CD Application

Create:

applications/nginx.yaml

Content:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx
  namespace: argocd

spec:
  project: default

  source:
    repoURL: https://github.com/aburayyanjeffry/gitops-101.git
    targetRevision: main
    path: charts/nginx

  destination:
    server: https://kubernetes.default.svc
    namespace: default

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Apply:

```bash
kubectl apply -f applications/nginx.yaml
```

---

# 17. Verify Argo CD Sync

Open Argo CD UI again:

https://localhost:8080

You should see:

nginx → Healthy → Synced

---

# 18. Test GitOps Workflow

Modify index.html:

```html
<h1>GitOps is Awesome</h1>
```

Commit and push:

```bash
git add .

git commit -m "Update nginx homepage"

git push
```

Argo CD will automatically detect the change and redeploy NGINX.

Refresh browser:

http://localhost:8081

You should see the updated page.

---

# Useful Commands

## List Kubernetes Resources

```bash
kubectl get all
```

## List Helm Releases

```bash
helm list
```

## List Helm Repositories

```bash
helm repo list
```

## Check Argo CD Pods

```bash
kubectl get pods -n argocd
```

---

# Cleanup

Remove NGINX:

```bash
helm uninstall nginx
```

Remove Argo CD:

```bash
kubectl delete namespace argocd
```

---

# References

- https://argo-cd.readthedocs.io/en/stable/
- https://helm.sh/docs/
- https://kubernetes.io/docs/home/
- https://docs.docker.com/desktop/
