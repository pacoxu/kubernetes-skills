# Kubernetes Basics

## Core Concepts

### Pods
- Smallest deployable unit in Kubernetes
- Contains one or more containers
- Shares network and storage resources
- Ephemeral by nature

### Deployments
- Manages Pod replicas
- Provides declarative updates
- Handles rollouts and rollbacks
- Ensures desired state

### Services
- Stable network endpoint for Pods
- Load balancing across Pods
- Service discovery mechanism
- Types: ClusterIP, NodePort, LoadBalancer, ExternalName

### Namespaces
- Virtual clusters within a physical cluster
- Resource isolation and organization
- Default namespaces: default, kube-system, kube-public

## Common Commands

```bash
# Get resources
kubectl get pods
kubectl get deployments
kubectl get services

# Describe resources
kubectl describe pod <pod-name>
kubectl describe deployment <deployment-name>

# Create resources
kubectl create -f <manifest.yaml>
kubectl apply -f <manifest.yaml>

# Delete resources
kubectl delete pod <pod-name>
kubectl delete -f <manifest.yaml>

# Logs and debugging
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- /bin/sh
```

## Basic YAML Structure

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

