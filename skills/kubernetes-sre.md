# Kubernetes SRE

## Overview

This skill covers Site Reliability Engineering (SRE) practices for Kubernetes, including daily inspection tasks, troubleshooting, monitoring, and incident response capabilities.

## Daily Inspection Tasks

### Cluster Health Checks

```bash
# Check cluster component status
kubectl get componentstatuses
kubectl get --raw /healthz
kubectl get --raw /livez
kubectl get --raw /readyz

# Check node status and resources
kubectl get nodes
kubectl top nodes
kubectl describe nodes

# Check critical system pods
kubectl get pods -n kube-system
kubectl get pods --all-namespaces --field-selector=status.phase!=Running

# Check API server responsiveness
kubectl cluster-info
kubectl version
```

### Resource Monitoring

```bash
# Monitor resource usage
kubectl top nodes
kubectl top pods --all-namespaces

# Check resource quotas and limits
kubectl get resourcequotas --all-namespaces
kubectl get limitranges --all-namespaces

# Identify pods with high resource usage
kubectl top pods --all-namespaces --sort-by=cpu
kubectl top pods --all-namespaces --sort-by=memory

# Check persistent volume status
kubectl get pv
kubectl get pvc --all-namespaces

# Monitor disk usage on nodes (requires metrics-server)
kubectl describe nodes | grep -A 5 "Allocated resources"
```

### Pod and Workload Health

```bash
# Check pod status across all namespaces
kubectl get pods --all-namespaces
kubectl get pods --all-namespaces --field-selector=status.phase=Pending
kubectl get pods --all-namespaces --field-selector=status.phase=Failed

# Check restarting pods
kubectl get pods --all-namespaces --sort-by='.status.containerStatuses[0].restartCount'

# Identify pods in CrashLoopBackOff
kubectl get pods --all-namespaces | grep CrashLoopBackOff

# Check deployment rollout status
kubectl get deployments --all-namespaces
kubectl rollout status deployment/<deployment-name> -n <namespace>

# Check for evicted pods
kubectl get pods --all-namespaces --field-selector=status.phase=Failed | grep Evicted
```

### Network Health

```bash
# Check services and endpoints
kubectl get services --all-namespaces
kubectl get endpoints --all-namespaces

# Verify DNS functionality
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup kubernetes.default

# Check network policies
kubectl get networkpolicies --all-namespaces

# Check ingress status
kubectl get ingress --all-namespaces
```

### Events and Logs

```bash
# Check recent cluster events
kubectl get events --all-namespaces --sort-by='.lastTimestamp'
kubectl get events --all-namespaces --field-selector type=Warning

# View recent events for a specific namespace
kubectl get events -n <namespace> --sort-by='.lastTimestamp'

# Check control plane logs (if accessible)
kubectl logs -n kube-system -l component=kube-apiserver
kubectl logs -n kube-system -l component=kube-scheduler
kubectl logs -n kube-system -l component=kube-controller-manager
```

## Troubleshooting and Incident Response

### Common Issues and Diagnostics

#### Pod Not Starting

```bash
# Get detailed pod information
kubectl describe pod <pod-name> -n <namespace>

# Check events
kubectl get events -n <namespace> --field-selector involvedObject.name=<pod-name>

# Check logs
kubectl logs <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --previous  # Previous container instance

# Check image pull issues
kubectl get events -n <namespace> | grep "Failed to pull image"

# Verify resource availability
kubectl describe nodes
```

#### CrashLoopBackOff

```bash
# Check logs from the failing container
kubectl logs <pod-name> -n <namespace> --previous

# Inspect container status
kubectl describe pod <pod-name> -n <namespace>

# Check liveness/readiness probes
kubectl get pod <pod-name> -n <namespace> -o yaml | grep -A 10 "livenessProbe"

# Debug with a shell (if possible)
kubectl exec -it <pod-name> -n <namespace> -- /bin/sh
```

#### Node Issues

```bash
# Check node conditions
kubectl describe node <node-name>

# Look for NotReady nodes
kubectl get nodes | grep NotReady

# Check node pressure conditions
kubectl describe nodes | grep -i "pressure\|taints"

# Drain node for maintenance
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# Uncordon node after maintenance
kubectl uncordon <node-name>
```

#### Resource Exhaustion

