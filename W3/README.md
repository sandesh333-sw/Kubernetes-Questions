# Week 3: Scaling, Probes & Health Checks

## 🎯 Learning Objectives

By the end of this week, you will:
- Master manual and automatic scaling with HPA
- Implement health checks using liveness, readiness, and startup probes
- Understand and configure resource requests and limits
- Perform rolling updates and rollbacks
- Work with DaemonSets and StatefulSets

## 📁 Files in this directory

### Theory
- `W3.md` - Complete Q&A guide with theory, best practices, and interview questions

### Hands-on Examples

#### Scaling
- `deployment-scalable.yaml` - Deployments ready for scaling
- `hpa.yaml` - Horizontal Pod Autoscaler examples (CPU, memory, custom metrics)

#### Health Checks
- `deployment-with-probes.yaml` - Comprehensive probe examples (HTTP, TCP, Exec)

#### Resource Management
- `deployment-with-resources.yaml` - Resource requests/limits, QoS classes, quotas

#### Updates & Rollbacks
- `deployment-rolling-update.yaml` - Rolling update strategies and PDB

#### Special Workloads
- `daemonset-example.yaml` - DaemonSet examples (monitoring, logging, networking)
- `statefulset-mysql.yaml` - StatefulSet examples (MySQL, MongoDB, Redis, Kafka)

## 🚀 Quick Start

### 1. Manual Scaling

```bash
# Deploy application
kubectl apply -f deployment-scalable.yaml

# Check current replicas
kubectl get deployment nginx-scalable

# Scale manually
kubectl scale deployment nginx-scalable --replicas=5

# Verify
kubectl get pods -l app=nginx
```

### 2. Horizontal Pod Autoscaler (HPA)

**Prerequisites:** Metrics Server must be installed

```bash
# Install Metrics Server (if not installed)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Verify Metrics Server
kubectl top nodes
kubectl top pods

# Create HPA
kubectl apply -f hpa.yaml

# Check HPA status
kubectl get hpa
kubectl describe hpa nginx-hpa

# Generate load to test autoscaling
kubectl run -it --rm load-generator --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://nginx-service; done"

# Watch HPA scale up
kubectl get hpa -w
kubectl get pods -l app=nginx -w

# Stop load generator (Ctrl+C) and watch scale down
```

### 3. Health Checks with Probes

```bash
# Deploy app with probes
kubectl apply -f deployment-with-probes.yaml

# Check probe status
kubectl describe pod <pod-name>

# Watch probe failures (in separate terminal)
kubectl get events --watch

# Test liveness probe failure (will restart container)
kubectl exec <pod-name> -- rm /tmp/healthy

# Test readiness probe failure (removes from service)
kubectl get endpoints webapp-service
```

### 4. Resource Management

```bash
# Deploy with resources
kubectl apply -f deployment-with-resources.yaml

# Check QoS class
kubectl get pod qos-guaranteed -o yaml | grep qosClass
kubectl get pod qos-burstable -o yaml | grep qosClass
kubectl get pod qos-besteffort -o yaml | grep qosClass

# Monitor resource usage
kubectl top pod qos-guaranteed
kubectl top pod qos-burstable

# Apply LimitRange and ResourceQuota
kubectl apply -f deployment-with-resources.yaml
kubectl describe limitrange resource-limits
kubectl describe resourcequota namespace-quota
```

### 5. Rolling Updates and Rollbacks

```bash
# Deploy initial version
kubectl apply -f deployment-rolling-update.yaml

# Check rollout status
kubectl rollout status deployment rolling-update-demo

# View rollout history
kubectl rollout history deployment rolling-update-demo

# Update image (simulate new version)
kubectl set image deployment/rolling-update-demo nginx=nginx:1.24 --record

# Watch rolling update
kubectl rollout status deployment rolling-update-demo
kubectl get pods -w

# Check history
kubectl rollout history deployment rolling-update-demo

# Rollback to previous version
kubectl rollout undo deployment rolling-update-demo

# Rollback to specific revision
kubectl rollout undo deployment rolling-update-demo --to-revision=1

# Pause/Resume rollout
kubectl rollout pause deployment rolling-update-demo
kubectl rollout resume deployment rolling-update-demo
```

