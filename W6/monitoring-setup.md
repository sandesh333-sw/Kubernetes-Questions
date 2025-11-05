# Prometheus & Grafana Monitoring Setup

## 📊 **Installing Prometheus Stack**

### Using Helm (Recommended)

```bash
# Add Prometheus Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus + Grafana + Alertmanager
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  --set prometheus.prometheusSpec.retention=30d \
  --set prometheus.prometheusSpec.storageSpec.volumeClaimTemplate.spec.resources.requests.storage=50Gi \
  --set grafana.persistence.enabled=true \
  --set grafana.persistence.size=10Gi

# Verify installation
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

---

## 🔍 **Accessing Prometheus & Grafana**

### Access Prometheus UI

```bash
# Port forward
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090

# Access: http://localhost:9090
```

### Access Grafana

```bash
# Port forward
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80

# Access: http://localhost:3000

# Get admin password
kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 -d

# Default username: admin
```

### Access via Ingress

```yaml
# prometheus-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prometheus-ingress
  namespace: monitoring
  annotations:
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: prometheus-basic-auth
spec:
  ingressClassName: nginx
  rules:
  - host: prometheus.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prometheus-kube-prometheus-prometheus
            port:
              number: 9090
  tls:
  - hosts:
    - prometheus.example.com
    secretName: prometheus-tls

---
# grafana-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  namespace: monitoring
spec:
  ingressClassName: nginx
  rules:
  - host: grafana.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prometheus-grafana
            port:
              number: 80
  tls:
  - hosts:
    - grafana.example.com
    secretName: grafana-tls
```

---

## 📈 **Creating ServiceMonitor**

ServiceMonitor tells Prometheus what to scrape:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: myapp-metrics
  namespace: production
  labels:
    app: myapp
spec:
  # Select services to monitor
  selector:
    matchLabels:
      app: myapp
  
  # Define endpoints to scrape
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
  
  # Optional: target labels
  targetLabels:
  - app
  - version
```

### Application with Metrics Endpoint

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: production
  labels:
    app: myapp
spec:
  selector:
    app: myapp
  ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: metrics  # Metrics endpoint
    port: 9090
    targetPort: 9090

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        ports:
        - containerPort: 8080
          name: http
        - containerPort: 9090  # Metrics port
          name: metrics
```

---

## 🎯 **PromQL Queries**

### Basic Queries

```promql
# CPU usage per pod
sum(rate(container_cpu_usage_seconds_total[5m])) by (pod)

# Memory usage per pod
sum(container_memory_usage_bytes) by (pod)

# Pod restarts
kube_pod_container_status_restarts_total

# Number of pods
count(kube_pod_info)

# HTTP request rate
rate(http_requests_total[5m])

# Error rate
rate(http_requests_total{status=~"5.."}[5m])

# 95th percentile latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

### Advanced Queries

```promql
# Top 10 pods by memory
topk(10, sum(container_memory_usage_bytes) by (pod))

# Pods using > 1GB memory
sum(container_memory_usage_bytes) by (pod) > 1073741824

# CPU throttling
rate(container_cpu_cfs_throttled_seconds_total[5m]) > 0

# Disk usage
(node_filesystem_size_bytes - node_filesystem_free_bytes) / node_filesystem_size_bytes * 100
```

---

## 🚨 **Alerting**

### PrometheusRule (Alert Definitions)

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: myapp-alerts
  namespace: monitoring
spec:
  groups:
  - name: myapp
    interval: 30s
    rules:
    
    # High CPU usage
    - alert: HighCPUUsage
      expr: |
        sum(rate(container_cpu_usage_seconds_total{namespace="production"}[5m])) by (pod) > 0.8
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High CPU usage on {{ $labels.pod }}"
        description: "Pod {{ $labels.pod }} CPU usage is above 80%"
    
    # High memory usage
    - alert: HighMemoryUsage
      expr: |
        sum(container_memory_usage_bytes{namespace="production"}) by (pod) / 
        sum(container_spec_memory_limit_bytes{namespace="production"}) by (pod) > 0.9
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High memory usage on {{ $labels.pod }}"
        description: "Pod {{ $labels.pod }} memory usage is above 90%"
    
    # Pod restarts
    - alert: PodRestarting
      expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Pod {{ $labels.pod }} restarting"
        description: "Pod {{ $labels.pod }} has restarted {{ $value }} times"
    
    # Service down
    - alert: ServiceDown
      expr: up{job="myapp"} == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "Service {{ $labels.job }} is down"
        description: "{{ $labels.job }} has been down for more than 1 minute"
    
    # High error rate
    - alert: HighErrorRate
      expr: |
        rate(http_requests_total{status=~"5.."}[5m]) / 
        rate(http_requests_total[5m]) > 0.05
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High error rate detected"
        description: "Error rate is {{ $value }}% (threshold: 5%)"
```

### Alertmanager Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: alertmanager-config
  namespace: monitoring
data:
  alertmanager.yml: |
    global:
      resolve_timeout: 5m
    
    route:
      group_by: ['alertname', 'cluster', 'service']
      group_wait: 10s
      group_interval: 10s
      repeat_interval: 12h
      receiver: 'slack'
      routes:
      - match:
          severity: critical
        receiver: 'pagerduty'
    
    receivers:
    - name: 'slack'
      slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'
        channel: '#alerts'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
    
    - name: 'pagerduty'
      pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_KEY'
```

---

## 📊 **Grafana Dashboards**

### Import Pre-built Dashboards

1. Login to Grafana
2. Go to Dashboards → Import
3. Enter dashboard ID:
   - **315**: Kubernetes cluster monitoring
   - **1860**: Node Exporter Full
   - **6417**: Kubernetes Deployment Statefulset Daemonset metrics
   - **7249**: Kubernetes Cluster
   - **3119**: Kubernetes Pods

### Create Custom Dashboard

```json
{
  "dashboard": {
    "title": "MyApp Metrics",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m])"
          }
        ],
        "type": "graph"
      }
    ]
  }
}
```

---

## 🔧 **Troubleshooting**

### Check if Prometheus is scraping targets

```bash
# Port forward Prometheus
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090

# Visit: http://localhost:9090/targets
```

### Check ServiceMonitor

```bash
# List ServiceMonitors
kubectl get servicemonitor -A

# Describe ServiceMonitor
kubectl describe servicemonitor myapp-metrics -n production

# Check if Prometheus operator is running
kubectl get pods -n monitoring | grep prometheus-operator
```

### View Prometheus logs

```bash
kubectl logs -n monitoring prometheus-prometheus-kube-prometheus-prometheus-0

# Follow logs
kubectl logs -n monitoring prometheus-prometheus-kube-prometheus-prometheus-0 -f
```

---

## 📚 **Additional Resources**

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [PromQL Basics](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Grafana Dashboards](https://grafana.com/grafana/dashboards/)

