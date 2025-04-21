# Lab 05: Docker Volumes

## Objective
In this lab, you will learn how to work with Docker volumes to persist data and share files between containers and the host system. You'll understand different types of volumes and their use cases.

## Prerequisites
- Completed Lab 04 (Dockerfile Basics)
- Docker Desktop running on your system
- Basic understanding of file systems and paths

## Key Concepts
- Volume: A persistent data storage mechanism in Docker
- Bind Mount: Direct mapping of a host directory to a container directory
- Named Volume: Docker-managed storage with a specific name
- Anonymous Volume: Temporary storage that gets removed with the container
- Volume Driver: Plugin that enables different storage backends

## Exercises

### 1. Working with Named Volumes
1. Create a named volume:
```powershell
docker volume create my-data
```

2. List all volumes:
```powershell
docker volume ls
```

3. Run a container with the volume:
```powershell
docker run -d --name nginx -v my-data:/usr/share/nginx/html nginx
```

4. Inspect the volume:
```powershell
docker volume inspect my-data
```

### 2. Using Bind Mounts
1. Create a directory on your host:
```powershell
mkdir C:\docker-data
```

2. Run a container with a bind mount:
```powershell
docker run -d --name nginx-bind -v C:\docker-data:/usr/share/nginx/html nginx
```

3. Create a test file in the mounted directory:
```powershell
echo "Hello from host" > C:\docker-data\test.txt
```

4. Verify the file in the container:
```powershell
docker exec nginx-bind cat /usr/share/nginx/html/test.txt
```

### 3. Volume Management
1. Remove a container with its volumes:
```powershell
docker rm -v nginx
```

2. Remove a specific volume:
```powershell
docker volume rm my-data
```

3. Remove all unused volumes:
```powershell
docker volume prune
```

## Common Volume Commands
```powershell
# Create a volume
docker volume create <volume-name>

# List volumes
docker volume ls

# Inspect a volume
docker volume inspect <volume-name>

# Remove a volume
docker volume rm <volume-name>

# Remove unused volumes
docker volume prune

# Run container with volume
docker run -v <volume-name>:<container-path> <image>

# Run container with bind mount
docker run -v <host-path>:<container-path> <image>
```

## Best Practices
1. Use named volumes for persistent data
2. Use bind mounts for development environments
3. Avoid writing to container's writable layer
4. Consider volume drivers for specific storage needs
5. Regularly clean up unused volumes

## Challenges
1. Create a MySQL container with persistent data storage
2. Set up a development environment with live code reloading
3. Share data between multiple containers
4. Backup and restore volume data
5. Configure a read-only volume

## Real-world Examples

### 1. Database with Persistent Storage
```powershell
docker run -d --name mysql \
  -v mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:5.7
```

### 2. Development Environment
```powershell
docker run -d --name dev-app \
  -v C:\projects\app:/app \
  -v /app/node_modules \
  node:14
```

### 3. Multiple Containers Sharing Data
```powershell
# Create a shared volume
docker volume create shared-data

# Run containers with the same volume
docker run -d --name app1 -v shared-data:/data nginx
docker run -d --name app2 -v shared-data:/data nginx
```

## Next Steps
- Once you're comfortable with volumes, move on to Lab 06: Docker Networking Basics
- Practice different volume scenarios to understand their behavior

## Additional Resources
- [Docker Volumes Documentation](https://docs.docker.com/storage/volumes/)
- [Bind Mounts Documentation](https://docs.docker.com/storage/bind-mounts/)
- [Volume Drivers Documentation](https://docs.docker.com/engine/extend/plugins_volume/) 