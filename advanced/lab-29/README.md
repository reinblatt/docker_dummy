# Lab 29: Docker Security Advanced

## Objective
In this lab, you will learn advanced Docker security concepts and techniques. You'll understand how to implement comprehensive security measures, including container hardening, runtime security, vulnerability scanning, and advanced security monitoring.

## Prerequisites
- Completed Lab 28 (Docker Orchestration Advanced)
- Docker Desktop running on your system
- Basic understanding of security concepts
- Familiarity with Docker Compose
- Access to Docker Swarm

## Key Concepts
- Container Hardening: Security best practices
- Runtime Security: Real-time protection
- Vulnerability Scanning: Security assessment
- Security Monitoring: Threat detection
- Access Control: Permission management

## Exercises

### 1. Container Hardening
1. Implement security profiles:
```yaml
# docker-compose.security.yml
version: '3.8'

services:
  web:
    image: nginx:latest
    security_opt:
      - no-new-privileges:true
      - seccomp:unconfined
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1G
```

2. Configure resource limits:
```yaml
# docker-compose.limits.yml
version: '3.8'

services:
  web:
    image: nginx:latest
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1G
        reservations:
          cpus: '1'
          memory: 512M
    ulimits:
      nofile:
        soft: 65535
        hard: 65535
      nproc:
        soft: 1024
        hard: 2048
```

### 2. Runtime Security
1. Set up Falco monitoring:
```yaml
# docker-compose.falco.yml
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
    environment:
      - FALCO_BPF_PROBE=""
    deploy:
      mode: global
```

2. Configure audit logging:
```yaml
# docker-compose.audit.yml
version: '3.8'

services:
  audit:
    image: docker/docker-bench-security:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /etc:/etc:ro
      - /usr/bin/docker-containerd:/usr/bin/docker-containerd:ro
      - /usr/bin/docker-runc:/usr/bin/docker-runc:ro
      - /usr/lib/systemd:/usr/lib/systemd:ro
      - /etc/docker:/etc/docker:ro
    deploy:
      mode: global
```

### 3. Vulnerability Scanning
1. Set up Trivy scanning:
```yaml
# docker-compose.trivy.yml
version: '3.8'

services:
  trivy:
    image: aquasec/trivy:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: server --listen 0.0.0.0:8080
    ports:
      - "8080:8080"
    deploy:
      mode: global
```

2. Configure automated scanning:
```yaml
# docker-compose.scan.yml
version: '3.8'

services:
  scanner:
    image: aquasec/trivy:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: image --severity HIGH,CRITICAL nginx:latest
    deploy:
      mode: global
```

### 4. Security Monitoring
1. Set up security monitoring:
```yaml
# docker-compose.monitor.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    deploy:
      mode: global

  grafana:
    image: grafana/grafana:latest
    volumes:
      - grafana-storage:/var/lib/grafana
    ports:
      - "3000:3000"
    deploy:
      mode: global

  alertmanager:
    image: prom/alertmanager:latest
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    ports:
      - "9093:9093"
    deploy:
      mode: global
```

2. Configure security alerts:
```yaml
# docker-compose.alerts.yml
version: '3.8'

services:
  alertmanager:
    image: prom/alertmanager:latest
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    ports:
      - "9093:9093"
    deploy:
      mode: global
    environment:
      - ALERTMANAGER_CONFIG=/etc/alertmanager/alertmanager.yml
```

## Best Practices

### 1. Container Security
- Implement security profiles
- Configure resource limits
- Use read-only filesystems
- Drop unnecessary capabilities
- Monitor container behavior

### 2. Runtime Security
- Set up runtime protection
- Configure audit logging
- Monitor system calls
- Implement security policies
- Document security measures

### 3. Vulnerability Management
- Perform regular scanning
- Update base images
- Monitor for vulnerabilities
- Implement patch management
- Document security findings

### 4. Monitoring
- Set up security monitoring
- Configure alerting
- Monitor system behavior
- Track security events
- Document monitoring setup

## Common Patterns

### 1. Security Configuration
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    security_opt:
      - no-new-privileges:true
      - seccomp:unconfined
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1G
```

### 2. Runtime Security Configuration
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
    environment:
      - FALCO_BPF_PROBE=""
    deploy:
      mode: global
```

### 3. Vulnerability Scanning Configuration
```yaml
version: '3.8'

services:
  trivy:
    image: aquasec/trivy:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: server --listen 0.0.0.0:8080
    ports:
      - "8080:8080"
    deploy:
      mode: global
```

### 4. Monitoring Configuration
```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    deploy:
      mode: global

  grafana:
    image: grafana/grafana:latest
    volumes:
      - grafana-storage:/var/lib/grafana
    ports:
      - "3000:3000"
    deploy:
      mode: global
```

## Challenges
1. Implement container hardening
2. Set up runtime security
3. Configure vulnerability scanning
4. Implement security monitoring
5. Create security policies

## Real-world Examples

### 1. Complete Security Stack
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
    environment:
      - FALCO_BPF_PROBE=""
    deploy:
      mode: global

  trivy:
    image: aquasec/trivy:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: server --listen 0.0.0.0:8080
    ports:
      - "8080:8080"
    deploy:
      mode: global

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    deploy:
      mode: global

  grafana:
    image: grafana/grafana:latest
    volumes:
      - grafana-storage:/var/lib/grafana
    ports:
      - "3000:3000"
    deploy:
      mode: global
```

### 2. Security Monitoring Stack
```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    deploy:
      mode: global

  grafana:
    image: grafana/grafana:latest
    volumes:
      - grafana-storage:/var/lib/grafana
    ports:
      - "3000:3000"
    deploy:
      mode: global

  alertmanager:
    image: prom/alertmanager:latest
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    ports:
      - "9093:9093"
    deploy:
      mode: global
```

## Next Steps
- Move on to Lab 30: Docker Enterprise Advanced
- Practice implementing advanced security in your projects
- Explore additional security tools and techniques

## Additional Resources
- [Docker Security Documentation](https://docs.docker.com/engine/security/)
- [Container Security Guide](https://docs.docker.com/engine/security/container-security/)
- [Runtime Security Guide](https://docs.docker.com/engine/security/security/)
- [Security Best Practices](https://docs.docker.com/engine/security/best-practices/) 