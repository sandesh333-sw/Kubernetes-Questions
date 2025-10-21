# Week 2: ConfigMaps, Secrets, and Volumes

## 🎯 Learning Objectives

By the end of this week, you will:
- Master ConfigMaps and Secrets management
- Understand different volume types and when to use them
- Work with PersistentVolumes and PersistentVolumeClaims
- Implement dynamic storage provisioning with StorageClass

## 📁 Files in this directory

### Theory
- `W2.md` - Complete Q&A guide with theory, best practices, and interview questions

### Hands-on Examples

#### ConfigMaps & Secrets
- `app-configmap.yaml` - ConfigMap examples (env vars, config files)
- `app-secret.yaml` - Secret examples (Opaque, TLS, Docker registry)
- `pod-with-config-secret.yaml` - Pods using ConfigMaps and Secrets

#### Persistent Storage
- `pv.yaml` - PersistentVolume examples (local, NFS, AWS EBS)
- `pvc.yaml` - PersistentVolumeClaim examples
- `pod-with-pvc.yaml` - Pods using PVCs with different volume types

#### Dynamic Provisioning
- `storageclass.yaml` - StorageClass examples (AWS, GCP, Azure, NFS)
- `dynamic-pvc.yaml` - Dynamic PVC examples with StatefulSet

## 🚀 Quick Start

### 1. Create ConfigMap and Secret

```bash
# Apply ConfigMap
kubectl apply -f app-configmap.yaml

# Apply Secret
kubectl apply -f app-secret.yaml

# Verify
kubectl get configmaps
kubectl get secrets
kubectl describe configmap app-config
kubectl describe secret app-secret
```

### 2. Use ConfigMap and Secret in Pod

```bash
# Create Pod
kubectl apply -f pod-with-config-secret.yaml

# Verify environment variables
kubectl exec app-with-config-secret -- env | grep -E "(DATABASE|LOG|USERNAME|PASSWORD|CONFIG_)"

# Check mounted files
kubectl exec app-with-config-secret -- ls -la /etc/config/
kubectl exec app-with-config-secret -- ls -la /etc/secrets/
kubectl exec app-with-config-secret -- cat /etc/config/app.properties
```

### 3. Work with Persistent Volumes

```bash
# Create PVs and PVCs
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml

# Check PV/PVC status
kubectl get pv
kubectl get pvc

# Verify binding
kubectl describe pvc pvc-app-storage

# Create Pod with PVC
kubectl apply -f pod-with-pvc.yaml

# Test persistence
kubectl exec app-with-persistent-storage -- sh -c 'echo "Hello, World!" > /usr/share/nginx/html/index.html'
kubectl exec app-with-persistent-storage -- cat /usr/share/nginx/html/index.html

# Delete and recreate Pod (data persists!)
kubectl delete pod app-with-persistent-storage
kubectl apply -f pod-with-pvc.yaml
kubectl exec app-with-persistent-storage -- cat /usr/share/nginx/html/index.html
```

### 4. Dynamic Provisioning (Cloud or local provisioner required)

```bash
# Create StorageClass (if not exists)
kubectl apply -f storageclass.yaml

# Create dynamic PVC
kubectl apply -f dynamic-pvc.yaml

# Watch PV auto-creation
kubectl get pvc -w
kubectl get pv
```

## 🔧 Useful Commands

### ConfigMaps

```bash
# Create ConfigMap from literal values
kubectl create configmap my-config \
  --from-literal=key1=value1 \
  --from-literal=key2=value2

# Create ConfigMap from file
kubectl create configmap app-config --from-file=config.properties

# Create ConfigMap from directory
kubectl create configmap configs --from-file=./configs/

# View ConfigMap
kubectl get configmap app-config -o yaml

# Edit ConfigMap
kubectl edit configmap app-config
```

### Secrets

```bash
# Create Secret from literal
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=secretpass

# Create Secret from file
kubectl create secret generic ssh-key --from-file=~/.ssh/id_rsa

# Create Docker registry secret
kubectl create secret docker-registry my-registry \
  --docker-server=docker.io \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=user@example.com

# Create TLS secret
kubectl create secret tls tls-secret \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key

# Decode secret
kubectl get secret app-secret -o jsonpath='{.data.password}' | base64 -d
```

### Volumes & Storage

```bash
# List PVs and PVCs
kubectl get pv
kubectl get pvc

# Describe PV/PVC
kubectl describe pv pv-local-storage
kubectl describe pvc pvc-app-storage

# Check PVC binding status
kubectl get pvc pvc-app-storage -o jsonpath='{.status.phase}'

# Delete PVC (check reclaim policy!)
kubectl delete pvc pvc-app-storage

# List StorageClasses
kubectl get storageclass
kubectl describe storageclass aws-ebs-gp3
```

## 🧪 Practice Exercises

### Exercise 1: Configuration Management
1. Create a ConfigMap with database connection details
2. Create a Secret with database credentials
3. Deploy a Pod that uses both
4. Verify the Pod can read the configuration

### Exercise 2: Volume Types
1. Create a Pod with emptyDir volume
2. Write data to it
3. Delete and recreate the Pod
4. Observe that data is lost

### Exercise 3: Persistent Storage
1. Create a PV with 5Gi storage
2. Create a PVC requesting 3Gi
3. Deploy a Pod using the PVC
4. Write data and verify persistence across Pod restarts

### Exercise 4: Multi-Container Sharing
1. Create a Pod with 2 containers
2. Use emptyDir to share data between them
3. One container writes, the other reads

### Exercise 5: ConfigMap Hot Reload
1. Create a ConfigMap mounted as volume
2. Deploy a Pod using it
3. Update the ConfigMap
4. Wait and verify changes appear in the Pod (may take ~60s)

## 📊 Storage Comparison

| Volume Type    | Lifecycle        | Shared | Use Case                   |
|----------------|------------------|--------|----------------------------|
| emptyDir       | Pod lifetime     | ✅      | Temporary cache, scratch   |
| hostPath       | Node lifetime    | ❌      | Node logs, system files    |
| PV/PVC         | Independent      | ✅*     | Databases, user data       |
| ConfigMap      | Independent      | ✅      | Configuration files        |
| Secret         | Independent      | ✅      | Sensitive data             |

*Depends on access mode (RWX)

## ⚠️ Common Pitfalls

1. **Secrets are not encrypted by default** - Enable encryption at rest
2. **ConfigMap updates don't update env vars** - Only mounted volumes get updated
3. **hostPath ties Pods to nodes** - Avoid in production
4. **PVC binding issues** - Check access modes, storage class, and capacity match
5. **Forgetting reclaim policy** - Use Retain for production data

## 🎓 Interview Questions to Practice

1. How do ConfigMaps differ from Secrets?
2. What happens when you update a ConfigMap that's mounted as a volume vs environment variable?
3. Explain the PV/PVC binding process
4. What are the different access modes and when would you use each?
5. How does dynamic provisioning work?
6. What is the difference between Retain and Delete reclaim policies?
7. How would you securely manage secrets in production?
8. Can multiple PVCs bind to the same PV?

## 📚 Additional Resources

- [Kubernetes ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- [External Secrets Operator](https://external-secrets.io/)

## ✅ Week 2 Checklist

- [ ] Understand ConfigMaps and Secrets
- [ ] Create and use ConfigMaps in Pods
- [ ] Create and use Secrets securely
- [ ] Differentiate between volume types
- [ ] Work with PV and PVC
- [ ] Understand StorageClass and dynamic provisioning
- [ ] Practice all hands-on examples
- [ ] Complete practice exercises

---

**Next:** Week 3 - Scaling, Probes & Health Checks 🚀

