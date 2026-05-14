# Log Monitoring with Loki
Application setup stack:
1. Microservices are running on docker containers.
2. Docker image directory location is changed into `/mnt/Data/docker`

## Purpose:
Monitor docker container logs into grafana with loki.

## Loki + Grafana + Promtain  Setup:
docker-compose.yaml
```YAML
version: "3.8"

networks:
  monitoring:
    driver: bridge

volumes:
  loki_data:
  grafana_data:

services:
  loki:
    image: grafana/loki:2.9.4
    container_name: loki
    restart: unless-stopped
    ports:
      - "3100:3100"
    volumes:
      - ./loki-config.yaml:/etc/loki/local-config.yaml
      - loki_data:/loki
    command: -config.file=/etc/loki/local-config.yaml
    networks:
      - monitoring

  promtail:
    image: grafana/promtail:2.9.4
    container_name: promtail
    restart: unless-stopped
    volumes:
      - ./promtail-config.yaml:/etc/promtail/config.yaml
      # Docker log path uses your custom data-root
      - /mnt/Data/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    command: -config.file=/etc/promtail/config.yaml
    depends_on:
      - loki
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:10.3.3
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3002:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_PATHS_PROVISIONING=/etc/grafana/provisioning
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    depends_on:
      - loki
    networks:
      - monitoring
```
loki-config.yaml
```YAML
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9096

common:
  path_prefix: /loki
  storage:
    filesystem:
      chunks_directory: /loki/chunks
      rules_directory: /loki/rules
  replication_factor: 1
  ring:
    instance_addr: 127.0.0.1
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

ruler:
  alertmanager_url: http://localhost:9093

limits_config:
  reject_old_samples: true
  reject_old_samples_max_age: 168h
  ingestion_rate_mb: 16
  ingestion_burst_size_mb: 32

```

promtail-config.yaml
```YAML
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: docker
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
        refresh_interval: 5s
        filters:
          - name: status
            values: ["running"]
    relabel_configs:
      # Static labels — always present
      - target_label: job
        replacement: docker

      - target_label: host
        replacement: "dockerhost"

      # Container name (strip leading slash)
      - source_labels: [__meta_docker_container_name]
        regex: "/(.*)"
        target_label: container
        replacement: "$1"

      # Image name
      - source_labels: [__meta_docker_container_image]
        target_label: image

      # Docker Compose project & service (empty if not compose)
      - source_labels: [__meta_docker_container_label_com_docker_compose_project]
        target_label: compose_project

      - source_labels: [__meta_docker_container_label_com_docker_compose_service]
        target_label: compose_service

      # Log file path using your custom data-root
      - source_labels: [__meta_docker_container_id]
        target_label: __path__
        replacement: "/var/lib/docker/containers/$1/$1-json.log"

    pipeline_stages:
      - docker: {} 
```
Start containers:
```
docker-compose up -d

## verify
docker-compose logs promtail
```

## Create Grafana Dashboard:
### Open in browser:
- http://localhost:3002 <br>
username - `admin` <br>
password - `admin` <br>

- Left sidebar → Dashboards → New → New Dashboard
- Click Add visualization
- Select Loki as the data source

### Panel 1 — Logs Panel (All Containers)
- Best for reading actual log lines.

- In panel editor, set visualization type to Logs (top right dropdown)
- In the query box, enter:
```
logql{job="docker"}
```
- Set Title → `All Container Logs`
- Click Apply

### Panel 2 — Log Volume Over Time (Bar Chart)
Shows how many log lines per container over time.

- Add new panel → visualization type: Time series or Bar chart
Query:
```logql
sum by (container) (count_over_time({job="docker"}[1m]))
```
OR Container as a variable
```
sum by (container) (count_over_time({job="docker", container=~"$container"}[1m]))
```
- Title → Log Volume by Container
- Apply

### Panel 3 — Error Rate Over Time
```logql
sum by (container) (count_over_time({job="docker"} |= "error" [1m]))
```
OR take container names as variable
```
sum by (container) (count_over_time({job="docker", container=~"$container"} |= "error" [1m]))
```
- Visualization: `Time series`
- Title: `Error Rate by Container`

### Panel 4 — Logs by Specific Container (Variable-driven)
- First set up a variable, then use it in panels
- Set visualizations to logs:
    - Enable Time, Wrap lines
```
{job="docker", container=~"$container"}
```

#### Create variable
<img width="928" height="812" alt="image" src="https://github.com/user-attachments/assets/e43c8248-ac31-499e-95d9-b9fcc90c102e" />
- check multi-value, include All options
- Apply.
