# Lab 23: Docker Networking Advanced

## Objective
In this lab, you will learn advanced Docker networking concepts and techniques. You'll understand how to implement overlay networks, service discovery, network policies, and troubleshooting strategies for complex containerized applications.

## Prerequisites
- Completed Lab 22 (Docker Security Scanning)
- Docker Desktop running on your system
- Basic understanding of networking concepts
- Familiarity with Docker Compose
- Access to Docker Swarm (for overlay networks)

## Key Concepts
- Overlay Networks: Multi-host networking
- Service Discovery: Container-to-container communication
- Network Policies: Security and access control
- Network Troubleshooting: Diagnostics and debugging
- Custom Network Drivers: Extending network capabilities

## Exercises

### 1. Overlay Networks
1. Create and manage overlay networks:
```powershell
# Create an overlay network
docker network create \
    --driver overlay \
    --attachable \
    --opt encrypted=true \
    --opt com.docker.network.driver.overlay.vxlanid_list=4096 \
    my-overlay

# List overlay networks
docker network ls --filter driver=overlay

# Inspect network details
docker network inspect my-overlay
```

2. Configure network options:
```powershell
# Create network with custom options
docker network create \
    --driver overlay \
    --attachable \
    --opt encrypted=true \
    --opt com.docker.network.driver.overlay.vxlanid_list=4096 \
    --opt com.docker.network.driver.overlay.vxlanudpport=4789 \
    --opt com.docker.network.driver.overlay.mtu=1450 \
    custom-overlay

# Configure network labels
docker network create \
    --driver overlay \
    --label com.example.environment=production \
    --label com.example.team=devops \
    labeled-overlay
```

### 2. Service Discovery
1. Configure DNS-based discovery:
```powershell
# Create a service with DNS name
docker service create \
    --name web \
    --network my-overlay \
    --dns-search example.com \
    --dns-option timeout:2 \
    --dns-option attempts:3 \
    nginx:latest

# Test DNS resolution
docker run --rm --network my-overlay \
    alpine:latest nslookup web
```

2. Implement service aliases:
```powershell
# Create service with aliases
docker service create \
    --name api \
    --network my-overlay \
    --network-alias backend \
    --network-alias api.example.com \
    nginx:latest

# Test alias resolution
docker run --rm --network my-overlay \
    alpine:latest nslookup backend
```

### 3. Network Policies
1. Configure network access:
```powershell
# Create network with access control
docker network create \
    --driver overlay \
    --opt com.docker.network.driver.overlay.vxlanid_list=4096 \
    --opt com.docker.network.driver.overlay.encrypted=true \
    --opt com.docker.network.driver.overlay.allow_network_access=false \
    secure-overlay

# Allow specific container access
docker network connect \
    --alias allowed-container \
    secure-overlay \
    my-container
```

2. Implement network segmentation:
```powershell
# Create isolated networks
docker network create \
    --driver overlay \
    --opt com.docker.network.driver.overlay.vxlanid_list=4097 \
    --opt com.docker.network.driver.overlay.encrypted=true \
    --opt com.docker.network.driver.overlay.allow_network_access=false \
    isolated-overlay

# Connect services to specific networks
docker service create \
    --name db \
    --network isolated-overlay \
    --network my-overlay \
    postgres:latest
```

### 4. Network Troubleshooting
1. Monitor network traffic:
```powershell
# Install tcpdump
docker run --rm --net=host \
    --cap-add=NET_ADMIN \
    nicolaka/netshoot \
    tcpdump -i eth0 -n

# Monitor network connections
docker run --rm --net=host \
    --cap-add=NET_ADMIN \
    nicolaka/netshoot \
    netstat -tuln
```

2. Debug network issues:
```powershell
# Check network connectivity
docker run --rm --network my-overlay \
    nicolaka/netshoot \
    ping -c 4 web

# Test DNS resolution
docker run --rm --network my-overlay \
    nicolaka/netshoot \
    dig web

# Check network configuration
docker run --rm --network my-overlay \
    nicolaka/netshoot \
    ip addr show
```

## Best Practices

### 1. Network Design
- Use appropriate network drivers
- Implement network segmentation
- Configure proper MTU settings
- Enable network encryption
- Use network labels

### 2. Service Discovery
- Implement DNS-based discovery
- Use service aliases
- Configure DNS options
- Monitor DNS resolution
- Implement fallback mechanisms

### 3. Security
- Enable network encryption
- Implement access control
- Use network policies
- Monitor network traffic
- Regular security audits

### 4. Performance
- Optimize network settings
- Monitor network metrics
- Implement load balancing
- Use appropriate MTU
- Regular performance testing

