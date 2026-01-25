# SMS Checker – Project Overview & Repository Guide

This repository (**operation**) serves as the entry point for the **SMS Checker** project.
The project is organized as a set of independent components that communicate via REST APIs. Each component is developed in its own repository. The overall architecture is gradually being evolved from a minimal working prototype into a fully modular, containerized system.

![Project Architecture](assets/image.png)

## 📦 Repository Structure

The organization consists of the following repositories:

| Component         | Description                                         | Repository                                                                                                           |
| ----------------- | --------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **operation**     | Entrypoint / meta-repository linking all components | [https://github.com/doda2025-team20/operation/tree/a1](https://github.com/doda2025-team20/operation/tree/a1)         |
| **model-service** | Backend service providing the model API             | [https://github.com/doda2025-team20/model-service/tree/a1](https://github.com/doda2025-team20/model-service/tree/a1) |
| **app**           | Frontend application consuming backend APIs         | [https://github.com/doda2025-team20/app/tree/a1](https://github.com/doda2025-team20/app/tree/a1)                     |
| **lib-version**   | Version-aware library                               | [https://github.com/doda2025-team20/lib-version/tree/a1](https://github.com/doda2025-team20/lib-version/tree/a1)     |

Each component is intended to be built, released, and deployed independently.

## 🚀 Project Context

The SMS Checker application currently includes only minimal functionality: a simple HTML/JavaScript frontend connected to a trivial backend.
Initially, the frontend is served through Spring Boot, with an API gateway forwarding requests to the model service to avoid cross-site scripting issues.

Throughout the course, the architecture is extended so that all components operate independently and interact solely via REST APIs or shared, versioned dependencies.
The **app** and **model-service** components are expected to be released as separate container images, deployable independently.
A version-aware library (**lib-version**) will also be introduced to demonstrate dependency versioning and reuse.

---

## 🐳 Docker Compose Setup

The `docker-compose.yml` file in this repository orchestrates the deployment of the entire system for local development and testing.

### Prerequisites
Ensure you have Docker and Docker Compose installed.

### Running the Application

You can start the application using either Docker Compose v2 (recommended) or v1.

**Using Docker Compose v2:**
```bash
docker compose up -d
```

**Using Docker Compose v1:**
```bash
docker-compose up -d
```

### Accessing the Application

Once the containers are running, the application is accessible at:
- **SMS Checker**: [http://localhost:8080/sms](http://localhost:8080/sms)
- **About Page (Lib Version Info)**: [http://localhost:8080/about](http://localhost:8080/about)

### ⚙️ Configuration (.env)

The setup is highly configurable via the `.env` file. Below is a detailed description of all available environment variables:

| Variable | Default Value | Description |
| :--- | :--- | :--- |
| **Images** | | |
| `APP_IMAGE` | `ghcr.io/doda2025-team20/app:latest` | The container image for the frontend application. |
| `MODEL_IMAGE` | `ghcr.io/doda2025-team20/model-service:latest` | The container image for the backend model service. |
| **Ports** | | |
| `APP_PORT` | `8080` | The internal port on which the app service listens. |
| `MODEL_PORT` | `8081` | The internal port on which the model service listens. |
| `HOST_APP_PORT` | `8080` | The port on your host machine that maps to the app service. |
| **Service Wiring** | | |
| `MODEL_HOST` | `http://model-service:8081` | The URL used by the app service to communicate with the model service internally. |
| **Model Service Config** | | |
| `MODEL_URL` | *(Latest Release URL)* | URL to download the model zip file. If empty, the service uses its internal default. |
| `DEBUG` | `false` | Enables Flask debug mode in the model service if set to `true`. |

### 💾 Volume Mappings

| Service           | Volume Mapping         | Description                                                                                                                                  |
| ----------------- | ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| **model-service** | `./output:/sms/output` | Directory for model used by `model-service`. Will be automatically populated on first startup if empty, otherwise its contents will be used. |

---

## ▶️ Kubernetes Deployment (Vagrant)

In order to run the VMs, and set up the Kubernetes Cluster, you can do:

```bash
cd infra
vagrant up
```

This will create a ctrl VM and two worker node VMs. After that is finished, you need to get the proper tools and then run the finalization.yaml playbook in order to set up MetalLB load balancer and Ingress. Before that make sure you have the community.kubernetes collection installed:

```bash
ansible-galaxy collection install community.kubernetes
```

After that is done, we run `finalization.yaml` to set up MetalLB and Ingress:

```bash
ansible-playbook -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory playbooks/finalization.yaml
```


---

## ⚠️ Upcoming Changes

The repository layout and startup procedures will be updated as the architectural extensions are introduced.
This README will be maintained to reflect those changes.
