# Helm Basics - Complete Guide

## 📦 **Installing Helm**

### Linux
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### macOS
```bash
brew install helm
```

### Windows
```powershell
choco install kubernetes-helm
```

### Verify Installation
```bash
helm version
```

---

## 🚀 **Getting Started**

### 1. Add Helm Repositories

```bash
# Add Bitnami (popular charts)
helm repo add bitnami https://charts.bitnami.com/bitnami

# Add Stable charts
helm repo add stable https://charts.helm.sh/stable

# Add Prometheus community
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Add Ingress Nginx
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# Update repos
helm repo update

# List repos
helm repo list
```

### 2. Search for Charts

```bash
# Search in all repos
helm search repo nginx

# Search specific repo
helm search repo bitnami/nginx

# Search Helm Hub (https://artifacthub.io)
helm search hub wordpress

# Show chart info
helm show chart bitnami/nginx
helm show values bitnami/nginx
helm show readme bitnami/nginx
```

---

## 📥 **Installing Charts**

### Basic Install

```bash
# Install chart
helm install my-nginx bitnami/nginx

# Install with custom name
helm install my-release bitnami/nginx

# Install in specific namespace
helm install my-nginx bitnami/nginx --namespace production --create-namespace

# Install with custom values
helm install my-nginx bitnami/nginx --set replicaCount=3

# Install with values file
helm install my-nginx bitnami/nginx -f custom-values.yaml

# Dry run (test without installing)
helm install my-nginx bitnami/nginx --dry-run --debug

# Wait for resources to be ready
helm install my-nginx bitnami/nginx --wait --timeout 5m
```

### Example: Install PostgreSQL

```bash
# Install PostgreSQL
helm install my-postgres bitnami/postgresql \
  --set auth.username=myuser \
  --set auth.password=mypassword \
  --set auth.database=mydb \
  --namespace database \
  --create-namespace

# Get connection info
export POSTGRES_PASSWORD=$(kubectl get secret --namespace database my-postgres-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
echo $POSTGRES_PASSWORD

# Connect to PostgreSQL
kubectl run my-postgres-postgresql-client --rm --tty -i --restart='Never' --namespace database \
  --image docker.io/bitnami/postgresql:15 \
  --env="PGPASSWORD=$POSTGRES_PASSWORD" \
  --command -- psql --host my-postgres-postgresql -U postgres -d mydb -p 5432
```

---

## 📋 **Managing Releases**

### List Releases

```bash
# List all releases
helm list

# List in all namespaces
helm list --all-namespaces

# List in specific namespace
helm list -n production

# Show all releases (including failed/deleted)
helm list --all
```

### Release Status

```bash
# Get release status
helm status my-nginx

# Get release status in specific namespace
helm status my-nginx -n production

# Get release values
helm get values my-nginx

# Get release manifest
helm get manifest my-nginx

# Get all release info
helm get all my-nginx
```

### Release History

```bash
# Show release history
helm history my-nginx

# Detailed history
helm history my-nginx --max 10
```

---

## 🔄 **Upgrading Releases**

```bash
# Upgrade with new values
helm upgrade my-nginx bitnami/nginx --set replicaCount=5

# Upgrade with values file
helm upgrade my-nginx bitnami/nginx -f new-values.yaml

# Upgrade or install (if not exists)
helm upgrade --install my-nginx bitnami/nginx

# Upgrade with dry run
helm upgrade my-nginx bitnami/nginx --dry-run --debug

# Force upgrade (recreate resources)
helm upgrade my-nginx bitnami/nginx --force

# Reuse previous values
helm upgrade my-nginx bitnami/nginx --reuse-values

# Reset values to defaults
helm upgrade my-nginx bitnami/nginx --reset-values
```

---

## ⏮️ **Rollback Releases**

```bash
# View history
helm history my-nginx

# Rollback to previous version
helm rollback my-nginx

# Rollback to specific revision
helm rollback my-nginx 2

# Rollback with dry run
helm rollback my-nginx 2 --dry-run

# Rollback and wait
helm rollback my-nginx --wait
```

---

## 🗑️ **Uninstalling Releases**

```bash
# Uninstall release
helm uninstall my-nginx

# Uninstall from specific namespace
helm uninstall my-nginx -n production

# Keep history (can rollback later)
helm uninstall my-nginx --keep-history

# Dry run
helm uninstall my-nginx --dry-run
```

