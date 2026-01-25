# Deployment Structure and Data Flow

## Overview

The system is a Kubernetes-based application augmented with **Istio service mesh** to support dynamic traffic routing and controlled experimentation. It consists of a user-facing frontend (**`app`**) and a backend machine-learning inference service (**`model-service`**) that classifies SMS messages as spam or non-spam using an LLM-based model.

The deployment supports **canary deployments** using Istio, allowing multiple versions of both the frontend and the model service to coexist while traffic is dynamically routed between them. Observability is provided via **Prometheus and Grafana**, and Istio is used for ingress traffic control, routing decisions, and telemetry collection.

---

## Deployed Components

### Frontend: `app`

- **Type:** Kubernetes Deployment
- **Role:** User-facing HTTP API
- **Function:** Accepts SMS messages and forwards them to the model service for classification
- **Versions:**
    - `v1` (stable)
    - `v2` (canary, optional)
- **Communication:** Sends HTTP requests to the internal `model-service`
- **Exposure:** Externally accessible via Istio IngressGateway

Each version of the frontend is deployed as a separate Kubernetes Deployment and distinguished using version labels. Both versions expose the same API and are functionally equivalent from the user’s perspective.

### Model Service: `model-service`

- **Type:** Kubernetes Deployment
- **Role:** Backend inference service
- **Function:** Runs an LLM-based spam classification model
- **Versions:**
    - `v1` (stable)
    - `v2` (canary, optional)
    - `v3` (shadow, optional)
- **State:** Requires persistent storage for model output
- **Storage:**
    - PersistentVolumeClaim (PVC) shared across model pods
- **Exposure:** Internal-only service, not directly accessible by users

The model service is invoked exclusively by the frontend and is abstracted behind a Kubernetes Service and Istio routing rules.

---

## Service Mesh and Traffic Management

### Istio Control Plane

- **Istiod:** Manages service mesh configuration, sidecar injection, and routing logic
- **Envoy sidecars:** Injected into application and model pods, enforcing traffic rules and collecting telemetry

### Istio Ingress Gateway

- Acts as the **single entry point** for all external HTTP traffic
- Forwards traffic into the mesh
- Applies routing logic defined by Istio VirtualServices

### Istio Routing Resources

For both the `app` and `model-service`, the following Istio resources are deployed:

- **Gateway:** Defines how external traffic enters the mesh
- **VirtualService:** Defines request routing and traffic splitting behavior
- **DestinationRule:** Defines versioned subsets (`v1`, `v2`) based on pod labels

---

## External Access

The application is accessed through the Istio IngressGateway:

- **Host:** `localhost`
- **Port:** Exposed by the IngressGateway
- **Path:** `/` (root path, prefix-based routing)
- **Authentication:** None
- **Headers:** No special headers required

Users interact only with the frontend service; the model service remains internal to the cluster.

---

## Request Flow

A typical request follows this path:

1. A user sends an HTTP request to the application via `localhost`
2. The request enters the cluster through the **Istio IngressGateway**
3. The IngressGateway forwards the request to the **Envoy proxy**
4. The **VirtualService for the frontend (`app`)** evaluates routing rules
5. The request is forwarded to either `app v1` or `app v2`, based on the configured traffic weights
6. The selected frontend instance processes the request
7. The frontend sends an internal HTTP request to the **model-service**
8. The **VirtualService for the model-service** selects the appropriate model version
9. The model performs inference and returns the result
10. The frontend returns the response to the user via the IngressGateway

All routing decisions are made dynamically by Istio Envoy sidecars and not by application logic.

---

## Traffic Management and Experimental Design

The deployment uses **Istio service mesh** to enable advanced traffic management strategies without modifying application code. Two experimental strategies are supported: **canary deployment** and **shadow launch (traffic mirroring)**. These strategies are mutually exclusive and can be enabled via configuration.

### Traffic Management Overview

- All external traffic enters the cluster through the **Istio IngressGateway**
- Routing decisions are defined using **Istio VirtualServices**
- Service versions are identified using **DestinationRule subsets** (`v1`, `v2`)
- All routing decisions are enforced at runtime by **Envoy sidecars**

---

## Canary Deployment

### Purpose

The canary deployment strategy is used to gradually expose a new service version to users by splitting incoming traffic between a stable and a canary version.

### Behavior

- User traffic is split between `v1` (stable) and `v2` (canary)
- Both versions serve user-facing responses
- Traffic split is percentage-based (e.g. 90% / 10%)

### Implementation

- **Configured in:** Istio `VirtualService`
- **Version definition:** Istio `DestinationRule`
- **Routing decision taken by:** Envoy proxy at the IngressGateway

This strategy introduces limited user exposure to the new version and is suitable once basic correctness has been established.

### Sticky Sessions and Manual Overrides

To manage user experience across versions, two distinct cookie-based mechanisms are employed:

#### 1. Automatic Sticky Sessions (Consistency)
To ensure that a user remains on the same version during their session, Istio is configured to use **consistent hashing** based on a specific cookie.

- **Cookie Name:** `user-session` (configured in `values.yaml`)
- **Mechanism:** The **DestinationRule** defines a consistent hash load balancer.
- **Behavior:** Istio (Envoy) generates this cookie for clients that don't have it and routes all subsequent requests with the same cookie value to the same backend instance. This prevents users from "flip-flopping" between `v1` and `v2`.

#### 2. Manual Version Pinning (Testing)
For testing purposes, developers can force routing to a specific version, bypassing the random split and sticky sessions.

- **Cookie Name:** `canary`
- **Mechanism:** The **VirtualService** uses regex matching on the cookie header.
- **Behavior:**
    - `canary=v1` -> Forces routing to **v1** (stable).
    - `canary=v2` -> Forces routing to **v2** (canary).
- **Usage:** Manually set via browser DevTools or `curl` (e.g., `curl --cookie "canary=v2" ...`) to test a specific version.

---

## Shadow Launch (Traffic Mirroring)

### Purpose

The shadow launch strategy is used to evaluate a new version of the **model service** under real production traffic **without exposing it to users**.

### Behavior

- 100% of user traffic is routed to the stable version (`v1`)
- Requests are **duplicated** and mirrored to the shadow version (`v3`)
- Responses from the shadow version are discarded
- Failures or increased latency in the shadow version do not impact users

### Implementation

- **Configured in:** Istio `VirtualService` for the `model-service`
- **Mirroring mechanism:** Istio traffic mirroring (`mirror` and `mirrorPercentage`)
- **Version definition:** Shared Istio `DestinationRule`
- **Routing decision taken by:** Envoy proxy enforcing VirtualService rules

The shadow launch allows safe validation of new model versions before promoting them to a canary or full rollout.

---

## Comparison of Deployment Strategies

| Strategy | User-visible responses | Traffic duplication | Risk to users |
|--------|-----------------------|--------------------|---------------|
| Canary | Yes (partial) | No | Medium |
| Shadow launch | No | Yes | None |
| Stable only | Yes | No | None |

---

## Configuration and Control

The deployment strategy is selected via Helm configuration values:

- Canary deployment is enabled using `istio.canary.enabled`
- Shadow launch is enabled using `istio.shadow.enabled`
- Only one strategy can be active at a time

This design ensures a clean separation of concerns and allows switching deployment strategies without modifying application code.

---


## Observability and Monitoring

### Prometheus

- Scrapes metrics from:
    - `app` pods
    - `model-service` pods
    - Istio Envoy sidecars
- Enables comparison of:
    - Request rate
    - Latency
    - Error rate per version

### Grafana

- Visualizes Prometheus metrics
- Used to evaluate the impact of canary deployments and compare versions

---

## Role of Istio in the Deployment

Istio is responsible for:

- Managing ingress traffic
- Enforcing service-to-service routing
- Enabling canary deployments without code changes
- Collecting telemetry via Envoy sidecars
- Supporting experimental traffic management strategies

The canary deployment experiment is implemented at the **service mesh layer**, demonstrating Istio’s ability to support advanced deployment patterns beyond simple request routing.

---

## Summary

This deployment combines Kubernetes and Istio to create a modular, observable, and experiment-friendly architecture. Kubernetes handles workload lifecycle and storage, while Istio enables dynamic traffic management and safe experimentation. The result is a flexible setup that allows iterative improvements without disrupting user-facing behavior.
