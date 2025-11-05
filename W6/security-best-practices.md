# Kubernetes Security Best Practices

## 🔐 **Security Layers**

```
┌─────────────────────────────────────┐
│  1. Container Security              │ ← Image scanning, non-root users
├─────────────────────────────────────┤
│  2. Pod Security                    │ ← Security contexts, admission
├─────────────────────────────────────┤
│  3. Network Security                │ ← NetworkPolicies, encryption
├─────────────────────────────────────┤
│  4. Access Control                  │ ← RBAC, ServiceAccounts
├─────────────────────────────────────┤
│  5. Secrets Management              │ ← Encryption, external vaults
├─────────────────────────────────────┤
│  6. Cluster Security                │ ← API server, etcd, updates
└─────────────────────────────────────┘
```

---

## 1️⃣ **Pod Security**

### Pod Security Standards

```yaml
# Enforce restricted security at namespace level
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

### Security Context (Pod-level)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true          # Must run as non-root
    runAsUser: 1000             # Specific UID
    runAsGroup: 3000            # Specific GID
    fsGroup: 2000               # File system group
    seccompProfile:             # Seccomp profile
      type: RuntimeDefault
  
  containers:
  - name: app
    image: myapp:latest
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL                   # Drop all capabilities
        add:
        - NET_BIND_SERVICE      # Only add required ones
```

### Example: Secure Nginx Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-nginx
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 101  # nginx user
    fsGroup: 101
  
  containers:
  - name: nginx
    image: nginx:1.24
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: [ALL]
        add: [NET_BIND_SERVICE]
    
    volumeMounts:
    - name: cache
      mountPath: /var/cache/nginx
    - name: run
      mountPath: /var/run
  
  volumes:
  - name: cache
    emptyDir: {}
  - name: run
    emptyDir: {}
```

---

## 2️⃣ **Image Security**

### Scan Images for Vulnerabilities

```bash
# Using Trivy
trivy image myapp:latest

# Fail on HIGH/CRITICAL
trivy image --severity HIGH,CRITICAL --exit-code 1 myapp:latest

# Scan Kubernetes manifests
trivy config deployment.yaml

# Scan running cluster
trivy k8s --report summary cluster
```

### Use Trusted Registries

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: gcr.io/my-project/myapp:v1.0.0  # Use specific tags, not :latest
  
  imagePullSecrets:
  - name: registry-credentials
```

### Image Pull Policy

```yaml
containers:
- name: app
  image: myapp:v1.0.0
  imagePullPolicy: Always  # or IfNotPresent, Never
```

---

## 3️⃣ **Network Security**

### Default Deny NetworkPolicy

```yaml
# Deny all ingress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress

---
# Deny all egress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Egress
```

### Allow Specific Traffic

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
    ports:
    - protocol: TCP
      port: 8080
```

---

## 4️⃣ **RBAC (Already covered in rbac-examples.yaml)**

### Principle of Least Privilege

```yaml
# Good: Minimal permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]

# Bad: Too permissive
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
```

---

## 5️⃣ **Secrets Management**

### External Secrets Operator

```yaml
# Install
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace

# SecretStore
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secretsmanager
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets

# ExternalSecret
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: app-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secretsmanager
    kind: SecretStore
  target:
    name: app-secret
  data:
  - secretKey: password
    remoteRef:
      key: prod/database
      property: password
```

### Sealed Secrets

```bash
# Install
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/controller.yaml

# Install kubeseal CLI
brew install kubeseal

# Create sealed secret
kubectl create secret generic mysecret --from-literal=password=mypassword --dry-run=client -o yaml | \
kubeseal -o yaml > mysealedsecret.yaml

# Apply (safe to commit to Git)
kubectl apply -f mysealedsecret.yaml
```

### Encrypt Secrets at Rest

```yaml
# /etc/kubernetes/encryption-config.yaml
apiVersion: apiextensions.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources:
  - secrets
  providers:
  - aescbc:
      keys:
      - name: key1
        secret: <base64-encoded-32-byte-key>
  - identity: {}
```

Add to kube-apiserver:
```
--encryption-provider-config=/etc/kubernetes/encryption-config.yaml
```

---

## 6️⃣ **Cluster Hardening**

### API Server Security

```bash
# Disable anonymous auth
--anonymous-auth=false

# Enable audit logging
--audit-log-path=/var/log/kubernetes/audit.log
--audit-log-maxage=30
--audit-log-maxbackup=10
--audit-log-maxsize=100

