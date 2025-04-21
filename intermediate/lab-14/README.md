# Lab 14: Docker Storage

## Objective
In this lab, you will learn advanced Docker storage concepts including volume drivers, storage plugins, persistent storage, and backup strategies. You'll understand how to manage and optimize storage for containerized applications.

## Prerequisites
- Completed Lab 13 (Docker Networking)
- Docker Desktop running on your system
- Basic understanding of storage concepts
- Familiarity with Docker Compose

## Key Concepts
- Volume Drivers: Different types of volume drivers and their use cases
- Storage Plugins: Extending Docker's storage capabilities
- Persistent Storage: Managing data persistence
- Backup and Restore: Data protection strategies
- Storage Optimization: Performance and resource management

## Exercises

### 1. Volume Drivers
1. Create and manage volumes:
```powershell
# Create a named volume
docker volume create my-volume

# Create a volume with specific driver
docker volume create --driver local \
    --opt type=none \
    --opt device=/path/to/storage \
    --opt o=bind my-bind-volume

# List volumes
docker volume ls

# Inspect a volume
docker volume inspect my-volume

# Remove a volume
docker volume rm my-volume
```

2. Use volumes in containers:
```powershell
# Mount a volume
docker run -d \
    --name web \
    -v my-volume:/app/data \
    nginx

# Mount a bind volume
docker run -d \
    --name web \
    -v /host/path:/container/path:ro \
    nginx

# Use volume in compose
docker-compose up -d
```

### 2. Storage Plugins
1. Install and configure a storage plugin:
```powershell
# Install a storage plugin
docker plugin install --grant-all-permissions \
    store/weaveworks/plugin:latest

# List installed plugins
docker plugin ls

# Configure plugin
docker volume create --driver weaveworks/plugin \
    --opt size=10GB \
    --opt redundancy=2 \
    my-shared-volume
```

2. Use plugin in compose:
```yaml
version: '3.8'

services:
  web:
    image: nginx
    volumes:
      - my-shared-volume:/app/data

volumes:
  my-shared-volume:
    driver: weaveworks/plugin
    driver_opts:
      size: "10GB"
      redundancy: "2"
```

### 3. Persistent Storage
1. Configure persistent storage:
```yaml
version: '3.8'

services:
  db:
    image: postgres:13
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./init:/docker-entrypoint-initdb.d
    environment:
      POSTGRES_PASSWORD: secret
    deploy:
      placement:
        constraints:
          - node.role == manager

  app:
    image: my-app
    volumes:
      - app-data:/app/data
    depends_on:
      - db

volumes:
  db-data:
    driver: local
    driver_opts:
      type: none
      device: /mnt/db-data
      o: bind
  app-data:
    driver: local
    driver_opts:
      type: none
      device: /mnt/app-data
      o: bind
```

2. Manage volume lifecycle:
```powershell
# Backup a volume
docker run --rm \
    -v db-data:/source \
    -v /backup:/backup \
    alpine tar -czf /backup/db-backup.tar.gz -C /source .

# Restore a volume
docker run --rm \
    -v db-data:/target \
    -v /backup:/backup \
    alpine sh -c "rm -rf /target/* && tar -xzf /backup/db-backup.tar.gz -C /target"
```

### 4. Storage Optimization
1. Configure storage options:
```yaml
version: '3.8'

services:
  db:
    image: postgres:13
    volumes:
      - db-data:/var/lib/postgresql/data
    deploy:
      resources:
        limits:
          memory: 1G
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_INITDB_ARGS: "--data-checksums"

volumes:
  db-data:
    driver: local
    driver_opts:
      type: none
      device: /mnt/db-data
      o: bind,noatime,nodiratime
```

2. Monitor storage usage:
```powershell
# View volume usage
docker system df -v

# Monitor container disk I/O
docker stats --format "table {{.Name}}\t{{.BlockIO}}"

# Check volume space
docker run --rm -v my-volume:/volume alpine df -h /volume
```

## Best Practices

### 1. Volume Management
- Use named volumes for persistence
- Implement proper backup strategies
- Monitor volume usage
- Clean up unused volumes
- Use appropriate volume drivers

### 2. Data Persistence
- Separate data from containers
- Use volume drivers for production
- Implement data backup
- Plan for data migration
- Consider data locality

### 3. Performance
- Use appropriate storage drivers
- Optimize mount options
- Monitor I/O performance
- Implement caching strategies
- Consider storage tiering

### 4. Security
- Use read-only mounts when possible
- Implement proper permissions
- Encrypt sensitive data
- Use secure volume drivers
- Monitor access patterns

## Common Patterns

### 1. Database Storage
```yaml
volumes:
  db-data:
    driver: local
    driver_opts:
      type: none
      device: /mnt/db-data
      o: bind,noatime
```

### 2. Shared Storage
```yaml
volumes:
  shared-data:
    driver: local
    driver_opts:
      type: none
      device: /mnt/shared
      o: bind
```

### 3. Backup Configuration
```yaml
services:
  backup:
    image: alpine
    volumes:
      - source:/source:ro
      - backup:/backup
    command: >
      sh -c "tar -czf /backup/backup.tar.gz -C /source ."
```

### 4. Performance Optimization
```yaml
volumes:
  fast-storage:
    driver: local
    driver_opts:
      type: none
      device: /mnt/ssd
      o: bind,noatime,nodiratime
```

## Challenges
1. Implement a backup strategy
2. Configure high-performance storage
3. Set up shared storage
4. Optimize storage performance
5. Implement data migration

## Real-world Examples

### 1. Database with Backup
```yaml
version: '3.8'

services:
  db:
    image: postgres:13
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./init:/docker-entrypoint-initdb.d
    environment:
      POSTGRES_PASSWORD: secret
    deploy:
      placement:
        constraints:
          - node.role == manager

  backup:
    image: alpine
    volumes:
      - db-data:/source:ro
      - backup:/backup
    command: >
      sh -c "while true; do
        tar -czf /backup/backup-$$(date +%Y%m%d).tar.gz -C /source .
        sleep 86400
      done"
    deploy:
      mode: global

volumes:
  db-data:
    driver: local
    driver_opts:
      type: none
      device: /mnt/db-data
      o: bind,noatime
  backup:
    driver: local
    driver_opts:
      type: none
      device: /mnt/backup
      o: bind
```

### 2. High-Performance Storage
```yaml
version: '3.8'

services:
  cache:
    image: redis:6
    volumes:
      - cache-data:/data
    deploy:
      resources:
        limits:
          memory: 1G
      placement:
        constraints:
          - node.labels.storage == ssd

  app:
    image: my-app
    volumes:
      - app-data:/app/data
    depends_on:
      - cache

volumes:
  cache-data:
    driver: local
    driver_opts:
      type: none
      device: /mnt/ssd/cache
      o: bind,noatime,nodiratime
  app-data:
    driver: local
    driver_opts:
      type: none
      device: /mnt/hdd/app
      o: bind
```

## Next Steps
- Once you're comfortable with Docker storage, move on to Lab 15: Docker Monitoring
- Practice implementing these storage patterns in your projects

## Additional Resources
- [Docker Storage Documentation](https://docs.docker.com/storage/)
- [Volume Drivers](https://docs.docker.com/engine/extend/plugins_volume/)
- [Storage Best Practices](https://docs.docker.com/storage/storagedriver/)
- [Backup and Restore](https://docs.docker.com/storage/volumes/#backup-restore-or-migrate-data-volumes) 