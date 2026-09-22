# MSc Distributed Systems — Flask App on Docker & Kubernetes

## Project Objective
This project containerizes the UBC Flask Sample App — a minimal Python REST API — 
using Docker and deploys it on a local Kubernetes cluster (Kind) with security 
best practices, health checks, vulnerability scanning and distributed systems 
demonstrations (self-healing, scaling, rolling update, rollback).

**Original starter application:** https://github.com/ubc/flask-sample-app


## Application Routes
| Method | Route | Description |
|--------|-------|-------------|
| GET | `/` | Returns `Hello, Flask! v2` |
| GET | `/health` | Health check |
| GET | `/items` | List all items |
| POST | `/items` | Add an item |
| GET | `/items/<id>` | Get item by ID |

## Prerequisites
- Python 3.11+
- Docker Desktop
- Kind
- kubectl

## 1. Run locally
```bash
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
flask run
```
Test: `curl http://localhost:5000/health`

## 2. Run with Docker
```bash
docker build -t flask-app:1.0.0 .
docker run -d -p 5000:5000 flask-app:1.0.0
```
Or with Docker Compose:
```bash
docker compose up -d
```
Test: `curl http://localhost:5000/health`

## 3. Run on Kubernetes (Kind)
```bash
# Create cluster
kind create cluster --name flask-cluster --config kind-config.yaml --image kindest/node:v1.31.0

# Deploy
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/networkpolicy.yaml

# Access
kubectl port-forward service/flask-app 8080:80 -n flask-app
curl http://localhost:8080/health
```

## 4. Key Demonstrations
```bash
# Self-healing
kubectl delete pod <pod-name> -n flask-app
kubectl get pods -n flask-app -w

# Scaling
kubectl scale deployment flask-app -n flask-app --replicas=4

# Rolling update
kubectl set image deployment/flask-app flask-app=hevia24/flask-app:2.0.0 -n flask-app
kubectl rollout status deployment/flask-app -n flask-app

# Rollback
kubectl rollout undo deployment/flask-app -n flask-app
```

## 5. Security
- Non-root user (UID 1000)
- Dropped ALL capabilities
- NetworkPolicy applied
- Vulnerability scan: 0 Critical, 3 High (no fix available)
- SBOM generated with Docker Scout

## Project Structure
```text
msc-de1-distributed-systems-docker-k8s-htde1/
|-- app/
|   |-- __init__.py
|   |-- routes.py
|
|-- k8s/
|   |-- deployment.yaml
|   |-- namespace.yaml
|   |-- networkpolicy.yaml
|   |-- service.yaml
|   |-- optional-config-or-secret.yaml
|
|-- tests/
|   |-- __init__.py
|   |-- test_app.py
|
|-- security/
|   |-- sbom.txt
|   |-- security-scan-v1.0.1.txt
|
|-- evidence/
|
|-- Dockerfile
|-- docker-compose.yml
|-- kind-config.yaml
|-- LICENSE
|-- README.md
|-- requirements.txt
|-- run.py
|-- .dockerignore
|-- .gitignore
```

## Docker Hub
Image available at: https://hub.docker.com/r/hevia24/flask-app

## Running Tests
```bash
python -m unittest discover tests
```