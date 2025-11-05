# Week 6: CI/CD, Monitoring, and Security

## 🎯 Learning Objectives

By the end of this week, you will:
- Implement CI/CD pipelines for Kubernetes
- Deploy applications using GitOps (ArgoCD)
- Configure RBAC for security
- Monitor clusters with Prometheus and Grafana
- Secure Kubernetes clusters
- Manage secrets properly
- Understand security best practices

## 📁 Files in this directory

### Theory
- `W6.md` - Complete Q&A guide with theory, best practices, and interview questions

### Hands-on Examples

#### CI/CD
- `github-actions-example.yaml` - GitHub Actions workflows for Kubernetes
- `argocd-setup.md` - Complete ArgoCD installation and usage guide

#### Security & Access Control
- `rbac-examples.yaml` - Comprehensive RBAC examples
- `security-best-practices.md` - Security hardening guide

#### Monitoring
- `monitoring-setup.md` - Prometheus & Grafana setup

## 🚀 Quick Start

### 1. GitHub Actions CI/CD

```bash
# Create .github/workflows/deploy.yml
# See github-actions-example.yaml for full examples

# Add secrets to GitHub repository
# - DOCKER_USERNAME
# - DOCKER_PASSWORD
# - KUBECONFIG (base64 encoded)

# Push to main branch to trigger deployment
git add .
git commit -m "Deploy app"
git push origin main
```

### 2. ArgoCD GitOps

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Create application
kubectl apply -f - <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/yourusername/your-repo
    targetRevision: HEAD
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF
```

### 3. RBAC Configuration

```bash
# Create ServiceAccount
kubectl create serviceaccount ci-cd -n production

# Create Role
kubectl apply -f rbac-examples.yaml

# Test permissions
kubectl auth can-i create deployments --as=system:serviceaccount:production:ci-cd -n production

# Get ServiceAccount token
kubectl create token ci-cd -n production
```

### 4. Prometheus & Grafana Monitoring

```bash
# Install Prometheus stack
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring --create-namespace

# Access Prometheus
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090

# Access Grafana
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80

# Get Grafana password
kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 -d
```

### 5. Security Hardening

```bash
# Enable Pod Security Admission
kubectl label namespace production \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/warn=restricted

# Scan images
trivy image nginx:latest

# Scan cluster
trivy k8s --report summary cluster

# Run CIS benchmark
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml
kubectl logs job/kube-bench
```

## 🔧 Useful Commands

### CI/CD

```bash
# GitHub Actions
gh workflow list
gh workflow run deploy.yml
gh run list
gh run view <run-id>

# ArgoCD
argocd app list
argocd app get myapp
argocd app sync myapp
argocd app history myapp
argocd app rollback myapp 5
```

### RBAC

```bash
# Test permissions
kubectl auth can-i create pods --as=user@example.com
kubectl auth can-i --list --as=user@example.com

# View roles
kubectl get roles,rolebindings -A
kubectl get clusterroles,clusterrolebindings

# Describe
kubectl describe role pod-reader
kubectl describe rolebinding read-pods
```

### Monitoring

```bash
# Prometheus queries
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090

# ServiceMonitors
kubectl get servicemonitor -A
kubectl describe servicemonitor myapp -n production

# Alerts
kubectl get prometheusrule -A
```

### Security

```bash
# Pod security
kubectl get pod secure-pod -o jsonpath='{.spec.securityContext}'

# Scan images
trivy image myapp:latest
trivy config deployment.yaml
trivy k8s cluster

