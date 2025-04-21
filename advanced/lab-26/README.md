# Lab 26: Docker Performance Advanced

## Objective
In this lab, you will learn advanced Docker performance optimization techniques. You'll understand how to implement sophisticated resource management, performance monitoring, and optimization strategies for containerized applications.

## Prerequisites
- Completed Lab 25 (Docker Security Advanced)
- Docker Desktop running on your system
- Basic understanding of performance concepts
- Familiarity with Docker Compose
- Access to Docker Swarm (for distributed performance)

## Key Concepts
- Resource Management: Advanced resource allocation
- Performance Monitoring: Real-time metrics collection
- Optimization Techniques: Performance tuning
- Load Balancing: Traffic distribution
- Caching Strategies: Performance optimization

## Exercises

### 1. Resource Management
1. Configure advanced resource limits:
```powershell
# Set CPU and memory limits
docker run --rm \
    --cpus=2 \
    --cpu-shares=1024 \
    --cpu-period=100000 \
    --cpu-quota=200000 \
    --memory=1g \
    --memory-swap=2g \
    --memory-reservation=512m \
    --memory-swappiness=0 \
    nginx:latest

# Configure I/O limits
docker run --rm \
    --device-read-bps=/dev/sda:1mb \
    --device-write-bps=/dev/sda:1mb \
    --device-read-iops=/dev/sda:100 \
    --device-write-iops=/dev/sda:100 \
    --blkio-weight=500 \
    nginx:latest
```

2. Implement resource reservations:
```powershell
# Set resource reservations
docker run --rm \
    --cpus=2 \
    --cpu-shares=1024 \
    --memory=1g \
    --memory-swap=2g \
    --memory-reservation=512m \
    --memory-swappiness=0 \
    --blkio-weight=500 \
    --oom-kill-disable \
    --oom-score-adj=100 \
    nginx:latest
```

### 2. Performance Monitoring
1. Set up performance monitoring:
```powershell
# Install cAdvisor
docker run -d \
    --name=cadvisor \
    --volume=/:/rootfs:ro \
    --volume=/var/run:/var/run:ro \
    --volume=/sys:/sys:ro \
    --volume=/var/lib/docker/:/var/lib/docker:ro \
    --volume=/dev/disk/:/dev/disk:ro \
    --publish=8080:8080 \
    --detach=true \
    gcr.io/cadvisor/cadvisor:latest

# Monitor container metrics
docker run --rm \
    --pid=host \
    --privileged \
    nicolaka/netshoot \
    top -b -n 1
```

2. Configure metrics collection:
```powershell
# Set up Prometheus
docker run -d \
    --name=prometheus \
    -p 9090:9090 \
    -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus:latest

# Configure Grafana
docker run -d \
    --name=grafana \
    -p 3000:3000 \
    -v grafana-storage:/var/lib/grafana \
    grafana/grafana:latest
```

### 3. Optimization Techniques
1. Implement performance optimizations:
```powershell
# Configure container performance
docker run --rm \
    --cpus=2 \
    --cpu-shares=1024 \
    --memory=1g \
    --memory-swap=2g \
    --memory-reservation=512m \
    --memory-swappiness=0 \
    --blkio-weight=500 \
    --oom-kill-disable \
    --oom-score-adj=100 \
    --device-read-bps=/dev/sda:1mb \
    --device-write-bps=/dev/sda:1mb \
    --device-read-iops=/dev/sda:100 \
    --device-write-iops=/dev/sda:100 \
    nginx:latest
```

2. Configure caching:
```powershell
# Set up caching
docker run --rm \
    --cpus=2 \
    --cpu-shares=1024 \
    --memory=1g \
    --memory-swap=2g \
    --memory-reservation=512m \
    --memory-swappiness=0 \
    --blkio-weight=500 \
    --oom-kill-disable \
    --oom-score-adj=100 \
    --device-read-bps=/dev/sda:1mb \
    --device-write-bps=/dev/sda:1mb \
    --device-read-iops=/dev/sda:100 \
    --device-write-iops=/dev/sda:100 \
    --tmpfs /tmp:rw,noexec,nosuid,size=65536k \
    nginx:latest
```

