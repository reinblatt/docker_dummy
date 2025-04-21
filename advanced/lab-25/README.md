# Lab 25: Docker Security Advanced

## Objective
In this lab, you will learn advanced Docker security concepts and techniques. You'll understand how to implement container hardening, security monitoring, runtime protection, and advanced security configurations for containerized applications.

## Prerequisites
- Completed Lab 24 (Docker Storage Advanced)
- Docker Desktop running on your system
- Basic understanding of security concepts
- Familiarity with Docker Compose
- Access to Docker Swarm (for distributed security)

## Key Concepts
- Container Hardening: Security best practices
- Security Monitoring: Real-time threat detection
- Runtime Protection: Container runtime security
- Security Policies: Access control and compliance
- Vulnerability Management: Scanning and remediation

## Exercises

### 1. Container Hardening
1. Implement security profiles:
```powershell
# Create a security profile
docker run --rm \
    --security-opt no-new-privileges \
    --security-opt seccomp=unconfined \
    --security-opt apparmor=unconfined \
    --cap-drop=ALL \
    --cap-add=NET_BIND_SERVICE \
    nginx:latest

# Apply security constraints
docker run --rm \
    --security-opt no-new-privileges \
    --security-opt seccomp=/path/to/profile.json \
    --security-opt apparmor=profile_name \
    --cap-drop=ALL \
    --cap-add=NET_BIND_SERVICE \
    --read-only \
    --tmpfs /tmp:rw,noexec,nosuid,size=65536k \
    nginx:latest
```

2. Configure resource limits:
```powershell
# Set resource limits
docker run --rm \
    --memory=512m \
    --memory-swap=1g \
    --cpus=1 \
    --cpu-shares=512 \
    --pids-limit=100 \
    --ulimit nofile=1024:1024 \
    nginx:latest

# Apply network restrictions
docker run --rm \
    --network=bridge \
    --dns=8.8.8.8 \
    --dns-search=example.com \
    --dns-opt=timeout:2 \
    --dns-opt=attempts:3 \
    nginx:latest
```

### 2. Security Monitoring
1. Set up security monitoring:
```powershell
# Install Falco
docker run -d \
    --name falco \
    --privileged \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /dev:/host/dev \
    -v /proc:/host/proc:ro \
    -v /boot:/host/boot:ro \
    -v /lib/modules:/host/lib/modules:ro \
    -v /usr:/host/usr:ro \
    falcosecurity/falco:latest

# Configure Falco rules
docker run --rm \
    -v /etc/falco:/etc/falco \
    falcosecurity/falco:latest \
    falco -c /etc/falco/falco.yaml -r /etc/falco/rules.d/
```

2. Implement audit logging:
```powershell
# Configure audit logging
docker run --rm \
    --log-driver=syslog \
    --log-opt syslog-address=udp://localhost:514 \
    --log-opt tag=docker \
    --log-opt syslog-facility=local7 \
    nginx:latest

# Set up audit rules
docker run --rm \
    --security-opt audit=1 \
    --security-opt audit-log=/var/log/docker-audit.log \
    nginx:latest
```

### 3. Runtime Protection
1. Configure runtime security:
```powershell
# Enable runtime protection
docker run --rm \
    --security-opt apparmor=profile_name \
    --security-opt seccomp=/path/to/profile.json \
    --security-opt no-new-privileges \
    --cap-drop=ALL \
    --cap-add=NET_BIND_SERVICE \
    nginx:latest

# Implement container isolation
docker run --rm \
    --security-opt apparmor=profile_name \
    --security-opt seccomp=/path/to/profile.json \
    --security-opt no-new-privileges \
    --cap-drop=ALL \
    --cap-add=NET_BIND_SERVICE \
    --read-only \
    --tmpfs /tmp:rw,noexec,nosuid,size=65536k \
    nginx:latest
```

2. Set up runtime monitoring:
```powershell
# Monitor container runtime
docker run --rm \
    --security-opt apparmor=profile_name \
    --security-opt seccomp=/path/to/profile.json \
    --security-opt no-new-privileges \
    --cap-drop=ALL \
    --cap-add=NET_BIND_SERVICE \
    --read-only \
    --tmpfs /tmp:rw,noexec,nosuid,size=65536k \
    --log-driver=syslog \
    --log-opt syslog-address=udp://localhost:514 \
    --log-opt tag=docker \
    --log-opt syslog-facility=local7 \
    nginx:latest
```

### 4. Security Policies
1. Implement access control:
```powershell
# Configure access control
docker run --rm \
    --security-opt apparmor=profile_name \
    --security-opt seccomp=/path/to/profile.json \
    --security-opt no-new-privileges \
    --cap-drop=ALL \
    --cap-add=NET_BIND_SERVICE \
    --read-only \
    --tmpfs /tmp:rw,noexec,nosuid,size=65536k \
    --log-driver=syslog \
    --log-opt syslog-address=udp://localhost:514 \
    --log-opt tag=docker \
    --log-opt syslog-facility=local7 \
    --user=nginx \
    --group=nginx \
    nginx:latest
```

