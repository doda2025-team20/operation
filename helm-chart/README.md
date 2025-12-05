Initial implementation of helm chart.

Deploys app & model, and starts services to expose them.

Template files were generated using
`kompose convert` and then modified to suit the application.

Quick start:
1. Check for erros in the chart
```bash
   helm lint helm-chart/
```

2. Install the chart
```bash
# Basic installation
helm install assignment-release ./helm-chart

# Installation with custom admin password (RECOMMENDED)
helm install assignment-release ./helm-chart \
  --set grafana.adminPassword=your-secure-password
```

```bash
   helm install assignment-release helm-chart/
```

2.5. Update the chart after changes
```bash
   helm upgrade assignment-release helm-chart/
```

3. Check that pods are running
```bash
   kubectl get pods
```

3.5. Check that services are running
```bash
   kubectl get services
```
4. Access the application using minikube
```
    minikube service app-service --url  # Copy the URL and paste it into your browser
```

## Configuration

The following table lists the configurable parameters of the chart and their default values.

### App Service

| Parameter | Description | Default |
|-----------|-------------|---------|
| `app.name` | Name of the app service deployment and service | `app-service` |
| `app.image` | Container image for the app service | `ghcr.io/doda2025-team20/app:latest` |
| `app.port` | Port the app service listens on | `8080` |

### Model Service

| Parameter | Description | Default |
|-----------|-------------|---------|
| `model.name` | Name of the model service deployment and service | `model-service` |
| `model.image` | Container image for the model service | `ghcr.io/doda2025-team20/model-service:latest` |
| `model.port` | Port the model service listens on | `8081` |
| `model.useHostPath` | Use hostPath volume instead of PVC. Set to `true` to use a VM shared folder. | `false` |
| `model.hostPath` | Path on the host for hostPath volume (only used when `useHostPath=true`) | `/mnt/shared` |
| `model.storage` | Storage size for the PVC (only used when `useHostPath=false`) | `100Mi` |
| `model.mountPath` | Mount path inside the container for the output volume | `/sms/output` |

### Overriding Values

You can override values at install/upgrade time:

```bash
helm install assignment-release helm-chart/ --set model.useHostPath=true
```

Or use a custom values file:

```bash
helm install assignment-release helm-chart/ -f my-values.yaml
```


# Grafana Monitoring Setup

## Overview

Grafana is deployed as part of the Helm chart and automatically provisions two dashboards:
1. **App Metrics Dashboard** - Monitors application performance and user behavior
2. **Experimental A/B Testing Dashboard** - Supports canary deployment decisions


## Installation

### Prerequisites

Before deploying Grafana, ensure you have:
- Kubernetes cluster running
- Prometheus deployed and collecting metrics from the app
- ServiceMonitor configured for Prometheus scraping

### Verify Installation

```bash
# Check if Grafana pod is running
kubectl get pods | grep grafana
# Should show: STATUS = Running

# Check if dashboards ConfigMap was created
kubectl get configmap | grep grafana-dashboards

# Check if datasource ConfigMap was created
kubectl get configmap | grep grafana-datasources
```

## Accessing Grafana

Access the application using minikube
```
   minikube service assignment-release-helm-chart-grafana --url  # Copy the URL and paste it into your browser
```

### Login Credentials

**Username:** `admin`  
**Password:** The value you set with `--set grafana.adminPassword=...` during installation

**To retrieve your password:**
```bash
# From Kubernetes Secret
kubectl get secret grafana-admin-secret -o jsonpath='{.data.admin-password}' | base64 -d

# From Helm values
helm get values assignment-release | grep adminPassword
```

**Default password (if not overridden):** `password123`

**To change the password:**
```bash
helm upgrade assignment-release ./helm-chart \
  --set grafana.adminPassword=password123
```

## Dashboards

### 1. App Metrics Dashboard

TO BE TESTED

### 2. Experimental A/B Testing Dashboard

TO BE TESTED


## Metrics Requirements

For the dashboards to work, the application must expose these metrics at the `/metrics` endpoint: