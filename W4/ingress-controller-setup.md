# Ingress Controller Setup Guide

## 📦 **Installing Nginx Ingress Controller**

### Method 1: Using Official Manifests (Recommended for Learning)

```bash
# Install Nginx Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# For bare-metal/local (Minikube, kind, etc.)
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/baremetal/deploy.yaml
```

### Method 2: Using Helm (Production)

```bash
# Add Helm repo
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Install
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=LoadBalancer

# For NodePort (local/testing)
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=NodePort
```

### Method 3: Minikube Addon

```bash
minikube addons enable ingress

# Verify
kubectl get pods -n ingress-nginx
```

---

## ✅ **Verify Installation**

```bash
# Check namespace
kubectl get namespace ingress-nginx

# Check pods
kubectl get pods -n ingress-nginx

# Check service
kubectl get svc -n ingress-nginx

# Check deployment
kubectl get deployment -n ingress-nginx

# Wait for controller to be ready
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

---

## 🔍 **Check Ingress Controller Status**

```bash
# Get controller version
kubectl exec -it -n ingress-nginx \
  $(kubectl get pods -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx -o jsonpath='{.items[0].metadata.name}') \
  -- /nginx-ingress-controller --version

# Get external IP (LoadBalancer)
kubectl get svc -n ingress-nginx ingress-nginx-controller

# For NodePort - get port
kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath='{.spec.ports[?(@.name=="http")].nodePort}'
```

---

## 🧪 **Test Installation**

### 1. Deploy Test Applications

```bash
# Create test namespace
kubectl create namespace ingress-test

# Deploy two test apps
kubectl create deployment web1 --image=gcr.io/google-samples/hello-app:1.0 -n ingress-test
kubectl create deployment web2 --image=gcr.io/google-samples/hello-app:2.0 -n ingress-test

# Expose as services
kubectl expose deployment web1 --port=8080 -n ingress-test
kubectl expose deployment web2 --port=8080 -n ingress-test

# Verify
kubectl get pods,svc -n ingress-test
```

### 2. Create Test Ingress

```yaml
# Save as test-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  namespace: ingress-test
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /v1
        pathType: Prefix
        backend:
          service:
            name: web1
            port:
              number: 8080
      - path: /v2
        pathType: Prefix
        backend:
          service:
            name: web2
            port:
              number: 8080
```

```bash
# Apply
kubectl apply -f test-ingress.yaml

# Check Ingress
kubectl get ingress -n ingress-test
kubectl describe ingress test-ingress -n ingress-test
```

### 3. Test Access

```bash
# For LoadBalancer
INGRESS_IP=$(kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
curl http://$INGRESS_IP/v1
curl http://$INGRESS_IP/v2

# For NodePort
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}')
NODE_PORT=$(kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath='{.spec.ports[?(@.name=="http")].nodePort}')
curl http://$NODE_IP:$NODE_PORT/v1
curl http://$NODE_IP:$NODE_PORT/v2

# For Minikube
minikube service ingress-nginx-controller -n ingress-nginx --url
# Then use the URL: curl <URL>/v1
```

---

## 🛠️ **Other Popular Ingress Controllers**

### Traefik

```bash
helm repo add traefik https://traefik.github.io/charts
helm install traefik traefik/traefik --namespace traefik --create-namespace
```

### HAProxy

```bash
helm repo add haproxy https://haproxy-ingress.github.io/charts
helm install haproxy haproxy/haproxy-ingress --namespace haproxy --create-namespace
```

### Kong

```bash
kubectl apply -f https://bit.ly/k4k8s
```

### Contour (Envoy-based)

```bash
kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
```

---

## 📋 **Multiple Ingress Controllers**

You can run multiple Ingress Controllers and specify which one to use:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  ingressClassName: nginx  # or traefik, haproxy, etc.
  rules:
  - ...
```

**List IngressClasses:**
```bash
kubectl get ingressclass
```

---

## 🐛 **Troubleshooting**

### Controller not starting

```bash
# Check controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx

# Check events
kubectl get events -n ingress-nginx --sort-by='.lastTimestamp'

# Describe pod
kubectl describe pod -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx
```

### Ingress not working

```bash
# Check Ingress status
kubectl describe ingress <ingress-name> -n <namespace>

# Check controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller

# Validate Ingress syntax
kubectl get ingress <ingress-name> -n <namespace> -o yaml

# Check backend service
kubectl get svc <service-name> -n <namespace>
kubectl get endpoints <service-name> -n <namespace>
```

### DNS not resolving

```bash
# Add to /etc/hosts for testing
echo "$INGRESS_IP myapp.local" | sudo tee -a /etc/hosts

# Test
curl http://myapp.local
```

---

## 🔧 **Configuration**

### Custom ConfigMap

```bash
# Get configmap
kubectl get configmap -n ingress-nginx ingress-nginx-controller -o yaml

# Edit
kubectl edit configmap -n ingress-nginx ingress-nginx-controller

# Add custom configs:
data:
  proxy-body-size: "100m"
  proxy-connect-timeout: "600"
  proxy-send-timeout: "600"
  proxy-read-timeout: "600"
```

---

## 🧹 **Cleanup**

```bash
# Delete test resources
kubectl delete namespace ingress-test

# Uninstall Nginx Ingress (Helm)
helm uninstall ingress-nginx -n ingress-nginx
kubectl delete namespace ingress-nginx

# Uninstall Nginx Ingress (Manifest)
kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Minikube
minikube addons disable ingress
```

---

## 📚 **References**

- [Nginx Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [Ingress API](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Annotations Reference](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/)

