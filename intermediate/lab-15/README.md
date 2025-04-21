# Lab 15: Docker Monitoring

## Objective
In this lab, you will learn how to monitor Docker containers and services using various tools and techniques. You'll understand how to collect metrics, analyze logs, set up alerts, and optimize container performance.

## Prerequisites
- Completed Lab 14 (Docker Storage)
- Docker Desktop running on your system
- Basic understanding of monitoring concepts
- Familiarity with Docker Compose

## Key Concepts
- Container Metrics: CPU, memory, network, and disk usage
- Logging: Container log collection and analysis
- Monitoring Tools: Prometheus, Grafana, and cAdvisor
- Alerting: Setting up notifications for critical events
- Performance Analysis: Identifying and resolving bottlenecks

## Exercises

### 1. Basic Monitoring
1. View container statistics:
```powershell
# View real-time stats for all containers
docker stats

# View stats for specific container
docker stats web

# View stats with custom format
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# View container events
docker events
```

2. Monitor container logs:
```powershell
# View container logs
docker logs web

# Follow logs in real-time
docker logs -f web

# View logs with timestamps
docker logs -t web

# View last N lines
docker logs --tail 100 web
```

### 2. Advanced Monitoring
1. Set up Prometheus and Grafana:
```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'

  grafana:
    image: grafana/grafana
    volumes:
      - grafana-data:/var/lib/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=secret
    depends_on:
      - prometheus

  cadvisor:
    image: gcr.io/cadvisor/cadvisor
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8080:8080"

volumes:
  grafana-data:
```

2. Configure Prometheus:
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

### 3. Log Management
1. Configure logging drivers:
```powershell
# Run container with specific logging driver
docker run -d \
    --log-driver=json-file \
    --log-opt max-size=10m \
    --log-opt max-file=3 \
    --name web nginx

# View logging configuration
docker inspect --format='{{.HostConfig.LogConfig}}' web
```

2. Set up centralized logging:
```yaml
version: '3.8'

services:
  web:
    image: nginx
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  logspout:
    image: gliderlabs/logspout
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: syslog://logstash:5000
    depends_on:
      - logstash

  logstash:
    image: docker.elastic.co/logstash/logstash:7.9.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5000:5000"
```

### 4. Alerting
1. Configure Prometheus alerts:
```yaml
# prometheus.yml
rule_files:
  - 'alerts.yml'

# alerts.yml
groups:
- name: example
  rules:
  - alert: HighMemoryUsage
    expr: container_memory_usage_bytes > 1e9
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: High memory usage
      description: Container {{ $labels.container }} is using more than 1GB of memory
```

2. Set up alert notifications:
```yaml
# alertmanager.yml
route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'email-notifications'

receivers:
- name: 'email-notifications'
  email_configs:
  - to: 'admin@example.com'
    from: 'alertmanager@example.com'
    smarthost: 'smtp.example.com:587'
    auth_username: 'alertmanager'
    auth_password: 'password'
```

## Best Practices

### 1. Metrics Collection
- Collect relevant metrics
- Set appropriate collection intervals
- Use efficient storage backends
- Implement metric aggregation
- Monitor metric cardinality

### 2. Log Management
- Use appropriate log drivers
- Implement log rotation
- Centralize log collection
- Structure log messages
- Monitor log volume

### 3. Alerting
- Set meaningful thresholds
- Avoid alert fatigue
- Use appropriate notification channels
- Implement alert grouping
- Document alert procedures

### 4. Performance
- Monitor resource usage
- Track container lifecycle
- Analyze performance trends
- Optimize monitoring overhead
- Scale monitoring infrastructure

## Common Patterns

### 1. Basic Monitoring
```yaml
services:
  web:
    image: nginx
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### 2. Prometheus Configuration
```yaml
scrape_configs:
  - job_name: 'docker'
    static_configs:
      - targets: ['cadvisor:8080']
    metrics_path: /metrics
```

### 3. Alert Configuration
```yaml
groups:
- name: docker
  rules:
  - alert: ContainerDown
    expr: up == 0
    for: 1m
    labels:
      severity: critical
```

### 4. Log Management
```yaml
services:
  web:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
        tag: "{{.Name}}"
```

## Challenges
1. Set up a complete monitoring stack
2. Configure custom alerts
3. Implement log analysis
4. Optimize monitoring performance
5. Create monitoring dashboards

## Real-world Examples

### 1. Complete Monitoring Stack
```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alerts.yml:/etc/prometheus/alerts.yml
    ports:
      - "9090:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'

  alertmanager:
    image: prom/alertmanager
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    ports:
      - "9093:9093"
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'

  grafana:
    image: grafana/grafana
    volumes:
      - grafana-data:/var/lib/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=secret
    depends_on:
      - prometheus

  cadvisor:
    image: gcr.io/cadvisor/cadvisor
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8080:8080"

  node-exporter:
    image: prom/node-exporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($$|/)'
    ports:
      - "9100:9100"

volumes:
  grafana-data:
```

### 2. Log Management System
```yaml
version: '3.8'

services:
  web:
    image: nginx
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
        tag: "{{.Name}}"

  logspout:
    image: gliderlabs/logspout
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: syslog://logstash:5000
    depends_on:
      - logstash

  logstash:
    image: docker.elastic.co/logstash/logstash:7.9.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5000:5000"

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.9.0
    environment:
      - discovery.type=single-node
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data

  kibana:
    image: docker.elastic.co/kibana/kibana:7.9.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

volumes:
  elasticsearch-data:
```

## Next Steps
- Once you're comfortable with Docker monitoring, move on to Lab 16: Docker CI/CD
- Practice implementing these monitoring patterns in your projects

## Additional Resources
- [Docker Monitoring Documentation](https://docs.docker.com/config/containers/runmetrics/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Logging Best Practices](https://docs.docker.com/config/containers/logging/) 