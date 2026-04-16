# CampusConnect Student Survey
##Docker, Kubernetes & CI/CD Pipeline

**Developer:** Chetana Patel, Gnanitha Suryadevara

**Application:** CampusConnect Student Survey  
**Docker Hub:** [chetsp2613/campusconnect](https://hub.docker.com/r/chetsp2613/campusconnect)  
**GitHub:** [https://github.com/chetsp/SWE-645-Spring-2026-](https://github.com/chetsp/SWE-645-Spring-2026-)

---

## Overview

CampusConnect is a multi-page HTML web application for student surveys, containerized with Docker and deployed to a Kubernetes cluster on AWS using Rancher. A Jenkins CI/CD pipeline automates builds and deployments on every GitHub push.

---

## Project Structure

```
SWE-645-Spring-2026-/
├── index.html               # Home page
├── survey.html              # Student survey form
├── error.html               # Error page
├── campus.jpg               # Campus image
├── Dockerfile               # Docker image build instructions
├── Jenkinsfile              # CI/CD pipeline definition
├── deployment.yaml          # Kubernetes Deployment manifest (3 pods)
├── service.yaml             # Kubernetes Service manifest (LoadBalancer)
└── README.md                # This file
```

---

## 1. Docker

### Build Image (multi-platform for AWS EC2)
```bash
docker buildx create --use
docker buildx build --platform linux/amd64,linux/arm64 -t chetsp2613/campusconnect:1.0 --push .
```

### Run Locally
```bash
docker run -it -p 8182:8080 chetsp2613/campusconnect:1.0
```
Access at: http://localhost:8182/CampusConnect

### Push to Docker Hub
```bash
docker login -u chetsp2613
docker push chetsp2613/campusconnect:1.0
```

---

## 2. Kubernetes Deployment

### Deploy to Cluster
```bash
kubectl create namespace campusconnect-ns
kubectl apply -f nginx-deployment.yaml
kubectl apply -f service.yaml
```

### Verify
```bash
kubectl get pods -n campusconnect-ns
kubectl get services -n campusconnect-ns
```

### Access Application
```
http://ec2-107-22-248-10.compute-1.amazonaws.com:30513/CampusConnect
```

---

## 3. CI/CD Pipeline (Jenkins)

The Jenkins pipeline automatically:
1. Pulls latest code from GitHub
2. Builds a new Docker image tagged with build timestamp
3. Pushes the image to Docker Hub
4. Deploys the new image to Kubernetes

### Trigger
Jenkins polls GitHub every minute (`* * * * *`). Any push to `main` triggers a new build and deployment.

### Jenkins Credentials Required
- `docker-hub-pass` – Docker Hub personal access token

---

## 4. Requirements

- Docker Desktop
- kubectl
- Jenkins (with Docker, Docker Pipeline, Build Timestamp, Kubernetes CLI plugins)
- AWS EC2 with Rancher installed
- Docker Hub account

---

## References

- [Docker Documentation](https://docs.docker.com)
- [Kubernetes Documentation](https://kubernetes.io/docs)
- [Rancher Documentation](https://rancher.com/docs)
- [Jenkins Documentation](https://www.jenkins.io/doc)
