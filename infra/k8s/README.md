# Installing required Kubernetes Tools

## Installing the Kubernetes Dashboard

Kubernetes Dashboard is a web-based user interface that allows you to manage and monitor your Kubernetes clusters. Follow the steps below to install and access the Kubernetes Dashboard.
### Prerequisites
- A running Kubernetes cluster (version 1.10 or higher).
- kubectl command-line tool configured to interact with your cluster.
- Helm

### Installation

```shell
# Add kubernetes-dashboard repository
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
# Deploy a Helm Release named "kubernetes-dashboard" using the kubernetes-dashboard chart
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
```

Wait until all pods are running ( takes 5-10 minutes )
```shell
watch kubectl get pods -n kubernetes-dashboard
```

### Accessing the Dashboard

1. Create a Service Account and ClusterRoleBinding for accessing the dashboard, using the kustomization in this folder:
```shell
kubectl apply -k kubernetes-dashboard
```
2. Get the access token for the Service Account:
```shell
kubectl create token cluster-admin -n kubernetes-dashboard
```

3. Expose the dashboard service using port-forwarding:
```shell
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443
```

4. Paste the token in the dashboard, exposed at https://localhost:8443

## Installing Istio

```shell
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
```

1. Install the base CRDs required for Istio operator:
```shell
helm install istio-base istio/base -n istio-system --set defaultRevision=default --create-namespace
```

2. Install the Istio discovery chart (Istiod):
```shell
helm install istiod istio/istiod -n istio-system --wait
```

3. Add the ingress gateway:
```shell
kubectl apply -k istio
```

4. Wait for the service and gateway to be running, then take the ip for the service and add it in the annotations of the gateway:
```shell
watch kubectl get svc -n istio-system
```

Make sure the gateway has the following annotations ( where {IP} is the ip of the istio-ingressgateway service ):
```yaml
  annotations:
    metallb.universe.tf/loadBalancerIPs: {IP}
    metallb.universe.tf/allow-shared-ip: secret-{IP}
    metallb.io/loadBalancerIPs: {IP}
    metallb.io/allow-shared-ip: secret-{IP}
```
