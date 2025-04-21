# Lab 08: Multi-stage Builds

## Objective
In this lab, you will learn how to use multi-stage builds to create smaller, more efficient Docker images. You'll understand how to separate build dependencies from runtime requirements and optimize your images.

## Prerequisites
- Completed Lab 07 (Docker Compose Basics)
- Docker Desktop running on your system
- Basic understanding of build tools and compilers
- Familiarity with programming languages (Python, Node.js, or Go)

## Key Concepts
- Build Stage: A temporary container used for compiling and building
- Runtime Stage: The final container that runs the application
- Builder Pattern: Using multiple stages to separate build and runtime
- Layer Caching: Optimizing build time by reusing layers
- Image Size Optimization: Reducing final image size

## Exercises

### 1. Basic Multi-stage Build
1. Create a simple Go application (main.go):
```go
package main

import "fmt"

func main() {
    fmt.Println("Hello from a multi-stage build!")
}
```

2. Create a Dockerfile with two stages:
```dockerfile
# Build stage
FROM golang:1.16-alpine AS builder

WORKDIR /app
COPY . .
RUN go build -o main .

# Runtime stage
FROM alpine:latest

WORKDIR /app
COPY --from=builder /app/main .
CMD ["./main"]
```

3. Build and run the image:
```powershell
docker build -t multi-stage-go .
docker run multi-stage-go
```

### 2. Node.js Multi-stage Build
1. Create a simple Node.js application (app.js):
```javascript
const http = require('http');

const server = http.createServer((req, res) => {
    res.end('Hello from Node.js multi-stage build!');
});

server.listen(3000);
```

2. Create a multi-stage Dockerfile:
```dockerfile
# Build stage
FROM node:14 AS builder

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Runtime stage
FROM node:14-alpine

WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./

EXPOSE 3000
CMD ["node", "dist/app.js"]
```

### 3. Python Multi-stage Build
1. Create a simple Python application (app.py):
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from Python multi-stage build!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

2. Create a multi-stage Dockerfile:
```dockerfile
# Build stage
FROM python:3.9-slim AS builder

WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Runtime stage
FROM python:3.9-slim

WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .

ENV PATH=/root/.local/bin:$PATH
EXPOSE 5000
CMD ["python", "app.py"]
```

## Best Practices
1. Use minimal base images for runtime stage
2. Copy only necessary files from build stage
3. Clean up build dependencies
4. Use specific version tags for base images
5. Leverage layer caching

## Common Patterns

### 1. Builder Pattern
```dockerfile
FROM node:14 AS builder
# Build steps...

FROM node:14-alpine
COPY --from=builder /app/dist ./dist
```

### 2. Development Tools
```dockerfile
FROM golang:1.16 AS builder
# Install development tools...

FROM golang:1.16-alpine
# Copy only the binary
```

### 3. Asset Compilation
```dockerfile
FROM node:14 AS assets
# Compile assets...

FROM nginx:alpine
COPY --from=assets /app/dist /usr/share/nginx/html
```

## Challenges
1. Create a multi-stage build for a React application
2. Build a Python application with compiled dependencies
3. Create a Java application with Maven build
4. Optimize a multi-stage build for a C++ application
5. Create a multi-architecture build

## Real-world Examples

### 1. React Application
```dockerfile
# Build stage
FROM node:14 AS builder

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Runtime stage
FROM nginx:alpine

COPY --from=builder /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 2. Java Spring Boot
```dockerfile
# Build stage
FROM maven:3.6-jdk-11 AS builder

WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn package

# Runtime stage
FROM openjdk:11-jre-slim

WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

## Next Steps
- Once you're comfortable with multi-stage builds, move on to Lab 09: Docker Best Practices
- Practice creating multi-stage builds for different types of applications

## Additional Resources
- [Multi-stage Builds Documentation](https://docs.docker.com/develop/develop-images/multistage-build/)
- [Builder Pattern Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Optimizing Docker Images](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#minimize-the-number-of-layers) 