```bash
# Check pods pending due to insufficient resources
kubectl describe pod <pod-name> -n <namespace> | grep "Insufficient"

# View cluster capacity
kubectl describe nodes | grep -A 5 "Allocated resources"

# Identify resource hogs
kubectl top pods --all-namespaces --sort-by=memory
kubectl top pods --all-namespaces --sort-by=cpu
```

#### Networking Issues

```bash
# Test pod-to-pod connectivity
kubectl run -it --rm debug --image=nicolaka/netshoot --restart=Never -- bash

# Check service endpoints
kubectl get endpoints <service-name> -n <namespace>

# Verify DNS resolution
kubectl exec -it <pod-name> -n <namespace> -- nslookup <service-name>

# Check network policies
kubectl describe networkpolicy <policy-name> -n <namespace>
```

### Debugging Tools and Techniques

```bash
# Run a debug pod
kubectl run debug --image=nicolaka/netshoot -it --rm --restart=Never -- bash

# Copy files from/to pods
kubectl cp <namespace>/<pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <namespace>/<pod-name>:/path/to/file

# Port-forward for local access
kubectl port-forward pod/<pod-name> 8080:80 -n <namespace>

# Execute commands in pods
kubectl exec <pod-name> -n <namespace> -- <command>

# Attach to a running container
kubectl attach <pod-name> -n <namespace> -c <container-name>

# Check resource definitions
kubectl get <resource-type> <resource-name> -n <namespace> -o yaml
kubectl get <resource-type> <resource-name> -n <namespace> -o json
```

## Observability and Monitoring

### Metrics Collection

```bash
# Install metrics-server (if not already installed)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# View cluster metrics
kubectl top nodes
kubectl top pods --all-namespaces

# Custom metrics (requires custom metrics API)
kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1
```

### Log Aggregation

```bash
# Stream logs from multiple pods
kubectl logs -f -l app=<label-value> -n <namespace>

# Get logs from all containers in a pod
kubectl logs <pod-name> -n <namespace> --all-containers=true

# Get logs with timestamps
kubectl logs <pod-name> -n <namespace> --timestamps=true

# Get recent logs
kubectl logs <pod-name> -n <namespace> --since=1h
kubectl logs <pod-name> -n <namespace> --tail=100
```

### Audit and Compliance

```bash
# Check RBAC configuration
kubectl get roles --all-namespaces
kubectl get rolebindings --all-namespaces
kubectl get clusterroles
kubectl get clusterrolebindings

# Audit service accounts
kubectl get serviceaccounts --all-namespaces

# Check security contexts
kubectl get pods --all-namespaces -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.securityContext}{"\n"}{end}'

# Verify pod security standards
kubectl label --dry-run=server --overwrite ns <namespace> pod-security.kubernetes.io/enforce=restricted
```

## Alerting and On-Call Procedures

### Alert Categories

#### Critical Alerts
- API server down or unreachable
- Nodes in NotReady state
- etcd cluster unhealthy
- Critical system pods down (kube-scheduler, kube-controller-manager)
- Cluster-wide resource exhaustion

#### High Priority Alerts
- Multiple pod failures in production namespace
- Persistent volume failures
- Certificate expiration warnings
- High error rates in applications
- Memory/CPU saturation on nodes

#### Medium Priority Alerts
- Pod restarts exceeding threshold
- Deployment rollout failures
- Slow API response times
- Persistent volume near capacity

### Response Procedures

#### Incident Response Workflow

1. **Acknowledge**: Confirm receipt of alert
2. **Assess**: Determine scope and impact
   ```bash
   kubectl get nodes
   kubectl get pods --all-namespaces
   kubectl get events --all-namespaces --sort-by='.lastTimestamp' | head -20
   ```
3. **Triage**: Identify root cause
4. **Mitigate**: Apply immediate fixes
5. **Communicate**: Update stakeholders
6. **Resolve**: Implement permanent solution
7. **Document**: Record incident details and learnings

#### Emergency Commands

```bash
# Scale deployment quickly
kubectl scale deployment <deployment-name> -n <namespace> --replicas=<count>

# Rollback problematic deployment
kubectl rollout undo deployment/<deployment-name> -n <namespace>

# Delete failed pods
kubectl delete pod <pod-name> -n <namespace> --force --grace-period=0

# Cordon node to prevent new pods
kubectl cordon <node-name>

# Emergency pod eviction
kubectl drain <node-name> --force --ignore-daemonsets

# Restart deployment
kubectl rollout restart deployment/<deployment-name> -n <namespace>
```

