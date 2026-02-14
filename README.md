# Industrial Grade Project 1 (IGP1)
# abctechnologies

## 🚀 End-to-End DevOps Pipeline with Kubernetes, CI/CD & Observability

**Industrial Grade Project 1 (IGP1)** is a full-stack DevOps implementation that demonstrates a complete, production-style delivery pipeline — from source code to monitored Kubernetes workloads.

This project reflects **real enterprise DevOps practices**, not a tutorial setup.

---

## 🧱 Architecture Overview




---

## 🛠️ Technology Stack

| Category | Tools |
|--------|------|
| Source Control | GitHub |
| CI/CD | Jenkins (Declarative Pipeline) |
| Build Tool | Maven |
| Containerization | Docker |
| Orchestration | Kubernetes (K3s) |
| Automation | Ansible |
| Monitoring | Prometheus, Grafana |
| OS | Ubuntu 24.04 LTS |

---

## 📦 Application Overview

- Java-based web application
- Packaged as a **WAR**
- Deployed on **Apache Tomcat**
- Containerized and orchestrated via Kubernetes
- Horizontally scalable using Kubernetes Deployments

---

## 🔁 CI/CD Pipeline (Jenkins)

The Jenkins pipeline is parameterized and supports **multiple deployment strategies**:

- `vm-tomcat` – Traditional VM-based deployment
- `docker-direct` – Standalone Docker container
- `ansible` – Docker deployment via Ansible
- `kubernetes` – Production-grade Kubernetes deployment
- `none` – Build-only mode

### Pipeline Stages
- SCM checkout
- Build & unit tests
- WAR packaging
- Artifact archiving
- Docker image build
- Deployment (based on selected method)
- Automated health checks

---

## 🐳 Dockerization

- Custom Dockerfile based on `tomcat:10.1-jdk21`
- WAR deployed as `ROOT.war`
- Exposed on port `8080`
- Versioned images using Jenkins build numbers

---

## ⚙️ Ansible Automation (Phase 4)

Ansible playbooks automate:
- Docker container lifecycle
- Image handling
- Kubernetes manifest updates
- Deployment rollouts
- Environment consistency

---

## ☸️ Kubernetes Orchestration (Phase 5)

- Lightweight **K3s cluster**
- Kubernetes Deployments & Services
- NodePort exposure
- Rolling updates
- Readiness probes for zero-downtime releases
- Ansible-driven Kubernetes deployments
- Image import into containerd (K3s runtime)

---

## 📊 Monitoring & Observability (Phase 6)

Monitoring is implemented using **kube-prometheus-stack** via Helm.

### Components
- Prometheus Operator
- Prometheus
- Alertmanager
- Node Exporter
- Grafana

### Features
- Cluster-level metrics
- Node CPU, memory, disk, and network metrics
- Pod and namespace resource usage
- Custom `ServiceMonitor` for the IGP1 application
- Preloaded Grafana dashboards

---

## 📈 Grafana Dashboards

Included dashboards:
- Kubernetes Cluster Overview
- Kubernetes Namespace (Pods & Workloads)
- Node Exporter Full
- API Server Metrics
- Network & Disk Monitoring

---

## 🔍 Verification & Health Checks

- Jenkins post-deployment health checks
- `kubectl rollout status`
- `kubectl top nodes / pods`
- Prometheus target health validation
- Grafana live metrics visualization

---

## ✅ Project Completion Status

**All phases successfully completed:**

1. Source Control & Build  
2. CI/CD Pipeline  
3. Dockerization  
4. Ansible Automation  
5. Kubernetes Orchestration  
6. Monitoring & Observability  

---

## 🎯 What This Project Demonstrates

- Enterprise-grade CI/CD pipelines
- Infrastructure as Code (IaC)
- Kubernetes production practices
- Observability-first architecture
- End-to-end DevOps lifecycle ownership

---

## 🏁 Final Note

This project is designed to reflect **real-world DevOps engineering standards** and is fully suitable for:

- Portfolio showcase
- Technical interviews
- DevOps / Cloud Engineer roles

**IGP1 — Complete.**


