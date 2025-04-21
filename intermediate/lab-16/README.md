# Lab 16: Docker CI/CD

## Objective
In this lab, you will learn how to implement Continuous Integration and Continuous Deployment (CI/CD) pipelines using Docker. You'll understand how to automate the build, test, and deployment processes for containerized applications.

## Prerequisites
- Completed Lab 15 (Docker Monitoring)
- Docker Desktop running on your system
- Basic understanding of CI/CD concepts
- Familiarity with Docker Compose
- GitHub account (for version control)

## Key Concepts
- CI/CD Pipeline: Automated build, test, and deployment process
- Docker Build: Automated image building
- Testing: Container testing strategies
- Deployment: Automated deployment methods
- Security: Secure CI/CD practices

## Exercises

### 1. Basic CI/CD Setup
1. Create a GitHub repository:
```powershell
# Initialize git repository
git init

# Add Dockerfile and application code
git add .

# Commit changes
git commit -m "Initial commit"

# Add remote repository
git remote add origin https://github.com/username/repo.git

# Push to GitHub
git push -u origin main
```

2. Set up GitHub Actions workflow:
```yaml
# .github/workflows/docker.yml
name: Docker CI/CD

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
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1
      
      - name: Login to DockerHub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: username/app:latest
```

### 2. Automated Testing
1. Create test configuration:
```yaml
# docker-compose.test.yml
version: '3.8'

services:
  test:
    build:
      context: .
      dockerfile: Dockerfile.test
    volumes:
      - ./tests:/app/tests
    environment:
      - TEST_ENV=ci
    command: pytest /app/tests
```

2. Add test stage to Dockerfile:
```dockerfile
# Dockerfile.test
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["pytest", "/app/tests"]
```

### 3. Deployment Automation
1. Configure deployment:
```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Deploy to production
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USERNAME }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            docker pull username/app:latest
            docker-compose -f docker-compose.prod.yml up -d
```

2. Create production compose file:
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  app:
    image: username/app:latest
    ports:
      - "80:80"
    environment:
      - NODE_ENV=production
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
```

### 4. Security Scanning
1. Add security scanning:
```yaml
# .github/workflows/security.yml
name: Security Scan

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: username/app:latest
          format: 'table'
          exit-code: '1'
          ignore-unfixed: true
          severity: 'CRITICAL,HIGH'
```

## Best Practices

### 1. Pipeline Design
- Keep pipelines simple and maintainable
- Use appropriate triggers
- Implement proper error handling
- Include rollback procedures
- Document pipeline processes

### 2. Testing Strategy
- Implement comprehensive testing
- Use appropriate test environments
- Automate test execution
- Monitor test coverage
- Handle test failures gracefully

### 3. Deployment
- Use blue-green deployments
- Implement canary releases
- Monitor deployment health
- Automate rollbacks
- Document deployment procedures

### 4. Security
- Scan for vulnerabilities
- Use secure secrets management
- Implement access controls
- Monitor security events
- Follow security best practices

## Common Patterns

### 1. Basic CI Pipeline
```yaml
name: CI Pipeline

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build Docker image
        run: docker build -t app .
```

### 2. Testing Configuration
```yaml
services:
  test:
    build:
      context: .
      dockerfile: Dockerfile.test
    environment:
      - TEST_ENV=ci
    command: pytest
```

### 3. Deployment Configuration
```yaml
services:
  app:
    image: app:latest
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
```

### 4. Security Scanning
```yaml
jobs:
  security:
    steps:
      - name: Scan for vulnerabilities
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: app:latest
```

## Challenges
1. Set up a complete CI/CD pipeline
2. Implement automated testing
3. Configure secure deployments
4. Add security scanning
5. Create deployment strategies

## Real-world Examples

### 1. Complete CI/CD Pipeline
```yaml
name: Complete CI/CD

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
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1
      
      - name: Login to DockerHub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: username/app:latest
      
      - name: Run tests
        run: docker-compose -f docker-compose.test.yml up --abort-on-container-exit
      
      - name: Security scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: username/app:latest
          format: 'table'
          exit-code: '1'
          ignore-unfixed: true
          severity: 'CRITICAL,HIGH'
      
      - name: Deploy to production
        if: github.ref == 'refs/heads/main'
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USERNAME }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            docker pull username/app:latest
            docker-compose -f docker-compose.prod.yml up -d
```

### 2. Multi-stage Deployment
```yaml
version: '3.8'

services:
  app:
    image: username/app:latest
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      rollback_config:
        parallelism: 0
        order: stop-first
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:80/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

## Next Steps
- Once you're comfortable with Docker CI/CD, move on to Lab 17: Docker Swarm Advanced
- Practice implementing these CI/CD patterns in your projects

## Additional Resources
- [Docker CI/CD Documentation](https://docs.docker.com/ci-cd/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Security Scanning](https://docs.docker.com/engine/scan/)
- [CI/CD Best Practices](https://docs.docker.com/ci-cd/best-practices/) 