# Structure du projet JENKINS EXAM

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


L’infrastructure repose sur les composants suivants :

##  1. Microservices FastAPI
movie-service
cast-service

## 2. Bases de données PostgreSQL
movie-db
cast-db

## 3. Ingress Controller
Traefik pour le routage HTTP/HTTPS
Certificats TLS automatiques via cert‑manager

## 4. Helm Chart multi‑environnements
values-dev.yaml
values-qa.yaml
values-staging.yaml
values-prod.yaml

## 5. Jenkins Pipeline
  
### Automatise

    - Build Docker
    - Push Docker Hub
    - Installation cert‑manager
    - Installation ClusterIssuer
    - Redémarrage CoreDNS
    - Nettoyage des nœuds NotReady
    - Déploiement Helm par environnement

Le pipeline Jenkins installe automatiquement :
  cert‑manager (via Helm)
  ClusterIssuer Let’s Encrypt (via kubectl)
  Le fichier cluster-issuer.yaml est appliqué une seule fois et ne doit pas être dans templates/