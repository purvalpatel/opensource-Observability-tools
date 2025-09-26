<img width="1280" height="1664" alt="image" src="https://github.com/user-attachments/assets/9f7b4428-b4af-4d4a-aaef-9fe8c9ba32de" />

What Can Be Monitored?
----------------------
- Infrastructure: CPU usage, memory usage, disk I/O, network traffic.
- Applications: Response times, error rates, throughput.
- Databases: Query performance, connection pool usage, transaction rates.
- Network: Latency, packet loss, bandwidth usage.
- Security: Unauthorized access attempts, vulnerability scans, firewall logs.

What Can Be Observed?
--------------------
- Logs: Detailed records of events and transactions within the system.
- Metrics: Quantitative data points like CPU load, memory consumption, and request counts.
- Traces: Data that shows the flow of requests through various services and components.



### Observability ( why and how ): 
What to observe - Logs, Metrics, traces 
  Metrics – Monitoring 
  Logs - 	Logging 
Traces – Tracing 

##### Use observability to: 

1. Diagnosis issues 
2. Understand behaviour 
3. Ensure availability 

##### What can be observed ? 

Logs: Detailed records of events and transactions within the system. 
Metrics: Quantitative data points like CPU load, memory consumption, and request counts. 
Traces: Data that shows the flow of requests through various services and components. 

### Monitoring ( when and what ): 

Monitoring is subset of observability. 
Use monitoring to: 
1. Detects problem early 
2. Understand behaviour 
3. Improve systems 

#### What can be monitored? 
Infrastructure: CPU usage, memory usage, disk I/O, network traffic. 
Applications: Response times, error rates, throughput. 
Databases: Query performance, connection pool usage, transaction rates. 
Network: Latency, packet loss, bandwidth usage. 
Security: Unauthorized access attempts, vulnerability scans, firewall logs. 

 

### Available tools: 
Monitoring Tools: Prometheus, Grafana, Nagios, Zabbix, PRTG. 
Observability Tools: 
ELK Stack (Elasticsearch, Logstash, Kibana), 
EFK Stack (Elasticsearch, FluentBit, Kibana) Splunk,  
Jaeger,  
Zipkin,  
New Relic,  
Dynatrace,  
Datadog. 

 

## Step by step flow: 

1. Instrumentation 
- OpenTelemetry SDK to Auto-instrument HTTP requests.	 

2. Collection 
- Gather telemetry from all services. 
- OpenTelemetry collector or Fluentbit. 

3. Storage 
- Storage telemetry in the right backend. 
Metrics --> Prometheus/Thanos 
Logs --> Elasticsearch/Loki 
Traces --> Jaeger/Tempo 	 

4. Visualization 
- Grafana	( Metrics ) 
- kibana	( Logs )	 

5. Alerting 
- Prometheus Alertmanager/ Grafana alerts.	 

So the flow looks like, 

App (OTel SDK) → OTel Collector → (Prometheus + Jaeger + Elasticsearch) → Grafana/Kibana → Alerts 

### Collection and instrumentation: 

#### Opentelemetry: 

Opensource observability framework for generating, collecting and exporting telemetry data (Metric, traces and logs ) to help monitor applications. Its not storage. 
Opentelemetry supports several languages. Think of it as a universal language + delivery truck for observability. 
It’s observability toolkit. 

The 3 pillers of OTel. 

Metrics 
Logs 
Traces 

Components: 

Instrumentation (SDK and Auto-instrumentation ) : Your app is instrumented with OTel SDK. 
Context propogation: Each request -> Generates metrics, logs and traces. 
OpenTelemetry collector: Telemetry -> Sent to Otel collector. 
Collector process it and exports to --> prometheus for metrics , Jager for traces, Elastic search for logos 

#### Fluentbit: 

A light weight log processor and forwarder. 
Same as OTel Collector but it sends logs only. Not metrices and traces. 
This runs as a Daemonset. 

 

#### OpenTelemetry vs. Fluentbit. 

In most cases tems uses them together. 

Fluent Bit → Collect logs from containers/nodes. 
OTel Collector → Collect metrics/traces + enrich logs + export everything to backends. 

 

Feature 

OpenTelemetry Collector 

Fluent Bit 
Telemetry types 
	

Metrics, Logs, Traces (all 3) 
	

Logs only 

Protocol 
	

OTLP (native) + many others 
	

Tail files, journald, syslog 

Use case 
	

Unified observability pipeline 
	

Log collection & shipping 

Kubernetes 
	

Runs as DaemonSet/sidecar/agent 
	

Typically runs as DaemonSet 

Processing 
	

Enrichment, batching, sampling 
	

Parsing, filtering, buffering 

Performance 
	

Moderate (multi-signal) 
	

Very lightweight (logs only) 

Best for 
	

Central pipeline (metrics/logs/traces) 
	

High-performance log shipping 

 

So the flow would be: 

Apps (OTel SDK) → OTel Collector → Prometheus / Jaeger / Tempo   

Container Logs → Fluent Bit → OTel Collector → Loki / Elasticsearch 

Note: 

    If your focus is only logs → Fluent Bit. 

    If your focus is metrics + traces (and logs) → OTel Collector. 

    If you want a full observability pipeline → combine both. 

