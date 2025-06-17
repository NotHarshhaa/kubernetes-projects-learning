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
  - As environment variables
  - As configuration files
  - As command-line arguments
- Working with Secrets
  - Creating encrypted secrets
  - Mounting secrets in pods
  - Automatic secret rotation
- Best practices for configuration management
  - Versioning configurations
  - Namespace isolation
  - Access control

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
The `pod-with-config.yaml` demonstrates how to inject ConfigMap values as environment variables.

### 5. Volume mounts for configuration
```yaml
volumes:
  - name: config-volume
    configMap:
      name: app-config
```

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
# Check ConfigMap creation
kubectl get configmaps

# Verify Pod environment variables
kubectl exec pod-name -- env

# View mounted configurations
kubectl exec pod-name -- ls /etc/config
```

## Troubleshooting

Common issues and solutions:

1. **ConfigMap Not Found**
   - Verify ConfigMap exists in the correct namespace
   - Check Pod and ConfigMap are in the same namespace
   - Validate RBAC permissions

2. **Configuration Not Updated**
   - Remember ConfigMap updates aren't automatically reflected
   - Restart Pod to pick up new configurations
   - Consider using a configuration reloader

3. **Volume Mount Issues**
   - Check volume mount paths
   - Verify file permissions
   - Validate ConfigMap keys match expected files

## Additional Resources

- [Official Kubernetes ConfigMap Documentation](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Kubernetes Secrets Documentation](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Configuration Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/) 