# Lab 06: Docker Networking Basics

## Objective
In this lab, you will learn the fundamentals of Docker networking, including different network types, container communication, and port mapping. You'll understand how to connect containers and expose services.

## Prerequisites
- Completed Lab 05 (Docker Volumes)
- Docker Desktop running on your system
- Basic understanding of networking concepts (IP addresses, ports)

## Key Concepts
- Bridge Network: Default network for container communication
- Host Network: Container shares host's network stack
- None Network: Container has no network access
- Port Mapping: Exposing container ports to the host
- Container Linking: Legacy method for container communication

## Exercises

### 1. Working with Bridge Networks
1. List existing networks:
```powershell
docker network ls
```

2. Create a new bridge network:
```powershell
docker network create my-network
```

3. Run containers in the network:
```powershell
docker run -d --name nginx1 --network my-network nginx
docker run -d --name nginx2 --network my-network nginx
```

4. Test communication between containers:
```powershell
docker exec nginx1 ping nginx2
```

### 2. Port Mapping
1. Run a container with port mapping:
```powershell
docker run -d --name web -p 8080:80 nginx
```

2. Access the service:
- Open http://localhost:8080 in your browser

3. Map multiple ports:
```powershell
docker run -d --name multi-port -p 8080:80 -p 8443:443 nginx
```

### 3. Network Inspection
1. Inspect a network:
```powershell
docker network inspect my-network
```

2. View container network details:
```powershell
docker inspect nginx1
```

3. Connect a running container to a network:
```powershell
docker network connect my-network nginx1
```

## Common Networking Commands
```powershell
# List networks
docker network ls

# Create a network
docker network create <network-name>

# Remove a network
docker network rm <network-name>

# Connect container to network
docker network connect <network-name> <container-name>

# Disconnect container from network
docker network disconnect <network-name> <container-name>

# Inspect a network
docker network inspect <network-name>
```

## Network Types

### Bridge Network
- Default network for containers
- Containers can communicate with each other
- Isolated from host network
- Example:
```powershell
docker run -d --network bridge nginx
```

### Host Network
- Container shares host's network stack
- No network isolation
- Better performance
- Example:
```powershell
docker run -d --network host nginx
```

### None Network
- Container has no network access
- Useful for security-sensitive applications
- Example:
```powershell
docker run -d --network none nginx
```

## Best Practices
1. Use custom bridge networks for related containers
2. Avoid using the default bridge network
3. Use specific port mappings instead of random ports
4. Consider network security when exposing ports
5. Use DNS names for container communication

## Challenges
1. Create a multi-container application with a web server and database
2. Set up a container that can only communicate with specific other containers
3. Configure a container to use the host network
4. Create a network that spans multiple Docker hosts
5. Implement a load balancer with multiple backend containers

## Real-world Examples

### 1. Web Application with Database
```powershell
# Create network
docker network create app-network

# Run database
docker run -d --name db --network app-network -e MYSQL_ROOT_PASSWORD=secret mysql:5.7

# Run web application
docker run -d --name web --network app-network -p 8080:80 nginx
```

### 2. Microservices Architecture
```powershell
# Create network
docker network create microservices

# Run services
docker run -d --name service1 --network microservices nginx
docker run -d --name service2 --network microservices nginx
docker run -d --name service3 --network microservices nginx
```

## Next Steps
- Once you're comfortable with basic networking, move on to Lab 07: Docker Compose Basics
- Practice different network configurations to understand their behavior

## Additional Resources
- [Docker Networking Documentation](https://docs.docker.com/network/)
- [Network Drivers Documentation](https://docs.docker.com/network/drivers/)
- [Container Networking Tutorial](https://docs.docker.com/network/network-tutorial-standalone/) 