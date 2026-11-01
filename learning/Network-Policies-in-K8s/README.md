# Network Policies in Kubernetes

This comprehensive guide explores Kubernetes Network Policies, which provide fine-grained control over how pods communicate with each other and other network endpoints.

## Understanding Network Policies

Network Policies are application-centric constructs that specify how groups of pods are allowed to communicate with each other and with other network endpoints. They are crucial for:

- Implementing zero-trust security models
- Enforcing micro-segmentation
- Controlling both ingress and egress traffic
- Protecting sensitive workloads

### Key Concepts

1. **Pod Selectors**: Define which pods the policy applies to
2. **Namespace Selectors**: Control traffic across namespaces
3. **CIDR Blocks**: Specify IP ranges for traffic control
4. **Port Definitions**: Control traffic on specific ports
5. **Rule Types**: Ingress (incoming) and Egress (outgoing) rules

## What you'll learn
- Understanding Network Policies
  - Basic concepts and architecture
  - Policy types and selectors
  - Rule evaluation and precedence
- Implementing pod isolation
  - Default deny policies
  - Allowlist approach
  - Namespace isolation
- Configuring ingress and egress rules
  - Port-based rules
  - Protocol-specific policies
  - CIDR-based rules
- Namespace-based network policies
  - Cross-namespace communication
  - Namespace isolation patterns
- Label-based network policies
  - Pod selection strategies
  - Dynamic policy application
  - Policy composition

## Prerequisites
- Kubernetes cluster with Network Policy support
  - Calico
  - Cilium
  - Antrea
  - WeaveNet
- kubectl CLI tool with admin access
- Basic understanding of:
  - Kubernetes networking
  - Pod-to-Pod communication
  - Service networking
  - CNI concepts

> ⚠️ **Common trap:** the default CNI of Minikube/kind **does not enforce** NetworkPolicies — pods apply them but traffic is not blocked. For Minikube start the cluster with `minikube start --cni=calico`; for kind install a CNI like Calico or Cilium afterwards. Verify with the test in the Implementation Guide below.

## Files in this folder

| File | Purpose |
|------|---------|
| `default-deny.yaml` | Baseline: deny all ingress + egress in the namespace |
| `allow-frontend.yaml` | Only `app: frontend` pods may reach `app: backend` pods on port 80 |
| `allow-database.yaml` | Only `app: backend` pods may reach `app: database` pods on port 5432 |

## Examples included

### 1. Default Deny All Traffic
```yaml
# default-deny.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

### 2. Allow Specific Pod-to-Pod Communication
```yaml
# allow-frontend.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

### 3. Namespace Isolation
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: namespace-isolation
spec:
  podSelector: {}
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          environment: production
```

## Best Practices

1. **Start with Default Deny**
   - Implement default deny policies first
   - Add specific allow rules as needed
   - Document all policy decisions

2. **Use Labels Effectively**
   - Design a consistent labeling strategy
   - Use role-based labels
   - Maintain label documentation

3. **Policy Organization**
   - Group policies by namespace
   - Use clear naming conventions
   - Implement change control

4. **Testing and Validation**
   - Test policies in non-production first
   - Use network policy validators
   - Maintain test cases

## Implementation Guide

1. **Prepare Your Environment**
```bash
# Deploy test pods (frontend, backend, database) with the labels used by the policies
kubectl run frontend --image=nginx:1.25-alpine -l app=frontend
kubectl run backend  --image=nginx:1.25-alpine -l app=backend
kubectl run database --image=nginx:1.25-alpine -l app=database

# Verify a CNI that enforces policies is installed
kubectl get pods -n kube-system | grep -Ei "calico|cilium|antrea|weave"
```

2. **Apply Base Policies**
```bash
# Apply default deny policy
kubectl apply -f default-deny.yaml

# Apply frontend access policy
kubectl apply -f allow-frontend.yaml

# Apply database access policy
kubectl apply -f allow-database.yaml
```

3. **Verify Policy Enforcement**
```bash
# frontend -> backend on port 80: should SUCCEED (allowed by allow-frontend.yaml)
kubectl exec frontend -- wget -qO- --timeout=5 http://backend:80

# frontend -> database on port 5432: should FAIL (no policy allows it)
kubectl exec frontend -- wget -qO- --timeout=5 http://database:5432

# Inspect the policies
kubectl describe networkpolicy allow-frontend
kubectl get networkpolicies -A
```

> 💡 Use `wget`/`curl` (TCP) rather than `ping` (ICMP) to test — ICMP traffic doesn't match the port-based rules used in these examples, so a successful ping proves nothing about HTTP access.

## Troubleshooting

### Common Issues

1. **Policy Not Applied**
   - Check CNI plugin status
   - Verify label selectors
   - Validate policy syntax

2. **Unexpected Blocking**
   - Review policy order
   - Check namespace selectors
   - Validate pod labels

3. **Performance Impact**
   - Monitor policy evaluation
   - Optimize selector usage
   - Consider policy aggregation

### Debugging Tools
```bash
# View applied policies
kubectl get networkpolicies -A

# Check pod labels
kubectl get pods --show-labels

# Validate connectivity
kubectl exec -it debugger -- ping target-pod
```

## Advanced Topics

1. **Egress Control**
   - External service access
   - CIDR-based rules
   - **DNS gotcha:** once you add a default-deny **egress** policy, pods can no longer resolve names! Always pair it with a DNS-allow policy:
```yaml
# allow-dns.yaml — required whenever default-deny egress is in place
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector: {}          # kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

2. **Multi-cluster Policies**
   - Cross-cluster communication
   - Federation considerations
   - Service mesh integration

3. **Policy Metrics**
   - Monitoring policy effectiveness
   - Traffic analysis
   - Compliance reporting

## Additional Resources

- [Official Kubernetes NetworkPolicy Documentation](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [Calico Network Policy Guide](https://docs.tigera.io/calico/latest/security/kubernetes-network-policy)
- [Network Policy Recipes](https://github.com/ahmetb/kubernetes-network-policy-recipes)
- [Cilium Network Policy Editor](https://editor.cilium.io/) 