### Dockerfile
```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
CMD ["python", "main.py"]
```
### docker-compose.yml
```yaml
version: "3.9"
services:
  myapp:
    build: .
    ports:
      - "8000:8000"
    environment:
      - ENV=local
```
### GitHub Actions workflow .github/workflows/ci-cd.yml
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Set up Docker
        uses: docker/setup-buildx-action@v2

      - name: Build and Push Docker image
        run: |
          docker build -t mydockerhub/myapp:${{ github.sha }} .
          echo $DOCKERHUB_TOKEN | docker login -u $DOCKERHUB_USER --password-stdin
          docker push mydockerhub/myapp:${{ github.sha }}

  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Set up kubectl
        uses: azure/setup-kubectl@v3
        with:
          version: 'v1.29.0'

      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/myapp myapp=mydockerhub/myapp:${{ github.sha }}
          kubectl rollout status deployment/myapp
```
### Kubernetes Deployment deployment.yml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: mydockerhub/myapp:latest
          ports:
            - containerPort: 8000
```