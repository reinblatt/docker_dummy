# Lab 20: Docker Enterprise Features

## Objective
In this lab, you will learn about Docker Enterprise features and capabilities. You'll understand how to use Universal Control Plane (UCP), Docker Trusted Registry (DTR), and other enterprise-grade features for managing containerized applications at scale.

## Prerequisites
- Completed Lab 19 (Docker Performance Optimization)
- Docker Desktop running on your system
- Basic understanding of enterprise IT concepts
- Familiarity with Docker Compose
- Access to Docker Enterprise Edition (or trial)

## Key Concepts
- Universal Control Plane (UCP): Enterprise-grade cluster management
- Docker Trusted Registry (DTR): Secure image storage and distribution
- Role-Based Access Control (RBAC): Enterprise security and access management
- Image Signing: Content trust and image verification
- High Availability: Enterprise-grade reliability features

## Exercises

### 1. Universal Control Plane (UCP)
1. Install UCP:
```powershell
# Install UCP on a manager node
docker container run --rm -it \
    --name ucp \
    -v /var/run/docker.sock:/var/run/docker.sock \
    docker/ucp:latest install \
    --host-address <node-ip> \
    --interactive

# Join additional nodes
docker container run --rm -it \
    --name ucp \
    -v /var/run/docker.sock:/var/run/docker.sock \
    docker/ucp:latest join \
    --host-address <node-ip> \
    --interactive
```

2. Configure UCP:
```powershell
# Configure UCP settings
docker container run --rm -it \
    --name ucp \
    -v /var/run/docker.sock:/var/run/docker.sock \
    docker/ucp:latest config \
    --interactive

# Backup UCP configuration
docker container run --rm -it \
    --name ucp \
    -v /var/run/docker.sock:/var/run/docker.sock \
    docker/ucp:latest backup \
    --interactive
```

### 2. Docker Trusted Registry (DTR)
1. Install DTR:
```powershell
# Install DTR on a manager node
docker container run --rm -it \
    --name dtr \
    -v /var/run/docker.sock:/var/run/docker.sock \
    docker/dtr:latest install \
    --ucp-url <ucp-url> \
    --ucp-node <node-name> \
    --dtr-external-url <dtr-url> \
    --interactive

# Join additional replicas
docker container run --rm -it \
    --name dtr \
    -v /var/run/docker.sock:/var/run/docker.sock \
    docker/dtr:latest join \
    --ucp-url <ucp-url> \
    --ucp-node <node-name> \
    --existing-replica-id <replica-id> \
    --interactive
```

2. Configure DTR:
```powershell
# Configure DTR settings
docker container run --rm -it \
    --name dtr \
    -v /var/run/docker.sock:/var/run/docker.sock \
    docker/dtr:latest config \
    --interactive

# Backup DTR configuration
docker container run --rm -it \
    --name dtr \
    -v /var/run/docker.sock:/var/run/docker.sock \
    docker/dtr:latest backup \
    --interactive
```

### 3. Role-Based Access Control (RBAC)
1. Configure RBAC:
```powershell
# Create a team
docker team create --name developers

# Create a user
docker user create --name john --password-stdin

# Add user to team
docker team add --user john developers

# Create a collection
docker collection create --name web-apps

# Grant team access to collection
docker collection grant --team developers --role contributor web-apps
```

2. Manage permissions:
```powershell
# View permissions
docker collection ls
docker team ls
docker user ls

# Modify permissions
docker collection grant --team developers --role owner web-apps
docker collection revoke --team developers web-apps
```

### 4. Image Signing
1. Configure content trust:
```powershell
# Enable content trust
export DOCKER_CONTENT_TRUST=1

# Sign an image
docker push myregistry.com/myimage:latest

# Verify image signatures
docker trust inspect myregistry.com/myimage:latest
```

2. Manage signing keys:
```powershell
# Generate signing keys
docker trust key generate mykey

# Add signer
docker trust signer add --key mykey.pub mykey myregistry.com/myimage

# Remove signer
docker trust signer remove mykey myregistry.com/myimage
```

## Best Practices

### 1. UCP Management
- Use high availability configuration
- Implement backup strategies
- Monitor cluster health
- Use appropriate node roles
- Implement security policies

### 2. DTR Management
- Configure image scanning
- Implement backup strategies
- Use appropriate storage backends
- Monitor registry health
- Implement security policies

### 3. Access Control
- Use least privilege principle
- Implement team-based access
- Use collections for organization
- Monitor access patterns
- Regular access reviews