### 6. DaemonSets

```bash
# Deploy DaemonSet
kubectl apply -f daemonset-example.yaml

# Verify one pod per node
kubectl get daemonset node-exporter
kubectl get pods -l app=node-exporter -o wide

# Check which nodes have the pod
kubectl get nodes
kubectl get pods -o wide | grep node-exporter

# Update DaemonSet
kubectl set image daemonset/node-exporter node-exporter=prom/node-exporter:v1.6.0
kubectl rollout status daemonset node-exporter
```

### 7. StatefulSets

```bash
# Deploy MySQL StatefulSet
kubectl apply -f statefulset-mysql.yaml

# Watch sequential creation
kubectl get pods -w

# Check Pod names (ordinal: mysql-0, mysql-1, mysql-2)
kubectl get pods -l app=mysql

# Check PVCs (one per pod)
kubectl get pvc

# Check stable DNS names
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup mysql-0.mysql

# Scale StatefulSet
kubectl scale statefulset mysql --replicas=5

# Watch sequential scaling
kubectl get pods -w

# Delete a pod (will recreate with same name and PVC)
kubectl delete pod mysql-1
kubectl get pods -w

# Access specific pod
kubectl exec -it mysql-0 -- mysql -u root -prootpassword
```

## 🔧 Useful Commands

### Scaling

```bash
# Manual scale
kubectl scale deployment myapp --replicas=10

# Autoscale (create HPA)
kubectl autoscale deployment myapp --cpu-percent=70 --min=2 --max=10

# View HPA
kubectl get hpa
kubectl describe hpa myapp

# Edit HPA
kubectl edit hpa myapp

# Delete HPA
kubectl delete hpa myapp
```

### Probes

```bash
# Check probe configuration
kubectl describe pod <pod-name> | grep -A 10 "Liveness\|Readiness\|Startup"

# View probe events
kubectl get events --field-selector involvedObject.name=<pod-name>

# Test endpoint manually
kubectl exec <pod-name> -- wget -O- http://localhost:8080/health
```

### Resources

```bash
# View resource usage
kubectl top nodes
kubectl top pods
kubectl top pods --all-namespaces

# Check resource requests/limits
kubectl describe pod <pod-name> | grep -A 5 "Limits\|Requests"

# View QoS class
kubectl get pod <pod-name> -o jsonpath='{.status.qosClass}'
```

### Updates & Rollbacks

```bash
# Update image
kubectl set image deployment/myapp container-name=image:tag --record

# View rollout status
kubectl rollout status deployment myapp

# View history
kubectl rollout history deployment myapp
kubectl rollout history deployment myapp --revision=2

# Rollback
kubectl rollout undo deployment myapp
kubectl rollout undo deployment myapp --to-revision=3

# Pause/Resume
kubectl rollout pause deployment myapp
kubectl rollout resume deployment myapp

# Restart deployment (recreate all pods)
kubectl rollout restart deployment myapp
```

## 🧪 Practice Exercises

### Exercise 1: Manual Scaling
1. Deploy nginx with 3 replicas
2. Scale to 7 replicas
3. Scale back to 2 replicas
4. Observe pod creation and deletion

### Exercise 2: Horizontal Pod Autoscaler
1. Deploy app with resource requests
2. Create HPA targeting 50% CPU
3. Generate load and watch scaling
4. Stop load and watch scale down

### Exercise 3: Probe Testing
1. Deploy app with all three probes
2. Trigger liveness probe failure (container restart)
3. Trigger readiness probe failure (remove from endpoints)
4. Observe startup probe delaying other probes

