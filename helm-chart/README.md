# Helm Chart – Application, Istio, and Monitoring

This Helm chart deploys the application and model services together with **Istio traffic management** and a full **monitoring stack** based on **Prometheus and Grafana**.

The application is instrumented with Prometheus metrics, which are scraped by a Prometheus instance provided via the **`kube-prometheus-stack` subchart** and visualized using **pre-provisioned Grafana dashboards**.

---

## Prerequisites

### Kubernetes Cluster

This setup is intended to run on **Minikube**.

```bash
minikube start
kubectl get nodes
```

### Helm

Ensure Helm is installed and initialized:

```bash
helm version
helm repo update
```

---

## Installation

### Update dependencies

This chart includes `kube-prometheus-stack` as a **required dependency**.

```bash
helm dependency update ./helm-chart
```

### Lint the chart

```bash
helm lint ./helm-chart
```

### Install the chart

```bash
helm install assignment-release ./helm-chart
```

### Upgrade after changes

```bash
helm upgrade assignment-release ./helm-chart
```

### Verify deployment

```bash
kubectl get pods
kubectl get services
```

---

## Istio Setup

### Enable sidecar injection

```bash
kubectl label namespace default istio-injection=enabled
```

Verify Istio is running:

```bash
kubectl get pods -n istio-system
```

---

## Accessing the Application

### Non-WSL users (Linux / macOS)

For non-WSL environments, Minikube exposes services directly via localhost.

The application should be reachable through Istio without additional steps once the ingress is created.

Example:

```bash
http://localhost/
```

### WSL users (Windows Subsystem for Linux)

WSL does **not** automatically expose Minikube services to the Windows host.
You must explicitly expose the Istio ingress gateway port.

Run the following and keep it open:

```bash
minikube service istio-ingressgateway -n istio-system --url
```

This will output a URL such as:

```text
http://127.0.0.1:<INGRESS_PORT>
```

Use that port to access the application.

---

## Monitoring Overview

* The application exposes Prometheus metrics at **`/metrics`**
* Metrics are scraped using **ServiceMonitor** resources
* Prometheus is provided by **`kube-prometheus-stack` (subchart)**
* Grafana dashboards are **auto-provisioned via ConfigMaps**

### Prometheus

Prometheus is **not exposed externally**.

Internal service name:

```
monitoring-kube-prometheus-prometheus
```

To access Prometheus manually:

```bash
kubectl port-forward monitoring-kube-prometheus-prometheus 9090:9090
```

Then open:

```text
http://localhost:9090
```

---

## Exposed Metrics

The application exports the following metrics:

* `sms_requests_total` (Counter)
* `sms_last_confidence` (Gauge)
* `sms_request_duration_seconds` (Histogram)
* `sms_ui_clicks_total` (UI interaction Counter)

Manual verification (via Istio ingress):

```bash
curl http://127.0.0.1:<INGRESS_PORT>/metrics | grep sms_
```

---

## Grafana

### Access Grafana

Grafana **is exposed** and should be reachable via:

```text
http://localhost/grafana
```

If Grafana is not reachable, use port-forwarding:

```bash
kubectl port-forward svc/assignment-release-helm-chart-grafana 3000:3000
```

Then open:

```text
http://localhost:3000
```

### Login credentials

* **Username:** `admin`
* **Password:** `password123`

---

## Dashboards

Two dashboards are pre-provisioned:

### SMS App Metrics Dashboard

Visualizes:

* Spam / ham confidence (gauges)
* Request rate
* Request latency (avg, p50, p95)
* Spam vs ham classification totals
* UI submit click count

### SMS Experimental A/B Testing Dashboard

Used with Istio canary routing:

* Traffic split between v1 and v2
* Latency comparison
* Total requests for v1 and v2
* Request rate per version
* Traffic distribution
* Mode preference (bulk or single message)

---

## Testing Workflow

### (WSL only) Get ingress port

```bash
minikube service istio-ingressgateway -n istio-system --url
```

Set the port:

```bash
INGRESS_PORT=<PORT>
```

### Generate traffic

```bash
for i in {1..50}; do
  curl -X POST http://127.0.0.1:$INGRESS_PORT/sms/ \
    -H "Content-Type: application/json" \
    -d "{\"sms\":\"test message $i\"}" \
    -s > /dev/null
done
```

Metrics should now appear in Grafana.

---

## Troubleshooting

### Dashboards show no data

* Ensure traffic was generated
* Verify the correct ingress port (WSL)
* Check Prometheus targets:

```bash
kubectl port-forward monitoring-kube-prometheus-prometheus 9090:9090
```

Query:

```text
sms_requests_total
```
