# Lab 13: Docker Networking

## Objective
In this lab, you will learn advanced Docker networking concepts including network drivers, overlays, service discovery, and load balancing. You'll understand how to create and manage complex network configurations for containerized applications.

## Prerequisites
- Completed Lab 12 (Docker Security)
- Docker Desktop running on your system
- Basic understanding of networking concepts
- Familiarity with Docker Compose

## Key Concepts
- Network Drivers: Different types of network drivers and their use cases
- Overlay Networks: Multi-host networking for Docker Swarm
- Service Discovery: Automatic service registration and discovery
- Load Balancing: Distributing traffic across multiple containers
- Network Troubleshooting: Tools and techniques for debugging

## Exercises

### 1. Network Drivers
1. Create different types of networks:
```powershell
# Bridge network (default)
docker network create --driver bridge my-bridge

# Host network
docker network create --driver host my-host

# Overlay network
docker network create --driver overlay my-overlay

# Macvlan network
docker network create --driver macvlan \
    --subnet=192.168.1.0/24 \
    --gateway=192.168.1.1 \
    -o parent=eth0 my-macvlan
```

2. List and inspect networks:
```powershell
# List all networks
docker network ls

# Inspect a network
docker network inspect my-bridge

# Connect a container to a network
docker run -d --network my-bridge --name web nginx

# Connect to multiple networks
docker network connect my-overlay web
```

### 2. Overlay Networks
1. Create an overlay network in Swarm mode:
```powershell
# Initialize swarm
docker swarm init

# Create overlay network
docker network create --driver overlay --attachable my-overlay

# Create a service on the overlay network
docker service create --name web \
    --network my-overlay \
    --replicas 3 \
    nginx
```

2. Configure network options:
```yaml
version: '3.8'

networks:
  my-overlay:
    driver: overlay
    attachable: true
    driver_opts:
      encrypted: "true"
    ipam:
      driver: default
      config:
        - subnet: 10.0.0.0/24
          gateway: 10.0.0.1
```

### 3. Service Discovery
1. Use Docker's built-in DNS:
```powershell
# Create a service
docker service create --name web --network my-overlay nginx

# Create another service that can resolve the first
docker service create --name client \
    --network my-overlay \
    alpine ping web
```

2. Configure custom DNS:
```yaml
version: '3.8'

services:
  web:
    image: nginx
    networks:
      - my-overlay
    dns:
      - 8.8.8.8
      - 8.8.4.4
    dns_search:
      - example.com

networks:
  my-overlay:
    driver: overlay
```

### 4. Load Balancing
1. Configure load balancing in Swarm:
```powershell
# Create a service with load balancing
docker service create --name web \
    --network my-overlay \
    --replicas 3 \
    --publish published=8080,target=80 \
    nginx
```

2. Set up custom load balancing:
```yaml
version: '3.8'

services:
  web:
    image: nginx
    deploy:
      replicas: 3
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
    networks:
      - my-overlay
    ports:
      - target: 80
        published: 8080
        protocol: tcp
        mode: host

networks:
  my-overlay:
    driver: overlay
```

## Best Practices

### 1. Network Design
- Use appropriate network drivers
- Implement network segmentation
- Configure proper IP addressing
- Use overlay networks for Swarm
- Implement network policies

### 2. Service Discovery
- Use Docker DNS
- Configure custom DNS when needed
- Implement health checks
- Use service labels
- Monitor service status

### 3. Load Balancing
- Use Swarm's built-in load balancing
- Configure proper health checks
- Implement rolling updates
- Monitor service performance
- Use appropriate scaling strategies

### 4. Security
- Use encrypted overlay networks
- Implement network policies
- Use internal networks when possible
- Monitor network traffic
- Implement access controls

## Common Patterns

### 1. Multi-host Networking
```yaml
networks:
  frontend:
    driver: overlay
    attachable: true
  backend:
    driver: overlay
    internal: true
```

### 2. Service Discovery
```yaml
services:
  web:
    image: nginx
    networks:
      - frontend
    dns:
      - 8.8.8.8
    dns_search:
      - example.com
```

### 3. Load Balancing
```yaml
services:
  web:
    deploy:
      replicas: 3
      update_config:
        parallelism: 2
        delay: 10s
    ports:
      - target: 80
        published: 8080
        mode: host
```

### 4. Network Security
```yaml
networks:
  secure:
    driver: overlay
    driver_opts:
      encrypted: "true"
    ipam:
      config:
        - subnet: 10.0.0.0/24
```

## Challenges
1. Set up a multi-host network
2. Implement custom load balancing
3. Configure service discovery
4. Create a secure network topology
5. Troubleshoot network issues

## Real-world Examples

### 1. Microservices Architecture
```yaml
version: '3.8'

services:
  frontend:
    image: nginx
    networks:
      - frontend
    ports:
      - "8080:80"
    deploy:
      replicas: 3
      update_config:
        parallelism: 2
        delay: 10s

  api:
    image: my-api
    networks:
      - frontend
      - backend
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '0.50'
          memory: 512M

  db:
    image: postgres
    networks:
      - backend
    volumes:
      - db-data:/var/lib/postgresql/data
    deploy:
      placement:
        constraints:
          - node.role == manager

networks:
  frontend:
    driver: overlay
    attachable: true
  backend:
    driver: overlay
    internal: true

volumes:
  db-data:
```

### 2. Load-Balanced Web Application
```yaml
version: '3.8'

services:
  web:
    image: nginx
    networks:
      - webnet
    deploy:
      replicas: 5
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
    ports:
      - target: 80
        published: 8080
        protocol: tcp
        mode: host

  lb:
    image: nginx
    networks:
      - webnet
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - "80:80"
    deploy:
      placement:
        constraints:
          - node.role == manager

networks:
  webnet:
    driver: overlay
    attachable: true
```

## Next Steps
- Once you're comfortable with Docker networking, move on to Lab 14: Docker Storage
- Practice implementing these networking patterns in your projects

## Additional Resources
- [Docker Networking Documentation](https://docs.docker.com/network/)
- [Overlay Networks](https://docs.docker.com/network/overlay/)
- [Service Discovery](https://docs.docker.com/engine/swarm/networking/)
- [Load Balancing](https://docs.docker.com/engine/swarm/ingress/) 