# Logging & Tracing Architecture (ELK + Jaeger)

```mermaid
graph TD
    subgraph Kubernetes Cluster
        subgraph logging-tracing namespace
            Filebeat[Filebeat DaemonSet]
            Logstash[Logstash]
            Elasticsearch[Elasticsearch]
            Kibana[Kibana]
            Jaeger[Jaeger]
        end
    end

    Filebeat -- Logs --> Logstash
    Logstash -- Processed Logs --> Elasticsearch
    Elasticsearch -- Data --> Kibana
    Kibana -- UI --> User
    Filebeat -- (optional direct) --> Elasticsearch
    Jaeger -- Traces --> Elasticsearch
    User -- Access --> Kibana
    User -- Access --> Jaeger

    subgraph Ingress
        KibanaIngress[Kibana Ingress]
        JaegerIngress[Jaeger Ingress]
    end
    Kibana -- Exposed via --> KibanaIngress
    Jaeger -- Exposed via --> JaegerIngress
    KibanaIngress -- External URL --> User
    JaegerIngress -- External URL --> User
```

**Description:**
- Filebeat DaemonSet collects logs from all nodes/pods.
- Logstash processes and forwards logs to Elasticsearch.
- Elasticsearch stores logs and traces.
- Kibana provides log visualization and dashboards.
- Jaeger provides distributed tracing and stores traces in Elasticsearch.
- Ingress exposes Kibana and Jaeger UIs externally.