Storage - logs 

Loki ( Log storage ): 

    Now fluentbit and opentelemetry collects logs from the application and forwards them. 

    Loki receives logs, store them efficiently and allows visualization. 

 

Storage –metrics 

Prometheus ->  thanos, Cortex, VictoriaMetrics 

Thanos: 

    If you dont want long term metrics, you might not need thanos. 

    Thanos is not a replacement for prometheus, its enhancement for long-term storage of prometheus. 

 

Prometheus → Thanos Sidecar → Object Storage → Thanos Store / Querier → Grafana 

Prometheus vs. Thanos  

Feature 
	

Prometheus 
	

Thanos 

Primary role 
	

Collects metrics from apps & stores them locally (time-series DB) 
	

Extends Prometheus for long-term storage, HA, and global view 

Data type 
	

Metrics (time-series) 
	

Metrics (aggregated from multiple Prometheus instances) 

High availability 
	

Limited → requires replicas manually 
	

Built-in HA with deduplication across replicas 

Long-term storage 
	

Local disk → retention limited to weeks/months 
	

Object storage (S3/GCS/Azure) → store for years 

Multi-cluster support 
	

No 
	

Yes → Thanos Querier provides global view across clusters 

Downsampling 
	

No 
	

Yes → reduces resolution for old data to save storage 

Querying 
	

PromQL for a single Prometheus 
	

PromQL across multiple Prometheus instances + historical data 

 

Cortex: 

    Same as thanos for long term storage. 

    Thanos is used for single per organization use case. 

    Cortex is use for multi tenant Saas Graded prometheus backend. 

    Setup is complex. 

Prometheus → Cortex Distributor → Ingester → Object Storage → Querier → Grafana dashboards 

VictoriaMetrics: 

    It’s simple and fast alternative of Thanos/Cortex for long-term metrics storage. 

    Optimized for huge data storage. High preformance and low resource. 

    Same queries as prometheus. 

    Long term retention. 

    Multi tenancy support. 

    Easiest to setup. 

App → Prometheus (scrapes metrics) → remote_write → VictoriaMetrics → Grafana 

 

Conculsion : VictoriaMetrics is simpler and cheaper for most orgranizations. 

 

 

Storage – traces 

 

Jaeger: 

Distributed tracing system used for monitoring and troubleshooting microservice-based architectures. 

Helps developers to understand how requests are flows through complex systems. 

Why use jaeger ? 

In morden applications, as specically microservices architecture, a single request can touch multiple services, when something goes wrong, its challenging to pinpoint the source. 

https://github.com/iam-veeramalla/observability-zero-to-hero/tree/main/day-6 

Tempo: 

    Store and query traces. 

    Stores traces in object storage. 

    Light weight. 

    Designed for cloud native high volume workloads. 

 

Jaeger vs. Tempo 

Feature 
	

Jaeger 
	

Tempo 

Primary function 
	

Distributed tracing 
	

Distributed tracing 

Storage 
	

Local disk, Elasticsearch, Cassandra 
	

Object storage (S3, GCS, Azure Blob) 

Scaling 
	

Medium → scaling requires DB tuning 
	

High → scalable for millions of traces/day 

Indexing 
	

Requires indexes (heavy on storage for large workloads) 
	

Index-free design → cheaper, less storage 

Integration 
	

Jaeger UI, Grafana (via plugin) 
	

Grafana native (directly query traces) 

Cost efficiency 
	

Moderate → DB-heavy, higher storage cost 
	

Very cost-efficient → object storage, minimal indexing 

Use case 
	

Small/medium-scale deployments 
	

Large-scale, cloud-native deployments 

Ease of setup 
	

Can run standalone with its UI 
	

Typically requires Grafana + OTel Collector 

 

 

Other Terms explanation: 

Instrumentation: 

    It’s a process of adding monitoring capabilities to your applications, systems or services. 

    This includes wring tools or using tools to collect metrics, logs and traces that provides insights into how the system is performing. 

How it works : 

For examlple, in your node js app you can use library called prom-client to expose the custom metrics. 

Instrumentation In prometheus: 

- Exporters – Node Exporters, Mysql Exporter, Postgres Exporter. 

- Custom metrics – You can write custom metrics to collect the custom data which you require. 

 

Logging: 

Loggin is crucial in any distributed systems, especially in kubernetes, to monitor applications behaviour, detect issues. 

Importance: 

Debugging 

Auditing 

Performance 

Security 

Tools available for logging in kubernetes: 

EFK Stack (ElasticSearch, FluentBit, Kibana), FlentD, Logstash 

Promtail + Loki + Grafana 

EFK Stack: 

Popular logging stack used for collect, store and analyze logs in kubernetes. 

FlentBit – A lightweight log forwarder that collects logs from different sources and sends them to Elasticsearch. 

Elasticsearch – Stores and index logs data for easy retrival. 

Kubana – Visualization tool that allowed users to explore and analyze the stored logs. 

 

Multi-tenancy 

Multi-tenancy means single systems servs multiple independent users, which keeping their data logically isolated. 

Each tenant feels like they have their own instance. But in reality each tenants share the same infrastructure. 

 

 

 

 

 

 
