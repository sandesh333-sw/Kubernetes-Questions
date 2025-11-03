# Week 4: Ingress, Networking, and Load Balancing

## 🎯 Learning Objectives

By the end of this week, you will:
- Master Ingress for HTTP/HTTPS routing
- Understand Kubernetes networking model
- Work with all Service types
- Implement NetworkPolicies for security
- Configure TLS/SSL termination
- Use Labels and Selectors effectively

## 📁 Files in this directory

### Theory
- `W4.md` - Complete Q&A guide with theory, best practices, and interview questions

### Hands-on Examples

#### Ingress Setup & Configuration
- `ingress-controller-setup.md` - Complete guide to install Nginx Ingress Controller
- `ingress-basic.yaml` - Basic Ingress examples
- `ingress-path-based.yaml` - Path-based routing
- `ingress-host-based.yaml` - Host-based routing and wildcard domains
- `ingress-tls.yaml` - TLS/SSL configuration and cert-manager
- `ingress-advanced.yaml` - Advanced features (rate limiting, auth, CORS, canary)

#### Services & Networking
- `service-types.yaml` - All service types with examples
- `network-policy.yaml` - Comprehensive NetworkPolicy examples

## 🚀 Quick Start

### 1. Install Ingress Controller

```bash
# For Minikube
minikube addons enable ingress

# For other clusters
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Verify
kubectl get pods -n ingress-nginx
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

### 2. Basic Ingress

```bash
# Deploy application and service
kubectl apply -f ingress-basic.yaml

# Check Ingress
kubectl get ingress
kubectl describe ingress basic-ingress

# Get Ingress IP/hostname
kubectl get ingress basic-ingress -o jsonpath='{.status.loadBalancer.ingress[0].ip}'

# Test (replace IP with actual)
curl http://<INGRESS-IP>/
```

### 3. Path-based Routing

```bash
# Deploy multiple services
kubectl apply -f ingress-path-based.yaml

# Test different paths
curl http://<INGRESS-IP>/api
curl http://<INGRESS-IP>/web
curl http://<INGRESS-IP>/admin
```

### 4. Host-based Routing

```bash
# Deploy
kubectl apply -f ingress-host-based.yaml

# Add to /etc/hosts (for testing)
echo "<INGRESS-IP> myapp.com api.myapp.com admin.myapp.com blog.myapp.com" | sudo tee -a /etc/hosts

# Test different hosts
curl http://myapp.com
curl http://api.myapp.com
curl http://admin.myapp.com
curl http://blog.myapp.com
```

### 5. TLS/SSL Configuration

```bash
# Generate self-signed certificate (testing)
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=myapp.com/O=myapp"

# Create TLS secret
kubectl create secret tls myapp-tls --cert=tls.crt --key=tls.key

# Deploy TLS Ingress
kubectl apply -f ingress-tls.yaml

# Test HTTPS
curl -k https://myapp.com  # -k to ignore self-signed cert
```

### 6. Service Types

```bash
# Deploy all service types
kubectl apply -f service-types.yaml

# ClusterIP (internal only)
kubectl get svc clusterip-service
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://clusterip-service

# NodePort (external via node IP)
kubectl get svc nodeport-service
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}')
NODE_PORT=$(kubectl get svc nodeport-service -o jsonpath='{.spec.ports[0].nodePort}')
curl http://$NODE_IP:$NODE_PORT