## Common Patterns

### 1. Overlay Network Configuration
```yaml
version: '3.8'

networks:
  app-network:
    driver: overlay
    attachable: true
    driver_opts:
      encrypted: "true"
      com.docker.network.driver.overlay.vxlanid_list: "4096"
      com.docker.network.driver.overlay.mtu: "1450"
    labels:
      com.example.environment: production
      com.example.team: devops

services:
  web:
    image: nginx:latest
    networks:
      - app-network
    deploy:
      replicas: 3
```

### 2. Service Discovery Setup
```yaml
version: '3.8'

networks:
  app-network:
    driver: overlay
    attachable: true

services:
  api:
    image: nginx:latest
    networks:
      app-network:
        aliases:
          - backend
          - api.example.com
    dns_search:
      - example.com
    dns_options:
      - timeout:2
      - attempts:3
    deploy:
      replicas: 3
```

### 3. Network Policy Configuration
```yaml
version: '3.8'

networks:
  secure-network:
    driver: overlay
    driver_opts:
      com.docker.network.driver.overlay.encrypted: "true"
      com.docker.network.driver.overlay.allow_network_access: "false"

services:
  database:
    image: postgres:latest
    networks:
      - secure-network
    deploy:
      placement:
        constraints:
          - node.role == manager
```

### 4. Monitoring Setup
```yaml
version: '3.8'

services:
  network-monitor:
    image: nicolaka/netshoot
    networks:
      - app-network
    command: >
      sh -c "
        while true; do
          netstat -tuln;
          sleep 60;
        done
      "
    deploy:
      mode: global
```

## Challenges
1. Set up a multi-host overlay network
2. Implement service discovery
3. Configure network policies
4. Set up network monitoring
5. Troubleshoot network issues

## Real-world Examples

### 1. Complete Network Stack
```yaml
version: '3.8'

networks:
  frontend:
    driver: overlay
    attachable: true
    driver_opts:
      encrypted: "true"
      com.docker.network.driver.overlay.vxlanid_list: "4096"
    labels:
      com.example.environment: production
      com.example.team: frontend

  backend:
    driver: overlay
    attachable: true
    driver_opts:
      encrypted: "true"
      com.docker.network.driver.overlay.vxlanid_list: "4097"
    labels:
      com.example.environment: production
      com.example.team: backend

  database:
    driver: overlay
    attachable: true
    driver_opts:
      encrypted: "true"
      com.docker.network.driver.overlay.vxlanid_list: "4098"
      com.docker.network.driver.overlay.allow_network_access: "false"
    labels:
      com.example.environment: production
      com.example.team: database

services:
  web:
    image: nginx:latest
    networks:
      - frontend
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure

  api:
    image: nginx:latest
    networks:
      - frontend
      - backend
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    image: postgres:latest
    networks:
      - backend
      - database
    deploy:
      placement:
        constraints:
          - node.role == manager
      restart_policy:
        condition: on-failure

  monitor:
    image: nicolaka/netshoot
    networks:
      - frontend
      - backend
      - database
    command: >
      sh -c "
        while true; do
          netstat -tuln;
          ping -c 4 web;
          ping -c 4 api;
          ping -c 4 db;
          sleep 60;
        done
      "
    deploy:
      mode: global
```

### 2. Network Troubleshooting Stack
```yaml
version: '3.8'

services:
  tcpdump:
    image: nicolaka/netshoot
    networks:
      - app-network
    command: >
      sh -c "
        tcpdump -i eth0 -n
      "
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager

  dns-monitor:
    image: nicolaka/netshoot
    networks:
      - app-network
    command: >
      sh -c "
        while true; do
          dig web;
          dig api;
          dig db;
          sleep 30;
        done
      "
    deploy:
      mode: global

  network-test:
    image: nicolaka/netshoot
    networks:
      - app-network
    command: >
      sh -c "
        while true; do
          ping -c 4 web;
          ping -c 4 api;
          ping -c 4 db;
          sleep 60;
        done
      "
    deploy:
      mode: global
```

## Next Steps
- Move on to Lab 24: Docker Storage Advanced
- Practice implementing advanced networking in your projects
- Explore additional networking tools and techniques

## Additional Resources
- [Docker Networking Documentation](https://docs.docker.com/network/)
- [Overlay Network Guide](https://docs.docker.com/network/overlay/)
- [Service Discovery Documentation](https://docs.docker.com/engine/swarm/networking/)
- [Network Troubleshooting Guide](https://docs.docker.com/network/troubleshoot/) 