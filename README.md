
# CI/CD Pipeline with Jenkins, Docker, and Argo CD on Kubernetes (KIND)

This project demonstrates a full CI/CD pipeline using **Jenkins**, **Docker**, **Argo CD**, and **Kubernetes** on a local development environment powered by **KIND** (Kubernetes IN Docker). The setup is divided into two GitHub repositories:

- [`nginx`](https://github.com/vivekdalsaniya12/nginx): Contains website code, `Dockerfile`, and `Jenkinsfile`.
- [`manifests`](https://github.com/vivekdalsaniya12/manifests): Contains Kubernetes manifests including `Deployment`, `Service`, and Argo CD `Application` definition.

---

![diagram-export-6-4-2025-11_49_15-PM](https://github.com/user-attachments/assets/d874468b-1a62-47ef-99b0-dd26b473bd71)


## 🔧 Prerequisites

Make sure you have the following installed on your system:

- [Docker](https://www.docker.com/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [KIND](https://kind.sigs.k8s.io/)
- [Jenkins](https://www.jenkins.io/) (locally or in Docker)
- `git`
- Internet access for fetching remote manifests

---

## 🚀 Quick Setup Guide

### Step 1: Clone Repositories

```bash
git clone https://github.com/vivekdalsaniya12/nginx.git
git clone https://github.com/vivekdalsaniya12/manifests.git
```

### Step 2: Create KIND Cluster

```bash
kind create cluster --name my-cluster --config cluster-config.yaml
```

### Step 3: Install Argo CD

```bash
kubectl create namespace argocd

kubectl apply -n argocd   -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl port-forward -n argocd svc/argocd-server 8081:443 &
```

### Step 4: Get Argo CD Admin Password

```bash
kubectl get secret argocd-initial-admin-secret -n argocd   -o jsonpath="{.data.password}" | base64 -d && echo
```

Login to Argo CD at: [https://localhost:8081](https://localhost:8081)  
**Username**: `admin`  
**Password**: (from the above command)

### Step 5: Deploy Your App via Argo CD

Apply the Argo CD `Application` resource:

```bash
kubectl apply -f Application.yaml -n argocd
```

This tells Argo CD to sync Kubernetes manifests from the [`manifests`](https://github.com/vivekdalsaniya12/manifests) repository.

### Step 6: Install Metrics Server (Optional for Monitoring)

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl patch deployment metrics-server -n kube-system   --type='json' -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
```

### Step 7: Generate Load (Optional)

Simulate traffic to the exposed service:

```bash
for i in {1..1000}; do
  curl http://localhost:30001/ > /dev/null &
done
```

### Step 8: Monitor Resource Usage

Use metrics server output or Argo CD dashboard:

```bash
kubectl top pod
```

Or open the [Argo CD UI](https://localhost:8081) and monitor deployment health.

---

## 🧪 Jenkins CI/CD Pipeline

> Assumes Docker and Jenkins are properly set up with credentials.

1. Jenkinsfile builds the Docker image using the `Dockerfile`.
2. Image is tagged and pushed to Docker Hub (`vivekdalsaniya/nginx-web:<tag>`).
3. Kubernetes manifests point to this image and are synced via Argo CD.
4. Any new push to the `nginx` repo with updated image triggers the pipeline.

---

## ✅ End Result

- Fully automated CI/CD using Git + Jenkins + Docker + Argo CD.
- GitOps workflow where deployment is triggered by code commits.
- Clean separation of application code and Kubernetes infrastructure.

---

## 📷 Screenshots (Optional)

> Add screenshots here (Argo CD UI, Jenkins console, kubectl output) for better clarity.

---

## 📄 License

MIT License
