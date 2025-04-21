# Lab 17: Docker Swarm Advanced

## Objective
In this lab, you will learn advanced Docker Swarm concepts and features. You'll understand how to implement sophisticated orchestration patterns, manage complex service deployments, and optimize Swarm performance.

## Prerequisites
- Completed Lab 16 (Docker CI/CD)
- Docker Desktop running on your system
- Basic understanding of Docker Swarm (Lab 10)
- Familiarity with Docker Compose
- Basic networking knowledge

## Key Concepts
- Advanced Service Configuration: Complex service definitions and constraints
- Swarm Networking: Overlay networks, ingress networks, and custom networks
- Resource Management: Resource limits, reservations, and placement constraints
- High Availability: Service resilience and failover strategies
- Security: Swarm security features and best practices

## Exercises

### 1. Advanced Service Configuration
1. Create a service with complex configuration:
```powershell
# Create a service with multiple replicas and update configuration
docker service create \
    --name web \
    --replicas 3 \
    --update-parallelism 1 \
    --update-delay 10s \
    --update-order start-first \
    --rollback-parallelism 0 \
    --rollback-order stop-first \
    --restart-condition on-failure \
    --restart-delay 5s \
    --restart-max-attempts 3 \
    --restart-window 120s \
    --health-cmd "curl -f http://localhost:80/health" \
    --health-interval 30s \
    --health-timeout 10s \
    --health-retries 3 \
    --health-start-period 40s \
    nginx:latest
```

2. Configure service constraints:
```powershell
# Create a service with placement constraints
docker service create \
    --name db \
    --constraint node.role==manager \
    --constraint node.labels.storage==ssd \
    --placement-pref spread=node.labels.zone \
    postgres:latest
```

### 2. Swarm Networking
1. Create overlay networks:
```powershell
# Create an overlay network
docker network create --driver overlay --attachable my-overlay

# Create a service using the overlay network
docker service create \
    --name web \
    --network my-overlay \
    --publish published=80,target=80 \
    nginx:latest
```

2. Configure network options:
```powershell
# Create a network with custom options
docker network create \
    --driver overlay \
    --subnet 10.0.9.0/24 \
    --gateway 10.0.9.1 \
    --opt encrypted=true \
    --opt com.docker.network.driver.overlay.vxlanid_list=4096 \
    secure-overlay
```

### 3. Resource Management
1. Configure resource limits:
```powershell
# Create a service with resource limits
docker service create \
    --name resource-limited \
    --limit-cpu 2 \
    --limit-memory 1GB \
    --reserve-cpu 1 \
    --reserve-memory 512MB \
    --replicas 3 \
    nginx:latest
```

2. Monitor resource usage:
```powershell
# View service resource usage
docker service ps --format "table {{.Name}}\t{{.Node}}\t{{.CPU %}}\t{{.MemUsage}}" resource-limited
```

### 4. High Availability
1. Configure service resilience:
```powershell
# Create a highly available service
docker service create \
    --name ha-service \
    --replicas 5 \
    --update-parallelism 2 \
    --update-delay 5s \
    --rollback-parallelism 1 \
    --rollback-delay 5s \
    --restart-condition any \
    --restart-max-attempts 5 \
    --restart-window 60s \
    --health-cmd "curl -f http://localhost:80/health" \
    --health-interval 10s \
    --health-timeout 5s \
    --health-retries 3 \
    nginx:latest
```

2. Implement failover strategy:
```powershell
# Create a service with failover configuration
docker service create \
    --name failover-service \
    --replicas 3 \
    --update-parallelism 1 \
    --update-delay 10s \
    --update-failure-action rollback \
    --update-monitor 30s \
    --update-max-failure-ratio 0.1 \
    --rollback-parallelism 1 \
    --rollback-delay 5s \
    --rollback-monitor 20s \
    --rollback-max-failure-ratio 0.1 \
    nginx:latest
```

## Best Practices

### 1. Service Configuration
- Use appropriate update strategies
- Implement health checks
- Configure proper restart policies
- Set resource limits and reservations
- Use placement constraints effectively

### 2. Networking
- Use overlay networks for service communication
- Implement network encryption
- Configure appropriate network drivers
- Use network segmentation
- Monitor network performance

### 3. Resource Management
- Set appropriate resource limits
- Use resource reservations
- Monitor resource usage
- Implement auto-scaling
- Optimize resource allocation

