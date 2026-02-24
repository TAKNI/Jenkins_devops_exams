# Jenkins DevOps Exam – CI/CD Pipeline with Kubernetes, Helm & FastAPI

This project implements a complete DevOps pipeline using **Jenkins**, **Docker**, **Kubernetes**, **Helm**, and **FastAPI**.  
It demonstrates a full CI/CD workflow including automated builds, multi‑environment deployments, TLS certificates, and microservices orchestration.

---

##  Project Overview

The application is composed of two FastAPI microservices:

- **movie-service**  
- **cast-service**

Each service has its own PostgreSQL database:

- **movie-db**
- **cast-db**

The entire system is deployed on a Kubernetes cluster using a **multi‑environment Helm chart** and exposed through **Traefik** with automatic TLS certificates provided by **cert‑manager**.

---

##  Project Structure

```

./Jenkins_devops_exams
│   .gitignore
│   docker-compose.yml
│   Jenkinsfile
│   nginx_config.conf
│   README.md
│
├── cast-service
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app/
│       ├── main.py
│       └── api/
│           ├── casts.py
│           ├── db.py
│           ├── db_manager.py
│           └── models.py
│
├── movie-service
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app/
│       ├── main.py
│       └── api/
│           ├── db.py
│           ├── db_manager.py
│           ├── models.py
│           ├── movies.py
│           └── service.py
│
├── charts
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── values-dev.yaml
│   ├── values-qa.yaml
│   ├── values-staging.yaml
│   ├── values-prod.yaml
│   ├── cluster-issuer.yaml   # Applied once manually
│   └── templates/
│       ├── movie-service-.yaml
│ ├── cast-service-.yaml
│       ├── movie-db-.yaml
│ ├── cast-db-.yaml
│       ├── ingress.yaml
│       └── tests/
```


## Kubernetes Environments

The cluster contains **four namespaces**, each representing a deployment environment:

- `dev`
- `qa`
- `staging`
- `prod`

Each environment uses its own Helm values file:

- `values-dev.yaml`
- `values-qa.yaml`
- `values-staging.yaml`
- `values-prod.yaml`

---

## TLS Certificates

TLS is handled by:

- **cert‑manager** (installed via Helm)
- **ClusterIssuer** (Let’s Encrypt)