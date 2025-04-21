# Lab 04: Dockerfile Basics

## Objective
In this lab, you will learn how to create and use Dockerfiles to build custom Docker images. You'll understand the basic Dockerfile instructions and how to create efficient images.

## Prerequisites
- Completed Lab 03 (Docker Images)
- Docker Desktop running on your system
- Basic understanding of Linux commands
- A text editor (VS Code recommended)

## Key Concepts
- Dockerfile: A text document containing commands to assemble an image
- Build Context: The set of files used to build an image
- Layer: Each instruction in a Dockerfile creates a new layer
- Caching: Docker uses layer caching to speed up builds

## Exercises

### 1. Create Your First Dockerfile
1. Create a new directory for your project:
```powershell
mkdir my-first-dockerfile
cd my-first-dockerfile
```

2. Create a simple Python application (app.py):
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello, Docker!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

3. Create a Dockerfile:
```dockerfile
# Use an official Python runtime as a parent image
FROM python:3.9-slim

# Set the working directory
WORKDIR /app

# Copy the requirements file
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application
COPY . .

# Make port 5000 available to the world outside this container
EXPOSE 5000

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

4. Create requirements.txt:
```
flask==2.0.1
```

### 2. Build and Run Your Image
1. Build the image:
```powershell
docker build -t my-flask-app .
```

2. Run the container:
```powershell
docker run -p 5000:5000 my-flask-app
```

3. Access your application at http://localhost:5000

### 3. Dockerfile Instructions Practice
1. Create a new Dockerfile with different instructions:
```dockerfile
FROM ubuntu:20.04

# Update package list and install curl
RUN apt-get update && apt-get install -y curl

# Create a directory and set it as working directory
WORKDIR /myapp

# Copy a file from host to container
COPY hello.txt .

# Set environment variables
ENV MY_VAR=value

# Expose a port
EXPOSE 8080

# Define the command to run
CMD ["echo", "Hello, Docker!"]
```

## Common Dockerfile Instructions
```dockerfile
# Base image
FROM <image>:<tag>

# Set working directory
WORKDIR /path/to/workdir

# Copy files
COPY <src> <dest>

# Add files (supports URLs and tar extraction)
ADD <src> <dest>

# Run commands
RUN <command>

# Set environment variables
ENV <key>=<value>

# Expose ports
EXPOSE <port>

# Define the command to run
CMD ["executable", "param1", "param2"]
```

## Best Practices
1. Use specific version tags for base images
2. Combine RUN commands to reduce layers
3. Clean up package manager caches in the same layer
4. Use .dockerignore to exclude unnecessary files
5. Order instructions from least to most frequently changing

## Challenges
1. Create a Dockerfile for a Node.js application
2. Build an image that serves a static HTML file using Nginx
3. Create a multi-service application using multiple Dockerfiles
4. Optimize a Dockerfile to reduce image size
5. Add health checks to your application

## Next Steps
- Once you're comfortable with basic Dockerfiles, move on to Lab 05: Docker Volumes
- Practice creating different types of Dockerfiles for various applications

## Additional Resources
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [Best Practices for Writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Dockerfile Examples](https://docs.docker.com/samples/) 