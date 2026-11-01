# Role-Based Access Control (RBAC) in Kubernetes

This comprehensive guide explores Kubernetes RBAC, the native authorization mechanism used to regulate access to Kubernetes cluster resources.

## Understanding RBAC

RBAC is a method of regulating access to computer or network resources based on the roles of individual users. In Kubernetes, RBAC:

- Defines what actions users can perform
- Controls access to cluster resources
- Implements principle of least privilege
- Provides fine-grained access control

### Core Concepts

1. **Rules**: Define what operations (verbs) can be performed on which resources
2. **Roles**: Collection of rules that apply within a namespace
3. **ClusterRoles**: Similar to Roles but apply cluster-wide
4. **Bindings**: Link roles to users, groups, or service accounts

## What you'll learn
- Understanding RBAC components
  - Rules and permissions
  - Role definitions
  - Binding mechanisms
  - Scope and inheritance
- Creating and managing Roles and ClusterRoles
  - Namespace-scoped roles
  - Cluster-wide permissions
  - Aggregated roles
  - Default roles
- Working with RoleBindings and ClusterRoleBindings
  - User bindings
  - Group bindings
  - ServiceAccount bindings
  - Cross-namespace bindings
- Service Account configuration
  - Creating service accounts
  - Token management
  - Pod association
  - Secret automation
- Best practices for RBAC implementation
  - Least privilege principle
  - Role composition
  - Binding strategies
  - Security considerations

## Files in this folder

| File | Purpose |
|------|---------|
| `service-account.yaml` | ServiceAccount the workload will run as |
| `role.yaml` | `pod-manager` Role — full CRUD on pods + read pod logs (namespace: default) |
| `role-binding.yaml` | Binds `pod-manager` to the ServiceAccount |
| `pod-with-sa.yaml` | Pod running with the custom ServiceAccount |

## Prerequisites
- Kubernetes cluster with RBAC enabled
  - Most clusters enable it by default
  - Check with `kubectl api-versions | grep rbac.authorization.k8s.io`
- kubectl CLI tool with admin access
- Basic understanding of:
  - Kubernetes API resources
  - Authentication mechanisms
  - Security concepts

## Examples included

### 1. Creating Service Accounts
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
  namespace: default
```

### 2. Defining Roles
```yaml
# Example of a pod-reader role (read-only). role.yaml in this folder contains a
# "pod-manager" variant with full CRUD on pods instead.
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

### 3. Setting up RoleBindings
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: app-service-account
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## Best Practices

### 1. Role Design
- Follow least privilege principle
- Group related permissions
- Use descriptive names
- Document role purposes

### 2. Binding Strategy
- Prefer namespace isolation
- Use groups for user management
- Limit cluster-wide access
- Regular access review

### 3. Service Accounts
- One service account per application
- Since Kubernetes 1.24, pods receive short-lived, auto-rotated **projected tokens** (no more permanent token Secrets)
- Set `automountServiceAccountToken: false` when the app never talks to the API
- Apply [Pod Security Admission](https://kubernetes.io/docs/concepts/security/pod-security-admission/) standards to restrict what pods can do (PodSecurityPolicy was removed in Kubernetes 1.25)

### 4. Security Considerations
- Regular audit of permissions
- Remove unused roles
- Monitor binding changes
- Implement detection controls

## Implementation Guide

### 1. Create Required Objects
```bash
# All from this folder:
kubectl apply -f service-account.yaml
kubectl apply -f role.yaml
kubectl apply -f role-binding.yaml

# Deploy pod with service account
kubectl apply -f pod-with-sa.yaml
```

### 2. Verify Setup
```bash
# Check service account
kubectl get serviceaccount app-service-account

# Verify role
kubectl get role pod-manager

# Check bindings
kubectl get rolebinding manage-pods

# Test permissions (impersonation)
kubectl auth can-i list pods --as system:serviceaccount:default:app-service-account   # yes
kubectl auth can-i delete nodes --as system:serviceaccount:default:app-service-account # no

# Full permission dump for the identity
kubectl auth can-i --list --as system:serviceaccount:default:app-service-account
```

## Troubleshooting

### Common Issues

1. **Permission Denied**
   - Verify role rules
   - Check binding configuration
   - Validate service account
   - Review API groups

2. **Role Not Applied**
   - Check namespace context
   - Verify binding subject
   - Validate role reference
   - Review aggregation labels

3. **Service Account Issues**
   - Check token mounting
   - Verify secret creation
   - Validate pod association
   - Review automount settings

### Debugging Commands
```bash
# Check effective permissions
kubectl auth can-i --list

# View role details
kubectl describe role role-name

# Check binding
kubectl describe rolebinding binding-name

# Validate service account
kubectl describe sa service-account-name
```

## Advanced Topics

### 1. Aggregated ClusterRoles
- Combining multiple roles
- Dynamic role updates
- Custom resource permissions
- Extension patterns

### 2. External Authentication
- Integration with external systems
- Token management
- Certificate handling
- Authentication plugins

### 3. Audit Logging
- Enabling audit logs
- Policy configuration
- Log analysis
- Compliance reporting

## Additional Resources

- [Official Kubernetes RBAC Documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [RBAC Good Practices](https://kubernetes.io/docs/concepts/security/rbac-good-practices/)
- [Authentication Overview](https://kubernetes.io/docs/concepts/security/authentication/)
- [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/overview/) 