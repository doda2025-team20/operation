# Helm Chart – Application, Istio, and Monitoring

This Helm chart deploys the application and model services together with Istio traffic management and a full monitoring stack based on Prometheus and Grafana.  
The initial Helm templates were generated using `kompose convert` and then manually adapted and extended to meet the assignment requirements.

---

## Installation

### Lint the chart

```bash
helm lint helm-chart/
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

## Prerequisites

### Minikube

```bash
minikube start
kubectl get nodes
```

### Prometheus Operator

This chart relies on `ServiceMonitor` resources, so the Prometheus Operator must be installed.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack
```

Verify CRDs:

```bash
kubectl get crds | grep monitoring.coreos.com
```

---

## Istio Setup

> From here onward, the process has only been tested on WSL, so it might be more complicated compared to Linux or macOS systems.

### Enable sidecar injection

```bash
kubectl label namespace default istio-injection=enabled
```

Verify Istio:

```bash
kubectl get pods -n istio-system
```

### Access the application through Istio

```bash
minikube service istio-ingressgateway -n istio-system --url
```

If multiple URLs are returned, use the one that responds correctly (typically the second).

Example:

```bash
curl http://127.0.0.1:<INGRESS_PORT>/
```

---

## Monitoring Overview

The application exposes Prometheus metrics at `/metrics`.
Prometheus scrapes these metrics using `ServiceMonitor` resources, and Grafana visualizes them using provisioned dashboards and datasources.

Prometheus instance:

```
monitoring-kube-prometheus-prometheus
```

---

## Prometheus Metrics

Example exposed metrics:

* `sms_requests_total`
* `sms_last_confidence`
* `sms_request_duration_seconds`

Manual verification:

```bash
curl http://127.0.0.1:<INGRESS_PORT>/metrics | grep sms_
```

---

## Grafana

### Access Grafana

```bash
minikube service assignment-release-helm-chart-grafana --url
```

Login:

* Username: `admin`
* Password: value configured via Helm values (should be `password123`)

### Provisioning

Grafana is provisioned using ConfigMaps:

* Prometheus datasource
* Dashboard provisioning config
* Dashboard JSON definitions

The Prometheus datasource URL:

```
http://monitoring-kube-prometheus-prometheus:9090
```

---

## Dashboards

### SMS App Metrics Dashboard

Visualizes:

* Spam and ham confidence (gauges)
* Request rate using `rate(sms_requests_total)`
* Request latency (average, p50, p95)
* Total spam vs ham classifications

### SMS Experimental A/B Testing Dashboard

Used when Istio canary routing is enabled.
Visualizes:

* Traffic split between v1 and v2
* Latency comparison between versions
* Request volume per version

---

## Testing Workflow

### Get ingress port

```bash
minikube service istio-ingressgateway -n istio-system --url
```

> This might not be needed on Linux or macOs

Keep it open in a terminal and in another terminal:

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

### Verify in Grafana

First port-forward:

```bash
kubectl port-forward svc/assignment-release-helm-chart-grafana 3000:3000
```

Then:

1. Open Grafana
2. Go to dashboard
3. Select under `general`
   - `SMS App Matrics Dashboard` to view the last spam/ham confidence, the sms request rate, the request duration diagram, total sms classification and the spam-ham distribution
   - `SMS Experimantal A/B Testing Dashboard` to view the request rate between stable and canary, the latency comparison, traffic distribution and v1/v2 total requests. 

or

1. Open Grafana
2. Go to **Explore**
3. Select the `prometheus` datasource
4. Query:

```
sms_requests_total
```

Then open the dashboards and set the time range to **Last 5–15 minutes**.

You can also generate more traffic to view the values update live


---

## Troubleshooting

### Dashboards show no data

* Ensure traffic was generated
* Verify correct ingress port
* Check Prometheus targets

```bash
kubectl port-forward svc/monitoring-kube-prometheus-prometheus 9090:9090
```

Query:

```
sms_requests_total
```

### Datasource not found

* Ensure the Prometheus datasource ConfigMap is mounted correctly
* Restart Grafana if needed
