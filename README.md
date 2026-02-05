<img width="1280" height="1664" alt="image" src="https://github.com/user-attachments/assets/9f7b4428-b4af-4d4a-aaef-9fe8c9ba32de" />


What can be monitored? 
--------

`Infrastructure`: CPU usage, memory usage, disk I/O, network traffic.  <br>
`Applications`: Response times, error rates, throughput.  <br>
`Databases`: Query performance, connection pool usage, transaction rates.  <br>
`Network`: Latency, packet loss, bandwidth usage.  <br>
`Security`: Unauthorized access attempts, vulnerability scans, firewall logs.  <br>

What Can Be Observed?
--------------------
- `Logs`: Detailed records of events and transactions within the system.
- `Metrics`: Quantitative data points like CPU load, memory consumption, and request counts.
- `Traces`: Data that shows the flow of requests through various services and components.

### Observability ( why and how ): 
What to observe - Logs, Metrics, traces <br>

- Metrics – Monitoring 
- Logs - 	Logging 
- Traces – Tracing 

##### Use observability to: 

1. Diagnosis issues 
2. Understand behaviour 
3. Ensure availability 

##### What can be observed ? 

`Logs` : Detailed records of events and transactions within the system.  <br>
`Metrics` : Quantitative data points like CPU load, memory consumption, and request counts.  <br>
`Traces` : Data that shows the flow of requests through various services and components.  <br>

### Monitoring ( when and what ): 

Monitoring is subset of observability.  <br>

Use monitoring to: 
1. Detects problem early 
2. Understand behaviour 
3. Improve systems 


Available tools: 
----

- Monitoring Tools: Prometheus, Grafana, Nagios, Zabbix, PRTG.  <br>
- Observability Tools: `ELK Stack` (Elasticsearch, Logstash, Kibana), `EFK Stack` (Elasticsearch, FluentBit, Kibana) `Splunk`, `Jaeger`,  `Zipkin`,  `New Relic`,  `Dynatrace`,  `Datadog`

Step by step flow: 
---

1️⃣ Instrumentation:
Applications are instrumented to generate telemetry:
- Metrics
- Logs
- Traces

`OpenTelemetry` SDK to Auto-instrument HTTP requests.	 

2️⃣  Collection 
- Gather telemetry from all services. 
- `OpenTelemetry collector` or `Fluentbit`. 

3️⃣ Storage 
- Storage telemetry in the right backend.

- Metrics --> `Prometheus`/`Thanos`
- Logs --> `Elasticsearch`/`Loki`
- Traces --> `Jaeger`/`Tempo`

4️⃣  Visualization
- `Grafana`	( Metrics ) 
- `kibana`	( Logs )	 

5️⃣ Alerting 
- `Prometheus Alertmanager`/ Grafana alerts.	 

So the flow looks like, 
```
App (OTel SDK)
   ↓
OpenTelemetry Collector
   ↓
Prometheus / Jaeger / Elasticsearch
   ↓
Grafana / Kibana
   ↓
Alerts
```
---------------------------------------------

### 1️⃣ instrumentation & Collection: 
- Instrumentation → create telemetry inside the app. Adding code to generate telemetry
- Collection → receive, process, and send telemetry out

**Instrumentation:** <br>
What instrumentation produces from your app:
- Spans (trace pieces)
- Metrics
- Logs

🔹 For that `OpenTelemetry` is used. <br>
Observability: <br>
- its a toolkit.

**The 3 pillers of OTel.** <br>
- Metrics 
- Logs 
- Traces 

2️⃣ **Collection:** <br>
- receiving telemetry from apps & infrastructure
```
App → sends telemetry → OpenTelemetry Collector
```
This is done by `OpenTelemetry Collector`.

🔹 **Fluentbit:** <br>
- A light weight log processor and forwarder. 
- Same as OTel Collector but it sends logs only. Not metrices and traces. 
- This runs as a Daemonset. 


**OpenTelemetry vs. Fluentbit.** 

- In most cases tems uses them together. 

- Fluent Bit → Collect logs from containers/nodes. 
- OTel Collector → Collect metrics/traces + enrich logs + export everything to backends. 

| Feature           | OpenTelemetry Collector                         | Fluent Bit                          |
|-------------------|--------------------------------------------------|-------------------------------------|
| Telemetry types   | Metrics, Logs, Traces (all three)               | Logs only                            |
| Protocol          | OTLP (native) + many others                     | Tail files, journald, syslog         |
| Use case          | Unified observability pipeline                  | Log collection & shipping            |
| Kubernetes        | Runs as DaemonSet / Sidecar / Agent             | Typically runs as DaemonSet          |
| Processing        | Enrichment, batching, sampling                  | Parsing, filtering, buffering        |
| Performance       | Moderate (multi-signal processing)              | Very lightweight (logs only)         |
| Best for          | Central pipeline for metrics, logs & traces     | High-performance log shipping        |

 

