# Lab 22: Docker Security Scanning

## Objective
In this lab, you will learn how to implement comprehensive security scanning for Docker images and containers. You'll understand how to use various scanning tools, interpret results, and implement security best practices to maintain a secure container environment.

## Prerequisites
- Completed Lab 21 (Docker Multi-Architecture Builds)
- Docker Desktop running on your system
- Basic understanding of security concepts
- Familiarity with Docker images and containers
- Access to Docker Hub or a private registry

## Key Concepts
- Vulnerability Scanning: Identifying security vulnerabilities in images
- Static Analysis: Analyzing Dockerfiles and configurations
- Runtime Security: Monitoring container behavior
- Compliance Scanning: Checking against security standards
- Security Policies: Implementing and enforcing security rules

## Exercises

### 1. Setting Up Security Scanners
1. Install and configure Trivy:
```powershell
# Install Trivy
docker pull aquasec/trivy:latest

# Basic image scan
docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    aquasec/trivy:latest image nginx:latest

# Scan with JSON output
docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    aquasec/trivy:latest -f json -o results.json image nginx:latest
```

2. Configure Docker Scout:
```powershell
# Enable Docker Scout
docker scout config enable

# Quick vulnerability scan
docker scout quickview nginx:latest

# Detailed vulnerability report
docker scout cves nginx:latest
```

### 2. Image Vulnerability Scanning
1. Scan images for vulnerabilities:
```powershell
# Scan with Trivy
docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    aquasec/trivy:latest image \
    --severity HIGH,CRITICAL \
    --ignore-unfixed \
    nginx:latest

# Scan with Docker Scout
docker scout cves \
    --only-severity critical,high \
    --exit-code 1 \
    nginx:latest
```

2. Generate security reports:
```powershell
# Generate HTML report with Trivy
docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v $PWD:/reports \
    aquasec/trivy:latest image \
    -f template \
    -t "@contrib/html.tpl" \
    -o /reports/scan-report.html \
    nginx:latest

# Generate SARIF report with Docker Scout
docker scout cves \
    --format sarif \
    --output scan-results.sarif \
    nginx:latest
```

### 3. Static Analysis
1. Analyze Dockerfiles:
```powershell
# Install Hadolint
docker pull hadolint/hadolint:latest

# Analyze Dockerfile
docker run --rm -i hadolint/hadolint < Dockerfile

# Analyze with custom rules
docker run --rm -i \
    -v ${PWD}/.hadolint.yaml:/.config/hadolint.yaml \
    hadolint/hadolint < Dockerfile
```

2. Check security configurations:
```powershell
# Install Dockle
docker pull goodwithtech/dockle:latest

# Check image configuration
docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    goodwithtech/dockle:latest nginx:latest

# Check with custom policies
docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v ${PWD}/policy.yaml:/policy.yaml \
    goodwithtech/dockle:latest --config /policy.yaml nginx:latest
```

### 4. Runtime Security
1. Configure runtime security:
```powershell
# Install Falco
docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /var/lib/falco:/var/lib/falco \
    falcosecurity/falco:latest

# Monitor container activity
docker run -d \
    --name falco \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /var/lib/falco:/var/lib/falco \
    falcosecurity/falco:latest
```

2. Set up alerts:
```powershell
# Configure Falco alerts
docker run -d \
    --name falco \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /var/lib/falco:/var/lib/falco \
    -v ${PWD}/falco.yaml:/etc/falco/falco.yaml \
    falcosecurity/falco:latest

# View security events
docker logs -f falco
```

## Best Practices

### 1. Scanning Strategy
- Regular automated scanning
- Multiple scanning tools
- Severity-based prioritization
- Automated remediation
- Continuous monitoring

### 2. Image Security
- Use minimal base images
- Regular updates
- Version pinning
- Layer optimization
- Signature verification

### 3. Configuration Security
- Least privilege principle
- Resource constraints
- Network segmentation
- Secret management
- Logging and monitoring

### 4. Compliance
- Policy enforcement
- Audit logging
- Regular assessments
- Documentation
- Incident response

## Common Patterns

### 1. Automated Scanning Pipeline
```yaml
version: '3.8'

services:
  trivy-scanner:
    image: aquasec/trivy:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./reports:/reports
    command: >
      image --format template 
      --template "@contrib/html.tpl" 
      --output /reports/scan-report.html 
      nginx:latest

  dockle-checker:
    image: goodwithtech/dockle:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./reports:/reports
    command: >
      --format json 
      --output /reports/config-check.json 
      nginx:latest

  hadolint:
    image: hadolint/hadolint:latest
    volumes:
      - ./Dockerfile:/Dockerfile
      - ./reports:/reports
    command: >
      --format json 
      > /reports/dockerfile-lint.json 
      /Dockerfile
```

