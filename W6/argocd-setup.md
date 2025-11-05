# ArgoCD Setup and Usage Guide

## 📦 **Installing ArgoCD**

### 1. Install ArgoCD

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Verify installation
kubectl get pods -n argocd
kubectl get svc -n argocd
```

### 2. Access ArgoCD UI

```bash
# Method 1: Port Forward
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Access: https://localhost:8080

# Method 2: LoadBalancer (production)
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
kubectl get svc argocd-server -n argocd

# Method 3: Ingress
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server
  namespace: argocd
  annotations:
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  rules:
  - host: argocd.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              number: 443
  tls:
  - hosts:
    - argocd.example.com
    secretName: argocd-tls
EOF
```

### 3. Get Admin Password

```bash
# Get initial password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Login with username: admin
# Password: <from above command>

# Change password (via CLI)
argocd login localhost:8080
argocd account update-password
```

---

## 🔧 **ArgoCD CLI Installation**

```bash
# Linux
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64

# macOS
brew install argocd

# Verify
argocd version
```

---

## 📱 **Creating Applications**

### Method 1: Using UI
1. Login to ArgoCD UI
2. Click "New App"
3. Fill in details:
   - Application Name: myapp
   - Project: default
   - Sync Policy: Automatic
   - Repository URL: https://github.com/yourusername/your-repo
   - Path: k8s/
   - Cluster: https://kubernetes.default.svc
   - Namespace: production
4. Click "Create"

### Method 2: Using CLI

```bash
# Login
argocd login localhost:8080 --username admin --password <password>

# Create app
argocd app create myapp \
  --repo https://github.com/yourusername/your-repo \
  --path k8s \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace production

# Sync app
argocd app sync myapp

# Get app status
argocd app get myapp
```

### Method 3: Using YAML (Declarative)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  # Project
  project: default
  
  # Source - Git repository
  source:
    repoURL: https://github.com/yourusername/your-repo
    targetRevision: HEAD  # or specific branch/tag
    path: k8s
    
    # Optional: Helm values
    helm:
      valueFiles:
      - values-production.yaml
      parameters:
      - name: image.tag
        value: v2.0.0
  
  # Destination - Kubernetes cluster
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  
  # Sync policy
  syncPolicy:
    automated:
      prune: true      # Delete resources not in Git
      selfHeal: true   # Auto-sync if drift detected
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
  
  # Ignore differences (optional)
  ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas  # Ignore HPA-managed replicas
```

Apply:
```bash
kubectl apply -f application.yaml
```

---

## 🔄 **Managing Applications**

### Sync Applications

```bash
# Manual sync
argocd app sync myapp

# Sync specific resource
argocd app sync myapp --resource deployment:myapp

# Force sync (ignore errors)
argocd app sync myapp --force

# Dry run
argocd app sync myapp --dry-run
```

### Application Status

```bash
# Get app details
argocd app get myapp

# List all apps
argocd app list

# Get app history
argocd app history myapp

# Get app logs
argocd app logs myapp

# Get app resources
argocd app resources myapp
```

### Rollback

```bash
# View history
argocd app history myapp

# Rollback to specific revision
argocd app rollback myapp 5

# Rollback to previous
argocd app rollback myapp
```

### Delete Application

```bash
# Delete app (keeps resources)
argocd app delete myapp

# Delete app and resources
argocd app delete myapp --cascade
```

---

## 🎯 **App of Apps Pattern**

```yaml
# apps/root-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/yourusername/your-repo
    targetRevision: HEAD
    path: apps  # Directory containing other app definitions
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Directory structure:
```
apps/
├── root-app.yaml
├── backend-app.yaml
├── frontend-app.yaml
└── database-app.yaml
```

---

## 🔐 **Private Git Repositories**

### HTTPS with Token

```bash
# Add repository
argocd repo add https://github.com/yourusername/private-repo \
  --username yourusername \
  --password ghp_YourGitHubPersonalAccessToken
```

### SSH Key

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "argocd@example.com" -f ~/.ssh/argocd

# Add public key to GitHub/GitLab
cat ~/.ssh/argocd.pub

# Add repository
argocd repo add git@github.com:yourusername/private-repo.git \
  --ssh-private-key-path ~/.ssh/argocd
```

---

## 🏗️ **Projects**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
  namespace: argocd
spec:
  description: Production applications
  
  # Allowed source repositories
  sourceRepos:
  - https://github.com/yourusername/*
  
  # Allowed destination clusters and namespaces
  destinations:
  - namespace: production
    server: https://kubernetes.default.svc
  - namespace: monitoring
    server: https://kubernetes.default.svc
  
  # Allowed resource types
  clusterResourceWhitelist:
  - group: '*'
    kind: '*'
  
  # Denied resources
  namespaceResourceBlacklist:
  - group: ''
    kind: ResourceQuota
  - group: ''
    kind: LimitRange
```

---

## 🔔 **Notifications**

```bash
# Install ArgoCD Notifications
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-notifications/stable/manifests/install.yaml

# Configure Slack notifications
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  service.slack: |
    token: $slack-token
  
  trigger.on-deployed: |
    - when: app.status.operationState.phase in ['Succeeded']
      send: [app-deployed]
  
  template.app-deployed: |
    message: |
      Application {{.app.metadata.name}} deployed successfully!
    slack:
      attachments: |
        [{
          "title": "{{.app.metadata.name}}",
          "color": "good"
        }]
EOF

# Subscribe app to notifications
argocd app set myapp --annotation notifications.argoproj.io/subscribe.on-deployed.slack=my-channel
```

---

## 📊 **Monitoring ArgoCD**

```bash
# Check ArgoCD components
kubectl get pods -n argocd

# ArgoCD server logs
kubectl logs -n argocd deployment/argocd-server

# Application controller logs
kubectl logs -n argocd deployment/argocd-application-controller

# Repo server logs
kubectl logs -n argocd deployment/argocd-repo-server

# Metrics
kubectl port-forward -n argocd svc/argocd-metrics 8082:8082
# Access: http://localhost:8082/metrics
```

---

## 🎨 **Best Practices**

1. ✅ Use Git as single source of truth
2. ✅ Enable automated sync with self-heal
3. ✅ Use App of Apps pattern for multiple apps
4. ✅ Organize by environment (dev, staging, prod)
5. ✅ Use ArgoCD Projects for RBAC
6. ✅ Enable notifications for important events
7. ✅ Use Helm for complex applications
8. ✅ Keep manifests DRY with Kustomize or Helm
9. ✅ Monitor ArgoCD metrics
10. ✅ Regular backups of ArgoCD configuration

---

## 🐛 **Troubleshooting**

### App stuck in "Progressing"

```bash
# Check sync status
argocd app get myapp

# Force refresh
argocd app refresh myapp --hard

# Check logs
kubectl logs -n production <pod-name>
```

### Sync failed

```bash
# Get detailed error
argocd app get myapp

# Check application events
kubectl get events -n production

# Manual sync with options
argocd app sync myapp --force --prune
```

### Out of sync

```bash
# Show diff
argocd app diff myapp

# Sync
argocd app sync myapp
```

---

## 📚 **Additional Resources**

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [ArgoCD Best Practices](https://argo-cd.readthedocs.io/en/stable/user-guide/best_practices/)
- [GitOps Guide](https://www.gitops.tech/)

