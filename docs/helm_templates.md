# Helm Templates Documentation

Reference documentation for the SMS-Checker Helm chart templates.

---

## Template Files

### Core Application

| File | Description |
|------|-------------|
| **app-deployment-stable.yaml** | Main app-service deployment (90% traffic). Runs the Spring Boot web API with Istio sidecar injection. |
| **app-deployment-canary.yaml** | Canary app-service deployment (10% traffic). Same app but can use different image for testing. |
| **app-service.yaml** | Kubernetes Service for app-service. Routes traffic to both stable and canary pods. |
| **app-ingress.yaml** | Nginx Ingress for app-service (used when Istio is disabled). |

### Model Service

| File | Description |
|------|-------------|
| **deployment-model-stable.yaml** | Main model-service deployment. Runs the Python/Flask ML model API. |
| **deployment-model-canary.yaml** | Canary model-service deployment. For testing new model versions with 10% traffic. |
| **deployment-model-shadow.yaml** | Shadow model-service deployment. Receives 100% mirrored traffic for evaluation (responses discarded). |
| **service-model.yaml** | Kubernetes Service for model-service. Routes to stable and canary pods. |
| **service-model-shadow.yaml** | Kubernetes Service for shadow model-service. Target for mirrored traffic. |

### Istio Configuration

| File | Description |
|------|-------------|
| **gateway.yaml** | Istio Gateway. Entry point for external traffic on port 80. |
| **virtualservice-app.yaml** | VirtualService for app-service. Implements 90/10 canary split and header-based routing (`x-version`). |
| **virtualservice-model.yaml** | VirtualService for model-service. Implements traffic mirroring to shadow (100% mirror). |
| **destinationrule.yaml** | DestinationRules for app and model services. Defines subsets: `stable`, `canary`, `shadow`. |

### Configuration

| File | Description |
|------|-------------|
| **configmap.yaml** | ConfigMap with environment variables: `MODEL_HOST`, `APP_PORT`, `MODEL_PORT`, `MODEL_URL`. |

### Monitoring

| File | Description |
|------|-------------|
| **monitoring.yaml** | ServiceMonitor resources for Prometheus. Scrapes metrics from app-service and model-service. |
| **prometheusrule.yaml** | PrometheusRules for alerting. Defines alert conditions. |
| **alertmanager-config.yaml** | AlertManager configuration. Defines notification routes and receivers. |
| **grafana-dashboards-configmap.yaml** | ConfigMap containing Grafana dashboard JSON files for visualization. |

