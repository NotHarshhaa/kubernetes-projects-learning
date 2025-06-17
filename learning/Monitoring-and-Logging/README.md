# Monitoring and Logging in Kubernetes

This comprehensive guide explores implementing robust monitoring and logging solutions in Kubernetes using industry-standard tools and best practices.

## Understanding Kubernetes Monitoring and Logging

Effective monitoring and logging in Kubernetes involves multiple components working together to provide:

- Real-time metrics collection
- Resource utilization tracking
- Application performance monitoring
- Centralized log aggregation
- Alerting and notification
- Visualization and dashboards

### Core Components

1. **Metrics Collection**
   - Prometheus for metrics scraping
   - Node Exporter for hardware/OS metrics
   - kube-state-metrics for cluster state
   - Custom metrics adapters

2. **Visualization**
   - Grafana for metric dashboards
   - Kibana for log visualization
   - Custom dashboards

3. **Log Aggregation**
   - Elasticsearch for log storage
   - Fluentd/Fluent Bit for collection
   - Logstash for processing
   - Vector for modern processing

## What you'll learn
- Setting up Prometheus and Grafana
  - Prometheus operator
  - Service monitors
  - Alert managers
  - Dashboard creation
- Implementing logging with EFK/ELK stack
  - Elasticsearch clusters
  - Fluentd configuration
  - Kibana dashboards
  - Log parsing
- Resource metrics collection
  - CPU and memory metrics
  - Network statistics
  - Storage metrics
  - Custom metrics
- Custom metrics and alerts
  - Prometheus rules
  - Alert configuration
  - Notification channels
  - SLO/SLI monitoring
- Log aggregation and visualization
  - Centralized logging
  - Log filtering
  - Pattern matching
  - Visual analysis

## Prerequisites
- Running Kubernetes cluster
  - Minimum 3 nodes recommended
  - Sufficient resources for monitoring stack
- kubectl CLI tool with admin access
- Helm (optional, for easy deployment)
- Basic understanding of:
  - Kubernetes architecture
  - Monitoring concepts
  - Log management
  - Metrics and alerting

## Component Setup

### 1. Prometheus Stack
```yaml
# prometheus-values.yaml
prometheus:
  prometheusSpec:
    retention: 15d
    resources:
      requests:
        memory: 1Gi
        cpu: 500m
      limits:
        memory: 2Gi
        cpu: 1000m

grafana:
  persistence:
    enabled: true
    size: 10Gi
```

### 2. Elasticsearch Configuration
```yaml
# elasticsearch.yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: logging
spec:
  version: 7.17.3
  nodeSets:
  - name: default
    count: 3
    config:
      node.store.allow_mmap: false
```

### 3. Fluentd DaemonSet
The `fluentd-ds.yaml` file in this directory contains a complete Fluentd configuration for log collection.

## Implementation Guide

### 1. Deploy Prometheus Stack
```bash
# Add Prometheus repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus stack
helm install monitoring prometheus-community/kube-prometheus-stack -f prometheus-values.yaml
```

### 2. Deploy EFK Stack
```bash
# Create namespace
kubectl create namespace logging

# Deploy Elasticsearch
kubectl apply -f elasticsearch.yaml

# Deploy Fluentd
kubectl apply -f fluentd-ds.yaml

# Deploy Kibana
kubectl apply -f kibana.yaml
```

### 3. Configure Service Monitors
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: app-monitor
spec:
  selector:
    matchLabels:
      app: your-app
  endpoints:
  - port: metrics
```

## Best Practices

### 1. Resource Management
- Size components appropriately
- Use resource limits
- Monitor monitoring components
- Implement retention policies

### 2. Security
- Implement authentication
- Encrypt communications
- Use secure endpoints
- Regular security updates

### 3. Performance
- Optimize scrape intervals
- Configure log rotation
- Use efficient queries
- Index management

### 4. High Availability
- Deploy redundant components
- Use persistent storage
- Implement backups
- Disaster recovery planning

## Troubleshooting

### Common Issues

1. **Prometheus Issues**
   - Target scraping failures
   - Storage problems
   - Rule evaluation errors
   - Resource constraints

2. **Logging Problems**
   - Log shipping delays
   - Missing logs
   - Index management
   - Storage capacity

3. **Visualization Issues**
   - Dashboard loading
   - Query performance
   - Data availability
   - User access

### Debugging Commands
```bash
# Check Prometheus status
kubectl get prometheuses -n monitoring
kubectl describe prometheus -n monitoring

# Verify Fluentd
kubectl get pods -n logging -l app=fluentd
kubectl logs -n logging -l app=fluentd

# Elasticsearch health
kubectl exec -it elasticsearch-0 -n logging -- curl localhost:9200/_cluster/health
```

## Advanced Configuration

### 1. Custom Metrics
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: custom-alerts
spec:
  groups:
  - name: custom.rules
    rules:
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) > 1
      for: 5m
      labels:
        severity: critical
```

### 2. Log Processing
```yaml
# Fluentd configuration
<match kubernetes.**>
  @type elasticsearch
  host elasticsearch
  port 9200
  logstash_format true
  <buffer>
    @type file
    path /var/log/fluentd-buffers/kubernetes.system.buffer
    flush_mode interval
    retry_type exponential_backoff
    flush_interval 5s
    retry_forever false
    retry_max_interval 30
    chunk_limit_size 2M
    queue_limit_length 8
    overflow_action block
  </buffer>
</match>
```

## Accessing Dashboards
- Prometheus: http://localhost:9090
- Grafana: http://localhost:3000
- Kibana: http://localhost:5601

Default credentials and access methods are documented in each component's deployment guide.

## Additional Resources

- [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Elasticsearch Documentation](https://www.elastic.co/guide/index.html)
- [Fluentd Documentation](https://docs.fluentd.org/)
- [Kubernetes Monitoring Guide](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/)
- [EFK Stack Tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-elasticsearch-fluentd-and-kibana-efk-logging-stack-on-kubernetes) 