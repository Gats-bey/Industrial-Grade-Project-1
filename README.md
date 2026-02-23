# IGP1 — Enterprise DevOps Pipeline
### Post Graduate Program in DevOps | Edureka × Purdue University

**Author:** Adewole Shitta Bey  
**Company:** ABC Technologies  
**Status:** ✅ Complete  
**Period:** January 24 – January 30, 2026

---

## Overview

This repository contains the complete source code, pipeline configuration, and infrastructure automation for **Industrial Grade Project 1 (IGP1)** — the capstone project of the Post Graduate Program in DevOps.

The project implements a full, end-to-end enterprise DevOps pipeline for a Java-based retail web application, taking it from raw source code all the way through to a containerised, orchestrated, and monitored production deployment — fully automated via CI/CD.

---

## Pipeline Architecture

```
Developer Push
      │
      ▼
   GitHub ──────────────────────────────────────────────┐
      │                                                  │
      ▼                                                  │
   Jenkins (CI/CD Orchestration)                         │
      │                                                  │
      ├──▶ Maven Build & Test ──▶ WAR Package            │
      │                                                  │
      ├──▶ [Option] Deploy to VM Tomcat                  │
      │                                                  │
      ├──▶ [Option] Docker Build & Run                   │
      │                                                  │
      ├──▶ [Option] Ansible Playbook ──▶ Docker Deploy   │
      │                                                  │
      └──▶ [Option] Kubernetes (K3s) ──▶ 2 Replicas     │
                                              │          │
                                              ▼          │
                                     Prometheus + Grafana│
                                     (Monitoring Stack)  │
                                                         │
      ◀───────────────────────── SCM Poll (every 5 min) ─┘
```

---

## Technology Stack

| Category | Technology | Version |
|---|---|---|
| Source Control | GitHub | — |
| Build Tool | Apache Maven | 3.x |
| CI/CD | Jenkins | Latest |
| Containerisation | Docker | Latest |
| Infrastructure Automation | Ansible | 2.16.3 |
| Container Orchestration | K3s (Kubernetes) | v1.34.3 |
| Package Manager (K8s) | Helm | v3.20.0 |
| Metrics Collection | Prometheus | v3.9.1 |
| Dashboards | Grafana | Latest |
| Alerting | Alertmanager | Latest |
| Runtime | OpenJDK | 11.0.29 |
| Operating System | Ubuntu | 24.04.3 LTS |

---

## Project Phases

### Phase 1 — Environment Setup, Build, Test & Deploy to Tomcat

Establishes the foundational environment and proves the application can be built, tested, and deployed reliably before any automation is introduced.

- Ubuntu 24.04 VM with Git, Java 11, Maven installed and verified
- Maven lifecycle: `compile → test → package`
- **4 JUnit tests passed, 0 failures**
- WAR artifact produced: `ABCtechnologies-1.0.war` (6.9 MB)
- Deployed to Apache Tomcat 10 via `webapps/` auto-deployer
- Application verified via browser at `http://10.214.99.251:8080/ABCtechnologies-1.0/`

---

### Phase 2 — Source Control with Git & GitHub

Places the application under version control and establishes secure, password-less SSH authentication between the development VM and GitHub.

- Git repository initialised; default branch renamed to `main`
- `.gitignore` configured to exclude `target/` and `.DS_Store`
- ED25519 SSH key generated and registered with GitHub
- Remote repository connected and code pushed
- Repository prepared as the single source of truth for all CI/CD automation

---

### Phase 3 — Jenkins CI/CD Pipeline

Implements a fully automated CI/CD pipeline using a declarative `Jenkinsfile` stored in source control (Pipeline as Code).

**Pipeline Stages:**

| Stage | Action |
|---|---|
| Checkout | Clone from GitHub via SSH |
| Build & Test | `mvn -B clean test` — 4 unit tests must pass |
| Package WAR | `mvn -B package -DskipTests` |
| Stage Artifact | Copy WAR to `/opt/jenkins-deploy/` |
| Deploy | Controlled root-owned deploy script |
| Health Check | `curl` loop — expects HTTP 200 within 50 seconds |
| Archive | WAR stored as Jenkins build artifact |

**Key design decisions:**
- Jenkins runs on port `8081` (Tomcat on `8080`) to avoid binding conflicts
- Least-privilege `sudoers` drop-in — Jenkins can only execute `/usr/local/bin/deploy_igp1.sh`
- Timestamped `.bak` WAR backup created on every deployment for instant rollback
- RSA PEM key used for Jenkins GitHub authentication (ED25519 incompatible with Jenkins Git client)
- Single **choice parameter** (`DEPLOYMENT_METHOD`) prevents conflicting deployments

**Supported deployment targets:**

| Option | Description |
|---|---|
| `ansible` | Ansible-automated Docker deployment (recommended) |
| `docker-direct` | Direct Docker CLI commands |
| `vm-tomcat` | Traditional VM Tomcat deployment |
| `none` | Build and test only, no deployment |

---

### Phase 4 — Ansible Infrastructure Automation

Introduces Infrastructure as Code (IaC) by replacing direct deployment commands with an idempotent Ansible playbook.