### 2. Security Monitoring
```yaml
version: '3.8'

services:
  falco:
    image: falcosecurity/falco:latest
    privileged: true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/falco:/var/lib/falco
      - ./falco.yaml:/etc/falco/falco.yaml
    command: >
      -k https://my-webhook.com/alert
      -o json_output=true
      -o file_output.enabled=true
      -o file_output.filename=/var/lib/falco/events.log

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'

  grafana:
    image: grafana/grafana:latest
    volumes:
      - grafana-data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=secret
    depends_on:
      - prometheus

volumes:
  prometheus-data:
  grafana-data:
```

### 3. Policy Configuration
```yaml
version: '3.8'

services:
  trivy-enforcer:
    image: aquasec/trivy:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - TRIVY_SEVERITY=HIGH,CRITICAL
      - TRIVY_EXIT_CODE=1
      - TRIVY_NO_PROGRESS=true
    command: image --exit-code 1 nginx:latest

  dockle-enforcer:
    image: goodwithtech/dockle:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - DOCKLE_EXIT_CODE=1
      - DOCKLE_NO_PROGRESS=true
    command: --exit-code 1 nginx:latest
```

### 4. Compliance Checking
```yaml
version: '3.8'

services:
  compliance-check:
    image: aquasec/trivy:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./reports:/reports
    environment:
      - TRIVY_FORMAT=template
      - TRIVY_TEMPLATE=@contrib/sarif.tpl
      - TRIVY_OUTPUT=/reports/compliance.sarif
    command: image --compliance=hipaa nginx:latest
```

## Challenges
1. Set up an automated scanning pipeline
2. Implement custom security policies
3. Configure runtime security monitoring
4. Create compliance reports
5. Implement automated remediation

## Real-world Examples

### 1. Complete Security Pipeline
```yaml
version: '3.8'

services:
  scanner:
    image: aquasec/trivy:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./reports:/reports
    environment:
      - TRIVY_SEVERITY=HIGH,CRITICAL
      - TRIVY_EXIT_CODE=1
      - TRIVY_NO_PROGRESS=true
    command: >
      image --format template 
      --template "@contrib/html.tpl" 
      --output /reports/scan-report.html 
      nginx:latest

  config-check:
    image: goodwithtech/dockle:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./reports:/reports
    environment:
      - DOCKLE_EXIT_CODE=1
      - DOCKLE_NO_PROGRESS=true
    command: >
      --format json 
      --output /reports/config-check.json 
      nginx:latest

  runtime-monitor:
    image: falcosecurity/falco:latest
    privileged: true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/falco:/var/lib/falco
      - ./falco.yaml:/etc/falco/falco.yaml
    command: >
      -k https://my-webhook.com/alert
      -o json_output=true
      -o file_output.enabled=true
      -o file_output.filename=/var/lib/falco/events.log

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'

  grafana:
    image: grafana/grafana:latest
    volumes:
      - grafana-data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=secret
    depends_on:
      - prometheus

  notification:
    image: alpine:latest
    volumes:
      - ./reports:/reports
    command: >
      sh -c "
        while true; do
          if [ -f /reports/scan-report.html ]; then
            curl -X POST -H 'Content-Type: application/json' 
              -d @/reports/scan-report.html 
              https://my-webhook.com/report;
          fi
          sleep 3600;
        done
      "

volumes:
  prometheus-data:
  grafana-data:
```

### 2. Automated Remediation Pipeline
```yaml
version: '3.8'

services:
  vulnerability-check:
    image: aquasec/trivy:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./reports:/reports
    environment:
      - TRIVY_SEVERITY=HIGH,CRITICAL
      - TRIVY_EXIT_CODE=1
    command: >
      image --format json 
      --output /reports/vulnerabilities.json 
      nginx:latest

  update-check:
    image: alpine:latest
    volumes:
      - ./reports:/reports
      - ./scripts:/scripts
    command: >
      sh -c "
        if [ -f /reports/vulnerabilities.json ]; then
          python /scripts/check_updates.py /reports/vulnerabilities.json;
        fi
      "
    depends_on:
      - vulnerability-check

  auto-update:
    image: docker:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./reports:/reports
    command: >
      sh -c "
        if [ -f /reports/update_required ]; then
          docker pull nginx:latest;
          docker-compose up -d --force-recreate web;
        fi
      "
    depends_on:
      - update-check

  notification:
    image: alpine:latest
    volumes:
      - ./reports:/reports
    command: >
      sh -c "
        if [ -f /reports/update_required ]; then
          curl -X POST -H 'Content-Type: application/json' 
            -d '{\"message\":\"Security updates applied\"}' 
            https://my-webhook.com/notification;
        fi
      "
    depends_on:
      - auto-update
```

## Next Steps
- Move on to Lab 23: Docker Networking Advanced
- Practice implementing security scanning in your projects
- Explore additional security tools and techniques

## Additional Resources
- [Docker Security Documentation](https://docs.docker.com/engine/security/)
- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- [Docker Scout Documentation](https://docs.docker.com/scout/)
- [Falco Documentation](https://falco.org/docs/) 