# Week 1: Core Concepts (Pods, Deployments, Services)

## 🎯 Learning Objectives

By the end of this week, you will:
- Understand Kubernetes architecture and core concepts
- Create and manage Pods
- Deploy applications using Deployments
- Expose applications with Services
- Master kubectl commands
- Organize resources with Namespaces

## 📁 Files in this directory

### Theory
- `W1.md` - Complete Q&A guide with theory, best practices, and interview questions

### Hands-on Examples

#### Pods
- `nginx-pod.yaml` - Simple Pod example
- `pod-examples.yaml` - Comprehensive Pod examples (10 scenarios)

#### Deployments
- `nginx-deployment.yaml` - Basic Deployment with Service
- `webapp-deployment.yaml` - Production-ready Deployment with sidecars

#### Services
- `nginx-service.yaml` - Simple NodePort Service
- `service-types.yaml` - All Service types (ClusterIP, NodePort, LoadBalancer, etc.)

#### Namespaces
- `namespace-examples.yaml` - Namespace management with ResourceQuotas

## 🚀 Quick Start

### 1. Create a Simple Pod

```bash
# Apply Pod manifest
kubectl apply -f nginx-pod.yaml

# Check Pod status
kubectl get pods

# Describe Pod
kubectl describe pod nginx

# View logs
kubectl logs nginx

# Access Pod
kubectl exec -it nginx -- bash
```

### 2. Create a Deployment

```bash
# Apply Deployment
kubectl apply -f nginx-deployment.yaml

# Check Deployment
kubectl get deployments
kubectl get pods

# Check ReplicaSet (created by Deployment)
kubectl get replicasets

# Scale Deployment
kubectl scale deployment nginx-deployment --replicas=5

# Check rollout status
kubectl rollout status deployment nginx-deployment
```

### 3. Expose with Service

```bash
# Apply Service
kubectl apply -f nginx-service.yaml

# Check Service
kubectl get services
kubectl describe service nginx-service

# Get service endpoints
kubectl get endpoints nginx-service

# Access Service (NodePort)
# Get node IP
kubectl get nodes -o wide
# Access: http://<NODE_IP>:30080
```

### 4. Work with Namespaces

```bash
# Create namespace
kubectl create namespace development

# Apply resources to namespace
kubectl apply -f nginx-deployment.yaml -n development

# Get resources in namespace
kubectl get all -n development

# Set default namespace
kubectl config set-context --current --namespace=development

# Delete namespace (deletes all resources!)
kubectl delete namespace development
```

## 🔧 Useful Commands

### Pod Management

```bash
# Create Pod (imperative)
kubectl run nginx --image=nginx:1.24 --port=80

# Get Pods
kubectl get pods
kubectl get pods -o wide
kubectl get pods -A  # All namespaces

# Describe Pod
kubectl describe pod nginx

# View logs
kubectl logs nginx
kubectl logs nginx -f  # Follow logs
kubectl logs nginx --tail=20
kubectl logs nginx --previous  # Previous container logs

# Execute command in Pod
kubectl exec nginx -- ls /
kubectl exec -it nginx -- bash

# Port forward to Pod
kubectl port-forward pod/nginx 8080:80
# Access: http://localhost:8080

# Delete Pod
kubectl delete pod nginx
```

### Deployment Management

```bash
# Create Deployment (imperative)
kubectl create deployment nginx --image=nginx:1.24 --replicas=3

# Get Deployments
kubectl get deployments
kubectl describe deployment nginx

# Scale Deployment
kubectl scale deployment nginx --replicas=5

# Update image
kubectl set image deployment/nginx nginx=nginx:1.25

# Rollout commands
kubectl rollout status deployment nginx
kubectl rollout history deployment nginx
kubectl rollout undo deployment nginx

# Delete Deployment
kubectl delete deployment nginx
```

### Service Management

```bash
# Expose Deployment
kubectl expose deployment nginx --port=80 --type=NodePort

# Get Services
kubectl get services
kubectl get svc  # Short form

# Describe Service
kubectl describe service nginx

# Get Service endpoints
kubectl get endpoints nginx

# Delete Service
kubectl delete service nginx
```

### Namespace Management

```bash
# Create namespace
kubectl create namespace production

# List namespaces
kubectl get namespaces
kubectl get ns  # Short form

# Switch namespace
kubectl config set-context --current --namespace=production

# Delete namespace
kubectl delete namespace production
```

### Debugging

```bash
# Get events (troubleshooting)
kubectl get events
kubectl get events --sort-by='.lastTimestamp'

# Describe resource (detailed info)
kubectl describe pod nginx
kubectl describe deployment nginx

# Check logs
kubectl logs nginx
kubectl logs nginx --previous

# Port forward for testing
kubectl port-forward pod/nginx 8080:80
```

