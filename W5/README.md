# Week 5: Helm, CRDs, and Operators

## 🎯 Learning Objectives

By the end of this week, you will:
- Master Helm for package management
- Create and manage custom Helm charts
- Understand Custom Resource Definitions (CRDs)
- Deploy and use Kubernetes Operators
- Extend Kubernetes API with custom resources

## 📁 Files in this directory

### Theory
- `W5.md` - Complete Q&A guide with theory, best practices, and interview questions

### Hands-on Examples

#### Helm
- `helm-basics.md` - Complete Helm guide (install, upgrade, rollback)
- `custom-chart/` - Example custom Helm chart
- `webapp-chart/` - Production-ready web application chart

#### CRDs & Operators
- `crd-example.yaml` - Custom Resource Definition examples
- `operator-example.md` - Installing and using popular operators

## 🚀 Quick Start

### 1. Install Helm

```bash
# macOS
brew install helm

# Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify
helm version
```

### 2. Basic Helm Usage

```bash
# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search for charts
helm search repo nginx

# Install chart
helm install my-nginx bitnami/nginx

# List releases
helm list

# Upgrade
helm upgrade my-nginx bitnami/nginx --set replicaCount=3

# Rollback
helm rollback my-nginx

# Uninstall
helm uninstall my-nginx
```

### 3. Create Custom Helm Chart

```bash
# Create chart scaffold
helm create myapp

# Edit values.yaml
vim myapp/values.yaml

# Lint chart
helm lint ./myapp

# Test rendering
helm template my-release ./myapp

# Install
helm install my-release ./myapp

# Package
helm package ./myapp
```

### 4. Working with CRDs

```bash
# Apply CRD
kubectl apply -f crd-example.yaml

# Verify CRD
kubectl get crd
kubectl describe crd databases.example.com

# Create custom resource
kubectl apply -f - <<EOF
apiVersion: example.com/v1
kind: Database
metadata:
  name: my-db
spec:
  size: 10Gi
  version: "15"
  replicas: 3
EOF

# List custom resources
kubectl get databases
kubectl get db  # short name

# Describe
kubectl describe database my-db

# Delete
kubectl delete database my-db
```

### 5. Install Operators

```bash
# Prometheus Operator
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring --create-namespace

# Cert-Manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Verify
kubectl get pods -n monitoring
kubectl get pods -n cert-manager
kubectl get pods -n argocd
```

## 🔧 Useful Commands

### Helm Commands

```bash
# Repository management
helm repo add <name> <url>
helm repo list
helm repo update
helm repo remove <name>

# Chart management
helm create <chart-name>
helm package <chart>
helm lint <chart>
helm template <release> <chart>

# Release management
helm install <release> <chart>
helm upgrade <release> <chart>
helm rollback <release> [revision]
helm uninstall <release>
helm list
helm status <release>
helm history <release>

# Values
helm get values <release>
helm show values <chart>

# Debugging
helm install <release> <chart> --dry-run --debug
helm template <release> <chart> --debug
```

### CRD & Operator Commands

```bash
# CRDs
kubectl get crd
kubectl describe crd <crd-name>
kubectl explain <custom-resource>

# Custom Resources
kubectl get <resource-type>
kubectl describe <resource-type> <name>
kubectl delete <resource-type> <name>

# Operator logs
kubectl logs -n <namespace> <operator-pod>
kubectl logs -n <namespace> <operator-pod> -f
```

## 🧪 Practice Exercises

### Exercise 1: Helm Basics
1. Install Helm
2. Add bitnami repository
3. Install nginx with custom replicas
4. Upgrade to different version
5. Rollback to previous version

### Exercise 2: Create Custom Chart
1. Create new chart with `helm create`
2. Modify values.yaml
3. Customize templates
4. Test with `helm template`
5. Install and verify

### Exercise 3: Chart with Dependencies
1. Create chart with PostgreSQL dependency
2. Configure dependency in Chart.yaml
3. Update dependencies
4. Override dependency values
5. Deploy full stack

### Exercise 4: CRD Creation
1. Create custom CRD (e.g., WebApp)
2. Add validation schema
3. Define additional printer columns
4. Apply CRD
5. Create custom resources