2. Enforce security policies:
```powershell
# Apply security policies
docker run --rm \
    --security-opt apparmor=profile_name \
    --security-opt seccomp=/path/to/profile.json \
    --security-opt no-new-privileges \
    --cap-drop=ALL \
    --cap-add=NET_BIND_SERVICE \
    --read-only \
    --tmpfs /tmp:rw,noexec,nosuid,size=65536k \
    --log-driver=syslog \
    --log-opt syslog-address=udp://localhost:514 \
    --log-opt tag=docker \
    --log-opt syslog-facility=local7 \
    --user=nginx \
    --group=nginx \
    --memory=512m \
    --memory-swap=1g \
    --cpus=1 \
    --cpu-shares=512 \
    --pids-limit=100 \
    --ulimit nofile=1024:1024 \
    nginx:latest
```

## Best Practices

### 1. Container Security
- Use minimal base images
- Implement security profiles
- Configure resource limits
- Apply access control
- Monitor container activity

### 2. Runtime Security
- Enable runtime protection
- Configure security policies
- Implement isolation
- Monitor runtime activity
- Regular security updates

### 3. Monitoring
- Set up security monitoring
- Configure audit logging
- Implement alerting
- Regular security audits
- Document security events

### 4. Compliance
- Follow security standards
- Implement access control
- Regular security assessments
- Document security policies
- Maintain audit trails

## Common Patterns

### 1. Security Configuration
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    security_opt:
      - no-new-privileges
      - seccomp=/path/to/profile.json
      - apparmor=profile_name
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid,size=65536k
    user: nginx
    group: nginx
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '1'
        reservations:
          memory: 256M
          cpus: '0.5'
```

### 2. Monitoring Setup
```yaml
version: '3.8'

services:
  falco:
    image: falcosecurity/falco:latest
    privileged: true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /dev:/host/dev
      - /proc:/host/proc:ro
      - /boot:/host/boot:ro
      - /lib/modules:/host/lib/modules:ro
      - /usr:/host/usr:ro
    deploy:
      mode: global

  syslog:
    image: nginx:latest
    logging:
      driver: syslog
      options:
        syslog-address: udp://localhost:514
        tag: docker
        syslog-facility: local7
```

### 3. Security Policy
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    security_opt:
      - no-new-privileges
      - seccomp=/path/to/profile.json
      - apparmor=profile_name
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid,size=65536k
    user: nginx
    group: nginx
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '1'
        reservations:
          memory: 256M
          cpus: '0.5'
      restart_policy:
        condition: on-failure
```

### 4. Audit Configuration
```yaml
version: '3.8'

services:
  audit:
    image: nginx:latest
    security_opt:
      - audit=1
      - audit-log=/var/log/docker-audit.log
    logging:
      driver: syslog
      options:
        syslog-address: udp://localhost:514
        tag: docker
        syslog-facility: local7
    deploy:
      mode: global
```

## Challenges
1. Implement container hardening
2. Set up security monitoring
3. Configure runtime protection
4. Enforce security policies
5. Create audit trails

## Real-world Examples

### 1. Complete Security Stack
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    security_opt:
      - no-new-privileges
      - seccomp=/path/to/profile.json
      - apparmor=profile_name
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid,size=65536k
    user: nginx
    group: nginx
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '1'
        reservations:
          memory: 256M
          cpus: '0.5'
      restart_policy:
        condition: on-failure

  falco:
    image: falcosecurity/falco:latest
    privileged: true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /dev:/host/dev
      - /proc:/host/proc:ro
      - /boot:/host/boot:ro
      - /lib/modules:/host/lib/modules:ro
      - /usr:/host/usr:ro
    deploy:
      mode: global

  audit:
    image: nginx:latest
    security_opt:
      - audit=1
      - audit-log=/var/log/docker-audit.log
    logging:
      driver: syslog
      options:
        syslog-address: udp://localhost:514
        tag: docker
        syslog-facility: local7
    deploy:
      mode: global
```

### 2. Security Monitoring Stack
```yaml
version: '3.8'

services:
  falco:
    image: falcosecurity/falco:latest
    privileged: true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /dev:/host/dev
      - /proc:/host/proc:ro
      - /boot:/host/boot:ro
      - /lib/modules:/host/lib/modules:ro
      - /usr:/host/usr:ro
    deploy:
      mode: global

  syslog:
    image: nginx:latest
    logging:
      driver: syslog
      options:
        syslog-address: udp://localhost:514
        tag: docker
        syslog-facility: local7
    deploy:
      mode: global

  audit:
    image: nginx:latest
    security_opt:
      - audit=1
      - audit-log=/var/log/docker-audit.log
    logging:
      driver: syslog
      options:
        syslog-address: udp://localhost:514
        tag: docker
        syslog-facility: local7
    deploy:
      mode: global
```

## Next Steps
- Move on to Lab 26: Docker Performance Advanced
- Practice implementing advanced security in your projects
- Explore additional security tools and techniques

## Additional Resources
- [Docker Security Documentation](https://docs.docker.com/engine/security/)
- [Container Security Guide](https://docs.docker.com/engine/security/container-security/)
- [Runtime Security Guide](https://docs.docker.com/engine/security/security/)
- [Security Best Practices](https://docs.docker.com/engine/security/security/) 