## 🧪 Practice Exercises

### Exercise 1: Create and Manage Pods
1. Create a Pod using `kubectl run`
2. Create a Pod from YAML file
3. View Pod logs
4. Execute command inside Pod
5. Delete Pod

### Exercise 2: Work with Deployments
1. Create Deployment with 3 replicas
2. Scale to 5 replicas
3. Update container image
4. View rollout history
5. Rollback to previous version

### Exercise 3: Service Types
1. Create ClusterIP Service
2. Create NodePort Service
3. Test accessing services
4. Check service endpoints
5. Understand when to use each type

### Exercise 4: Labels and Selectors
1. Create Pods with different labels
2. Create Service with selector
3. Use `kubectl get` with label selectors
4. Update labels on running Pods

### Exercise 5: Namespaces
1. Create multiple namespaces (dev, staging, prod)
2. Deploy same application to different namespaces
3. Access service across namespaces
4. Set ResourceQuota per namespace

### Exercise 6: Multi-Container Pods
1. Create Pod with 2 containers
2. Share volume between containers
3. Test inter-container communication
4. View logs from specific container

### Exercise 7: Full Application Stack
1. Create namespace
2. Deploy backend (Deployment + ClusterIP Service)
3. Deploy frontend (Deployment + NodePort Service)
4. Test frontend accessing backend
5. Scale both applications

## 📊 Comparison Tables

### Pod vs Deployment

| Aspect              | Pod                     | Deployment               |
|---------------------|-------------------------|--------------------------|
| Purpose             | Run containers          | Manage Pods              |
| Self-healing        | ❌ No                    | ✅ Yes                    |
| Scaling             | Manual                  | Automatic/Manual         |
| Rolling updates     | ❌ No                    | ✅ Yes                    |
| Rollback            | ❌ No                    | ✅ Yes                    |
| Production use      | Rare                    | Standard                 |

### Service Types

| Type          | Access     | Port Range     | Cost  | Use Case          |
|---------------|------------|----------------|-------|-------------------|
| ClusterIP     | Internal   | Any            | Free  | Microservices     |
| NodePort      | External   | 30000-32767    | Free  | Dev/Testing       |
| LoadBalancer  | External   | Any            | $$    | Production        |
| ExternalName  | External   | N/A            | Free  | External services |

### kubectl Commands

| Task                  | Command                                    |
|-----------------------|--------------------------------------------|
| Create resource       | `kubectl apply -f file.yaml`               |
| Get resources         | `kubectl get pods/deployments/services`    |
| Describe resource     | `kubectl describe pod nginx`               |
| View logs             | `kubectl logs nginx`                       |
| Execute command       | `kubectl exec -it nginx -- bash`           |
| Port forward          | `kubectl port-forward pod/nginx 8080:80`   |
| Scale                 | `kubectl scale deployment nginx --replicas=3` |
| Delete resource       | `kubectl delete pod nginx`                 |

## ⚠️ Common Pitfalls

1. **Using bare Pods in production** - Always use Deployments
2. **Not setting resource limits** - Can cause resource starvation
3. **Using :latest tag** - Not version controlled
4. **Wrong Service selector** - Service won't find Pods
5. **Forgetting namespace** - Resources in default namespace
6. **Not using labels properly** - Hard to manage resources
7. **Creating too many LoadBalancers** - Expensive on cloud
8. **Hardcoding values** - Use ConfigMaps instead (Week 2)

## 🎓 Interview Questions to Practice

1. What is Kubernetes and why use it?
2. Explain the difference between a Pod and a Container
3. Why use Deployments instead of bare Pods?
4. What are the different Service types and when to use each?
5. How does Service discovery work in Kubernetes?
6. What is the difference between kubectl apply and create?
7. Explain the relationship between Deployment, ReplicaSet, and Pod
8. How do you troubleshoot a Pod that won't start?
9. What is a Namespace and why use it?
10. How do labels and selectors work?

## 📚 Additional Resources

- [Kubernetes Official Docs](https://kubernetes.io/docs/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Pod Documentation](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Deployment Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Service Documentation](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Kubernetes by Example](https://kubernetesbyexample.com/)

## ✅ Week 1 Checklist

- [ ] Understand what Kubernetes is and why it's used
- [ ] Create Pods using kubectl and YAML
- [ ] Deploy applications using Deployments
- [ ] Expose applications with Services
- [ ] Master basic kubectl commands
- [ ] Work with Namespaces
- [ ] Understand labels and selectors
- [ ] Practice all examples in this directory
- [ ] Complete all practice exercises
- [ ] Troubleshoot common issues

---

**Next:** Week 2 - ConfigMaps, Secrets, and Volumes 🚀

