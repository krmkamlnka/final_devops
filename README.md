# DevOps Final Exam Project — Karim Japparov

## Overview
This repository contains a complete DevOps workflow for a Spring Boot ToDo application:
- Source control: GitHub
- Containerization: Docker and Docker Compose
- CI/CD: Jenkins (installed on the VM)
- Orchestration: Kubernetes (Minikube)
- Automation: Ansible (idempotent) and Ansible Vault
- Verification evidence (Part 5)

Application stack:
- Spring Boot (ToDo REST API)
- PostgreSQL

Docker image pushed by Jenkins:
- krmka/todo-app:1.1

## Architecture
High-level flow:
Developer (Mac) -> GitHub -> Jenkins (VM) -> Docker Hub -> VM deploy (Docker Compose) / Kubernetes (Minikube)

Runtime architecture (Kubernetes):
Client
  |
  | NodePort 30889
  v
Service todo-app (NodePort) -> Deployment todo-app -> Pod(s) (Spring Boot)
                                      |
                                      | ClusterIP 5432
                                      v
                               Service todo-db (ClusterIP) -> Deployment todo-db -> Pod (PostgreSQL)

## Repository structure
- ToDoApp/                         Spring Boot application source code
  - Dockerfile_KarimJapparov
- docker/                          Docker Compose deployment files
  - docker-compose_KarimJapparov.yml
- k8s/                             Kubernetes manifests
  - deployment_KarimJapparov.yaml
  - service_KarimJapparov.yaml
  - configmap_KarimJapparov.yaml
  - secret_KarimJapparov.yaml
  - hpa_KarimJapparov.yaml
- ansible/                         Ansible automation
  - inventory_KarimJapparov.ini
  - playbook_KarimJapparov.yml
  - vault_KarimJapparov.yml (encrypted)

## Part 1 — Docker / Docker Compose
Start the application using Docker Compose (PostgreSQL + App):

cd /opt/devops-project/docker
docker compose -f docker-compose_KarimJapparov.yml --env-file .env up -d --build
docker compose -f docker-compose_KarimJapparov.yml ps

## Part 2 — Jenkins CI/CD
Jenkins is installed on the VM and configured with:
- GitHub credentials (PAT)
- Docker Hub credentials

Pipeline:
- Checkout from GitHub
- Build (tests skipped in current Jenkinsfile)
- Build and push Docker image to Docker Hub
- Deploy on the VM using Docker Compose

Pipeline file:
- Jenkinsfile_KarimJapparov

## Part 3 — Kubernetes (Minikube)
Start Minikube and enable metrics-server:

minikube config set driver docker
minikube start --driver=docker
minikube addons enable metrics-server

Apply manifests:

cd /opt/devops-project/k8s
kubectl apply -f deployment_KarimJapparov.yaml
kubectl apply -f configmap_KarimJapparov.yaml
kubectl apply -f secret_KarimJapparov.yaml
kubectl apply -f service_KarimJapparov.yaml
kubectl apply -f hpa_KarimJapparov.yaml

Check resources:

kubectl get pods -n todo -o wide
kubectl get svc -n todo
kubectl get hpa -n todo

Access service:

minikube service todo-app -n todo --url

Rolling update / rollback example:

kubectl rollout history deployment/todo-app -n todo
kubectl set image deployment/todo-app -n todo app=krmka/todo-app:1.1
kubectl rollout status deployment/todo-app -n todo
kubectl rollout undo deployment/todo-app -n todo
kubectl rollout status deployment/todo-app -n todo

## Part 4 — Ansible (Infrastructure Automation)
Ansible automates:
- Docker installation and service setup
- Kubernetes tooling installation (kubectl, minikube) and metrics-server addon
- Jenkins installation and service setup
- Rendering and applying Kubernetes manifests
- Sensitive variables are stored in Ansible Vault

Run:

cd /opt/devops-project/app/ansible
ansible-playbook -i inventory_KarimJapparov.ini playbook_KarimJapparov.yml --ask-vault-pass --ask-become-pass

Idempotency check (run twice):

ansible-playbook -i inventory_KarimJapparov.ini playbook_KarimJapparov.yml --ask-vault-pass --ask-become-pass
ansible-playbook -i inventory_KarimJapparov.ini playbook_KarimJapparov.yml --ask-vault-pass --ask-become-pass

## Part 5 — Verification evidence
Collect outputs or screenshots for:
1) Jenkins pipeline success (Console Output)
2) Kubernetes resources:

kubectl get pods -n todo -o wide
kubectl get svc -n todo
kubectl get hpa -n todo

3) Application response (use the URL returned by minikube):

curl -i http://<minikube_service_url>/

Note: Requesting "/" returns HTTP 500 in this API because there is no static resource mapped for "/".
Use a real API endpoint (for example: /api/todos) to verify functionality.

## File naming requirements
Key deliverable files include the suffix KarimJapparov, for example:
- Dockerfile_KarimJapparov
- docker-compose_KarimJapparov.yml
- Jenkinsfile_KarimJapparov
- deployment_KarimJapparov.yaml, service_KarimJapparov.yaml, configmap_KarimJapparov.yaml, secret_KarimJapparov.yaml, hpa_KarimJapparov.yaml
- playbook_KarimJapparov.yml, inventory_KarimJapparov.ini, vault_KarimJapparov.yml