### 4. Security
- Enable content trust
- Use image signing
- Implement RBAC
- Monitor security events
- Regular security audits

## Common Patterns

### 1. UCP Configuration
```yaml
version: '3.8'

services:
  ucp:
    image: docker/ucp:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - UCP_ADMIN_USERNAME=admin
      - UCP_ADMIN_PASSWORD=password
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager
```

### 2. DTR Configuration
```yaml
version: '3.8'

services:
  dtr:
    image: docker/dtr:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - DTR_EXTERNAL_URL=https://dtr.example.com
      - UCP_URL=https://ucp.example.com
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager
```

### 3. RBAC Configuration
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    deploy:
      labels:
        com.docker.ucp.access.label: /web-apps
        com.docker.ucp.collection: web-apps
```

### 4. Content Trust
```yaml
version: '3.8'

services:
  app:
    image: myregistry.com/myimage:latest
    environment:
      - DOCKER_CONTENT_TRUST=1
```

## Challenges
1. Set up a UCP cluster
2. Configure DTR with high availability
3. Implement RBAC policies
4. Configure image signing
5. Set up monitoring and backup

## Real-world Examples

### 1. Complete Enterprise Stack
```yaml
version: '3.8'

networks:
  ucp:
    driver: overlay
    driver_opts:
      encrypted: "true"
  dtr:
    driver: overlay
    driver_opts:
      encrypted: "true"

services:
  ucp:
    image: docker/ucp:latest
    networks:
      - ucp
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - UCP_ADMIN_USERNAME=admin
      - UCP_ADMIN_PASSWORD=password
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager
      resources:
        limits:
          cpus: '4'
          memory: 4G
        reservations:
          cpus: '2'
          memory: 2G

  dtr:
    image: docker/dtr:latest
    networks:
      - dtr
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - dtr-data:/var/lib/docker/registry
    environment:
      - DTR_EXTERNAL_URL=https://dtr.example.com
      - UCP_URL=https://ucp.example.com
      - DOCKER_CONTENT_TRUST=1
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager
      resources:
        limits:
          cpus: '4'
          memory: 4G
        reservations:
          cpus: '2'
          memory: 2G

  web:
    image: myregistry.com/web:latest
    networks:
      - ucp
    deploy:
      labels:
        com.docker.ucp.access.label: /web-apps
        com.docker.ucp.collection: web-apps
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G

  api:
    image: myregistry.com/api:latest
    networks:
      - ucp
    deploy:
      labels:
        com.docker.ucp.access.label: /api-apps
        com.docker.ucp.collection: api-apps
      resources:
        limits:
          cpus: '4'
          memory: 4G
        reservations:
          cpus: '2'
          memory: 2G

volumes:
  dtr-data:
```

### 2. Monitoring and Backup Stack
```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    networks:
      - ucp
    deploy:
      labels:
        com.docker.ucp.access.label: /monitoring
        com.docker.ucp.collection: monitoring

  grafana:
    image: grafana/grafana
    volumes:
      - grafana-data:/var/lib/grafana
    networks:
      - ucp
    deploy:
      labels:
        com.docker.ucp.access.label: /monitoring
        com.docker.ucp.collection: monitoring

  backup:
    image: alpine:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - backup-data:/backup
    networks:
      - ucp
    deploy:
      labels:
        com.docker.ucp.access.label: /backup
        com.docker.ucp.collection: backup
    command: >
      sh -c "while true; do
        docker container run --rm -v /var/run/docker.sock:/var/run/docker.sock
          docker/ucp:latest backup --interactive > /backup/ucp-backup-$(date +%Y%m%d).tar;
        docker container run --rm -v /var/run/docker.sock:/var/run/docker.sock
          docker/dtr:latest backup --interactive > /backup/dtr-backup-$(date +%Y%m%d).tar;
        sleep 86400;
      done"

volumes:
  prometheus-data:
  grafana-data:
  backup-data:
```

## Next Steps
- Practice implementing these enterprise features in your projects
- Explore additional Docker Enterprise capabilities
- Consider certification paths for Docker Enterprise

## Additional Resources
- [Docker Enterprise Documentation](https://docs.docker.com/ee/)
- [UCP Documentation](https://docs.docker.com/ee/ucp/)
- [DTR Documentation](https://docs.docker.com/ee/dtr/)
- [Docker Enterprise Security](https://docs.docker.com/ee/security/) 