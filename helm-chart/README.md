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


## Prerequisites

### Minikube must be running
The Kubernetes API must be reachable before installing the Helm chart. If Minikube is not started, Helm will fail with errors such as:

```bash
kubernetes cluster unreachable: dial tcp 127.0.0.1:xxxxx: connect: connection refused
```



Start Minikube:

```bash
minikube start
````

Verify the cluster is reachable:

```bash
kubectl get nodes
```

### Prometheus Operator CRDs must be installed

This chart includes resources such as `AlertmanagerConfig` and `PrometheusRule`, which belong to the Prometheus Operator API group (`monitoring.coreos.com`). These will fail unless the Prometheus Operator (commonly installed via `kube-prometheus-stack`) is present.

Install the CRDs by installing the stack:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack
```

Verify that the CRDs exist:

```bash
kubectl get crds | grep monitoring.coreos.com
```

If these are missing, Helm will report errors such as:

```
no matches for kind "AlertmanagerConfig"
ensure CRDs are installed first
```

## Installing the Helm Chart

After ensuring that Minikube is running and the Prometheus Operator CRDs are installed, install the chart:

```bash
helm install assignment-release ./helm-chart
```

# Istio Deployment

This section describes how to deploy the application with Istio for traffic management, canary releases, and monitoring.

## Prerequisites

- Istio must be installed in your cluster. You can use Istio’s official Helm chart or `istioctl install`.
- Ensure the `default` namespace has sidecar injection enabled:

```bash
kubectl label namespace default istio-injection=enabled
```

* Verify Istio system pods are running:

```bash
kubectl get pods -n istio-system
```

## Deploying with Istio

After installing the Helm chart for the app:

1. **Verify the Istio ingress gateway:**

```bash
kubectl get svc -n istio-system istio-ingressgateway
```

* This service exposes the application externally via ports 80/443.
* minikube must be running:

```bash
minikube tunnel
```

3. **Accessing the app via Istio:**

```bash
curl -v http://<ISTIO_INGRESS_IP>/ # Should be 127.0.0.1
```

* Replace `<ISTIO_INGRESS_IP>` with IP of `istio-ingressgateway` under `EXTERNAL-IP`.

4. **Testing v1/v2 routing:**

* Confirm both versions of `app-service` are registered with Istio:

```bash
istioctl proxy-config endpoints <INGRESS_GATEWAY_POD> -n istio-system | grep app-service
```

* To get the INGRESS_GATEWAY_POD run:

```bash
kubectl get pods -n istio-system -l istio=ingressgateway
```

* Sample output should include endpoints for `v1` and `v2`.

* To verify sticky sessions:

```bash
cookie=$(curl -v -s http://<ISTIO_INGRESS_IP>/ 2>&1 | grep "set-cookie" | cut -d: -f2- | xargs)
for i in {1..10}; do curl -s --header "Cookie: $cookie" http://<ISTIO_INGRESS_IP>/; echo; done
```

Requests with the same cookie should consistently hit the same version of the service.

To view to which version of the service the request was routed to, open and inspect the logs:

```bash
kubectl logs -l app=app-service -c istio-proxy -f
```

## Troubleshooting

* **`kubectl run` fails or pod deleted immediately:**
  Use a supported image like `curlimages/curl` and ensure the namespace is not restricted.

* **Ingress gateway reports 503:**

  * Check that app pods are healthy and sidecars are injected.
  * Verify that the virtual service and destination rules are applied.

```bash
kubectl get virtualservice
kubectl get destinationrule
```

* **Istioctl commands fail to find pods:**
  Ensure you are using the correct namespace and pod name:

```bash
kubectl get pod -n istio-system -l istio=ingressgateway
```

## Notes

* Always keep the `minikube tunnel` terminal open for LoadBalancer access.
* For canary deployments, you can adjust traffic split by updating the `DestinationRule` and `VirtualService` objects for `app-service`.



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