### 4. High Availability
- Implement proper failover strategies
- Use multiple replicas
- Configure health checks
- Monitor service health
- Implement backup strategies

## Common Patterns

### 1. Complex Service
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
        failure_action: rollback
        monitor: 30s
        max_failure_ratio: 0.1
      rollback_config:
        parallelism: 1
        delay: 5s
        order: stop-first
        failure_action: pause
        monitor: 20s
        max_failure_ratio: 0.1
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
      placement:
        constraints:
          - node.role == worker
        preferences:
          - spread: node.labels.zone
      resources:
        limits:
          cpus: '2'
          memory: 1G
        reservations:
          cpus: '1'
          memory: 512M
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### 2. Network Configuration
```yaml
networks:
  secure-overlay:
    driver: overlay
    driver_opts:
      encrypted: "true"
    ipam:
      driver: default
      config:
        - subnet: 10.0.9.0/24
          gateway: 10.0.9.1
```

### 3. Resource Management
```yaml
services:
  resource-limited:
    image: nginx:latest
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1G
        reservations:
          cpus: '1'
          memory: 512M
      placement:
        constraints:
          - node.labels.storage == ssd
```

### 4. High Availability
```yaml
services:
  ha-service:
    image: nginx:latest
    deploy:
      replicas: 5
      update_config:
        parallelism: 2
        delay: 5s
        order: start-first
      restart_policy:
        condition: any
        max_attempts: 5
        window: 60s
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]
      interval: 10s
      timeout: 5s
      retries: 3
```

## Challenges
1. Create a highly available web service
2. Implement a secure overlay network
3. Configure resource management
4. Set up service failover
5. Optimize service deployment

## Real-world Examples

### 1. Complete Swarm Stack
```yaml
version: '3.8'

networks:
  secure-overlay:
    driver: overlay
    driver_opts:
      encrypted: "true"
    ipam:
      driver: default
      config:
        - subnet: 10.0.9.0/24
          gateway: 10.0.9.1

services:
  web:
    image: nginx:latest
    networks:
      - secure-overlay
    deploy:
      replicas: 5
      update_config:
        parallelism: 2
        delay: 5s
        order: start-first
        failure_action: rollback
      rollback_config:
        parallelism: 1
        delay: 5s
        order: stop-first
      restart_policy:
        condition: any
        max_attempts: 5
        window: 60s
      placement:
        constraints:
          - node.role == worker
        preferences:
          - spread: node.labels.zone
      resources:
        limits:
          cpus: '2'
          memory: 1G
        reservations:
          cpus: '1'
          memory: 512M
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]
      interval: 10s
      timeout: 5s
      retries: 3

  db:
    image: postgres:latest
    networks:
      - secure-overlay
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        order: stop-first
      placement:
        constraints:
          - node.labels.storage == ssd
      resources:
        limits:
          cpus: '4'
          memory: 4G
        reservations:
          cpus: '2'
          memory: 2G
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 3
```

### 2. Multi-tier Application
```yaml
version: '3.8'

networks:
  frontend:
    driver: overlay
  backend:
    driver: overlay
    driver_opts:
      encrypted: "true"

services:
  frontend:
    image: nginx:latest
    networks:
      - frontend
    deploy:
      replicas: 5
      update_config:
        parallelism: 2
        delay: 5s
      placement:
        constraints:
          - node.role == worker
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]

  backend:
    image: node:latest
    networks:
      - frontend
      - backend
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      placement:
        constraints:
          - node.labels.tier == backend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]

  database:
    image: postgres:latest
    networks:
      - backend
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        order: stop-first
      placement:
        constraints:
          - node.labels.storage == ssd
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
```

## Next Steps
- Once you're comfortable with advanced Docker Swarm concepts, move on to Lab 18: Docker Security
- Practice implementing these advanced Swarm patterns in your projects

## Additional Resources
- [Docker Swarm Documentation](https://docs.docker.com/engine/swarm/)
- [Swarm Mode Tutorial](https://docs.docker.com/engine/swarm/swarm-tutorial/)
- [Swarm Mode Key Concepts](https://docs.docker.com/engine/swarm/key-concepts/)
- [Swarm Mode Best Practices](https://docs.docker.com/engine/swarm/swarm-mode-best-practices/) 