# Restrict access to localhost
--insecure-bind-address=127.0.0.1
```

### etcd Security

```bash
# Enable TLS
--cert-file=/path/to/server.crt
--key-file=/path/to/server.key
--peer-cert-file=/path/to/peer.crt
--peer-key-file=/path/to/peer.key
--trusted-ca-file=/path/to/ca.crt
--peer-trusted-ca-file=/path/to/ca.crt

# Client authentication
--client-cert-auth=true
```

### Kubelet Security

```bash
# Disable anonymous access
--anonymous-auth=false

# Enable authentication
--client-ca-file=/path/to/ca.crt

# Enable authorization
--authorization-mode=Webhook
```

---

## 7️⃣ **Runtime Security**

### Falco (Runtime Security)

```bash
# Install Falco
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm install falco falcosecurity/falco \
  --namespace falco \
  --create-namespace \
  --set falco.grpc.enabled=true

# View alerts
kubectl logs -n falco -l app.kubernetes.io/name=falco
```

### Example Falco Rules

```yaml
# Detect shell in container
- rule: Shell in Container
  desc: Detect shell execution in container
  condition: >
    spawned_process and
    container and
    proc.name in (bash, sh, zsh)
  output: Shell spawned in container (user=%user.name container=%container.name shell=%proc.name)
  priority: WARNING

# Detect privilege escalation
- rule: Privilege Escalation
  desc: Detect privilege escalation
  condition: >
    spawned_process and
    proc.name in (sudo, su)
  output: Privilege escalation detected (user=%user.name command=%proc.cmdline)
  priority: CRITICAL
```

---

## 8️⃣ **Security Scanning**

### Kube-bench (CIS Benchmark)

```bash
# Run kube-bench
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml

# View results
kubectl logs job/kube-bench

# Run on master
kube-bench master

# Run on node
kube-bench node
```

### Kube-hunter (Penetration Testing)

```bash
# Run as pod
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-hunter/main/job.yaml

# View results
kubectl logs job/kube-hunter
```

---

## 9️⃣ **Security Checklist**

### Container Security
- [ ] Run as non-root user
- [ ] Read-only root filesystem
- [ ] Drop all capabilities
- [ ] No privilege escalation
- [ ] Scan images for vulnerabilities
- [ ] Use specific image tags (not :latest)
- [ ] Use trusted registries

### Pod Security
- [ ] Enable Pod Security Admission
- [ ] Set resource limits
- [ ] Configure security contexts
- [ ] Use seccomp profiles
- [ ] Enable AppArmor/SELinux

### Network Security
- [ ] Enable NetworkPolicies
- [ ] Default deny all traffic
- [ ] Allow only necessary traffic
- [ ] Use TLS for all communication
- [ ] Restrict egress traffic

### Access Control
- [ ] Enable RBAC
- [ ] Least privilege principle
- [ ] Avoid cluster-admin
- [ ] Use ServiceAccounts
- [ ] Regular access reviews

### Secrets Management
- [ ] Enable encryption at rest
- [ ] Use external secret managers
- [ ] Rotate secrets regularly
- [ ] Don't commit secrets to Git
- [ ] Mount secrets as volumes (not env vars)

### Cluster Security
- [ ] Secure API server
- [ ] Secure etcd
- [ ] Secure kubelet
- [ ] Enable audit logging
- [ ] Regular security updates
- [ ] Backup etcd regularly

### Monitoring & Detection
- [ ] Deploy Falco or similar
- [ ] Monitor audit logs
- [ ] Set up security alerts
- [ ] Regular security scans
- [ ] Incident response plan

---

## 🚨 **Common Security Mistakes**

### ❌ DON'T

```yaml
# Running as root
securityContext:
  runAsUser: 0

# Privileged container
securityContext:
  privileged: true

# Host network
hostNetwork: true

# Host path volumes
volumes:
- name: host
  hostPath:
    path: /

# All capabilities
securityContext:
  capabilities:
    add: [ALL]

# Latest tag
image: nginx:latest

# Secrets in env vars
env:
- name: PASSWORD
  value: "hardcoded"
```

### ✅ DO

```yaml
# Non-root user
securityContext:
  runAsNonRoot: true
  runAsUser: 1000

# Minimal capabilities
securityContext:
  allowPrivilegeEscalation: false
  capabilities:
    drop: [ALL]

# Specific image tags
image: nginx:1.24.0

# Secrets from volumes
volumeMounts:
- name: secrets
  mountPath: /secrets
  readOnly: true
volumes:
- name: secrets
  secret:
    secretName: app-secrets
```

---

## 📚 **Additional Resources**

- [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/)
- [CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes)
- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [OWASP Kubernetes Top 10](https://owasp.org/www-project-kubernetes-top-ten/)
- [NSA Kubernetes Hardening Guide](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF)

