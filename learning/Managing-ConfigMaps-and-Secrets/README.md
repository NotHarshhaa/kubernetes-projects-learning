# Managing ConfigMaps and Secrets in Kubernetes

This guide provides a comprehensive walkthrough of ConfigMaps and Secrets in Kubernetes, essential components for managing application configuration and sensitive data.

## What are ConfigMaps and Secrets?

### ConfigMaps
ConfigMaps are API objects that store non-confidential data in key-value pairs. They allow you to decouple environment-specific configuration from your container images, making your applications more portable.

Use cases for ConfigMaps:
- Application configuration files
- Command-line arguments
- Environment variables
- Port numbers
- Feature flags

### Secrets
Secrets are similar to ConfigMaps but are specifically designed to hold sensitive information such as:
- API tokens
- TLS certificates
- SSH keys
- Database credentials

## What you'll learn
- Creating and managing ConfigMaps
  - Using literal values
  - From configuration files
  - From directories
- Injecting configuration data into Pods
  - As environment variables (`valueFrom` and `envFrom`)
  - As configuration files (volume mounts)
  - As command-line arguments
- Working with Secrets
  - Creating secrets (`stringData` vs base64 `data`)
  - Mounting secrets in pods
  - Understanding how secret updates propagate
- Best practices for configuration management
  - Versioning configurations
  - Namespace isolation
  - Access control

> ⚠️ **Important:** Secrets are *base64-encoded, not encrypted*. Anyone with read access can decode them. Real protection = RBAC + encryption-at-rest + tools like [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) or [External Secrets Operator](https://external-secrets.io/).

## Files in this folder

| File | Purpose |
|------|---------|
| `configmap-literal.yaml` | ConfigMap with individual key-value pairs |
| `configmap-file.yaml` | ConfigMap holding a whole properties file as one key |
| `secret.yaml` | Opaque Secret with database credentials |
| `pod-with-config.yaml` | Pod consuming config as env vars, `envFrom`, and mounted files |

## Prerequisites
- A running Kubernetes cluster (minikube, kind, or cloud provider)
- kubectl CLI tool installed and configured
- Basic understanding of Kubernetes concepts (Pods, Deployments)
- Text editor for YAML file manipulation

## Examples included

### 1. ConfigMap from literal values
```bash
# Create ConfigMap from literal values
kubectl create configmap app-config \
    --from-literal=APP_COLOR=blue \
    --from-literal=APP_MODE=prod
```

### 2. ConfigMap from files
```bash
# Create ConfigMap from a configuration file
kubectl create configmap app-config \
    --from-file=app-config.properties
```

### 3. Secret management
```bash
# Create a secret for database credentials
kubectl create secret generic db-secret \
    --from-literal=username=myuser \
    --from-literal=password=mypassword
```

### 4. Environment variables injection
The `pod-with-config.yaml` demonstrates **three** ways to consume configuration:
```yaml
env:                    # selected keys as env vars
- name: APP_COLOR
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: APP_COLOR

envFrom:                # ALL keys of a ConfigMap as env vars
- configMapRef:
    name: app-config-file
```

### 5. Volume mounts for configuration
```yaml
volumes:
  - name: config-volume
    configMap:
      name: app-config
```
Each ConfigMap key becomes a file under the mount path. Unlike env vars, volume-mounted ConfigMaps/Secrets are **updated in place** by the kubelet (eventually — usually within a minute), so apps watching their config files can reload without a restart.

## Best Practices

1. **Namespace Isolation**
   - Use separate ConfigMaps for different environments
   - Keep configurations namespace-scoped
   - Use RBAC to control access

2. **Version Control**
   - Store ConfigMap definitions in git
   - Use labels for versioning
   - Implement change tracking

3. **Security**
   - Never store sensitive data in ConfigMaps
   - Use Secrets for confidential information
   - Implement proper RBAC policies

4. **Resource Management**
   - Keep ConfigMaps small
   - Use multiple ConfigMaps for different aspects
   - Consider using a configuration management tool

## Getting Started

1. Apply the ConfigMap manifests:
```bash
# Create the literal ConfigMap
kubectl apply -f configmap-literal.yaml

# Create the file-based ConfigMap
kubectl apply -f configmap-file.yaml

# Create the secret
kubectl apply -f secret.yaml

# Deploy the pod with configurations
kubectl apply -f pod-with-config.yaml
```

2. Verify the configuration:
```bash
# Check ConfigMap and Secret creation
kubectl get configmaps,secrets

# Verify Pod environment variables
kubectl exec app-pod -- env | grep -E "APP_|LOG_LEVEL"

# View mounted configurations
kubectl exec app-pod -- ls /etc/app-config
kubectl exec app-pod -- cat /etc/app-config/APP_COLOR
```

## Troubleshooting

Common issues and solutions:

1. **ConfigMap Not Found**
   - Verify ConfigMap exists in the correct namespace
   - Check Pod and ConfigMap are in the same namespace
   - Validate RBAC permissions

2. **Configuration Not Updated**
   - **Env vars are injected at Pod creation only** — editing a ConfigMap never updates them; restart the Pod (`kubectl rollout restart deployment <name>`)
   - Volume-mounted configs are refreshed automatically (eventually), but apps must reload the files themselves
   - Remember the 1 MiB size limit for ConfigMaps/Secrets

3. **Volume Mount Issues**
   - Check volume mount paths
   - Verify file permissions
   - Validate ConfigMap keys match expected files

## Additional Resources

- [Official Kubernetes ConfigMap Documentation](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Kubernetes Secrets Documentation](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Configuration Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/) 