# Lab 02: Your First Container

## Objective
In this lab, you will learn how to create and manage your first Docker container. You'll understand basic Docker commands and container lifecycle management.

## Prerequisites
- Completed Lab 01 (Docker Installation and Setup)
- Docker Desktop running on your system
- Basic understanding of command line interface

## Key Concepts
- Container: A lightweight, standalone, executable package that includes everything needed to run a piece of software
- Image: A read-only template used to create containers
- Docker Hub: A cloud-based registry service for sharing and managing container images

## Exercises

### 1. Pull and Run a Simple Container
1. Pull the official Nginx image:
```powershell
docker pull nginx:latest
```

2. Run the Nginx container:
```powershell
docker run -d -p 80:80 --name my-nginx nginx
```

3. Verify the container is running:
```powershell
docker ps
```

4. Access the Nginx welcome page by opening http://localhost in your web browser

### 2. Container Lifecycle Management
1. Stop the container:
```powershell
docker stop my-nginx
```

2. Start the container again:
```powershell
docker start my-nginx
```

3. Restart the container:
```powershell
docker restart my-nginx
```

4. Remove the container:
```powershell
docker rm -f my-nginx
```

### 3. Container Inspection
1. View container logs:
```powershell
docker logs my-nginx
```

2. Get detailed information about the container:
```powershell
docker inspect my-nginx
```

3. View resource usage:
```powershell
docker stats my-nginx
```

## Challenges

1. Create a container using the official Python image
2. Run a simple Python web server in the container
3. Access the web server from your host machine
4. Modify the container's port mapping
5. Clean up all containers and images

## Common Commands Reference
```powershell
# List running containers
docker ps

# List all containers (including stopped ones)
docker ps -a

# Pull an image
docker pull <image-name>

# Run a container
docker run [options] <image-name>

# Stop a container
docker stop <container-name>

# Start a container
docker start <container-name>

# Remove a container
docker rm <container-name>

# View container logs
docker logs <container-name>
```

## Next Steps
- Once you're comfortable with basic container management, move on to Lab 03: Docker Images
- Practice the commands until you can execute them without referring to the documentation

## Additional Resources
- [Docker Run Reference](https://docs.docker.com/engine/reference/run/)
- [Docker CLI Reference](https://docs.docker.com/engine/reference/commandline/cli/)
- [Docker Hub](https://hub.docker.com/) 