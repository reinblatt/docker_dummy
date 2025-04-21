# Lab 27: Docker CI/CD Advanced

## Objective
In this lab, you will learn advanced Docker CI/CD concepts and techniques. You'll understand how to implement sophisticated continuous integration and continuous deployment pipelines, including multi-stage builds, automated testing, deployment strategies, and monitoring.

## Prerequisites
- Completed Lab 26 (Docker Performance Advanced)
- Docker Desktop running on your system
- Basic understanding of CI/CD concepts
- Familiarity with Docker Compose
- Access to a CI/CD platform (e.g., GitHub Actions, GitLab CI, Jenkins)

## Key Concepts
- Multi-stage Builds: Optimized image creation
- Automated Testing: Integration and deployment testing
- Deployment Strategies: Rolling updates and blue-green deployments
- Pipeline Orchestration: Complex workflow management
- Monitoring and Rollback: Deployment health and recovery

## Exercises

### 1. Multi-stage Builds
1. Create an optimized multi-stage build:
```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

2. Implement caching strategies:
```dockerfile
# Build stage with caching
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 2. Automated Testing
1. Set up test automation:
```yaml
# .github/workflows/test.yml
name: Test Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build and test
        run: |
          docker build -t test-image .
          docker run test-image npm test
```

2. Configure integration tests:
```yaml
# docker-compose.test.yml
version: '3.8'

services:
  test:
    build:
      context: .
      target: test
    environment:
      - NODE_ENV=test
    command: npm run test:integration
    depends_on:
      - db
      - redis

  db:
    image: postgres:13
    environment:
      - POSTGRES_USER=test
      - POSTGRES_PASSWORD=test
      - POSTGRES_DB=test

  redis:
    image: redis:6
```

### 3. Deployment Strategies
1. Implement rolling updates:
```yaml
# docker-compose.deploy.yml
version: '3.8'

services:
  web:
    image: ${DOCKER_REGISTRY}/web:${VERSION}
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      rollback_config:
        parallelism: 1
        delay: 10s
        order: stop-first
      restart_policy:
        condition: on-failure
```

2. Configure blue-green deployment:
```yaml
# docker-compose.blue-green.yml
version: '3.8'

services:
  web-blue:
    image: ${DOCKER_REGISTRY}/web:${BLUE_VERSION}
    deploy:
      replicas: 3
      labels:
        - "traefik.http.routers.web-blue.rule=Host(`blue.example.com`)"

  web-green:
    image: ${DOCKER_REGISTRY}/web:${GREEN_VERSION}
    deploy:
      replicas: 3
      labels:
        - "traefik.http.routers.web-green.rule=Host(`green.example.com`)"
```

### 4. Pipeline Orchestration
1. Create a complete CI/CD pipeline:
```yaml
# .github/workflows/pipeline.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build and test
        run: |
          docker build -t test-image .
          docker run test-image npm test
      - name: Build production image
        run: |
          docker build -t prod-image --target production .
      - name: Push to registry
        run: |
          docker push ${DOCKER_REGISTRY}/web:${GITHUB_SHA}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to production
        run: |
          docker stack deploy -c docker-compose.prod.yml web
```

2. Implement deployment monitoring:
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

  grafana:
    image: grafana/grafana:latest
    volumes:
      - grafana-storage:/var/lib/grafana
    ports:
      - "3000:3000"

  alertmanager:
    image: prom/alertmanager:latest
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    ports:
      - "9093:9093"
```

## Best Practices

### 1. Build Optimization
- Use multi-stage builds
- Implement build caching
- Optimize layer creation
- Minimize image size
- Use appropriate base images

### 2. Testing
- Implement comprehensive testing
- Use test containers
- Configure test environments
- Automate test execution
- Monitor test coverage

### 3. Deployment
- Use appropriate deployment strategies
- Implement health checks
- Configure rollback procedures
- Monitor deployments
- Document deployment processes

### 4. Monitoring
- Set up deployment monitoring
- Configure alerting
- Track deployment metrics
- Monitor application health
- Document monitoring setup

## Common Patterns

### 1. Multi-stage Build
```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Test stage
FROM builder AS test
RUN npm run test

# Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 2. CI/CD Pipeline
```yaml
# .github/workflows/pipeline.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build and test
        run: |
          docker build -t test-image .
          docker run test-image npm test
      - name: Build production image
        run: |
          docker build -t prod-image --target production .
      - name: Push to registry
        run: |
          docker push ${DOCKER_REGISTRY}/web:${GITHUB_SHA}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to production
        run: |
          docker stack deploy -c docker-compose.prod.yml web
```

### 3. Deployment Configuration
```yaml
# docker-compose.deploy.yml
version: '3.8'

services:
  web:
    image: ${DOCKER_REGISTRY}/web:${VERSION}
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      rollback_config:
        parallelism: 1
        delay: 10s
        order: stop-first
      restart_policy:
        condition: on-failure
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### 4. Monitoring Setup
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

  grafana:
    image: grafana/grafana:latest
    volumes:
      - grafana-storage:/var/lib/grafana
    ports:
      - "3000:3000"

  alertmanager:
    image: prom/alertmanager:latest
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    ports:
      - "9093:9093"
```

## Challenges
1. Implement multi-stage builds
2. Set up automated testing
3. Configure deployment strategies
4. Create CI/CD pipelines
5. Implement monitoring and rollback

## Real-world Examples

### 1. Complete CI/CD Stack
```yaml
# docker-compose.ci.yml
version: '3.8'

services:
  jenkins:
    image: jenkins/jenkins:lts
    volumes:
      - jenkins-data:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - "8080:8080"
      - "50000:50000"

  gitlab:
    image: gitlab/gitlab-ce:latest
    volumes:
      - gitlab-data:/var/opt/gitlab
      - gitlab-logs:/var/log/gitlab
      - gitlab-config:/etc/gitlab
    ports:
      - "80:80"
      - "443:443"
      - "22:22"

  sonarqube:
    image: sonarqube:latest
    volumes:
      - sonarqube-data:/opt/sonarqube/data
      - sonarqube-logs:/opt/sonarqube/logs
      - sonarqube-extensions:/opt/sonarqube/extensions
    ports:
      - "9000:9000"

volumes:
  jenkins-data:
  gitlab-data:
  gitlab-logs:
  gitlab-config:
  sonarqube-data:
  sonarqube-logs:
  sonarqube-extensions:
```

### 2. Deployment Pipeline
```yaml
# .github/workflows/deploy.yml
name: Deployment Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build and test
        run: |
          docker build -t test-image .
          docker run test-image npm test
      - name: Build production image
        run: |
          docker build -t prod-image --target production .
      - name: Push to registry
        run: |
          docker push ${DOCKER_REGISTRY}/web:${GITHUB_SHA}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to staging
        run: |
          docker stack deploy -c docker-compose.staging.yml web
      - name: Run integration tests
        run: |
          docker-compose -f docker-compose.test.yml run test
      - name: Deploy to production
        run: |
          docker stack deploy -c docker-compose.prod.yml web

  monitor:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - name: Monitor deployment
        run: |
          docker stack deploy -c docker-compose.monitor.yml monitor
```

## Next Steps
- Move on to Lab 28: Docker Orchestration Advanced
- Practice implementing advanced CI/CD in your projects
- Explore additional CI/CD tools and techniques

## Additional Resources
- [Docker CI/CD Documentation](https://docs.docker.com/ci-cd/)
- [Multi-stage Builds Guide](https://docs.docker.com/develop/develop-images/multistage-build/)
- [Deployment Strategies Guide](https://docs.docker.com/engine/swarm/swarm-tutorial/rolling-update/)
- [CI/CD Best Practices](https://docs.docker.com/ci-cd/best-practices/) 