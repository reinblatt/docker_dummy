# Lab 11: Advanced Dockerfile Techniques

## Objective
In this lab, you will learn advanced Dockerfile techniques including build arguments, secrets management, health checks, and advanced multi-stage builds. You'll understand how to create more sophisticated and secure Docker images.

## Prerequisites
- Completed Lab 10 (Basic Container Orchestration)
- Docker Desktop running on your system
- Basic understanding of shell scripting
- Familiarity with environment variables and secrets management

## Key Concepts
- Build Arguments: Runtime variables for Docker builds
- Secrets Management: Secure handling of sensitive data
- Advanced Health Checks: Custom health monitoring
- Build Context Optimization: Efficient build processes
- Layer Optimization: Advanced layer management

## Exercises

### 1. Build Arguments and Environment Variables
1. Create a Dockerfile with build arguments:
```dockerfile
ARG NODE_VERSION=14
FROM node:${NODE_VERSION}-alpine

ARG APP_VERSION
ENV APP_VERSION=${APP_VERSION}

WORKDIR /app
COPY . .
RUN npm install

EXPOSE 3000
CMD ["node", "app.js"]
```

2. Build with arguments:
```powershell
docker build --build-arg NODE_VERSION=16 --build-arg APP_VERSION=1.0.0 -t myapp .
```

### 2. Secrets Management
1. Create a Dockerfile with secrets:
```dockerfile
FROM alpine:latest

# Create a secret
RUN --mount=type=secret,id=mysecret \
    cat /run/secrets/mysecret > /app/secret.txt

# Use the secret
CMD ["sh", "-c", "cat /app/secret.txt"]
```

2. Build and run with secrets:
```powershell
echo "my-secret-value" > secret.txt
docker build --secret id=mysecret,src=secret.txt -t secret-app .
docker run secret-app
```

### 3. Advanced Health Checks
1. Create a Dockerfile with custom health check:
```dockerfile
FROM nginx:alpine

# Install curl for health check
RUN apk add --no-cache curl

# Custom health check script
COPY healthcheck.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/healthcheck.sh

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD /usr/local/bin/healthcheck.sh
```

2. Create healthcheck.sh:
```bash
#!/bin/sh
curl -f http://localhost/ || exit 1
```

### 4. Advanced Multi-stage Builds
1. Create a complex multi-stage build:
```dockerfile
# Build stage 1: Dependencies
FROM node:14 AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Build stage 2: Builder
FROM node:14 AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Build stage 3: Production
FROM node:14-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=deps /app/node_modules ./node_modules
COPY package*.json ./

# Security hardening
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
RUN chown -R appuser:appgroup /app
USER appuser

EXPOSE 3000
CMD ["node", "dist/app.js"]
```

## Best Practices

### 1. Build Arguments
- Use default values for build arguments
- Document required build arguments
- Use build arguments for version pinning
- Avoid using build arguments for secrets

### 2. Secrets Management
- Never store secrets in the image
- Use Docker secrets or build secrets
- Rotate secrets regularly
- Use minimal secret scope

### 3. Health Checks
- Use appropriate intervals
- Set reasonable timeouts
- Implement graceful degradation
- Monitor health check results

### 4. Layer Optimization
- Combine related RUN commands
- Clean up temporary files
- Use .dockerignore
- Leverage build cache

## Common Patterns

### 1. Version Pinning
```dockerfile
ARG NODE_VERSION=14
FROM node:${NODE_VERSION}-alpine
```

### 2. Build-time Secrets
```dockerfile
RUN --mount=type=secret,id=mysecret \
    cat /run/secrets/mysecret > /app/config.json
```

### 3. Custom Health Checks
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost/health || exit 1
```

### 4. Layer Optimization
```dockerfile
RUN apk add --no-cache curl && \
    rm -rf /var/cache/apk/* && \
    chmod +x /usr/local/bin/script.sh
```

## Challenges
1. Create a multi-architecture build
2. Implement build-time configuration
3. Set up complex health checks
4. Optimize build performance
5. Implement secure secret handling

## Real-world Examples

### 1. Production Node.js Application
```dockerfile
# Build arguments
ARG NODE_VERSION=14
ARG APP_VERSION=1.0.0

# Dependencies stage
FROM node:${NODE_VERSION} AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Builder stage
FROM node:${NODE_VERSION} AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Production stage
FROM node:${NODE_VERSION}-alpine
WORKDIR /app

# Copy built assets
COPY --from=builder /app/dist ./dist
COPY --from=deps /app/node_modules ./node_modules
COPY package*.json ./

# Security
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
RUN chown -R appuser:appgroup /app
USER appuser

# Environment
ENV NODE_ENV=production
ENV APP_VERSION=${APP_VERSION}

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

EXPOSE 3000
CMD ["node", "dist/app.js"]
```

### 2. Python Application with Secrets
```dockerfile
# Build stage
FROM python:3.9-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Runtime stage
FROM python:3.9-slim
WORKDIR /app

# Copy dependencies
COPY --from=builder /root/.local /root/.local
ENV PATH=/root/.local/bin:$PATH

# Copy application
COPY . .

# Security
RUN useradd -m appuser
RUN chown -R appuser:appuser /app
USER appuser

# Secrets
RUN --mount=type=secret,id=db_password \
    cat /run/secrets/db_password > /app/db_password.txt

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:5000/health || exit 1

EXPOSE 5000
CMD ["python", "app.py"]
```

## Next Steps
- Once you're comfortable with advanced Dockerfile techniques, move on to Lab 12: Docker Security
- Practice implementing these techniques in your projects

## Additional Resources
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [Build Arguments](https://docs.docker.com/engine/reference/builder/#arg)
- [Secrets Management](https://docs.docker.com/engine/reference/builder/#run---mounttypesecret)
- [Health Checks](https://docs.docker.com/engine/reference/builder/#healthcheck) 