## Performance Optimization

### Resource Management

```bash
# Analyze resource requests and limits
kubectl describe nodes | grep -A 5 "Allocated resources"

# Check for over-committed resources
kubectl get pods --all-namespaces -o json | jq '.items[] | {name: .metadata.name, namespace: .metadata.namespace, requests: .spec.containers[].resources.requests, limits: .spec.containers[].resources.limits}'

# Identify pods without resource limits
kubectl get pods --all-namespaces -o json | jq -r '.items[] | select(.spec.containers[].resources.limits == null) | "\(.metadata.namespace)/\(.metadata.name)"'
```

### Cost Optimization

```bash
# Find unused PVCs
kubectl get pvc --all-namespaces -o json | jq -r '.items[] | select(.status.phase=="Bound") | select(.spec.volumeName) | "\(.metadata.namespace)/\(.metadata.name)"'

# Identify idle workloads (requires metrics-server)
kubectl top pods --all-namespaces | awk '{if (NR>1 && $3+0 < 1) print $0}'

# Check for terminated pods not cleaned up
kubectl get pods --all-namespaces --field-selector=status.phase=Succeeded
kubectl get pods --all-namespaces --field-selector=status.phase=Failed
```

## Capacity Planning

### Growth Analysis

```bash
# Track resource usage trends over time
kubectl top nodes --sort-by=memory
kubectl top nodes --sort-by=cpu

# Project future needs based on current usage
kubectl describe nodes | grep -A 5 "Allocated resources"

# Identify scaling bottlenecks
kubectl get hpa --all-namespaces
kubectl describe hpa <hpa-name> -n <namespace>
```

### Cluster Scaling

```bash
# Check current cluster autoscaler status (if enabled)
kubectl get configmap cluster-autoscaler-status -n kube-system -o yaml

# Manual node addition considerations
# - Verify node capacity matches workload requirements
# - Check node labels and taints
# - Ensure proper networking configuration

# Plan for pod distribution
kubectl get pods --all-namespaces -o wide | awk '{print $8}' | sort | uniq -c
```

## Best Practices for SRE

### Regular Maintenance Tasks

1. **Certificate Management**
   - Monitor certificate expiration dates
   - Automate certificate rotation
   - Verify certificate chains

2. **Backup and Recovery**
   - Regular etcd backups
   - Test restore procedures
   - Document recovery runbooks

3. **Version Management**
   - Track Kubernetes version updates
   - Plan upgrade windows
   - Test upgrades in staging

4. **Security Hygiene**
   - Regular security audits
   - Update container images
   - Review RBAC permissions

### Automation Opportunities

- Automated health checks and reporting
- Self-healing mechanisms with operators
- Automated scaling based on metrics
- Continuous compliance validation
- Proactive alerting based on trends

### Documentation and Runbooks

- Maintain up-to-date runbooks for common incidents
- Document cluster architecture and dependencies
- Keep contact lists current
- Record post-incident reviews
- Share knowledge across team

## Tools and Integrations

### Essential SRE Tools

- **kubectl**: Primary CLI tool
- **k9s**: Terminal UI for cluster management
- **stern**: Multi-pod log tailing
- **kubectx/kubens**: Context and namespace switching
- **kube-ps1**: Kubectl prompt info
- **popeye**: Cluster sanitizer
- **kube-bench**: Security benchmark
- **trivy**: Vulnerability scanner

### Monitoring Solutions

- Prometheus + Grafana
- ELK/EFK Stack
- Datadog
- New Relic
- Dynatrace

### GitOps and IaC

- Flux/ArgoCD for continuous deployment
- Terraform for infrastructure provisioning
- Helm for application packaging
- Kustomize for configuration management

## Emergency Contacts and Escalation

### Escalation Paths

1. **Level 1**: On-call SRE (initial response)
2. **Level 2**: Senior SRE/Platform team
3. **Level 3**: Engineering leads and architects
4. **Level 4**: Vendor support (for managed services)

### Communication Channels

- Incident management platform (PagerDuty, Opsgenie)
- Team chat (Slack, Teams)
- Status page updates
- Stakeholder notifications

## Useful Resources

- Kubernetes Official Documentation: https://kubernetes.io/docs/
- SRE Book: https://sre.google/books/
- Kubernetes Best Practices: https://kubernetes.io/docs/concepts/configuration/overview/
- CNCF Landscape: https://landscape.cncf.io/