- Ansible 2.16.3 with `community.docker` collection
- Playbook: `/opt/ansible-igp1/deploy-docker.yml`
- Validates Dockerfile and WAR exist before deploying
- Builds Docker image, stops/removes old container, starts new container
- HTTP health check confirms `200 OK` before marking deployment successful
- Playbook committed to Git — all infrastructure changes are version-controlled and auditable
- Jenkins invokes Ansible via pipeline stage; clean separation of CI/CD orchestration from deployment execution

---

### Phase 5 — Kubernetes Orchestration with K3s

Transitions from single-container Docker deployment to full Kubernetes orchestration with high availability and self-healing.

- K3s v1.34.3 installed (lightweight, fully Kubernetes-conformant)
- `kubectl` configured for both `ubuntu` and `jenkins` users
- Kubernetes manifests: `Deployment` (2 replicas) + `NodePort` Service
- Ansible K8s playbook automates image import and manifest application
- **Rolling updates** — zero-downtime deployments on every pipeline run
- **Self-healing** — Kubernetes automatically restarts failed pods
- Application accessible at `http://localhost:30080`
- End-to-end validated: Jenkins Build #19 → running K8s pods confirmed

---

### Phase 6 — Monitoring with Prometheus & Grafana

Implements full observability across both the infrastructure and application layers.

- `kube-prometheus-stack` deployed via Helm (includes Prometheus Operator, Grafana, Alertmanager)
- `ServiceMonitor` resource configured to scrape application pods at `/metrics` every 30 seconds
- Live metrics verified via both Grafana dashboards and `kubectl top`

**Node metrics observed:**

| Metric | Value |
|---|---|
| CPU Utilisation | 8% (346m cores) |
| Memory Usage | 79% (3116 MiB) |
| Disk Usage | 51.4% of 24 GiB |

**Pod metrics observed:**

| Pod | CPU | Memory |
|---|---|---|
| `igp1-app-85469959f5-5lj58` | 9m cores | 170 MiB |
| `igp1-app-85469959f5-fj7jw` | 9m cores | 166 MiB |

---

## Service Access Points

| Service | URL | Purpose |
|---|---|---|
| ABC Technologies App (K8s) | `http://localhost:30080` | Primary deployment |
| ABC Technologies App (Docker) | `http://localhost:8082` | Docker/Ansible deployment |
| ABC Technologies App (Tomcat) | `http://localhost:8080` | Traditional VM deployment |
| Jenkins CI/CD | `http://localhost:8081` | Build pipeline management |
| Grafana | `http://localhost:30683` | Metrics dashboards |
| Prometheus | `http://localhost:30808` | Metrics storage & querying |

---

## Repository Structure

```
IGP1/
├── src/                        # Java application source code
│   └── main/java/
│       ├── RetailModule.java
│       ├── RetailDataImp.java
│       └── RetailAccessObject.java
├── src/test/                   # JUnit test classes
│   └── ProductImpTest.java
├── Dockerfile                  # Container image definition
├── Jenkinsfile                 # Declarative CI/CD pipeline (Pipeline as Code)
├── pom.xml                     # Maven build configuration
└── ansible/
    └── deploy-docker.yml       # Ansible deployment playbook
```

---

## Getting Started

### Prerequisites

| Tool | Version | Install |
|---|---|---|
| Java (OpenJDK) | 11 | `sudo apt install openjdk-11-jdk -y` |
| Maven | 3.x | `sudo apt install maven -y` |
| Git | Latest | `sudo apt install git -y` |
| Docker | Latest | `sudo apt install docker.io -y` |
| Jenkins | Latest | See [Jenkins install guide](https://www.jenkins.io/doc/book/installing/linux/) |
| K3s | v1.34.3 | `curl -sfL https://get.k3s.io | sh -` |
| Ansible | 2.16.3 | Pre-installed on Ubuntu 24.04 |
| Helm | v3.20.0 | See [Helm install guide](https://helm.sh/docs/intro/install/) |

### Build Locally

```bash
# Clone the repository
git clone git@github.com:YOUR_USERNAME/IGP1.git
cd IGP1

# Compile, test, and package
mvn clean test
mvn package

# Verify the WAR was produced
ls -lh target/*.war
```

### Run with Docker

```bash
# Build the image
docker build -t igp1-app:latest .

# Run the container
docker run -d --name igp1-app -p 8082:8080 igp1-app:latest

# Verify
curl http://localhost:8082/
```

### Deploy with Ansible

```bash
# Run the deployment playbook
ansible-playbook /opt/ansible-igp1/deploy-docker.yml

# Dry run (check mode)
ansible-playbook /opt/ansible-igp1/deploy-docker.yml --check
```

### Deploy to Kubernetes

```bash
# Apply manifests
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

# Verify pods are running
kubectl get pods
kubectl get svc

# Check application
curl http://localhost:30080/
```

---

## CI/CD Trigger

This pipeline uses **Jenkins SCM Polling** — Jenkins checks this repository every 5 minutes for new commits. Any push to the `main` branch will automatically trigger a full pipeline run.

To trigger a build manually: Jenkins UI → Select job → **Build with Parameters** → Choose deployment method → **Build**.

---

## Supplementary Files

| File | Description |
|---|---|
| `IGP1_Documentation2.docx` | Full project documentation across all 6 phases |
| `IGP1_Commands_Chronological.txt` | Every terminal command executed, in order |

---

## Acknowledgements

This project was completed as the capstone of the **Post Graduate Program in DevOps**, jointly delivered by **[Edureka](https://www.edureka.co)** and **Purdue University**.
