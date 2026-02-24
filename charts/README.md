### Chart structure

To create helm chart run following command and edit values, chart files and files in templates directory based on your k8s manifests:

```
$ helm create fastapiapp
```
This creates the default Helm chart structure

Now move to the charts directory and update default files .yaml:

```
$ cd charts 
$ tree

./charts
├── Chart.yaml
├── README.md
├── values.yaml
│
├── values-dev.yaml
├── values-qa.yaml
├── values-staging.yaml
├── values-prod.yaml
│
├── cluster-issuer.yaml        # Applied manually or by Jenkins (NOT in templates)
│
└── templates
    ├── ingress.yaml
    ├── movie-service-deployment.yaml
    ├── movie-service-service.yaml
    ├── cast-service-deployment.yaml
    ├── cast-service-service.yaml
    ├── movie-db-deployment.yaml
    ├── movie-db-service.yaml
    ├── cast-db-deployment.yaml
    ├── cast-db-service.yaml
    ├── _helpers.tpl
    └── NOTES.txt
```