### Exercise 5: Use Operators
1. Install Prometheus Operator
2. Create ServiceMonitor
3. Access Prometheus UI
4. Create custom alerts
5. View metrics in Grafana

### Exercise 6: Cert-Manager
1. Install cert-manager
2. Create ClusterIssuer (Let's Encrypt)
3. Create Certificate resource
4. Verify certificate creation
5. Use in Ingress

### Exercise 7: GitOps with ArgoCD
1. Install ArgoCD
2. Connect Git repository
3. Create Application
4. Enable auto-sync
5. Test GitOps workflow

## 📊 Comparison Tables

### Helm vs kubectl

| Aspect              | kubectl              | Helm                     |
|---------------------|----------------------|--------------------------|
| Deployment          | Individual manifests | Packaged charts          |
| Configuration       | Multiple files       | Single values.yaml       |
| Versioning          | Manual (Git)         | Built-in releases        |
| Rollback            | Manual               | One command              |
| Templates           | No                   | Yes (Go templates)       |
| Dependencies        | Manual               | Automatic                |
| Reusability         | Copy files           | Charts                   |

### CRD vs ConfigMap

| Aspect              | ConfigMap            | CRD                      |
|---------------------|----------------------|--------------------------|
| Purpose             | Configuration data   | Custom API resources     |
| Validation          | None                 | OpenAPI schema           |
| Controllers         | No                   | Yes (Operators)          |
| API Integration     | No                   | Full Kubernetes API      |
| Lifecycle           | Simple               | Complex (with Operator)  |

## ⚠️ Common Pitfalls

### Helm
1. **Forgetting `--dry-run`** - Always test before deploying
2. **Not version bumping** - Update Chart.yaml version on changes
3. **Hard-coded values** - Use values.yaml for all configuration
4. **Ignoring lint warnings** - Fix all lint issues
5. **No rollback plan** - Always know how to rollback

### CRDs & Operators
1. **No validation schema** - Always add OpenAPI validation
2. **Breaking CRD changes** - Version CRDs properly (v1alpha1 → v1beta1 → v1)
3. **Missing RBAC** - Operators need proper permissions
4. **No status subresource** - Add status for better UX
5. **Operator restarts loop** - Check reconciliation logic
6. **Resource leaks** - Ensure cleanup on resource deletion

## 🎓 Interview Questions to Practice

1. What is Helm and why use it?
2. Explain Helm chart structure
3. How do Helm templates work?
4. What's the difference between `helm upgrade` and `helm install`?
5. How do you rollback a failed Helm release?
6. What are CRDs?
7. Explain the Operator pattern
8. Difference between CRD and Operator?
9. When would you use an Operator vs Helm?
10. How do Operators reconcile state?
11. What is the Operator SDK?
12. Explain Helm hooks and their use cases
13. How do you manage secrets in Helm?
14. What are Helm dependencies?
15. How do you version Helm charts?

## 📚 Additional Resources

- [Helm Documentation](https://helm.sh/docs/)
- [Artifact Hub](https://artifacthub.io/) - Helm charts catalog
- [Helm Best Practices](https://helm.sh/docs/chart_best_practices/)
- [OperatorHub.io](https://operatorhub.io/) - Operator catalog
- [Operator SDK](https://sdk.operatorframework.io/)
- [Kubebuilder Book](https://book.kubebuilder.io/)
- [CNCF Operator White Paper](https://www.cncf.io/wp-content/uploads/2021/07/CNCF_Operator_WhitePaper.pdf)

## ✅ Week 5 Checklist

- [ ] Install and configure Helm
- [ ] Use Helm to deploy applications
- [ ] Create custom Helm charts
- [ ] Understand Helm templates and functions
- [ ] Manage Helm releases (install, upgrade, rollback)
- [ ] Create Custom Resource Definitions
- [ ] Understand OpenAPI validation schemas
- [ ] Install and use Operators
- [ ] Deploy Prometheus Operator
- [ ] Deploy Cert-Manager
- [ ] Deploy ArgoCD for GitOps
- [ ] Understand the Operator pattern
- [ ] Complete all practice exercises

---

**Next:** Week 6 - CI/CD, Monitoring, and Security 🚀