So the flow would be: 
```
Apps (OTel SDK) → OTel Collector → Prometheus / Jaeger / Tempo   

Container Logs → Fluent Bit → OTel Collector → Loki / Elasticsearch 
```
### Note: 
- If your focus is only logs → Fluent Bit.
- If your focus is metrics + traces (and logs) → OTel Collector.
- If you want a full observability pipeline → combine both. 

3️⃣ Storage 

#### Storage - logs

**Loki ( Log storage ):** 
Now fluentbit and opentelemetry collects logs from the application and forwards them. <br>
- Loki receives logs, store them efficiently and allows visualization. 


📊 Storage – metrics 
**Prometheus**
```
Prometheus ->  thanos, Cortex, VictoriaMetrics 
```
**Thanos:**
- If you dont want long term metrics, you might not need thanos. 
- `Thanos` is not a replacement for prometheus, its enhancement for long-term storage of prometheus. 

```
Prometheus → Thanos Sidecar → Object Storage → Thanos Store / Querier → Grafana 
```

#### Prometheus vs. Thanos  
| Feature | Prometheus | Thanos |
|------|------------|--------|
| **Primary role** | Collects metrics from apps and stores them locally (time-series DB) | Extends Prometheus for long-term storage, high availability, and global view |
| **Data type** | Metrics (time-series) | Metrics (aggregated from multiple Prometheus instances) |
| **High availability** | Limited – requires manual replica setup | Built-in HA with deduplication across replicas |
| **Long-term storage** | Local disk (retention limited to weeks/months) | Object storage (S3 / GCS / Azure) – can store data for years |
| **Multi-cluster support** | No | Yes – Thanos Querier provides a global view across clusters |
| **Downsampling** | No | Yes – reduces resolution of old data to save storage |
| **Querying** | PromQL on a single Prometheus instance | PromQL across multiple Prometheus instances + historical data |

 
### Cortex: 
- Same as thanos for long term storage.
- Thanos is used for single per organization use case.
- Cortex is use for multi tenant Saas Graded prometheus backend. 
- Setup is complex. 
```
Prometheus → Cortex Distributor → Ingester → Object Storage → Querier → Grafana dashboards 
```

### VictoriaMetrics: 
- It’s simple and fast alternative of Thanos/Cortex for long-term metrics storage. 
- Optimized for huge data storage. High preformance and low resource. 
- Same queries as prometheus. 
- Long term retention. 
- Multi tenancy support.
- Easiest to setup. 
```
App → Prometheus (scrapes metrics) → remote_write → VictoriaMetrics → Grafana 
```
Conculsion : VictoriaMetrics is simpler and cheaper for most orgranizations. 

📜 Logs storage
- Elasticsearch
   - Full-text search <br>
   - Heavy but powerful <br>
- Loki
   - Label-based <br>
   - Cheap and scalable <br>
   - Works natively with Grafana <br>
Flow:
```
Fluent Bit → OTel Collector (optional) → Loki / Elasticsearch
```
 
🧭 Storage – traces 

- Jaeger: 
   - Distributed tracing system used for monitoring and troubleshooting microservice-based architectures. <br>
   - Helps developers to understand how requests are flows through complex systems.  <br>

Why use jaeger ?  <br>

- In morden applications, as specically microservices architecture, a single request can touch multiple services, when something goes wrong, its challenging to pinpoint the source.
- https://github.com/iam-veeramalla/observability-zero-to-hero/tree/main/day-6 

Tempo: <br>
   - Store and query traces. <br>
   - Stores traces in object storage. <br>
   - Light weight. <br>
   - Designed for cloud native high volume workloads.  <br>

### Jaeger vs. Tempo

| Feature | Jaeger | Tempo |
|------|--------|-------|
| **Primary function** | Distributed tracing | Distributed tracing |
| **Storage** | Local disk, Elasticsearch, Cassandra | Object storage (S3, GCS, Azure Blob) |
| **Scaling** | Medium – scaling requires database tuning | High – scalable to millions of traces per day |
| **Indexing** | Requires indexes (storage-heavy at scale) | Index-free design – cheaper and less storage |
| **Integration** | Jaeger UI, Grafana (via plugin) | Native Grafana integration (direct trace queries) |
| **Cost efficiency** | Moderate – DB-heavy, higher storage cost | Very cost-efficient – object storage, minimal indexing |
| **Use case** | Small to medium-scale deployments | Large-scale, cloud-native deployments |
| **Ease of setup** | Can run standalone with built-in UI | Typically requires Grafana + OTel Collector |


4️⃣ Visualization (Understand data)
- Humans need dashboards and UIs to make sense of telemetry.
### Tools used 
📈 Grafana <br> 
📄 Kibana <br>

5️⃣ Alerting (Act on issues)
- 🚨 Prometheus Alertmanager
- 🚦 Grafana Alerts

## 🔁 End-to-End Flow (Production)
```
Application
  ↓ (OTel SDK)
Instrumentation
  ↓
OTel Collector  ← Fluent Bit (logs)
  ↓
Prometheus / Thanos (metrics)
Loki / Elasticsearch (logs)
Jaeger / Tempo (traces)
  ↓
Grafana / Kibana
  ↓
Alertmanager / Grafana Alerts
```
