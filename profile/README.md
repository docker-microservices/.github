# DevOps Microservices Platform

This organization hosts a complete end‑to‑end DevOps project demonstrating microservices architecture, container orchestration with Docker Swarm, and a full CI/CD pipeline using GitHub Actions and a self‑hosted runner.

Screenshots and diagrams related to this setup are available under the `/docs` directory and are referenced throughout this document.

---

## 1. Architecture Overview

The system is built using a microservices architecture with service discovery and centralized routing.

**Core Components:**

* **Eureka Discovery Server** – Service registration and discovery.
* **API Gateway** – Single entry point for backend services.
* **CRUD Microservices**

  * Books Service
  * Authors Service
  * Implemented as Spring Boot REST APIs
  * Uses H2 in‑memory database
  * Swagger / OpenAPI enabled for API documentation and testing
* **Frontend** – Angular application served via Nginx.

**Architecture Diagram:**

* See [Architecture Diagram](docs/architecture.png)

---

## 2. Infrastructure & Deployment Topology

The deployment uses two virtual machines with bridged networking to enable direct IP‑based communication.

### Virtual Machines

**VM1**

* Eureka Discovery Server
* API Gateway
* Docker Swarm Manager

**VM2**

* Angular Frontend
* Backend REST APIs
* Docker Swarm Worker

Both VMs:

* Use bridged network interfaces
* Communicate using static IPs
* Are accessible via SSH by the GitHub self‑hosted runner

**Network Flow Diagram:**

* See [Network Topology](docs/network-topology.png)

---

## 3. Docker Swarm Setup

Docker Swarm is used for container orchestration across the two VMs.

### Swarm Configuration

* Swarm initialized on **VM1** as **Manager**
* **VM2** joined as **Worker** node

### Overlay Network

* A shared Docker overlay network is created from the manager node
* All services communicate through this network

### Node Labels

Node labels are used to control service placement:

* **Manager Node (VM1):** `role=gateway`
* **Worker Node (VM2):** `role=apis`

Services are constrained to nodes based on these labels.
* See [Docker Swarm Screenshots](docs/docker-swarm/)

---

## 4. CI/CD Pipeline

A fully automated CI/CD pipeline is implemented using GitHub Actions and Docker.

### 4.1 Repositories

* All services (frontend, gateway, discovery, and microservices) are hosted in GitHub repositories
* Repositories are grouped under a single GitHub Organization

### 4.2 Self‑Hosted GitHub Runner

* A self‑hosted runner is registered at the **organization level**
* Configured as a **group runner**
* Granted access to all repositories (public and private)

### 4.3 Organization‑Level Secrets

The following secrets are defined at the organization level:

**Docker Hub:**

* `DOCKER_USERNAME`
* `DOCKER_PASSWORD`

**Production VM (VM1):**

* `PROD_HOST`
* `PROD_USER`
* `PROD_SSH_KEY`

These secrets are reused across all repositories.

### 4.4 GitHub Actions Workflow

Each repository contains a CI/CD workflow that includes:

**Package Stage**

* Backend: Maven build and package
* Frontend: Angular production build
* Docker image build
* Push image to Docker Hub

**Deploy Stage**

* SSH into the target VM
* Copy Docker Compose configuration
* Pull latest Docker images
* Deploy or update services in Docker Swarm

### 4.5 Docker Configuration

Each repository includes:

* A Dockerfile for building the service image
* A Docker Compose file for Swarm deployment

Compose configurations:

* Use the shared overlay network
* Apply node constraints using role labels
  
* See [CI/CD Screenshots](docs/ci-cd/)

---

## 5. API Documentation (Swagger / OpenAPI)

All backend microservices expose interactive API documentation using Swagger (OpenAPI).

**Swagger Integration:**

* Enabled on each Spring Boot CRUD service
* Provides interactive UI for testing endpoints
* Automatically documents request/response models

**Access Pattern:**

* Swagger UIs are exposed per service
* Accessible directly via service ports

**Available Swagger Endpoints:**

* Books Service: `http://192.168.1.8:8081/swagger-ui/index.html`
* Authors Service: `http://192.168.1.8:8082/swagger-ui/index.html`

Swagger access can also be routed through the API Gateway if configured accordingly

Screenshots of Swagger endpoints are available in:

* [Swagger Screenshots](docs/swagger/)

---

## 6. Docker Swarm Service Discovery

To ensure proper service discovery and routing inside Docker Swarm, all Eureka client services (API Gateway, Authors Service, Books Service) are configured with:

* Centralized Eureka server URL
* Hostname‑based service registration
* Swarm‑compatible networking behavior

This allows services to resolve each other correctly across nodes.

---

## 6. Nginx, Local DNS, and SSL

### 6.1 Nginx Usage

* Nginx is used to serve the Angular frontend
* Acts as a reverse proxy to the API Gateway

### 6.2 Local DNS Mapping

* Local domain names are mapped to VM IPs using the host machine’s `hosts` file
* Example domain: `local.bookstore`

### 6.3 Local SSL

* Self‑signed SSL certificates are generated for the local domain
* Nginx is configured to:

  * Redirect HTTP to HTTPS
  * Serve the frontend over HTTPS
  * Proxy API requests securely to the API Gateway

* See [Frontend Page](docs/frontend-page.png)

---

## 7. Documentation & Screenshots

Additional documentation, screenshots, and diagrams can be found in the [Docs](docs/) directory:

* Architecture diagrams
* Network topology
* CI/CD pipeline screenshots
* Docker Swarm services and nodes
* Swagger UI endpoints

---

## 8. Summary

This project demonstrates:

* Microservices architecture with Spring Boot and Angular
* Service discovery using Eureka
* Container orchestration with Docker Swarm
* Automated CI/CD with GitHub Actions
* Secure local access using Nginx, HTTPS, and custom DNS

It is designed as a complete reference implementation for DevOps practices in a multi‑service environment.
