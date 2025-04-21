# Lab 12: Docker Security

## Objective
In this lab, you will learn advanced Docker security concepts including image scanning, runtime security, network security, and access control. You'll understand how to secure your Docker environment and protect your containerized applications.

## Prerequisites
- Completed Lab 11 (Advanced Dockerfile Techniques)
- Docker Desktop running on your system
- Basic understanding of security concepts
- Familiarity with Linux permissions and networking

## Key Concepts
- Image Security: Scanning and vulnerability management
- Runtime Security: Container isolation and resource limits
- Network Security: Network policies and encryption
- Access Control: User permissions and authentication
- Security Scanning: Vulnerability detection and remediation

## Exercises

### 1. Image Security
1. Scan a Docker image for vulnerabilities:
```powershell
# Install Docker Scan plugin
docker scan --version

# Scan an image
docker scan nginx:latest

# Scan with specific options
docker scan --severity high nginx:latest
```

2. Create a secure base image:
```dockerfile
FROM alpine:3.14

# Update and upgrade packages
RUN apk update && apk upgrade && \
    rm -rf /var/cache/apk/*

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Set working directory
WORKDIR /app

# Copy application
COPY --chown=appuser:appgroup . .

# Switch to non-root user
USER appuser

# Set environment variables
ENV NODE_ENV=production

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

# Start application
CMD ["node", "app.js"]
```

### 2. Runtime Security
1. Run a container with security options:
```powershell
# Run with read-only filesystem
docker run --read-only -d nginx

# Run with resource limits
docker run --memory=512m --cpus=1 -d nginx

# Run with security options
docker run --security-opt=no-new-privileges \
           --cap-drop=ALL \
           --cap-add=NET_BIND_SERVICE \
           -d nginx
```

2. Create a secure container configuration:
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    security_opt:
      - no-new-privileges
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
    tmpfs:
      - /tmp
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
```

### 3. Network Security
1. Configure network security:
```powershell
# Create a secure network
docker network create --driver bridge \
                     --opt com.docker.network.bridge.enable_icc=false \
                     secure-network

# Run container on secure network
docker run --network secure-network -d nginx
```

2. Set up network policies:
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    networks:
      - frontend
    ports:
      - "8080:80"

  api:
    image: my-api:latest
    networks:
      - frontend
      - backend
    environment:
      - DB_HOST=db

  db:
    image: mysql:5.7
    networks:
      - backend
    environment:
      MYSQL_ROOT_PASSWORD: secret

networks:
  frontend:
    driver: bridge
    internal: true
  backend:
    driver: bridge
    internal: true
```

### 4. Access Control
1. Configure user namespaces:
```powershell
# Create user namespace mapping
echo "dockremap:100000:65536" >> /etc/subuid
echo "dockremap:100000:65536" >> /etc/subgid

# Configure Docker daemon
echo '{
  "userns-remap": "dockremap"
}' > /etc/docker/daemon.json

# Restart Docker daemon
systemctl restart docker
```

2. Set up authentication:
```powershell
# Create Docker registry
docker run -d \
  --name registry \
  --restart=always \
  -p 5000:5000 \
  -v registry-data:/var/lib/registry \
  registry:2

# Login to registry
docker login localhost:5000

# Tag and push image
docker tag nginx:latest localhost:5000/nginx:latest
docker push localhost:5000/nginx:latest
```

## Best Practices

### 1. Image Security
- Use minimal base images
- Keep images updated
- Scan for vulnerabilities
- Remove unnecessary tools
- Use multi-stage builds

### 2. Runtime Security
- Run as non-root user
- Use read-only filesystems
- Drop unnecessary capabilities
- Set resource limits
- Enable security options

### 3. Network Security
- Use internal networks
- Implement network policies
- Encrypt network traffic
- Isolate sensitive services
- Monitor network activity

### 4. Access Control
- Use user namespaces
- Implement authentication
- Restrict registry access
- Use secrets management
- Monitor access logs

## Common Patterns

### 1. Secure Container Configuration
```yaml
services:
  web:
    security_opt:
      - no-new-privileges
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
    tmpfs:
      - /tmp
```

### 2. Network Isolation
```yaml
networks:
  frontend:
    driver: bridge
    internal: true
  backend:
    driver: bridge
    internal: true
```

### 3. Resource Limits
```yaml
deploy:
  resources:
    limits:
      cpus: '0.50'
      memory: 512M
    reservations:
      cpus: '0.25'
      memory: 256M
```

### 4. Authentication
```yaml
services:
  registry:
    environment:
      - REGISTRY_AUTH=htpasswd
      - REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd
      - REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm
```

## Challenges
1. Implement a secure CI/CD pipeline
2. Set up container monitoring
3. Configure network encryption
4. Implement access control
5. Create a security scanning workflow

## Real-world Examples

### 1. Production-Ready Secure Configuration
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    security_opt:
      - no-new-privileges
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
    tmpfs:
      - /tmp
    networks:
      - frontend
    ports:
      - "8080:80"
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M

  api:
    image: my-api:latest
    security_opt:
      - no-new-privileges
    cap_drop:
      - ALL
    networks:
      - frontend
      - backend
    environment:
      - DB_HOST=db
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M

  db:
    image: mysql:5.7
    security_opt:
      - no-new-privileges
    cap_drop:
      - ALL
    networks:
      - backend
    environment:
      MYSQL_ROOT_PASSWORD: secret
    volumes:
      - db-data:/var/lib/mysql
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 1G

networks:
  frontend:
    driver: bridge
    internal: true
  backend:
    driver: bridge
    internal: true

volumes:
  db-data:
```

### 2. Secure Registry Configuration
```yaml
version: '3.8'

services:
  registry:
    image: registry:2
    ports:
      - "5000:5000"
    environment:
      - REGISTRY_AUTH=htpasswd
      - REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd
      - REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm
    volumes:
      - registry-data:/var/lib/registry
      - ./auth:/auth
    security_opt:
      - no-new-privileges
    cap_drop:
      - ALL
    networks:
      - registry-net

networks:
  registry-net:
    driver: bridge
    internal: true

volumes:
  registry-data:
```

## Next Steps
- Once you're comfortable with Docker security, move on to Lab 13: Docker Networking
- Practice implementing these security measures in your projects

## Additional Resources
- [Docker Security Documentation](https://docs.docker.com/engine/security/)
- [Container Security Best Practices](https://docs.docker.com/develop/security-best-practices/)
- [Docker Security Scanning](https://docs.docker.com/engine/scan/)
- [Docker Security Benchmarks](https://docs.docker.com/engine/security/benchmarks/) 