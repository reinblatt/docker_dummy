# Lab 28: Docker Orchestration Advanced

## Objective
In this lab, you will learn advanced Docker orchestration concepts and techniques. You'll understand how to implement sophisticated container orchestration, including service discovery, load balancing, scaling strategies, and advanced deployment patterns.

## Prerequisites
- Completed Lab 27 (Docker CI/CD Advanced)
- Docker Desktop running on your system
- Basic understanding of orchestration concepts
- Familiarity with Docker Compose
- Access to Docker Swarm

## Key Concepts
- Service Discovery: Dynamic service registration
- Load Balancing: Advanced traffic distribution
- Scaling Strategies: Dynamic resource allocation
- Deployment Patterns: Complex service deployment
- High Availability: Fault tolerance and recovery

## Exercises

### 1. Service Discovery
1. Configure service discovery:
```yaml
# docker-compose.discovery.yml
version: '3.8'

services:
  consul:
    image: consul:latest
    ports:
      - "8500:8500"
    volumes:
      - consul-data:/consul/data
    command: agent -server -bootstrap-expect=1 -ui -client=0.0.0.0

  registrator:
    image: gliderlabs/registrator:latest
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock
    command: -internal consul://consul:8500

  web:
    image: nginx:latest
    environment:
      - SERVICE_NAME=web
      - SERVICE_TAGS=nginx,web
    depends_on:
      - consul
```

2. Implement health checks:
```yaml
# docker-compose.health.yml
version: '3.8'

services:
  web:
    image: nginx:latest
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    deploy:
      mode: replicated
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
```

### 2. Load Balancing
1. Configure advanced load balancing:
```yaml
# docker-compose.lb.yml
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
      placement:
        constraints:
          - node.role == manager

  web:
    image: nginx:latest
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      restart_policy:
        condition: on-failure
```

2. Implement sticky sessions:
```yaml
# docker-compose.sticky.yml
version: '3.8'

services:
  traefik:
    image: traefik:latest
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.swarmmode=true"
      - "--entrypoints.web.address=:80"
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    deploy:
      mode: global

  web:
    image: nginx:latest
    deploy:
      replicas: 3
      labels:
        - "traefik.http.services.web.loadbalancer.sticky=true"
        - "traefik.http.services.web.loadbalancer.sticky.cookie.name=sticky"
```

### 3. Scaling Strategies
1. Configure auto-scaling:
```yaml
# docker-compose.scale.yml
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
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      restart_policy:
        condition: on-failure
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

2. Implement resource-based scaling:
```yaml
# docker-compose.resource.yml
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
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      restart_policy:
        condition: on-failure
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### 4. Deployment Patterns
1. Implement blue-green deployment:
```yaml
# docker-compose.blue-green.yml
version: '3.8'

services:
  web-blue:
    image: nginx:latest
    deploy:
      replicas: 3
      labels:
        - "traefik.http.routers.web-blue.rule=Host(`blue.example.com`)"
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      restart_policy:
        condition: on-failure

  web-green:
    image: nginx:latest
    deploy:
      replicas: 3
      labels:
        - "traefik.http.routers.web-green.rule=Host(`green.example.com`)"
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      restart_policy:
        condition: on-failure
```

2. Configure canary deployment:
```yaml
# docker-compose.canary.yml
version: '3.8'

services:
  web:
    image: nginx:latest
    deploy:
      replicas: 10
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      restart_policy:
        condition: on-failure
    labels:
      - "traefik.http.routers.web.rule=Host(`example.com`)"
      - "traefik.http.services.web.loadbalancer.weights=90,10"

  web-canary:
    image: nginx:canary
    deploy:
      replicas: 1
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      restart_policy:
        condition: on-failure
    labels:
      - "traefik.http.routers.web-canary.rule=Host(`example.com`)"
      - "traefik.http.services.web-canary.loadbalancer.weights=90,10"
```

## Best Practices

### 1. Service Discovery
- Implement dynamic service registration
- Configure health checks
- Use appropriate discovery tools
- Monitor service health
- Document service configurations

### 2. Load Balancing
- Configure appropriate load balancing
- Implement sticky sessions
- Monitor traffic distribution
- Set up health checks
- Document load balancing rules

### 3. Scaling
- Implement auto-scaling
- Configure resource limits
- Monitor resource usage
- Set up scaling triggers
- Document scaling policies

### 4. Deployment
- Use appropriate deployment strategies
- Implement health checks
- Configure rollback procedures
- Monitor deployments
- Document deployment processes

## Common Patterns

### 1. Service Discovery Configuration
```yaml
version: '3.8'

services:
  consul:
    image: consul:latest
    ports:
      - "8500:8500"
    volumes:
      - consul-data:/consul/data
    command: agent -server -bootstrap-expect=1 -ui -client=0.0.0.0

  registrator:
    image: gliderlabs/registrator:latest
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock
    command: -internal consul://consul:8500

  web:
    image: nginx:latest
    environment:
      - SERVICE_NAME=web
      - SERVICE_TAGS=nginx,web
    depends_on:
      - consul
```

### 2. Load Balancer Configuration
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
      placement:
        constraints:
          - node.role == manager

  web:
    image: nginx:latest
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      restart_policy:
        condition: on-failure
```

### 3. Scaling Configuration
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
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      restart_policy:
        condition: on-failure
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### 4. Deployment Configuration
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
      rollback_config:
        parallelism: 1
        delay: 10s
        order: stop-first
      restart_policy:
        condition: on-failure
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

## Challenges
1. Implement service discovery
2. Configure load balancing
3. Set up auto-scaling
4. Implement deployment strategies
5. Configure high availability

## Real-world Examples

### 1. Complete Orchestration Stack
```yaml
version: '3.8'

services:
  consul:
    image: consul:latest
    ports:
      - "8500:8500"
    volumes:
      - consul-data:/consul/data
    command: agent -server -bootstrap-expect=1 -ui -client=0.0.0.0

  registrator:
    image: gliderlabs/registrator:latest
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock
    command: -internal consul://consul:8500

  haproxy:
    image: haproxy:latest
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg
    ports:
      - "80:80"
      - "443:443"
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager

  web:
    image: nginx:latest
    environment:
      - SERVICE_NAME=web
      - SERVICE_TAGS=nginx,web
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '2'
          memory: 1G
        reservations:
          cpus: '1'
          memory: 512M
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      restart_policy:
        condition: on-failure
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  consul-data:
```

### 2. High Availability Stack
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
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      restart_policy:
        condition: on-failure
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  haproxy:
    image: haproxy:latest
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg
    ports:
      - "80:80"
      - "443:443"
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

## Next Steps
- Move on to Lab 29: Docker Security Advanced
- Practice implementing advanced orchestration in your projects
- Explore additional orchestration tools and techniques

## Additional Resources
- [Docker Swarm Documentation](https://docs.docker.com/engine/swarm/)
- [Service Discovery Guide](https://docs.docker.com/engine/swarm/services/)
- [Load Balancing Guide](https://docs.docker.com/engine/swarm/ingress/)
- [Scaling Guide](https://docs.docker.com/engine/swarm/swarm-tutorial/scale-service/) 