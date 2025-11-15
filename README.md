# SMS Checker – Project Overview & Repository Guide

This repository (**operation**) serves as the entry point for the **SMS Checker** project.
The project is organized as a set of independent components that communicate via REST APIs or shared versioned libraries. The overall architecture is gradually being evolved from a minimal working prototype into a fully modular, containerized system.

![project architecture](assets/image.png)

## 📦 Repository Structure

The organization consists of the following repositories:

| Component         | Description                                           | Repository                                                                                           |
| ----------------- | ----------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **operation**     | Entry point / meta-repository linking all components  | [https://github.com/doda2025-team20/operation](https://github.com/doda2025-team20/operation)         |
| **model-service** | Backend service providing the model API               | [https://github.com/doda2025-team20/model-service](https://github.com/doda2025-team20/model-service) |
| **app**           | Frontend application (HTML/JS) consuming backend APIs | [https://github.com/doda2025-team20/app](https://github.com/doda2025-team20/app)                     |
| **lib-version**   | Version-aware shared library (to be implemented)      | [https://github.com/doda2025-team20/lib-version](https://github.com/doda2025-team20/lib-version)     |

Each component is intended to be built, released, and deployed independently.

---

## 🚀 Project Context

The SMS Checker application currently includes only minimal functionality: a simple HTML/JavaScript frontend connected to a trivial backend.
Initially, the frontend is served through Spring Boot, with an API gateway forwarding requests to the model service to avoid cross-site scripting issues.

Throughout the course, the architecture is extended so that all components operate independently and interact solely via REST APIs or shared, versioned dependencies.
The **app** and **model-service** components are expected to be released as separate container images, deployable independently.
A version-aware library (**lib-version**) will also be introduced to demonstrate dependency versioning and reuse.

---

## ▶️ Running the Project Locally (Current State)

At the moment, the system runs by starting each component separately:

### 1️⃣ Backend: model-service

Navigate to the **model-service** repository and follow the setup instructions in its README.
Repository: [https://github.com/doda2025-team20/model-service](https://github.com/doda2025-team20/model-service)

### 2️⃣ Frontend: app

Navigate to the **app** repository and follow the setup instructions in its README.
Repository: [https://github.com/doda2025-team20/app](https://github.com/doda2025-team20/app)

### 3️⃣ Versioned Library: lib-version

The **lib-version** component is not yet implemented.
Repository: [https://github.com/doda2025-team20/lib-version](https://github.com/doda2025-team20/lib-version)

---

## ⚠️ Upcoming Changes

The repository layout and startup procedures will be updated as the architectural extensions are introduced.
This README will be maintained to reflect those changes.
