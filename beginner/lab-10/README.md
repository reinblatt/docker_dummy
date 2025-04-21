# Lab 10: Basic Container Orchestration

## Objective
In this lab, you will learn the fundamentals of container orchestration using Docker Swarm. You'll understand how to create and manage a swarm, deploy services, and handle basic scaling and failover scenarios.

## Prerequisites
- Completed Lab 09 (Docker Best Practices)
- Docker Desktop running on your system
- Basic understanding of networking concepts
- Familiarity with Docker Compose

## Key Concepts
- Swarm: A cluster of Docker engines running in swarm mode
- Node: A single Docker engine participating in the swarm
- Service: A definition of tasks to execute on nodes
- Task: A running container that is part of a service
- Replica: Multiple instances of the same service

## Exercises

### 1. Initialize a Swarm
1. Initialize a swarm on your local machine:
```powershell
docker swarm init
```

2. View swarm information:
```powershell
docker info
```

3. List nodes in the swarm:
```powershell
docker node ls
```

### 2. Create and Manage Services
1. Create a simple service:
```powershell
docker service create --name web --replicas 3 -p 8080:80 nginx
```

2. List services:
```powershell
docker service ls
```

3. View service details:
```powershell
docker service inspect web
```

4. Scale the service:
```powershell
docker service scale web=5
```

### 3. Service Updates and Rollbacks
1. Update a service:
```powershell
docker service update --image nginx:1.19 web
```

2. Rollback a service:
```powershell
docker service rollback web
```

3. Update service configuration:
```powershell
docker service update --replicas 3 --update-parallelism 2 --update-delay 10s web
```

## Common Swarm Commands
```powershell
# Initialize swarm
docker swarm init

# Join swarm
docker swarm join --token <token> <manager-ip>:<port>

# Create service
docker service create [options] <image>

# List services
docker service ls

# Inspect service
docker service inspect <service-name>

# Scale service
docker service scale <service-name>=<replicas>

# Update service
docker service update [options] <service-name>

# Remove service
docker service rm <service-name>
```

## Service Configuration

### 1. Basic Service
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    deploy:
      replicas: 3
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
    ports:
      - "8080:80"
```

### 2. Service with Resources
```yaml
services:
  web:
    image: nginx:latest
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
      placement:
        constraints:
          - node.role == worker
```

### 3. Service with Health Checks
```yaml
services:
  web:
    image: nginx:latest
    deploy:
      replicas: 3
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
```

## Best Practices
1. Use appropriate number of replicas
2. Configure update strategies
3. Set resource limits
4. Implement health checks
5. Use placement constraints

## Challenges
1. Create a load-balanced web application
2. Implement rolling updates
3. Configure service discovery
4. Set up a multi-node swarm
5. Implement failover scenarios

## Real-world Examples

### 1. Load-Balanced Web Application
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    deploy:
      replicas: 3
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
    ports:
      - "8080:80"
    networks:
      - webnet

networks:
  webnet:
    driver: overlay
```

### 2. Multi-Service Application
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    deploy:
      replicas: 3
    ports:
      - "8080:80"
    depends_on:
      - api

  api:
    image: my-api:latest
    deploy:
      replicas: 2
    environment:
      - DB_HOST=db
    depends_on:
      - db

  db:
    image: mysql:5.7
    deploy:
      replicas: 1
    environment:
      MYSQL_ROOT_PASSWORD: secret
    volumes:
      - db-data:/var/lib/mysql

volumes:
  db-data:
```

## Next Steps
- Once you're comfortable with basic orchestration, move on to Lab 11: Advanced Dockerfile Techniques
- Practice creating and managing different types of services

## Additional Resources
- [Docker Swarm Documentation](https://docs.docker.com/engine/swarm/)
- [Swarm Mode Tutorial](https://docs.docker.com/engine/swarm/swarm-tutorial/)
- [Service Configuration Reference](https://docs.docker.com/engine/reference/commandline/service_create/) 