### 4. Load Balancing
1. Configure load balancing:
```powershell
# Set up load balancer
docker run -d \
    --name=nginx \
    -p 80:80 \
    -p 443:443 \
    -v /path/to/nginx.conf:/etc/nginx/nginx.conf \
    nginx:latest

# Configure load balancing
docker run -d \
    --name=haproxy \
    -p 80:80 \
    -p 443:443 \
    -v /path/to/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg \
    haproxy:latest
```

2. Implement traffic distribution:
```powershell
# Configure traffic distribution
docker run -d \
    --name=traefik \
    -p 80:80 \
    -p 443:443 \
    -v /var/run/docker.sock:/var/run/docker.sock \
    traefik:latest
```

## Best Practices

### 1. Resource Management
- Set appropriate resource limits
- Configure resource reservations
- Monitor resource usage
- Implement resource isolation
- Regular resource optimization

### 2. Performance Monitoring
- Set up comprehensive monitoring
- Configure metrics collection
- Implement alerting
- Regular performance analysis
- Document performance metrics

### 3. Optimization
- Implement caching strategies
- Configure load balancing
- Optimize container settings
- Regular performance testing
- Capacity planning

### 4. Load Balancing
- Configure traffic distribution
- Implement health checks
- Monitor load balancer
- Regular load testing
- Document load patterns

## Common Patterns

### 1. Resource Configuration
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1G
        reservations:
          cpus: '1'
          memory: 512M
      restart_policy:
        condition: on-failure
```

### 2. Monitoring Setup
```yaml
version: '3.8'

services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    ports:
      - "8080:8080"
    deploy:
      mode: global

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:latest
    volumes:
      - grafana-storage:/var/lib/grafana
    ports:
      - "3000:3000"

volumes:
  grafana-storage:
```

### 3. Load Balancer Configuration
```yaml
version: '3.8'

services:
  haproxy:
    image: haproxy:latest
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg
    ports:
      - "80:80"
      - "443:443"
    deploy:
      mode: global

  web:
    image: nginx:latest
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '2'
          memory: 1G
        reservations:
          cpus: '1'
          memory: 512M
      restart_policy:
        condition: on-failure
```

### 4. Performance Optimization
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1G
        reservations:
          cpus: '1'
          memory: 512M
      restart_policy:
        condition: on-failure
    tmpfs:
      - /tmp:rw,noexec,nosuid,size=65536k
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
```

## Challenges
1. Implement resource optimization
2. Set up performance monitoring
3. Configure load balancing
4. Implement caching strategies
5. Optimize container performance

## Real-world Examples

### 1. Complete Performance Stack
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '2'
          memory: 1G
        reservations:
          cpus: '1'
          memory: 512M
      restart_policy:
        condition: on-failure
    tmpfs:
      - /tmp:rw,noexec,nosuid,size=65536k
    ulimits:
      nofile:
        soft: 65536
        hard: 65536

  haproxy:
    image: haproxy:latest
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg
    ports:
      - "80:80"
      - "443:443"
    deploy:
      mode: global

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    ports:
      - "8080:8080"
    deploy:
      mode: global

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:latest
    volumes:
      - grafana-storage:/var/lib/grafana
    ports:
      - "3000:3000"

volumes:
  grafana-storage:
```

### 2. Load Balancing Stack
```yaml
version: '3.8'

services:
  haproxy:
    image: haproxy:latest
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg
    ports:
      - "80:80"
      - "443:443"
    deploy:
      mode: global

  web:
    image: nginx:latest
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '2'
          memory: 1G
        reservations:
          cpus: '1'
          memory: 512M
      restart_policy:
        condition: on-failure
    tmpfs:
      - /tmp:rw,noexec,nosuid,size=65536k
    ulimits:
      nofile:
        soft: 65536
        hard: 65536

  monitor:
    image: nicolaka/netshoot
    command: >
      sh -c "
        while true; do
          curl -s http://haproxy/stats | grep -i 'web';
          sleep 5;
        done
      "
    deploy:
      mode: global
```

## Next Steps
- Move on to Lab 27: Docker CI/CD Advanced
- Practice implementing advanced performance in your projects
- Explore additional performance tools and techniques

## Additional Resources
- [Docker Performance Documentation](https://docs.docker.com/config/containers/resource_constraints/)
- [Performance Tuning Guide](https://docs.docker.com/config/containers/resource_constraints/)
- [Load Balancing Guide](https://docs.docker.com/engine/swarm/ingress/)
- [Monitoring Guide](https://docs.docker.com/config/daemon/prometheus/) 