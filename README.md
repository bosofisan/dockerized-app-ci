# Dockerized Node.js App with CI/CD & Kubernetes Deployment
This project showcases a complete DevOps delivery pipeline, from containerization to automated builds to kubernetes deployment.
It demonstrates practical experience with Docker, GitHub Actions, Docker Hub, and Kubernetes manifests running on a local cluster. 

## Overview
The application is a lightweight Node.js service packaged as a Docker image, automatically built and pushed via GitHub Actions, and deployed to a Kubernetes cluster using declarative YAML configuration. 

This workflow mirrors a modern cloud-native delivery model used by Platform, DevOps, and Cloud Engineering teams worldwide. 

## Architecture

           ┌────────────┐
           │  Developer │
           └──────┬─────┘
                  │  Push code to GitHub
                  ▼
          ┌──────────────────┐
          │  GitHub Actions  │
          │   (CI Pipeline)  │
          └──────┬───────────┘
                  │ Builds Docker image
                  │ Pushes to Docker Hub
                  ▼
         ┌───────────────────┐
         │     Docker Hub    │
         └──────┬────────────┘
                 │ Pull latest image
                 ▼
        ┌────────────────────┐
        │     Kubernetes     │
        │ Deployment + Svc   │
        └────────────────────┘

## Tech Stack
- Node.js – application runtime
- Docker – build & containerize
- GitHub Actions – CI pipeline
- Docker Hub – container registry
- Kubernetes (kind) – deployment & orchestration
- YAML – infrastructure as code

## CI Pipeline (GitHub Actions)
The CI workflow automatically triggers every time code is pushed to the main branch: 
1. GitHub Actions checks out the repo
2. Docker Buildx builds the image
3. Github Actions authenticates with Docker Hub
4. The image is pushed with the tag: ``` bosofisan/dockerized-app-ci:latest ```
5. Kubernetes pulls the updated image on redeploy

## CI Pipeline (``` .github/workflows/ci.yml ```)
```yaml
name: CI Pipeline

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: bosofisan/dockerized-app-ci:latest
```


## Kubernetes Deployment
### Deployment (deployment.yaml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dockerized-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: dockerized-app
  template:
    metadata:
      labels:
        app: dockerized-app
    spec:
      containers:
        - name: dockerized-app
          image: bosofisan/dockerized-app-ci:latest
          ports:
            - containerPort: 3000
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "250m"
              memory: "256Mi"
```

### Service (service.yaml)
``` yaml
apiVersion: v1
kind: Service
metadata:
  name: dockerized-app-service
spec:
  type: NodePort
  selector:
    app: dockerized-app
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
      nodePort: 30080
```

## Run App Locally
Apply Kubernetes manifests: 
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
``` 
Verify:
```bash
kubectl get pods
kubectl get svc
```
Post-forward service to local machine:
```bash
kubectl port-forward service/dockerized-app-service 3000:3000
```

Visit: http://localhost:3000

## Project Structure 
```bash
project-root/
│
├── .github/workflows/
│   └── ci.yml
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── src/
│   └── app.js
├── Dockerfile
└── README.md
```

## Key Learnings
- Built a production-grade CI pipeline using GitHub Actions
- Implemented containerization via Docker
- Stored images securely in Docker Hub
- Deployed to Kubernetes using declarative IaC
- Configured resource requests/limits to prevent noisy-neighbor issue 
- Practiced real-world DevOps workflows from code -> container -> deploy

## Future Enhancements 
- Add Helm chart packaging 
- Implement GitOps with ArgoCD
- Add automated tests into CI workflow
- Deploy to EKS with Terraform
- Add Horizontal Pod Autoscaling 
- Add Ingress resource with NGINX

## Author
### Lulu Osofisan
DevOps • Cloud • Platform Engineer

- GitHub: bosofisan
- Docker Hub: bosofisan