# Secrets
kubectl get secrets
kubectl describe secret app-secret
```

## 🧪 Practice Exercises

### Exercise 1: GitHub Actions Pipeline
1. Create GitHub Actions workflow
2. Build and push Docker image
3. Deploy to Kubernetes
4. Add image scanning step
5. Test multi-environment deployment

### Exercise 2: GitOps with ArgoCD
1. Install ArgoCD
2. Create Git repository with manifests
3. Create ArgoCD Application
4. Enable auto-sync
5. Make changes in Git and watch sync
6. Test rollback

### Exercise 3: RBAC Setup
1. Create ServiceAccount for CI/CD
2. Create Role with minimal permissions
3. Create RoleBinding
4. Test permissions
5. Create ClusterRole for cluster-wide access

### Exercise 4: Monitoring Stack
1. Install Prometheus + Grafana
2. Create ServiceMonitor
3. Write PromQL queries
4. Create custom dashboard
5. Set up alerts
6. Test alert notifications

### Exercise 5: Security Hardening
1. Enable Pod Security Admission
2. Create secure Pod with security context
3. Implement NetworkPolicies
4. Scan images with Trivy
5. Run CIS benchmark
6. Fix security issues

### Exercise 6: Secret Management
1. Create secret manually
2. Use External Secrets Operator
3. Configure Sealed Secrets
4. Mount secrets as volumes
5. Test secret rotation

### Exercise 7: End-to-End Secure Pipeline
1. GitHub Actions with security scanning
2. Push to secure registry
3. Deploy with ArgoCD
4. RBAC for deployment
5. Monitor with Prometheus
6. NetworkPolicies for isolation

## 📊 Comparison Tables

### CI/CD Tools

| Tool           | Type        | Complexity | Best For                |
|----------------|-------------|------------|-------------------------|
| GitHub Actions | Cloud       | Low        | GitHub users            |
| GitLab CI      | Cloud/Self  | Medium     | GitLab users            |
| Jenkins        | Self-hosted | High       | Complex pipelines       |
| ArgoCD         | GitOps      | Medium     | Declarative deployments |
| Tekton         | Native      | High       | Cloud-native pipelines  |

### Monitoring Solutions

| Tool       | Purpose         | Complexity | CNCF        |
|------------|-----------------|------------|-------------|
| Prometheus | Metrics         | Medium     | Graduated   |
| Grafana    | Visualization   | Low        | No          |
| Loki       | Logs            | Medium     | No          |
| Jaeger     | Tracing         | Medium     | Graduated   |
| Falco      | Runtime Security| Medium     | Incubating  |

### Secret Management

| Solution             | Type       | Complexity | Cost    |
|----------------------|------------|------------|---------|
| Kubernetes Secrets   | Built-in   | Low        | Free    |
| Sealed Secrets       | Encrypted  | Medium     | Free    |
| External Secrets     | Operator   | Medium     | Free    |
| HashiCorp Vault      | Enterprise | High       | $$      |
| AWS Secrets Manager  | Cloud      | Low        | $       |

## ⚠️ Common Pitfalls

### CI/CD
1. **Hardcoded credentials** - Use secrets management
2. **No testing stage** - Always test before deploy
3. **Using :latest tag** - Use specific versions
4. **No rollback plan** - Always have rollback strategy
5. **Deploying to prod directly** - Use staging first

### Monitoring
1. **No alerts** - Set up critical alerts
2. **Alert fatigue** - Don't alert on everything
3. **No retention policy** - Metrics storage grows
4. **Ignoring logs** - Logs are as important as metrics
5. **No SLOs** - Define service level objectives

### Security
1. **Running as root** - Always use non-root users
2. **Privileged containers** - Avoid unless absolutely necessary
3. **No NetworkPolicies** - Implement network segmentation
4. **Weak RBAC** - Follow least privilege
5. **Secrets in Git** - Never commit secrets
6. **No security scanning** - Scan images and manifests
7. **Ignoring updates** - Keep cluster and components updated

## 🎓 Interview Questions to Practice

1. Explain GitOps and its benefits
2. How does ArgoCD work?
3. What are the components of RBAC?
4. How does Prometheus collect metrics?
5. What is the difference between ServiceMonitor and PodMonitor?
6. Explain the 4 Golden Signals of monitoring
7. How do you secure a Kubernetes cluster?
8. What is Pod Security Admission?
9. How do you manage secrets in Kubernetes?
10. Explain zero-downtime deployment strategies
11. What is the difference between liveness and readiness probes?
12. How do you implement network policies?
13. What is etcd and why is it critical?
14. How do you backup and restore etcd?
15. Explain the Kubernetes security admission control

## 📚 Additional Resources

- [Kubernetes Security](https://kubernetes.io/docs/concepts/security/)
- [RBAC Documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes)
- [OWASP Kubernetes Top 10](https://owasp.org/www-project-kubernetes-top-ten/)
- [NSA Kubernetes Hardening Guide](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)

## ✅ Week 6 Checklist

- [ ] Understand CI/CD concepts
- [ ] Create GitHub Actions workflow
- [ ] Install and configure ArgoCD
- [ ] Create GitOps application
- [ ] Understand RBAC components
- [ ] Configure ServiceAccounts and Roles
- [ ] Install Prometheus + Grafana
- [ ] Create ServiceMonitors
- [ ] Write PromQL queries
- [ ] Set up alerts
- [ ] Enable Pod Security Admission
- [ ] Implement security contexts
- [ ] Create NetworkPolicies
- [ ] Scan images for vulnerabilities
- [ ] Secure etcd
- [ ] Implement secret management
- [ ] Complete all practice exercises

---

## 🎉 **Congratulations!**

You've completed the **6-Week Kubernetes Mastery Program**!

### What You've Learned:
- ✅ Week 1: Core Concepts (Pods, Deployments, Services)
- ✅ Week 2: ConfigMaps, Secrets, and Volumes
- ✅ Week 3: Scaling, Probes & Health Checks
- ✅ Week 4: Ingress, Networking, and Load Balancing
- ✅ Week 5: Helm, CRDs, and Operators
- ✅ Week 6: CI/CD, Monitoring, and Security

### Next Steps:
1. 🎯 **Practice** - Build real projects
2. 📜 **Certifications** - CKA, CKAD, CKS
3. 🌐 **Advanced Topics** - Service Mesh, Multi-cluster
4. 🤝 **Contribute** - Open source Kubernetes projects
5. 📚 **Stay Updated** - Follow Kubernetes blog and releases

### Career Opportunities:
- Kubernetes Administrator
- DevOps Engineer
- Site Reliability Engineer (SRE)
- Cloud Architect
- Platform Engineer

**You're now ready for production Kubernetes!** 🚀

