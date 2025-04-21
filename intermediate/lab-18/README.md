# Lab 18: Docker Security

## Objective
In this lab, you will learn about Docker security best practices, vulnerability scanning, and secure container configurations. You'll understand how to implement security measures at different levels of the Docker stack.

## Prerequisites
- Completed Lab 17 (Docker Swarm Advanced)
- Docker Desktop running on your system
- Basic understanding of security concepts
- Familiarity with Docker Compose
- Access to Docker Hub or a private registry

## Key Concepts
- Container Security: Isolation, resource limits, and runtime security
- Image Security: Secure base images, vulnerability scanning, and image signing
- Network Security: Network isolation, encryption, and access control
- Runtime Security: Seccomp, AppArmor, and capabilities
- Secrets Management: Secure handling of sensitive data

## Exercises

### 1. Container Security
1. Run a container with security constraints:
```powershell
# Run container with limited capabilities
docker run -d \
    --cap-drop=ALL \
    --cap-add=NET_BIND_SERVICE \
    --security-opt=no-new-privileges \
    --security-opt=apparmor=unconfined \
    --security-opt=seccomp=unconfined.json \
    --memory=512m \
    --cpus=1 \
    --pids-limit=100 \
    --read-only \
    --tmpfs=/tmp:rw,noexec,nosuid \
    nginx:latest
```

2. Configure container security:
```powershell
# Create a seccomp profile
docker run --rm -it --security-opt seccomp=unconfined \
    --security-opt apparmor=unconfined \
    --cap-add=NET_ADMIN \
    --cap-add=NET_RAW \
    --cap-drop=ALL \
    --read-only \
    --tmpfs=/tmp:rw,noexec,nosuid \
    --memory=512m \
    --cpus=1 \
    --pids-limit=100 \
    nginx:latest
```

### 2. Image Security
1. Scan images for vulnerabilities:
```powershell
# Install Trivy
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
    aquasec/trivy image nginx:latest

# Scan with Snyk
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
    snyk/snyk-cli:docker test nginx:latest
```

2. Create a secure Dockerfile:
```dockerfile
# Use minimal base image
FROM alpine:latest

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Set working directory
WORKDIR /app

# Copy application files
COPY --chown=appuser:appgroup . .

# Drop privileges
USER appuser

# Set entrypoint
ENTRYPOINT ["/app/start.sh"]
```

### 3. Network Security
1. Configure network security:
```powershell
# Create a secure overlay network
docker network create \
    --driver overlay \
    --opt encrypted=true \
    --opt com.docker.network.driver.overlay.vxlanid_list=4096 \
    secure-overlay

# Run container with network constraints
docker run -d \
    --network secure-overlay \
    --publish published=80,target=80,protocol=tcp \
    --publish published=443,target=443,protocol=tcp \
    --publish-all=false \
    nginx:latest
```

2. Implement network policies:
```yaml
# docker-compose.yml
version: '3.8'

networks:
  secure:
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
      - secure
    ports:
      - "80:80"
      - "443:443"
    security_opt:
      - no-new-privileges
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
```

### 4. Secrets Management
1. Use Docker secrets:
```powershell
# Create a secret
echo "mysecretpassword" | docker secret create db_password -

# Use secret in a service
docker service create \
    --name db \
    --secret db_password \
    --env MYSQL_ROOT_PASSWORD_FILE=/run/secrets/db_password \
    mysql:latest
```

2. Configure secrets in compose:
```yaml
# docker-compose.yml
version: '3.8'

secrets:
  db_password:
    file: ./secrets/db_password.txt

services:
  db:
    image: mysql:latest
    secrets:
      - db_password
    environment:
      - MYSQL_ROOT_PASSWORD_FILE=/run/secrets/db_password
```

## Best Practices

### 1. Container Security
- Use minimal base images
- Run containers as non-root
- Implement resource limits
- Use read-only filesystems
- Configure security profiles

