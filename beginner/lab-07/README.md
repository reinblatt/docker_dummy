# Lab 07: Docker Compose Basics

## Objective
In this lab, you will learn how to use Docker Compose to define and run multi-container Docker applications. You'll understand how to create and manage services, networks, and volumes using a single configuration file.

## Prerequisites
- Completed Lab 06 (Docker Networking Basics)
- Docker Desktop running on your system
- Docker Compose installed (comes with Docker Desktop)
- Basic understanding of YAML syntax

## Key Concepts
- Service: A container definition in a Compose file
- Compose File: YAML file that defines services, networks, and volumes
- Project: A set of services defined in a Compose file
- Environment Variables: Configuration values passed to containers
- Dependencies: Service relationships and startup order

## Exercises

### 1. Create Your First Compose File
1. Create a new directory for your project:
```powershell
mkdir my-compose-app
cd my-compose-app
```

2. Create a docker-compose.yml file:
```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - app-network

  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: mydb
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  mysql-data:
```

3. Create a simple HTML file:
```powershell
mkdir html
echo "<h1>Hello from Docker Compose!</h1>" > html/index.html
```

### 2. Managing Services
1. Start all services:
```powershell
docker-compose up -d
```

2. View running services:
```powershell
docker-compose ps
```

3. View service logs:
```powershell
docker-compose logs
```

4. Stop all services:
```powershell
docker-compose down
```

### 3. Service Configuration
1. Add environment variables:
```yaml
services:
  web:
    environment:
      - NODE_ENV=production
      - DEBUG=false
```

2. Configure health checks:
```yaml
services:
  web:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
```

3. Set resource limits:
```yaml
services:
  web:
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
```

## Common Compose Commands
```powershell
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View service status
docker-compose ps

# View service logs
docker-compose logs

# Execute command in service
docker-compose exec <service> <command>

# Build services
docker-compose build

# Scale services
docker-compose up -d --scale <service>=<count>
```

## Compose File Structure
```yaml
version: '3.8'

services:
  service1:
    image: image:tag
    ports:
      - "host:container"
    volumes:
      - source:target
    environment:
      - VAR=value
    networks:
      - network-name
    depends_on:
      - service2

  service2:
    # service configuration

networks:
  network-name:
    driver: bridge

volumes:
  volume-name:
```

## Best Practices
1. Use specific version tags for images
2. Define all services in a single Compose file
3. Use environment variables for configuration
4. Define proper service dependencies
5. Use named volumes for persistent data

## Challenges
1. Create a WordPress site with MySQL database
2. Set up a development environment with hot-reloading
3. Configure a load-balanced web application
4. Create a microservices architecture
5. Implement health checks and monitoring

## Real-world Examples

### 1. WordPress with MySQL
```yaml
version: '3.8'

services:
  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress-data:/var/www/html
    depends_on:
      - db

  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  wordpress-data:
  mysql-data:
```

### 2. Development Environment
```yaml
version: '3.8'

services:
  frontend:
    build: ./frontend
    volumes:
      - ./frontend:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development

  backend:
    build: ./backend
    volumes:
      - ./backend:/app
      - /app/node_modules
    ports:
      - "4000:4000"
    environment:
      - NODE_ENV=development
      - DB_HOST=db

  db:
    image: postgres:13
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: dev
    volumes:
      - postgres-data:/var/lib/postgresql/data
```

## Next Steps
- Once you're comfortable with basic Compose, move on to Lab 08: Multi-stage Builds
- Practice creating different types of multi-service applications

## Additional Resources
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Compose File Reference](https://docs.docker.com/compose/compose-file/)
- [Compose CLI Reference](https://docs.docker.com/compose/reference/) 