---

## 📝 **Creating Custom Charts**

### Create New Chart

```bash
# Create chart scaffold
helm create myapp

# Chart structure
myapp/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default values
├── charts/             # Dependencies
├── templates/          # Templates
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl
│   └── NOTES.txt
└── .helmignore

# Lint chart
helm lint ./myapp

# Package chart
helm package ./myapp

# Install local chart
helm install my-release ./myapp
```

### Example: Custom Chart Structure

```yaml
# Chart.yaml
apiVersion: v2
name: myapp
description: My awesome application
version: 1.0.0
appVersion: "2.0.0"
keywords:
  - web
  - api
maintainers:
  - name: Your Name
    email: you@example.com
```

```yaml
# values.yaml
replicaCount: 3

image:
  repository: nginx
  tag: "1.24"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  hostname: myapp.local

resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

---

## 🎨 **Template Examples**

### Deployment Template

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - containerPort: {{ .Values.service.port }}
        {{- if .Values.resources }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
        {{- end }}
```

### Test Template Rendering

```bash
# Render templates locally
helm template my-release ./myapp

# Render with custom values
helm template my-release ./myapp -f custom-values.yaml

# Render specific template
helm template my-release ./myapp --show-only templates/deployment.yaml
```

---

## 🔧 **Advanced Helm Commands**

### Plugin Management

```bash
# List plugins
helm plugin list

# Install plugin
helm plugin install https://github.com/databus23/helm-diff

# Use plugin (helm-diff example)
helm diff upgrade my-release ./myapp -f new-values.yaml
```

### Environment

```bash
# Show Helm environment
helm env

# Get specific env var
helm env | grep HELM_NAMESPACE
```

### Dependencies

```bash
# Update chart dependencies
helm dependency update ./myapp

# List dependencies
helm dependency list ./myapp

# Build dependencies
helm dependency build ./myapp
```

---

## 🧪 **Testing & Validation**

```bash
# Lint chart
helm lint ./myapp

# Test with dry-run
helm install my-release ./myapp --dry-run --debug

# Validate against Kubernetes
helm install my-release ./myapp --dry-run --debug --validate

# Template and pipe to kubectl
helm template my-release ./myapp | kubectl apply --dry-run=client -f -
```

---

## 📚 **Common Helm Values Patterns**

### Custom Values File

```yaml
# production-values.yaml
replicaCount: 5

image:
  tag: "v2.0.0"
  pullPolicy: Always

service:
  type: LoadBalancer

ingress:
  enabled: true
  hostname: myapp.example.com
  tls:
    enabled: true
    secretName: myapp-tls

resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi

env:
  - name: NODE_ENV
    value: production
  - name: LOG_LEVEL
    value: info

secrets:
  DATABASE_URL: "postgresql://user:pass@db:5432/mydb"
```

### Use Custom Values

```bash
helm install my-release ./myapp -f production-values.yaml
```

---

## 🎯 **Best Practices**

1. ✅ Always use `--dry-run` before actual deployment
2. ✅ Lint charts before packaging
3. ✅ Use semantic versioning
4. ✅ Document in README.md and NOTES.txt
5. ✅ Use values.yaml for configuration
6. ✅ Don't hard-code values in templates
7. ✅ Test in dev/staging first
8. ✅ Use namespaces
9. ✅ Keep charts simple and focused
10. ✅ Version control your charts

---

## 🔍 **Troubleshooting**

### Debug Failed Installation

```bash
# Show detailed error
helm install my-release ./myapp --debug

# Check release status
helm status my-release

# Get release manifest
helm get manifest my-release

# Check Kubernetes resources
kubectl get all -l app.kubernetes.io/instance=my-release

# View logs
kubectl logs -l app.kubernetes.io/instance=my-release
```

### Fix Failed Release

```bash
# Delete and reinstall
helm uninstall my-release
helm install my-release ./myapp

# Or upgrade
helm upgrade my-release ./myapp --force
```

---

## 📖 **Additional Resources**

- [Helm Documentation](https://helm.sh/docs/)
- [Artifact Hub](https://artifacthub.io/) - Find charts
- [Helm Best Practices](https://helm.sh/docs/chart_best_practices/)
- [Chart Template Guide](https://helm.sh/docs/chart_template_guide/)

