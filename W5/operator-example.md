# Kubernetes Operators - Practical Examples

## 📦 **Installing Popular Operators**

### 1. Prometheus Operator (Monitoring)

```bash
# Add Prometheus Helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus Operator
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace

# Verify
kubectl get pods -n monitoring
kubectl get crd | grep monitoring

# Access Prometheus UI
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090

# Access Grafana
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
# Default credentials: admin / prom-operator
```

### 2. Cert-Manager (TLS Certificate Management)

```bash
# Install Cert-Manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# Verify
kubectl get pods -n cert-manager
kubectl get crd | grep cert-manager

# Create ClusterIssuer for Let's Encrypt
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
EOF

# Create Certificate
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: myapp-tls
  namespace: default
spec:
  secretName: myapp-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - myapp.example.com
  - www.myapp.example.com
EOF

# Check certificate
kubectl get certificate
kubectl describe certificate myapp-tls
```

### 3. MySQL Operator

```bash
# Install MySQL Operator
helm repo add mysql-operator https://mysql.github.io/mysql-operator/
helm repo update

helm install mysql-operator mysql-operator/mysql-operator \
  --namespace mysql-operator \
  --create-namespace

# Verify
kubectl get pods -n mysql-operator
kubectl get crd | grep mysql

# Create MySQL Cluster
cat <<EOF | kubectl apply -f -
apiVersion: mysql.oracle.com/v2
kind: InnoDBCluster
metadata:
  name: mycluster
spec:
  secretName: mypwds
  tlsUseSelfSigned: true
  instances: 3
  router:
    instances: 1
  datadirVolumeClaimTemplate:
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: 10Gi
EOF

# Create secret for MySQL
kubectl create secret generic mypwds \
  --from-literal=rootUser=root \
  --from-literal=rootHost=% \
  --from-literal=rootPassword=MyS3cr3t

# Check cluster
kubectl get innodbcluster
kubectl get pods
```

### 4. ArgoCD (GitOps)

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Verify
kubectl get pods -n argocd

# Access ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Create Application
cat <<EOF | kubectl apply -f -
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

### 5. Strimzi Kafka Operator

```bash
# Install Strimzi Operator
kubectl create namespace kafka
kubectl apply -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka

# Verify
kubectl get pods -n kafka
kubectl get crd | grep kafka

# Create Kafka Cluster
cat <<EOF | kubectl apply -f -
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: my-cluster
  namespace: kafka
spec:
  kafka:
    version: 3.5.0
    replicas: 3
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: tls
        port: 9093
        type: internal
        tls: true
    storage:
      type: persistent-claim
      size: 10Gi
      deleteClaim: false
  zookeeper:
    replicas: 3
    storage:
      type: persistent-claim
      size: 5Gi
      deleteClaim: false
  entityOperator:
    topicOperator: {}
    userOperator: {}
EOF

# Create Kafka Topic
cat <<EOF | kubectl apply -f -
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: my-topic
  namespace: kafka
  labels:
    strimzi.io/cluster: my-cluster
spec:
  partitions: 3
  replicas: 2
  config:
    retention.ms: 7200000
    segment.bytes: 1073741824
EOF

# Check resources
kubectl get kafka -n kafka
kubectl get kafkatopic -n kafka
```

---

## 🛠️ **Building a Simple Operator**

### Using Operator SDK (Go)

```bash
# Install Operator SDK
brew install operator-sdk  # macOS
# or
curl -LO https://github.com/operator-framework/operator-sdk/releases/latest/download/operator-sdk_linux_amd64
chmod +x operator-sdk_linux_amd64
sudo mv operator-sdk_linux_amd64 /usr/local/bin/operator-sdk

# Create new operator project
mkdir myapp-operator
cd myapp-operator

operator-sdk init --domain=example.com --repo=github.com/yourusername/myapp-operator

# Create API and Controller
operator-sdk create api --group apps --version v1 --kind MyApp --resource --controller

# Edit the API (api/v1/myapp_types.go)
# Edit the Controller (controllers/myapp_controller.go)

# Build and push
make docker-build docker-push IMG=yourusername/myapp-operator:v0.1.0

# Deploy
make deploy IMG=yourusername/myapp-operator:v0.1.0

# Create custom resource
kubectl apply -f config/samples/apps_v1_myapp.yaml
```

### Using Helm-based Operator

```bash
# Create Helm-based operator
operator-sdk init --plugins=helm --domain=example.com --group=apps --version=v1 --kind=MyApp

# Use existing Helm chart
operator-sdk create api --helm-chart=./mychart

# Build and deploy
make docker-build docker-push IMG=yourusername/myapp-operator:v0.1.0
make deploy IMG=yourusername/myapp-operator:v0.1.0
```

---

## 🧪 **Testing Operators**

### Test CRD Installation

```bash
# Apply CRD
kubectl apply -f crd-example.yaml

# List CRDs
kubectl get crd

# Describe CRD
kubectl describe crd databases.example.com

# Create custom resource
kubectl apply -f - <<EOF
apiVersion: example.com/v1
kind: Database
metadata:
  name: test-db
spec:
  size: 10Gi
  version: "15"
  replicas: 2
EOF

# Get custom resources
kubectl get databases
kubectl get db  # short name

# Describe custom resource
kubectl describe database test-db

# Delete custom resource
kubectl delete database test-db
```

---

## 📊 **Operator Monitoring**

### Check Operator Logs

```bash
# Find operator pod
kubectl get pods -n mysql-operator

# View logs
kubectl logs -n mysql-operator <operator-pod-name>

# Follow logs
kubectl logs -n mysql-operator <operator-pod-name> -f

# Previous logs (if restarted)
kubectl logs -n mysql-operator <operator-pod-name> --previous
```

### Check Operator Events

```bash
# Events for custom resource
kubectl get events --field-selector involvedObject.name=mycluster

# Events in namespace
kubectl get events -n kafka --sort-by='.lastTimestamp'
```

---

## 🔍 **Troubleshooting Operators**

### Common Issues

**1. CRD not found**
```bash
# Check if CRD exists
kubectl get crd | grep example.com

# Apply CRD
kubectl apply -f crd-example.yaml
```

**2. Operator not reconciling**
```bash
# Check operator logs
kubectl logs -n operator-namespace <operator-pod>

# Check RBAC permissions
kubectl describe clusterrole <operator-role>
kubectl describe clusterrolebinding <operator-binding>
```

**3. Custom resource validation failed**
```bash
# Check CRD schema
kubectl get crd databases.example.com -o yaml

# Validate your CR
kubectl apply --dry-run=server -f my-database.yaml
```

---

## 📚 **Useful Operator Resources**

- [OperatorHub.io](https://operatorhub.io/) - Operator catalog
- [Operator SDK](https://sdk.operatorframework.io/) - Build operators
- [Kubebuilder](https://book.kubebuilder.io/) - Operator framework
- [KUDO](https://kudo.dev/) - Declarative operators

---

## ✅ **Best Practices**

1. ✅ Use existing operators when possible
2. ✅ Test operators in dev/staging first
3. ✅ Monitor operator logs
4. ✅ Set resource limits on operator pods
5. ✅ Use RBAC with least privilege
6. ✅ Version your CRDs properly
7. ✅ Document custom resources
8. ✅ Implement proper validation
9. ✅ Handle edge cases in reconciliation
10. ✅ Add observability (metrics, logs, traces)

