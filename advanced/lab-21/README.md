# Lab 21: Docker Multi-Architecture Builds

## Objective
In this lab, you will learn how to build and manage Docker images for multiple CPU architectures. You'll understand how to use Docker Buildx to create multi-architecture images, manage build contexts, and optimize builds for different platforms.

## Prerequisites
- Completed Lab 20 (Docker Enterprise Features)
- Docker Desktop running on your system
- Basic understanding of CPU architectures
- Familiarity with Docker Build
- Access to Docker Hub or a private registry

## Key Concepts
- Multi-Architecture Builds: Creating images for different CPU architectures
- Buildx: Docker's extended build capabilities
- Platform Support: Understanding different CPU architectures
- Build Context: Managing build environments
- Image Manifest: Managing multi-architecture image lists

## Exercises

### 1. Setting Up Buildx
1. Create and use a new builder:
```powershell
# Create a new builder instance
docker buildx create --name multi-arch --driver docker-container --use

# Verify builder setup
docker buildx inspect --bootstrap

# List available builders
docker buildx ls
```

2. Configure build context:
```powershell
# Set up QEMU for cross-platform builds
docker run --privileged --rm tonistiigi/binfmt --install all

# Verify QEMU setup
docker buildx inspect --bootstrap
```

### 2. Multi-Platform Builds
1. Build for multiple platforms:
```powershell
# Build for multiple architectures
docker buildx build \
    --platform linux/amd64,linux/arm64,linux/arm/v7 \
    -t myregistry.com/myapp:latest \
    --push .

# Build with specific platform features
docker buildx build \
    --platform linux/amd64,linux/arm64 \
    --build-arg BUILDPLATFORM=linux/amd64 \
    --build-arg TARGETPLATFORM=linux/arm64 \
    -t myregistry.com/myapp:latest \
    --push .
```

2. Optimize builds:
```powershell
# Use build cache
docker buildx build \
    --platform linux/amd64,linux/arm64 \
    --cache-from=type=registry,ref=myregistry.com/myapp:buildcache \
    --cache-to=type=registry,ref=myregistry.com/myapp:buildcache \
    -t myregistry.com/myapp:latest \
    --push .

# Parallel builds
docker buildx build \
    --platform linux/amd64,linux/arm64 \
    --parallel \
    -t myregistry.com/myapp:latest \
    --push .
```

### 3. Platform-Specific Optimizations
1. Create platform-specific Dockerfiles:
```dockerfile
# Base image with platform-specific optimizations
FROM --platform=$BUILDPLATFORM golang:1.21 AS builder
ARG TARGETPLATFORM
ARG BUILDPLATFORM
WORKDIR /app
COPY . .
RUN GOOS=$(echo $TARGETPLATFORM | cut -d/ -f1) \
    GOARCH=$(echo $TARGETPLATFORM | cut -d/ -f2) \
    go build -o app

# Final image
FROM --platform=$TARGETPLATFORM alpine:latest
COPY --from=builder /app/app /app
CMD ["/app"]
```

2. Use platform-specific features:
```dockerfile
# Platform-specific optimizations
FROM --platform=$TARGETPLATFORM nginx:alpine
ARG TARGETPLATFORM
RUN if [ "$TARGETPLATFORM" = "linux/arm64" ]; then \
        apk add --no-cache arm64-optimized-package; \
    elif [ "$TARGETPLATFORM" = "linux/amd64" ]; then \
        apk add --no-cache amd64-optimized-package; \
    fi
```

### 4. Image Manifest Management
1. Create and manage manifests:
```powershell
# Create a manifest list
docker manifest create myregistry.com/myapp:latest \
    myregistry.com/myapp:amd64 \
    myregistry.com/myapp:arm64 \
    myregistry.com/myapp:armv7

# Annotate manifests
docker manifest annotate \
    myregistry.com/myapp:latest \
    myregistry.com/myapp:arm64 \
    --os linux --arch arm64 --variant v8

# Push manifest list
docker manifest push myregistry.com/myapp:latest
```

2. Inspect and manage manifests:
```powershell
# Inspect manifest
docker manifest inspect myregistry.com/myapp:latest

# Remove manifest
docker manifest rm myregistry.com/myapp:latest
```

## Best Practices

### 1. Build Configuration
- Use appropriate base images
- Implement platform-specific optimizations
- Leverage build cache
- Use parallel builds when possible
- Implement proper error handling

### 2. Platform Support
- Test on target platforms
- Use platform-specific features
- Implement fallback mechanisms
- Monitor build performance
- Document platform requirements

### 3. Image Management
- Use semantic versioning
- Implement proper tagging
- Maintain manifest lists
- Monitor image sizes
- Implement cleanup strategies

### 4. Performance
- Optimize build times
- Minimize image sizes
- Use appropriate compression
- Implement caching strategies
- Monitor resource usage

## Common Patterns

