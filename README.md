# RD Final Project - GitOps Kubernetes Platform

This project demonstrates a full GitOps-based Kubernetes platform using FluxCD, Helm, Operators, and multiple environments.

The application is a simple Next.js app that checks connectivity to:

- PostgreSQL database

- Redis-compatible Dragonfly database

Infrastructure is managed fully through GitOps principles.

## Stack

- Kubernetes (Docker Desktop)
- FluxCD
- Helm
- NGINX Ingress Controller
- CloudNativePG Operator
- Dragonfly Operator
- Next.js
- PostgreSQL
- DragonflyDB
- Docker Hub

## Architecture

Environments:

- staging namespace
- production namespace

Database:

- PostgreSQL via CloudNativePG Operator
- DragonflyDB via Dragonfly Operator

Deployment:

- HelmRelease + FluxCD

Ingress:

- NGINX Ingress Controller
- Self-signed TLS via mkcert

## Repository Structure

```txt
apps/
charts/
clusters/
infrastructure/
```

## Self-healing

Self-healing was validated by manually deleting the Deployment and reconciling the HelmRelease through FluxCD.

![alt text](<screenshots/Screenshot 2026-05-26 at 22.50.15.png>)

## Flux status

![alt text](<screenshots/Screenshot 2026-05-26 at 22.36.40.png>)
![alt text](<screenshots/Screenshot 2026-05-26 at 22.36.56.png>)
![alt text](<screenshots/Screenshot 2026-05-26 at 22.37.29.png>)
![alt text](<screenshots/Screenshot 2026-05-26 at 22.37.47.png>)

## How to access

Production:
https://app.local:8443

Staging:
https://app.staging.local:8443

![alt text](<screenshots/Screenshot 2026-05-26 at 22.40.07.png>)
![alt text](<screenshots/Screenshot 2026-05-26 at 22.40.16.png>)

## Useful Commands

### FluxCD

Check all Flux kustomizations:

```bash
flux get kustomizations
```

Check all Helm releases:

```bash
flux get helmreleases -A
```

Force reconcile Git source:

```bash
flux reconcile source git flux-system
```

Reconcile staging environment:

```bash
flux reconcile kustomization app-staging --with-source
```

Reconcile production environment:

```bash
flux reconcile kustomization app-production --with-source
```

Reconcile HelmRelease:

```bash
flux reconcile helmrelease simple-app -n staging --with-source
```

---

### Kubernetes

Check all pods:

```bash
kubectl get pods -A
```

Watch staging pods:

```bash
kubectl get pods -n staging -w
```

Check services:

```bash
kubectl get svc -A
```

Check ingress resources:

```bash
kubectl get ingress -A
```

Check deployments:

```bash
kubectl get deploy -A
```

Check HPA:

```bash
kubectl get hpa -A
```

View application logs:

```bash
kubectl logs -f deploy/simple-app -n staging
```

Restart deployment:

```bash
kubectl rollout restart deployment simple-app -n staging
```

Delete pod:

```bash
kubectl delete pod <pod-name> -n staging
```

Self-healing test:

```bash
kubectl delete deploy simple-app -n staging
```

---

### Helm

Render Helm templates locally:

```bash
helm template simple-app charts/simple-app
```

List Helm releases:

```bash
helm list -A
```

View rendered manifests:

```bash
helm get manifest simple-app -n staging
```

---

### Ingress / TLS

Port-forward ingress controller HTTPS:

```bash
kubectl port-forward -n ingress-nginx svc/ingress-nginx-ingress-nginx-controller 8443:443
```

Port-forward ingress controller HTTP:

```bash
kubectl port-forward -n ingress-nginx svc/ingress-nginx-ingress-nginx-controller 8080:80
```

Generate self-signed certificates:

```bash
mkcert -install
mkcert app.local app.staging.local
```

Create TLS secret:

```bash
kubectl create secret tls simple-app-tls \
  --cert=app.local+1.pem \
  --key=app.local+1-key.pem \
  -n staging
```

---

### Docker

Build Docker image:

```bash
docker build -f .docker/prod.dockerfile -t username/app:v0.0.1 .
```

Push Docker image:

```bash
docker push username/app:v0.0.1
```

---

### Local Access

Add local domains:

```bash
sudo nano /etc/hosts
```

```txt
127.0.0.1 app.local
127.0.0.1 app.staging.local
```

Production URL:

```txt
https://app.local:8443
```

Staging URL:

```txt
https://app.staging.local:8443
```
