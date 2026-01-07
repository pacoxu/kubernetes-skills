# Kubernetes Deployment

## Deployment Strategies

### Rolling Update (Default)
- Gradually replaces old Pods with new ones
- Zero downtime
- Configurable with `maxSurge` and `maxUnavailable`

### Recreate
- Terminates all old Pods before creating new ones
- Brief downtime
- Use for stateful applications that can't run multiple versions

### Blue/Green
- Deploy new version alongside old
- Switch traffic when ready
- Requires external load balancer

### Canary
- Deploy new version to subset of users
- Gradually increase traffic
- Rollback if issues detected

## Deployment Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app:v1.0.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
```

## Health Checks

### Liveness Probe
- Determines if container is running
- Restarts container if fails
- Use for detecting deadlocks

### Readiness Probe
- Determines if container is ready to serve traffic
- Removes Pod from Service endpoints if fails
- Use for startup dependencies

### Startup Probe
- Used for slow-starting containers
- Disables liveness/readiness until succeeds
- Prevents premature restarts

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

## Scaling

```bash
# Manual scaling
kubectl scale deployment my-app --replicas=5
```

```yaml
# Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

