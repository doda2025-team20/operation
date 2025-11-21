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

## ▶️ Deployment

The `docker-compose.yml` file in this repository orchestrates the deployment of the entire system. To deploy the application, ensure Docker and Docker Compose are installed. Download the `docker-compose.yml` file and run the following command in the terminal:

```bash
docker-compose up -d
```

The application will then be accessible at `http://localhost:8080`.

The `docker-compose.yml` file specifies the use of container images for the **app** and **model-service** components, which are hosted on GitHub Container Registry, at `ghcr.io/doda2025-team20/app:latest` and `ghcr.io/doda2025-team20/model-service:latest`, respectively.

The following sections detail the configurations that can be adjusted in the `docker-compose.yml` file.

### 🚪 Ports

| Service           | Default Port Mapping | Description                                                                |
| ----------------- | -------------------- | -------------------------------------------------------------------------- |
| **app-service**   | `8080:8080`          | Frontend application access point.                                         |
| **model-service** | `8081:8081`          | Backend model service API endpoint, **not exposed externally by default**. |

### ⚙️ Environment Variables

| Service           | Environment Variable | Default Value               | Description                                                                                                        |
| ----------------- | -------------------- | --------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **app-service**   | `MODEL_HOST`         | `http://model-service:8081` | Complete URL of the model service for API requests. Use the internal Docker network hostname and port.             |
|                   | `APP_PORT`           | `8080`                      | Port on which the app service listens internally. Ensure to update the relevant port mapping accordingly.          |
| **model-service** | `MODEL_PORT`         | `8081`                      | Port on which the model service listens internally. Ensure to update the relevant port mapping accordingly.        |
|                   | `MODEL_URL`          |                             | URL to download the model for the service. The empty default will download from the URL hard-coded in the service. |
|                   | `DEBUG`              | `false`                     | Enable or disable the Flask debug mode.                                                                            |

### 💾 Volume Mappings

| Service           | Volume Mapping         | Description                                                                                                                                  |
| ----------------- | ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| **model-service** | `./output:/sms/output` | Directory for model used by `model-service`. Will be automatically populated on first startup if empty, otherwise its contents will be used. |

---

## ⚠️ Upcoming Changes

The repository layout and startup procedures will be updated as the architectural extensions are introduced.
This README will be maintained to reflect those changes.
