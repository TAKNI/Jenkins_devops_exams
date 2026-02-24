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
├───cast-service
│   │   Dockerfile
│   │   requirements.txt
│   │
│   └───app
│       │   main.py
│       │
│       └───api
│               casts.py
│               db.py
│               db_manager.py
│               models.py
│
├───charts
│   │   .helmignore
│   │   Chart.yaml
│   │   cluster-issuer.yaml
│   │   README.md
│   │   values-dev.yaml
│   │   values-prod.yaml
│   │   values-qa.yaml
│   │   values-staging.yaml
│   │   values.yaml
│   │
│   └───templates
│       │   cast-db-deployment.yaml
│       │   cast-db-service.yaml
│       │   cast-service-deployment.yaml
│       │   cast-service-service.yaml
│       │   ingress.yaml
│       │   movie-db-deployment.yaml
│       │   movie-db-service.yaml
│       │   movie-service-deployment.yaml
│       │   movie-service-service.yaml
│       │   _helpers.tpl
│       │
│       └───tests
└───movie-service
    │   Dockerfile
    │   requirements.txt
    │
    └───app
        │   main.py
        │
        └───api
                db.py
                db_manager.py
                models.py
                movies.py
                service.py
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