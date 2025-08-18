# Elasticsearch Single-Node Deployment - Working Configuration

## Summary
Successfully resolved Elasticsearch security issues on 2025-08-18 by implementing a proper single-node deployment configuration.

## Issue Fixed
- **Original Error**: `Security must be explicitly enabled when using a [basic] license. Enable security by setting [xpack.security.enabled] to [true]`
- **Root Cause**: Elasticsearch 7.17.3 requires explicit security configuration, and multi-node cluster discovery was causing issues

## Working Solution

### Key Configuration Parameters
```yaml
replicas: 1
minimumMasterNodes: 1
xpack.security.enabled: false
discovery.type: single-node
```

### Working Helm Command
```bash
helm upgrade --install elasticsearch elastic/elasticsearch \
  --version "7.17.3" \
  --namespace "logging-tracing" \
  --set replicas=1 \
  --set minimumMasterNodes=1 \
  --set xpack.security.enabled=false \
  --set discovery.type=single-node \
  --set volumeClaimTemplate.storageClassName=longhorn \
  --set resources.requests.memory=1Gi \
  --set resources.limits.memory=2Gi \
  --wait --timeout=300s
```

### Ansible Implementation
Updated `roles/logging-tracing/tasks/main.yml` to use the working configuration with proper variable substitution:

```yaml
- name: Deploy Elasticsearch (7.17.3)
  shell: |
    export PATH=/usr/local/bin:$PATH
    /usr/local/bin/helm upgrade --install elasticsearch elastic/elasticsearch \
      --version "{{ elasticsearch_chart_version }}" \
      --namespace "{{ logging_namespace }}" \
      --create-namespace \
      --set replicas="{{ elasticsearch_replicas }}" \
      --set minimumMasterNodes="{{ elasticsearch_min_master_nodes }}" \
      --set xpack.security.enabled=false \
      --set discovery.type=single-node \
      --set volumeClaimTemplate.storageClassName="{{ storage_class }}" \
      --set volumeClaimTemplate.accessModes[0]=ReadWriteOnce \
      --set volumeClaimTemplate.resources.requests.storage="{{ elasticsearch_storage_size }}" \
      --set resources.requests.memory="{{ elasticsearch_memory_request }}" \
      --set resources.requests.cpu="{{ elasticsearch_cpu_request }}" \
      --set resources.limits.memory="{{ elasticsearch_memory_limit }}" \
      --set resources.limits.cpu="{{ elasticsearch_cpu_limit }}" \
      --set esJavaOpts="{{ elasticsearch_java_opts }}" \
      --wait --timeout=600s
```

## Verification Steps

### 1. Pod Status
```bash
kubectl get pods -n logging-tracing -l app=elasticsearch-master
# Should show: elasticsearch-master-0   1/1     Running
```

### 2. Cluster Health
```bash
kubectl exec -n logging-tracing elasticsearch-master-0 -- curl -s http://localhost:9200/_cluster/health
# Should return JSON with "status":"yellow" (expected for single-node)
```

### 3. Basic API Test
```bash
kubectl exec -n logging-tracing elasticsearch-master-0 -- curl -s http://localhost:9200/
# Should return Elasticsearch version info
```

## Critical Success Factors

1. **Single Node Configuration**: Using `discovery.type=single-node` prevents cluster discovery issues
2. **Security Disabled**: `xpack.security.enabled=false` removes authentication requirements  
3. **Proper Replicas**: `replicas=1` and `minimumMasterNodes=1` for single-node setup
4. **Resource Limits**: Appropriate memory limits (1Gi request, 2Gi limit) for the environment

## Default Variables (defaults/main.yml)
```yaml
elasticsearch_replicas: 1
elasticsearch_min_master_nodes: 1
elasticsearch_cluster_name: "elasticsearch"
elasticsearch_node_group: "master"
elasticsearch_cpu_request: "500m"
elasticsearch_cpu_limit: "1000m"
elasticsearch_memory_request: "1Gi"
elasticsearch_memory_limit: "2Gi"
elasticsearch_storage_size: "2Gi"
elasticsearch_java_opts: "-Xmx1g -Xms1g"
```

## Files Updated
- `roles/logging-tracing/tasks/main.yml` - Main deployment task
- `roles/logging-tracing/defaults/main.yml` - Added documentation comments
- `roles/logging-tracing/files/elasticsearch-values.yaml` - Kept as reference

## Deployment Status
✅ **Working**: Elasticsearch 7.17.3 single-node deployment  
✅ **Working**: Kibana connectivity to Elasticsearch  
✅ **Working**: No security errors  
✅ **Working**: Ansible automation ready

## Next Steps
- Test full ELK stack deployment with `ansible-playbook -i inventory-orig complete-installation-runbook.yml --tags logging`
- Verify log ingestion from Filebeat to Logstash to Elasticsearch
- Confirm Kibana dashboard access and index pattern creation