# LoadBalancer (cloud provider)
kubectl get svc loadbalancer-service
# Wait for EXTERNAL-IP
kubectl get svc loadbalancer-service -w
EXTERNAL_IP=$(kubectl get svc loadbalancer-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
curl http://$EXTERNAL_IP
```

### 7. NetworkPolicies

```bash
# Create test namespace
kubectl create namespace production

# Label namespace (for selectors)
kubectl label namespace production environment=production

# Deploy applications
kubectl apply -f network-policy.yaml -n production

# Test before NetworkPolicy
kubectl run frontend --image=nginx -n production --labels=tier=frontend
kubectl run backend --image=nginx -n production --labels=tier=backend
kubectl run database --image=nginx -n production --labels=tier=database

# All can communicate (default)
kubectl exec -n production frontend -- wget -O- http://backend
kubectl exec -n production frontend -- wget -O- http://database
kubectl exec -n production backend -- wget -O- http://database

# Apply NetworkPolicies
kubectl apply -f network-policy.yaml -n production

# Test after (should fail where not allowed)
kubectl exec -n production frontend -- wget -T 2 -O- http://database  # Should timeout
kubectl exec -n production frontend -- wget -O- http://backend  # Should work
```

### 8. Advanced Ingress Features

```bash
# Deploy advanced Ingress
kubectl apply -f ingress-advanced.yaml

# Test rate limiting
for i in {1..20}; do curl http://api.myapp.com; done

# Test basic auth
kubectl apply -f ingress-advanced.yaml
curl -u admin:password http://admin.myapp.com

# Test canary deployment (10% traffic to canary)
kubectl apply -f ingress-advanced.yaml
for i in {1..20}; do curl http://myapp.com; done

# Test canary with header
curl -H "X-Canary: always" http://myapp.com
```

## 🔧 Useful Commands

### Ingress

```bash
# List Ingress
kubectl get ingress
kubectl get ingress -A

# Describe Ingress
kubectl describe ingress <ingress-name>

# Get Ingress YAML
kubectl get ingress <ingress-name> -o yaml

# Edit Ingress
kubectl edit ingress <ingress-name>

# Check Ingress Controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx

# Get Ingress Class
kubectl get ingressclass

# Test Ingress backend
kubectl get ingress <ingress-name> -o jsonpath='{.spec.rules[0].http.paths[0].backend.service.name}'
```

### Services

```bash
# List services
kubectl get svc
kubectl get svc -A

# Describe service
kubectl describe svc <service-name>

# Get service endpoints
kubectl get endpoints <service-name>

# Test service DNS
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup <service-name>

# Port-forward to service
kubectl port-forward svc/<service-name> 8080:80
```

### NetworkPolicies

```bash
# List NetworkPolicies
kubectl get networkpolicy
kubectl get netpol  # Short form

# Describe NetworkPolicy
kubectl describe networkpolicy <policy-name>

# Test connectivity
kubectl run -it --rm test --image=busybox --restart=Never -- wget -T 5 -O- http://service-name

# Check pod labels (for selectors)
kubectl get pods --show-labels
```

### DNS & Networking

```bash
# Test DNS resolution
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup kubernetes.default

# Full service DNS
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup myservice.default.svc.cluster.local

# Check CoreDNS
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns

# Test pod-to-pod communication
kubectl get pods -o wide
kubectl exec <pod1> -- ping <pod2-ip>
```

## 🧪 Practice Exercises

### Exercise 1: Basic Ingress
1. Deploy two applications
2. Create Ingress with path-based routing
3. Test both paths
4. Check Ingress Controller logs

### Exercise 2: Multi-Host Ingress
1. Deploy apps for different subdomains
2. Create host-based Ingress
3. Add entries to /etc/hosts
4. Test all subdomains

### Exercise 3: TLS Configuration
1. Generate self-signed certificate
2. Create TLS secret
3. Configure Ingress with TLS
4. Test HTTPS access

### Exercise 4: Service Types
1. Deploy same app with different service types
2. Access via ClusterIP (from inside cluster)
3. Access via NodePort (from outside)
4. Access via LoadBalancer (if available)

### Exercise 5: NetworkPolicy Security
1. Deploy 3-tier app (frontend, backend, database)
2. Apply default deny-all policy
3. Add policies to allow only necessary traffic
4. Test that restrictions work

### Exercise 6: Canary Deployment
1. Deploy production service
2. Deploy canary service
3. Configure canary Ingress (10% traffic)
4. Test traffic distribution

### Exercise 7: Advanced Ingress
1. Add rate limiting
2. Add basic authentication
3. Configure CORS
4. Add custom headers

## 📊 Comparison Tables

### Service Types

| Type          | Exposure | Cost   | Use Case           | Access Method         |
|---------------|----------|--------|--------------------|-----------------------|
| ClusterIP     | Internal | Free   | Internal services  | DNS name              |
| NodePort      | External | Free   | Dev/testing        | NodeIP:NodePort       |
| LoadBalancer  | External | $$     | Production (cloud) | External IP           |
| ExternalName  | External | Free   | DNS alias          | DNS CNAME             |

### Ingress vs Service

| Feature                 | Service      | Ingress                |
|-------------------------|--------------|------------------------|
| Layer                   | L4 (TCP/UDP) | L7 (HTTP/HTTPS)        |
| Path routing            | ❌            | ✅                      |
| Host routing            | ❌            | ✅                      |
| TLS termination         | ❌            | ✅                      |
| Advanced features       | ❌            | ✅ (auth, rate limit)  |

### NetworkPolicy Selectors

| Selector Type     | Use Case                          | Example                |
|-------------------|-----------------------------------|------------------------|
| podSelector       | Select pods by labels             | app: backend           |
| namespaceSelector | Select entire namespace           | environment: prod      |
| ipBlock           | Select IP ranges                  | 10.0.0.0/8             |
| Combined          | Pod in specific namespace         | namespace + pod labels |

## ⚠️ Common Pitfalls

1. **Forgetting to install Ingress Controller** - Ingress resources do nothing without a controller
2. **No DNS/hosts entry** - Host-based routing needs DNS or /etc/hosts
3. **Wrong IngressClass** - Specify correct class if multiple controllers
4. **NetworkPolicy no effect** - CNI must support NetworkPolicies
5. **Default deny breaks everything** - Add DNS and essential traffic first
6. **TLS secret in wrong namespace** - Must be in same namespace as Ingress
7. **NodePort port conflicts** - Range is 30000-32767
8. **LoadBalancer without cloud provider** - Stays in pending state

## 🎓 Interview Questions to Practice

1. How does Ingress differ from Service?
2. What is an Ingress Controller and why is it needed?
3. Explain the Kubernetes networking model
4. How does kube-proxy work?
5. What are NetworkPolicies and when would you use them?
6. Explain the difference between NodePort and LoadBalancer
7. How does DNS resolution work for Services?
8. What is a headless service and when would you use it?
9. How do you implement zero-downtime deployments with Ingress?
10. Explain taints and tolerations

## 📚 Additional Resources

- [Ingress Documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Nginx Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [NetworkPolicies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [DNS for Services](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
- [Cert-Manager](https://cert-manager.io/)

## ✅ Week 4 Checklist

- [ ] Understand Ingress and Ingress Controllers
- [ ] Install and configure Nginx Ingress Controller
- [ ] Implement path-based routing
- [ ] Implement host-based routing
- [ ] Configure TLS/SSL termination
- [ ] Work with all Service types
- [ ] Understand Kubernetes networking model
- [ ] Implement NetworkPolicies
- [ ] Test advanced Ingress features
- [ ] Complete all practice exercises

---

**Next:** Week 5 - Helm, CRDs, and Operators 🚀

