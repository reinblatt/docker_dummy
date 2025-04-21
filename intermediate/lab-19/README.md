# Lab 19: Docker Performance Optimization

## Objective
In this lab, you will learn how to optimize Docker performance through various techniques and configurations. You'll understand how to improve container performance, resource utilization, and overall system efficiency.

## Prerequisites
- Completed Lab 18 (Docker Security)
- Docker Desktop running on your system
- Basic understanding of system performance concepts
- Familiarity with Docker Compose
- Access to performance monitoring tools

## Key Concepts
- Resource Optimization: CPU, memory, and I/O management
- Image Optimization: Layer optimization and build caching
- Network Performance: Network driver optimization and bandwidth management
- Storage Performance: Volume optimization and storage driver selection
- Monitoring and Tuning: Performance metrics and optimization techniques

## Exercises

### 1. Resource Optimization
1. Configure resource limits:
```powershell
# Run container with optimized resources
docker run -d \
    --cpus=2 \
    --memory=2g \
    --memory-swap=3g \
    --memory-reservation=1g \
    --cpu-shares=512 \
    --cpu-period=100000 \
    --cpu-quota=50000 \
    --blkio-weight=500 \
    --device-read-bps=/dev/sda:1mb \
    --device-write-bps=/dev/sda:1mb \
    nginx:latest
```

2. Monitor resource usage:
```powershell
# View container stats
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}\t{{.BlockIO}}"

# View detailed container metrics
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
    --privileged \
    --pid=host \
    --name cadvisor \
    google/cadvisor:latest
```

### 2. Image Optimization
1. Optimize Dockerfile:
```dockerfile
# Multi-stage build
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production image
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

2. Use build cache effectively:
```powershell
# Build with cache optimization
docker build \
    --cache-from=myapp:latest \
    --build-arg BUILDKIT_INLINE_CACHE=1 \
    -t myapp:latest .

# Use BuildKit for better performance
DOCKER_BUILDKIT=1 docker build -t myapp:latest .
```

### 3. Network Performance
1. Configure network optimization:
```powershell
# Create optimized network
docker network create \
    --driver bridge \
    --opt com.docker.network.bridge.name=br0 \
    --opt com.docker.network.bridge.enable_icc=true \
    --opt com.docker.network.bridge.enable_ip_masquerade=true \
    --opt com.docker.network.bridge.mtu=1500 \
    optimized-network

# Run container with network optimization
docker run -d \
    --network optimized-network \
    --sysctl net.core.somaxconn=1024 \
    --sysctl net.ipv4.tcp_max_syn_backlog=1024 \
    --sysctl net.ipv4.tcp_syncookies=1 \
    nginx:latest
```

2. Monitor network performance:
```powershell
# Install iperf3
docker run -it --rm networkstatic/iperf3 -s

# Test network performance
docker run -it --rm networkstatic/iperf3 -c <server-ip>
```

### 4. Storage Performance
1. Configure storage optimization:
```powershell
# Create optimized volume
docker volume create \
    --driver local \
    --opt type=tmpfs \
    --opt device=tmpfs \
    --opt o=size=100m,uid=1000 \
    optimized-volume

# Run container with storage optimization
docker run -d \
    --mount type=volume,source=optimized-volume,target=/data \
    --mount type=tmpfs,target=/tmp,tmpfs-size=100m \
    nginx:latest
```

2. Monitor storage performance:
```powershell
# Install ioping
docker run -it --rm --privileged \
    -v /:/host \
    --pid=host \
    --name ioping \
    alpine:latest \
    apk add ioping && \
    ioping -c 10 /host/var/lib/docker
```

## Best Practices

### 1. Resource Management
- Set appropriate resource limits
- Use resource reservations
- Monitor resource usage
- Implement auto-scaling
- Optimize resource allocation

### 2. Image Optimization
- Use multi-stage builds
- Minimize layers
- Leverage build cache
- Use appropriate base images
- Remove unnecessary files

### 3. Network Optimization
- Choose appropriate network driver
- Optimize network settings
- Monitor network performance
- Use network segmentation
- Implement load balancing

### 4. Storage Optimization
- Choose appropriate storage driver
- Use volumes effectively
- Implement caching strategies
- Monitor I/O performance
- Optimize file system

## Common Patterns

### 1. Optimized Container
```yaml
services:
  optimized-app:
    image: nginx:latest
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G
    sysctls:
      net.core.somaxconn: 1024
      net.ipv4.tcp_max_syn_backlog: 1024
    volumes:
      - type: tmpfs
        target: /tmp
        tmpfs:
          size: 100m
```

### 2. Multi-stage Build
```dockerfile
# Build stage
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 3. Network Configuration
```yaml
networks:
  optimized:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: br0
      com.docker.network.bridge.enable_icc: "true"
      com.docker.network.bridge.enable_ip_masquerade: "true"
      com.docker.network.bridge.mtu: "1500"
```

### 4. Storage Configuration
```yaml
volumes:
  optimized-data:
    driver: local
    driver_opts:
      type: tmpfs
      device: tmpfs
      o: size=100m,uid=1000
```

## Challenges
1. Optimize container resource usage
2. Implement multi-stage builds
3. Configure network performance
4. Optimize storage performance
5. Set up performance monitoring

## Real-world Examples

### 1. Complete Optimized Stack
```yaml
version: '3.8'

networks:
  optimized:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: br0
      com.docker.network.bridge.enable_icc: "true"
      com.docker.network.bridge.enable_ip_masquerade: "true"
      com.docker.network.bridge.mtu: "1500"

volumes:
  app-data:
    driver: local
    driver_opts:
      type: tmpfs
      device: tmpfs
      o: size=100m,uid=1000

services:
  web:
    image: nginx:latest
    networks:
      - optimized
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G
    sysctls:
      net.core.somaxconn: 1024
      net.ipv4.tcp_max_syn_backlog: 1024
    volumes:
      - type: tmpfs
        target: /tmp
        tmpfs:
          size: 100m
    ports:
      - "80:80"

  api:
    image: node:latest
    networks:
      - optimized
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 4G
        reservations:
          cpus: '2'
          memory: 2G
    volumes:
      - app-data:/app/data
    environment:
      - NODE_ENV=production
      - NODE_OPTIONS=--max-old-space-size=2048

  db:
    image: postgres:latest
    networks:
      - optimized
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 4G
        reservations:
          cpus: '2'
          memory: 2G
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    secrets:
      - db_password
```

### 2. Performance Monitoring Stack
```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

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

## Next Steps
- Once you're comfortable with Docker performance optimization, move on to Lab 20: Docker Enterprise Features
- Practice implementing these optimization patterns in your projects

## Additional Resources
- [Docker Performance Documentation](https://docs.docker.com/config/containers/runmetrics/)
- [Docker Build Performance](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Docker Storage Drivers](https://docs.docker.com/storage/storagedriver/)
- [Docker Network Performance](https://docs.docker.com/network/network-tutorial-standalone/) 