## Overview

This project demonstrates how I would take a simple Node.js application and make it production-ready for deployment on AWS EKS.

The goal was not just to “make it work”, but to ensure:

* the container is secure and lightweight
* deployments are stable and observable
* CI/CD is automated and reproducible

---
## Architecture

* **Kubernetes (EKS)** → application runtime
* **Jenkins** → CI/CD pipeline
* **Docker** → containerization
* **Artifactory** → image storage
* **Prometheus + Grafana** → metrics & dashboards
* **ELK Stack** → logging and debugging

---
## Key Decisions & Reasoning

### Dockerfile Changes

The original Dockerfile worked locally but had a few issues:

* `node:latest` is unpredictable → replaced with a fixed version
* Running as root → switched to non-root user
* Large image size → used multi-stage build
* Dev dependencies included → removed in final image

---
### Kubernetes Setup

* Used a **Deployment** with 3 replicas for basic high availability
* Added **readiness & liveness probes** to avoid bad pods receiving traffic
* Defined **resource limits** to prevent resource exhaustion
* Named port as `api-web` as required (instead of default names)

---
### CI/CD Pipeline

The pipeline is intentionally simple but practical:

1. Pull code
2. Install dependencies and run tests
3. Build Docker image
4. Push to Artifactory (using Jenkins secret)
5. Update deployment in EKS

I kept the scan step as a placeholder — in a real setup I’d integrate Trivy or Snyk.

---
### Observability

#### Metrics

* Prometheus annotations added to pods
* Assumes `/metrics` endpoint is exposed

#### Logging

* App logs go to stdout
* Can be picked up by Fluentd/Filebeat → sent to ELK

---
## How to Run

### Build & Push Image

```bash
docker build -t <repo>/node-api .
docker push <repo>/node-api
```

### Deploy

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

### Verify

```bash
kubectl get pods
kubectl get svc
kubectl get ingress
```
## Assumptions

* EKS cluster is already set up
* ALB Ingress Controller is installed
* Prometheus/Grafana and ELK stack are available in the cluster

---
## Final Thoughts

The focus of this solution was to keep things simple but realistic — something that can actually be used as a base in a real project rather than an over-engineered setup.
