# Kubernetes Security

## Service Accounts

- Identity for Pods
- Default service account per namespace
- Used for API authentication

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
```

## RBAC (Role-Based Access Control)

### Role
- Namespace-scoped permissions
- Defines what actions can be performed

### ClusterRole
- Cluster-scoped permissions
- Applies to all namespaces

### RoleBinding
- Grants Role to users/groups/service accounts
- Namespace-scoped

### ClusterRoleBinding
- Grants ClusterRole to users/groups/service accounts
- Cluster-scoped

## RBAC Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: ServiceAccount
  name: my-service-account
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## Secrets

- Store sensitive data
- Base64 encoded (not encrypted)
- Types: Opaque, kubernetes.io/tls, kubernetes.io/dockerconfigjson

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded
  password: cGFzc3dvcmQ=
```

## Using Secrets

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: username
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secret
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: my-secret
```

## Security Context

- Pod-level and container-level settings
- User ID, group ID
- Capabilities, SELinux, AppArmor

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: sec-ctx-demo
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
        add:
        - NET_BIND_SERVICE
```

## Pod Security Standards

- **Privileged**: Unrestricted
- **Baseline**: Minimally restrictive
- **Restricted**: Hardened

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

## Network Policies

- Control network traffic
- Default deny or allow
- Ingress and egress rules

## Image Security

- Use trusted registries
- Scan images for vulnerabilities
- Use specific tags (avoid :latest)
- Implement image pull policies