### 2. Image Security
- Scan images regularly
- Use trusted base images
- Sign images
- Keep images updated
- Remove unnecessary tools

### 3. Network Security
- Use encrypted networks
- Implement network segmentation
- Restrict network access
- Use secure protocols
- Monitor network traffic

### 4. Runtime Security
- Drop unnecessary capabilities
- Use security profiles
- Implement resource limits
- Monitor container behavior
- Use secure defaults

## Common Patterns

### 1. Secure Container
```yaml
services:
  secure-app:
    image: nginx:latest
    security_opt:
      - no-new-privileges
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid
    mem_limit: 512m
    cpus: 1
    pids_limit: 100
```

### 2. Secure Network
```yaml
networks:
  secure:
    driver: overlay
    driver_opts:
      encrypted: "true"
    ipam:
      driver: default
      config:
        - subnet: 10.0.9.0/24
          gateway: 10.0.9.1
```

### 3. Secrets Configuration
```yaml
secrets:
  db_password:
    file: ./secrets/db_password.txt

services:
  db:
    secrets:
      - db_password
    environment:
      - MYSQL_ROOT_PASSWORD_FILE=/run/secrets/db_password
```

### 4. Security Scanning
```yaml
services:
  scanner:
    image: aquasec/trivy
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: image nginx:latest
```

## Challenges
1. Create a secure container configuration
2. Implement image scanning
3. Set up secure networking
4. Configure secrets management
5. Implement runtime security

## Real-world Examples

### 1. Complete Secure Stack
```yaml
version: '3.8'

networks:
  secure:
    driver: overlay
    driver_opts:
      encrypted: "true"
    ipam:
      driver: default
      config:
        - subnet: 10.0.9.0/24
          gateway: 10.0.9.1

secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    file: ./secrets/api_key.txt

services:
  web:
    image: nginx:latest
    networks:
      - secure
    security_opt:
      - no-new-privileges
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid
    mem_limit: 512m
    cpus: 1
    pids_limit: 100
    ports:
      - "80:80"
      - "443:443"

  api:
    image: node:latest
    networks:
      - secure
    secrets:
      - api_key
    environment:
      - API_KEY_FILE=/run/secrets/api_key
    security_opt:
      - no-new-privileges
    cap_drop:
      - ALL
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid

  db:
    image: mysql:latest
    networks:
      - secure
    secrets:
      - db_password
    environment:
      - MYSQL_ROOT_PASSWORD_FILE=/run/secrets/db_password
    security_opt:
      - no-new-privileges
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
```

### 2. Security Monitoring Stack
```yaml
version: '3.8'

services:
  trivy:
    image: aquasec/trivy
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: server --listen 0.0.0.0:8080

  falco:
    image: falcosecurity/falco
    privileged: true
    volumes:
      - /var/run/docker.sock:/host/var/run/docker.sock
      - /dev:/host/dev
      - /proc:/host/proc:ro
      - /boot:/host/boot:ro
      - /lib/modules:/host/lib/modules:ro
      - /usr:/host/usr:ro
    environment:
      - FALCO_BPF_PROBE=""

  sysdig:
    image: sysdig/sysdig
    privileged: true
    volumes:
      - /var/run/docker.sock:/host/var/run/docker.sock
      - /dev:/host/dev
      - /proc:/host/proc:ro
      - /boot:/host/boot:ro
      - /lib/modules:/host/lib/modules:ro
      - /usr:/host/usr:ro
```

## Next Steps
- Once you're comfortable with Docker security, move on to Lab 19: Docker Performance Optimization
- Practice implementing these security patterns in your projects

## Additional Resources
- [Docker Security Documentation](https://docs.docker.com/engine/security/)
- [Docker Security Best Practices](https://docs.docker.com/engine/security/security/)
- [Container Security Guide](https://docs.docker.com/engine/security/container-security/)
- [Docker Security Scanning](https://docs.docker.com/engine/scan/) 