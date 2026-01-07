# Kubernetes Networking

## Service Types

### ClusterIP (Default)
- Internal IP address within cluster
- Only accessible from within cluster
- Use for inter-service communication

### NodePort
- Exposes service on each Node's IP at static port
- Accessible from outside cluster
- Port range: 30000-32767

### LoadBalancer
- Creates external load balancer (cloud provider)
- Automatically creates NodePort and ClusterIP
- Use for public-facing services

### ExternalName
- Maps service to external DNS name
- Returns CNAME record
- No proxy created

## Service Configuration

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
```

## Ingress

- HTTP/HTTPS routing to services
- SSL/TLS termination
- Name-based virtual hosting
- Path-based routing

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
  tls:
  - hosts:
    - example.com
    secretName: example-tls
```

## Network Policies

- Controls traffic flow between Pods
- Pod-to-Pod communication rules
- Namespace isolation

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-network-policy
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
```

## DNS

- Service DNS: `<service-name>.<namespace>.svc.cluster.local`
- Pod DNS: `<pod-ip>.<namespace>.pod.cluster.local`
- CoreDNS handles DNS resolution