### Exercise 4: Resource Limits
1. Create pods with different QoS classes
2. Apply LimitRange to namespace
3. Apply ResourceQuota to namespace
4. Try exceeding quota

### Exercise 5: Rolling Update
1. Deploy app with v1 image
2. Update to v2 with rolling update
3. Watch gradual replacement
4. Rollback to v1

### Exercise 6: DaemonSet
1. Deploy monitoring agent as DaemonSet
2. Add new node and verify pod creation
3. Use node selector to limit nodes
4. Update DaemonSet image

### Exercise 7: StatefulSet
1. Deploy StatefulSet database
2. Verify stable names and storage
3. Delete a pod and verify recreation
4. Scale up and down

## 📊 Comparison Tables

### Probe Types

| Probe Type | Purpose                    | Action on Failure      | When to Use                  |
|------------|----------------------------|------------------------|------------------------------|
| Liveness   | Is container alive?        | Restart container      | Detect deadlocks, hung apps  |
| Readiness  | Ready for traffic?         | Remove from endpoints  | Warm-up, dependency checks   |
| Startup    | Has container started?     | Wait before other checks | Slow-starting legacy apps    |

### QoS Classes

| Class      | Condition               | Eviction Priority | Use Case                |
|------------|-------------------------|-------------------|-------------------------|
| Guaranteed | requests = limits       | Lowest (last)     | Critical workloads      |
| Burstable  | Some requests/limits    | Medium            | Most applications       |
| BestEffort | No requests/limits      | Highest (first)   | Non-critical batch jobs |

### Workload Types

| Type        | Use Case                | Pod Names    | Storage    | Order      |
|-------------|-------------------------|--------------|------------|------------|
| Deployment  | Stateless apps          | Random       | Shared     | Parallel   |
| StatefulSet | Stateful apps           | Ordered      | Individual | Sequential |
| DaemonSet   | Node agents             | Per-node     | Shared     | Per-node   |
| Job         | Run-to-completion       | Random       | Shared     | Parallel   |
| CronJob     | Scheduled tasks         | Random       | Shared     | Scheduled  |

## ⚠️ Common Pitfalls

1. **HPA without resource requests** - HPA needs CPU/memory requests to calculate utilization
2. **Too aggressive probes** - Causes restart loops
3. **Same liveness/readiness endpoint** - Should be different
4. **No startup probe for slow apps** - Liveness kills slow-starting containers
5. **Limits = requests unnecessarily** - Wastes burst capacity
6. **No PodDisruptionBudget** - Node drains can take down too many pods
7. **Forgetting minReadySeconds** - Pods considered ready too fast
8. **StatefulSet without headless service** - No stable DNS names

## 🎓 Interview Questions to Practice

1. Explain the difference between HPA and VPA
2. When would you use a startup probe?
3. What happens when a pod exceeds memory limit vs CPU limit?
4. How does rolling update work? What are maxSurge and maxUnavailable?
5. When should you use a StatefulSet vs Deployment?
6. What is a Pod Disruption Budget and why is it important?
7. Explain QoS classes and eviction order
8. How do you troubleshoot a failing liveness probe?
9. What is the difference between DaemonSet and Deployment replicas?
10. How do you perform zero-downtime deployments?

## 📚 Additional Resources

- [HPA Documentation](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [Configure Liveness, Readiness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [Managing Resources](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

## ✅ Week 3 Checklist

- [ ] Understand manual and automatic scaling
- [ ] Configure HPA with CPU/memory metrics
- [ ] Implement liveness, readiness, and startup probes
- [ ] Set appropriate resource requests and limits
- [ ] Understand QoS classes
- [ ] Perform rolling updates and rollbacks
- [ ] Work with Pod Disruption Budgets
- [ ] Deploy and manage DaemonSets
- [ ] Deploy and manage StatefulSets
- [ ] Complete all practice exercises

---

**Next:** Week 4 - Ingress, Networking, and Load Balancing 🚀

