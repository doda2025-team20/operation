# SMS Checker – Project Overview & Repository Guide

This repository (**operation**) serves as the main entry point for the **SMS Checker** project.
The project is organized as a set of independent components that communicate via REST APIs. Each component is developed in its own repository and released independently. Over the course of the project, the architecture evolved from a minimal prototype into a containerized system with Kubernetes, ingress, service mesh, and monitoring support.

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

### Cluster Provisioning (Vagrant + Ansible)

To provision a Kubernetes cluster locally using virtual machines:

```bash
cd infra
vagrant up
```

This creates:

* one control-plane node
* two worker nodes

After this is finished, you need to get the proper tools and then run the `finalization.yaml` playbook in order to set up **MetalLB** load balancer and **Ingress**. Before that make sure you have the `community.kubernetes` collection installed:

```bash
ansible-galaxy collection install community.kubernetes
```
If Ansible Galaxy returns an HTTP 500 error, install the collection directly from GitHub instead:
```bash
ansible-galaxy collection install "git+https://github.com/ansible-collections/community.kubernetes.git,2.0.0" --force
```
After installation is verified, we run `finalization.yaml` to set up MetalLB and Ingress:

```bash
ansible-playbook -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory playbooks/finalization.yaml
```

All Kubernetes tooling, access instructions, and operational details are documented here:

* **Cluster services & tooling** -> [`infra/k8s/README.md`](./infra/k8s/README.md)
---

## Assignment 3 - Kubernetes Deployment and Monitoring

The SMS Checker application is deployed to Kubernetes using a **single Helm chart** located in `helm-chart/`.

The Helm chart installs the following components:

* App and model Deployments and Services
* Istio Gateway, VirtualServices, and DestinationRules
* Prometheus `ServiceMonitor` resources
* Prometheus alert rules
* Grafana with pre-provisioned dashboards and datasources
* Secrets and ConfigMaps for configuration

To deploy the Helm chart, upon editing `values.yaml` to your preferences as required (remember to disable Istio config if not available), follow these steps:

```bash
helm repo update
helm dependency update ./helm-chart
helm install assignment-release ./helm-chart
```

Verify that all components are running with `kubectl get pods`.

The `app` Deployment should be available either through an Nginx Ingress or Istio Ingress Gateway, depending on your cluster setup, at `http://localhost` (no special host configuration has been made). The Grafana dashboard should also be available at `http://localhost/grafana` (default credentials: `admin` / `password123`, unles otherwise set in your `values.yaml`).

The `model-service` and all other components are not publically available through Ingress, and would require port-forwarding. Ensure canary and shadow versions are disabled in all relevant options, when not using Istio, as not doing so may have adverse effects in routing.

Further information and commands regarding access in adverse scenarios may be available in the `helm-chart/README.md`, as well as the `docs/` directory.

## Assignment 4 - Istio & Traffic Management

The project uses **Istio** to implement advanced traffic management. Ensure to activate Istio sidecar injection in the target namespace before deploying the Helm chart. For the default namespace:

```bash
kubectl label namespace default istio-injection=enabled
```

It is recommended to do a full uninstall and install of the Helm chart after enabling Istio injection, to ensure all pods have sidecars injected.

The service mesh allows activating further options in the `values.yaml` file of the Helm chart, such as a canary deployment of the `app` and `model-service`, as well as shadow launching a third version of the `model-service`. These options leverage Istio's VirtualServices and DestinationRules to route traffic accordingly. Other than activating all relevant options in `values.yaml`, no further configuration is required, as the Helm chart fully integrates Istio resources.

### Architecture

* **Istio Ingress Gateway**: Handles all incoming traffic.
* **VirtualServices & DestinationRules**: Manage routing between `v1` (stable), `v2` (canary), and `v3` (shadow) versions of the services, where available. By default, 10% of traffic is routed to the canary version. For version-sticky sessions, the `canary` cookie can be set to `v1` or `v2` manually. No separate hostname has been defined for the canary traffic, but setting that cookie to the desired version in the browser developer tools will route traffic accordingly.
* **Envoy Sidecars**: Injected into every application pod to intercept and control traffic.

Further documentation on the deployment strategies can be found in [`docs/deployment.md`](./docs/deployment.md). Our experimental function using the service mesh is documented in [`docs/continuous-experimentation.md`](./docs/continuous-experimentation.md).

### Project Extension: GitOps

For the optional extension, we implemented a **GitOps-based deployment pipeline using Flux**.

This replaces the manual Helm commands with an automated, declarative workflow where the cluster state is continuously reconciled with a Git repository.

* **Extension Documentation & Proposal**: [`docs/extension.md`](./docs/extension.md)
* **Flux Configuration Repository**: [doda2025-team20/team20-flux](https://github.com/doda2025-team20/team20-flux)

### Configuring Flux with Vagrant

To set up Flux in the Vagrant-provisioned cluster, follow these steps:
1. Go to the flux playbook for vagrant `./infra/playbooks/flux.yaml`
2. Update the GitHub token variable with your personal access token that has repo access
3. Run the playbook with the following command:
```bash
ansible-playbook -i inventory.cfg playbooks/flux.yaml
```

(In case `inventory.cfg` is not available consult the Assignment 2 section for further information.)

## Additional Documentation

* **Team activity log**: `ACTIVITY.md`