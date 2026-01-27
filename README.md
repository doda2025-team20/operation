# SMS Checker – Project Overview & Repository Guide

This repository (**operation**) serves as the main entry point for the **SMS Checker** project.
The project is organized as a set of independent components that communicate via REST APIs. Each component is developed in its own repository and released independently. Over the course of the project, the architecture evolved from a minimal prototype into a containerized system with Kubernetes, ingress, service mesh, and monitoring support.

This document serves as the single source of truth for setting up the different components of the system. This document takes precendence over any other documentation found in the individual repositories or subdirectories.

![Project Architecture](assets/image.png)

---

## Repository Structure

The organization consists of the following repositories:

| Component         | Description                                | Repository                                                                                           |
| ----------------- | ------------------------------------------ | ---------------------------------------------------------------------------------------------------- |
| **operation**     | Infrastructure, deployment, and operations | [https://github.com/doda2025-team20/operation](https://github.com/doda2025-team20/operation)         |
| **model-service** | Backend ML inference service               | [https://github.com/doda2025-team20/model-service](https://github.com/doda2025-team20/model-service) |
| **app**           | Frontend application and API gateway       | [https://github.com/doda2025-team20/app](https://github.com/doda2025-team20/app)                     |
| **lib-version**   | Version-aware shared library               | [https://github.com/doda2025-team20/lib-version](https://github.com/doda2025-team20/lib-version)     |
| **team20-flux**   | GitOps configuration (Extension)           | [https://github.com/doda2025-team20/team20-flux](https://github.com/doda2025-team20/team20-flux)     |

Each component is built, versioned, and deployed independently.

---

## Assignment 1 - Release Pipelines and Containerization

For local development and testing, the entire system can be started using Docker Compose. Both images for the `app` and `model-service` are available on the GitHub container registry, and are publicly accessible. A provided `.env` file sets deployment parameters, such as images, published ports, and model version used, and can be modified directly to update the Docker Compose deployment without touching its `.yaml` configuration.

### Prerequisites

* Docker
* Docker Compose

### Running the application

Adjust `.env` to your preferences as required (more details available below). Then, just run:

```bash
docker compose up -d
```

### Accessing the application

* **SMS Checker**: [http://localhost:8080](http://localhost:8080) => Redirects to `/sms`
* **Metrics**: [http://localhost:8080/metrics](http://localhost:8080/metrics)
* **About**: [http://localhost:8080/about](http://localhost:8080/about)

### Configuration (`.env`)

The deployment is configurable via the `.env` file:

| Variable        | Default                                        | Description            |
| --------------- | ---------------------------------------------- | ---------------------- |
| `APP_IMAGE`     | `ghcr.io/doda2025-team20/app:latest`           | Frontend image         |
| `MODEL_IMAGE`   | `ghcr.io/doda2025-team20/model-service:latest` | Backend image          |
| `APP_PORT`      | `8080`                                         | App container port     |
| `MODEL_PORT`    | `8081`                                         | Model container port   |
| `HOST_APP_PORT` | `8080`                                         | Host port              |
| `MODEL_HOST`    | `http://model-service:8081`                    | App -> model routing   |
| `MODEL_VERSION` | `latest`                                       | Model download version |
| `DEBUG`         | `false`                                        | Flask debug mode       |

### Volumes

| Service       | Mapping                | Purpose                 |
| ------------- | ---------------------- | ----------------------- |
| model-service | `./output:/sms/output` | Persisted model storage |

---

### Assignment 2 - Cluster Provisioning

The `infra/` directory contains all infrastructure-as-code required to provision a Kubernetes cluster locally using Vagrant and Ansible. For testing, we have used a VirtualBox provider.

The `Vagrantfile` contains at the top the configuration parameters for the cluster nodes, such as CPU and RAM of controller and worker nodes, the number of the latter being adjustable as well.

To provision the cluster, navigate to the `infra/` directory and run:

```bash
vagrant up
ansible-playbook -i inventory.cfg playbooks/finalization.yaml
```

(The `inventory.cfg` file will be created by Vagrant during the `vagrant up` process. If not, the file at `.vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory` may be used.)

By default, this creates:

* one control-plane node (4 CPUs, 4GB RAM)
* two worker nodes (2 CPUs, 2GB RAM each)

Please adjust the above parameters to your system capabilities as needed.

Upon succesful provisioning, the `kubeconfig` file will be available at `infra/kubeconfig`, and may be used to access the cluster using `kubectl` from the host machine:

Further documentation can be found in `infra/k8s/README.md`, including instructions for installing and accessing the Kubernetes Dashboard.

By default, the cluster uses the following IP addresses for the nodes:
* Controller: `192.168.56.100`
* Workers: `192.168.56.100 + N` (where N is 1, 2, ... for each worker)
* MetalLB IP range: `192.168.56.90 - 192.168.56.99`

## Kubernetes Application Deployment (Helm)

The SMS Checker application is deployed to Kubernetes using a **single Helm chart** located in `helm-chart/`.

### What the Helm chart installs

* App and model Deployments and Services
* Istio Gateway, VirtualServices, and DestinationRules
* Prometheus `ServiceMonitor` resources
* Prometheus alert rules
* Grafana with pre-provisioned dashboards and datasources
* Secrets and ConfigMaps for configuration

Detailed Helm usage, prerequisites, and verification steps are documented here:

* **Helm chart, monitoring & dashboards** -> [`helm-chart/README.md`](./helm-chart/README.md)

---

## Accessing the Application via Ingress

### Istio Ingress (Kubernetes)

After installing Istio and deploying the Helm chart:

```bash
kubectl get svc -n istio-system istio-ingressgateway
```

Retrieve the external IP or port (depending on environment).

#### Minikube

```bash
minikube service istio-ingressgateway -n istio-system --url
```

Then access:

```
http://<INGRESS_ADDRESS>/sms
```

Metrics are available at:

```
http://<INGRESS_ADDRESS>/metrics
```

---

## Monitoring: Prometheus & Grafana

Monitoring is deployed as part of the Helm chart.

### Prometheus

* Scrapes metrics exposed by both app and model services
* Uses `ServiceMonitor` resources
* Includes alert rules

Example metrics:

* `sms_requests_total`
* `sms_last_confidence`
* `sms_request_duration_seconds`

### Grafana

* Automatically provisioned dashboards
* Prometheus datasource configured via ConfigMaps

Access instructions and dashboard descriptions are available in:

* `helm-chart/README.md`

---

## Service Mesh & Traffic Management

The project uses **Istio** to implement advanced traffic management.

### Architecture

* **Istio Ingress Gateway**: Handles all incoming traffic.
* **VirtualServices & DestinationRules**: Manage routing between `v1` (stable), `v2` (canary), and `v3` (shadow) versions of the services.
* **Envoy Sidecars**: Injected into every application pod to intercept and control traffic.

Detailed explanation of the deployment strategies (Canary vs. Shadow Launch) and data flow can be found in:

* **Deployment & Traffic Flow Documentation** -> [`docs/deployment.md`](./docs/deployment.md)

### Implementation Details

The Istio configuration is **fully integrated into the Helm chart**.
The resources (VirtualServices, Gateways, etc.) are defined in `helm-chart/templates/`.

---

## Additional Documentation

* **Team activity log**: `ACTIVITY.md`

---

## Project Extension: GitOps (Assignment A5)

For the optional extension, we implemented a **GitOps-based deployment pipeline using Flux**.

This replaces the manual Helm commands with an automated, declarative workflow where the cluster state is continuously reconciled with a Git repository.

* **Extension Documentation & Proposal** -> [`docs/extension.md`](./docs/extension.md)
* **Flux Configuration Repository** -> [doda2025-team20/team20-flux](https://github.com/doda2025-team20/team20-flux)

### Configuring Flux with Vagrant

To set up Flux in the Vagrant-provisioned cluster, follow these steps:
1. Go to the flux playbook for vagrant [flux.yaml](./infra/playbooks/flux.yaml)
2. Update the github token variable with your personal access token that has repo access
3. Run the playbook with the following command:
```bash
ansible-playbook -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory playbooks/flux.yaml
```

---


## Ongoing Evolution

The repository layout and startup procedures will be updated as the architectural extensions are introduced.
This README will be maintained to reflect those changes.
