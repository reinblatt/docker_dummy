# Lab 09: Docker Best Practices

## Objective
In this lab, you will learn the best practices for working with Docker, including image optimization, security considerations, and production deployment strategies. You'll understand how to create efficient, secure, and maintainable Docker environments.

## Prerequisites
- Completed Lab 08 (Multi-stage Builds)
- Docker Desktop running on your system
- Basic understanding of security concepts
- Familiarity with production deployment concepts

## Key Concepts
- Image Optimization: Techniques to reduce image size and build time
- Security Hardening: Methods to secure Docker containers
- Production Readiness: Features needed for production deployment
- Monitoring and Logging: Tools and practices for container observability
- Resource Management: Controlling container resources

## Exercises

### 1. Image Optimization
1. Create a .dockerignore file:
```text
# Version control
.git
.gitignore

# Dependencies
node_modules
__pycache__
*.pyc

# Build artifacts
dist
build
*.log

# Environment files
.env
.env.*
```

2. Optimize a Dockerfile:
```dockerfile
# Use specific version tags
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Install dependencies first (better caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Use non-root user
RUN useradd -m appuser
USER appuser

# Set environment variables
ENV PYTHONUNBUFFERED=1

# Expose port
EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:5000/health || exit 1

# Define entrypoint
ENTRYPOINT ["python"]
CMD ["app.py"]
```

### 2. Security Hardening
1. Create a security-focused Dockerfile:
```dockerfile
FROM node:14-alpine

# Add security updates
RUN apk update && apk upgrade

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Set permissions
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

# Set environment variables
ENV NODE_ENV=production

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -q --spider http://localhost:3000/health || exit 1

# Start application
CMD ["node", "app.js"]
```

### 3. Resource Management
1. Create a resource-limited container:
```powershell
docker run -d \
  --name limited-container \
  --memory=512m \
  --cpus=1 \
  --pids-limit=100 \
  nginx
```

2. Monitor resource usage:
```powershell
docker stats limited-container
```

## Best Practices

### 1. Image Building
- Use specific version tags
- Leverage multi-stage builds
- Minimize the number of layers
- Use .dockerignore
- Clean up package manager caches

### 2. Security
- Run containers as non-root users
- Keep base images updated
- Scan images for vulnerabilities
- Use minimal base images
- Implement security scanning

### 3. Production Deployment
- Use orchestration tools
- Implement health checks
- Configure logging
- Set resource limits
- Use environment variables

### 4. Monitoring
- Implement health checks
- Configure logging
- Use monitoring tools
- Set up alerts
- Track metrics

## Common Patterns

### 1. Health Checks
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/health || exit 1
```

### 2. Resource Limits
```yaml
services:
  web:
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
```

### 3. Security Scanning
```powershell
docker scan <image-name>
```

## Challenges
1. Optimize a large Docker image
2. Implement security scanning in CI/CD
3. Set up container monitoring
4. Configure resource limits
5. Implement health checks

## Real-world Examples

### 1. Production-Ready Node.js Application
```dockerfile
FROM node:14-alpine

# Security updates
RUN apk update && apk upgrade

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Set permissions
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

# Environment variables
ENV NODE_ENV=production

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -q --spider http://localhost:3000/health || exit 1

# Expose port
EXPOSE 3000

# Start application
CMD ["node", "app.js"]
```

### 2. Secure Python Application
```dockerfile
FROM python:3.9-slim

# Security updates
RUN apt-get update && apt-get upgrade -y && \
    rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN useradd -m appuser

# Set working directory
WORKDIR /app

# Copy requirements
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Set permissions
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

# Environment variables
ENV PYTHONUNBUFFERED=1

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:5000/health || exit 1

# Expose port
EXPOSE 5000

# Start application
CMD ["python", "app.py"]
```

## Next Steps
- Once you're comfortable with best practices, move on to Lab 10: Basic Container Orchestration
- Practice implementing these best practices in your projects

## Additional Resources
- [Docker Security Best Practices](https://docs.docker.com/engine/security/)
- [Production Checklist](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Container Security Scanning](https://docs.docker.com/engine/scan/) 