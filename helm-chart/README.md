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