### 1. Multi-Platform Build
```yaml
version: '3.8'

services:
  builder:
    image: tonistiigi/binfmt
    command: --install all
    privileged: true

  app:
    build:
      context: .
      platforms:
        - linux/amd64
        - linux/arm64
        - linux/arm/v7
      args:
        - BUILDPLATFORM=linux/amd64
        - TARGETPLATFORM=linux/arm64
    image: myregistry.com/myapp:latest
```

### 2. Platform-Specific Dockerfile
```dockerfile
# Build stage
FROM --platform=$BUILDPLATFORM golang:1.21 AS builder
ARG TARGETPLATFORM
ARG BUILDPLATFORM
WORKDIR /app
COPY . .
RUN GOOS=$(echo $TARGETPLATFORM | cut -d/ -f1) \
    GOARCH=$(echo $TARGETPLATFORM | cut -d/ -f2) \
    go build -o app

# Final stage
FROM --platform=$TARGETPLATFORM alpine:latest
COPY --from=builder /app/app /app
CMD ["/app"]
```

### 3. Build Configuration
```yaml
version: '3.8'

services:
  build:
    image: docker/binfmt:latest
    command: --install all
    privileged: true

  app:
    build:
      context: .
      platforms:
        - linux/amd64
        - linux/arm64
      cache_from:
        - type=registry,ref=myregistry.com/myapp:buildcache
      cache_to:
        - type=registry,ref=myregistry.com/myapp:buildcache
    image: myregistry.com/myapp:latest
```

### 4. Manifest Management
```yaml
version: '3.8'

services:
  manifest:
    image: alpine:latest
    command: >
      sh -c "
        docker manifest create myregistry.com/myapp:latest \
          myregistry.com/myapp:amd64 \
          myregistry.com/myapp:arm64;
        docker manifest push myregistry.com/myapp:latest
      "
```

## Challenges
1. Create a multi-architecture build pipeline
2. Implement platform-specific optimizations
3. Set up automated testing for different architectures
4. Optimize build performance
5. Implement manifest management

## Real-world Examples

### 1. Complete Multi-Architecture Stack
```yaml
version: '3.8'

services:
  builder:
    image: tonistiigi/binfmt
    command: --install all
    privileged: true

  app:
    build:
      context: .
      platforms:
        - linux/amd64
        - linux/arm64
        - linux/arm/v7
      args:
        - BUILDPLATFORM=linux/amd64
        - TARGETPLATFORM=linux/arm64
      cache_from:
        - type=registry,ref=myregistry.com/myapp:buildcache
      cache_to:
        - type=registry,ref=myregistry.com/myapp:buildcache
    image: myregistry.com/myapp:latest
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G

  test:
    image: myregistry.com/myapp:latest
    command: >
      sh -c "
        docker run --platform linux/amd64 myregistry.com/myapp:latest /app/test;
        docker run --platform linux/arm64 myregistry.com/myapp:latest /app/test;
        docker run --platform linux/arm/v7 myregistry.com/myapp:latest /app/test
      "
    depends_on:
      - app

  manifest:
    image: alpine:latest
    command: >
      sh -c "
        docker manifest create myregistry.com/myapp:latest \
          myregistry.com/myapp:amd64 \
          myregistry.com/myapp:arm64 \
          myregistry.com/myapp:armv7;
        docker manifest push myregistry.com/myapp:latest
      "
    depends_on:
      - app
```

### 2. Build Pipeline
```yaml
version: '3.8'

services:
  builder:
    image: tonistiigi/binfmt
    command: --install all
    privileged: true

  build:
    image: docker:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: >
      sh -c "
        docker buildx create --name multi-arch --driver docker-container --use;
        docker buildx build \
          --platform linux/amd64,linux/arm64,linux/arm/v7 \
          --cache-from=type=registry,ref=myregistry.com/myapp:buildcache \
          --cache-to=type=registry,ref=myregistry.com/myapp:buildcache \
          -t myregistry.com/myapp:latest \
          --push .;
        docker buildx rm multi-arch
      "

  test:
    image: myregistry.com/myapp:latest
    command: >
      sh -c "
        for platform in linux/amd64 linux/arm64 linux/arm/v7; do
          docker run --platform $platform myregistry.com/myapp:latest /app/test;
        done
      "
    depends_on:
      - build

  manifest:
    image: alpine:latest
    command: >
      sh -c "
        docker manifest create myregistry.com/myapp:latest \
          myregistry.com/myapp:amd64 \
          myregistry.com/myapp:arm64 \
          myregistry.com/myapp:armv7;
        docker manifest push myregistry.com/myapp:latest
      "
    depends_on:
      - test
```

## Next Steps
- Move on to Lab 22: Docker Security Scanning
- Practice implementing multi-architecture builds in your projects
- Explore additional build optimization techniques

## Additional Resources
- [Docker Buildx Documentation](https://docs.docker.com/buildx/working-with-buildx/)
- [Multi-Platform Build Guide](https://docs.docker.com/build/building/multi-platform/)
- [Docker Manifest Documentation](https://docs.docker.com/engine/reference/commandline/manifest/)
- [BuildKit Documentation](https://github.com/moby/buildkit) 