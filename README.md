# Dockerized Node.js App with CI/CD & Kubernetes Deployment
- A complete, production-style DevOps workflow demonstrating:
- Containerization with Docker
- Continuous Integration with GitHub Actions
- Image registry with Docker Hub
- Continuous Deployment to Kubernetes
- Fully declarative YAML manifests
- Automated build → push → deploy pipeline



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
- Node.js – lightweight web app
- Docker – build & containerize
- GitHub Actions – CI pipeline
- Docker Hub – container registry
- Kubernetes (kind) – deployment & orchestration
- YAML – infrastructure as code

## CI Pipeline (GitHub Actions)
Every push to main triggers:
1. Checkout repo
2. Build Docker image
3. Authenticate to Docker Hub
4. Push latest image tag
5. Kubernetes downloads the new image on next deploy

.github/workflows/ci.yml
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

## Run the App Locally on Kubernetes
```yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get pods
kubectl port-forward service/dockerized-app-service 3000:3000
```

Visit: http://localhost:3000

## Key Learnings
- Automated Docker image builds using GitHub Actions
- Kubernetes Deployments + Services for container orchestration
- Resource requests/limits to prevent noisy-neighbor issues
- Container registry authentication via GitHub Secrets
- End-to-end CI/CD using industry-standard tooling

## Author
### Lulu Osofisan
DevOps • Cloud • Platform Engineer

- GitHub: bosofisan
- Docker Hub: bosofisan