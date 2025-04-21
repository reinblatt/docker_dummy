# Lab 24: Docker Storage Advanced

## Objective
In this lab, you will learn advanced Docker storage concepts and techniques. You'll understand how to implement sophisticated volume management, storage drivers, backup strategies, and performance optimization for containerized applications.

## Prerequisites
- Completed Lab 23 (Docker Networking Advanced)
- Docker Desktop running on your system
- Basic understanding of storage concepts
- Familiarity with Docker Compose
- Access to Docker Swarm (for distributed storage)

## Key Concepts
- Volume Management: Advanced volume operations
- Storage Drivers: Understanding and configuring drivers
- Backup and Recovery: Data protection strategies
- Performance Optimization: Storage performance tuning
- Distributed Storage: Multi-host storage solutions

## Exercises

### 1. Advanced Volume Management
1. Create and manage named volumes:
```powershell
# Create a named volume with specific options
docker volume create \
    --driver local \
    --opt type=none \
    --opt device=/path/to/storage \
    --opt o=bind \
    my-volume

# Create a volume with labels
docker volume create \
    --driver local \
    --label com.example.environment=production \
    --label com.example.team=devops \
    labeled-volume

# List volumes with filters
docker volume ls --filter label=com.example.environment=production
```

2. Configure volume options:
```powershell
# Create a volume with size limit
docker volume create \
    --driver local \
    --opt type=tmpfs \
    --opt device=tmpfs \
    --opt o=size=100m \
    limited-volume

# Create a volume with specific mount options
docker volume create \
    --driver local \
    --opt type=none \
    --opt device=/path/to/storage \
    --opt o=bind,ro \
    read-only-volume
```

### 2. Storage Drivers
1. Configure storage driver options:
```powershell
# Check current storage driver
docker info --format '{{.Driver}}'

# Configure overlay2 driver options
docker run --rm \
    --storage-opt overlay2.override_kernel_check=true \
    --storage-opt overlay2.size=100G \
    nginx:latest

# Configure btrfs driver options
docker run --rm \
    --storage-opt btrfs.min_space=10G \
    --storage-opt btrfs.mountopt=compress \
    nginx:latest
```

2. Implement storage driver features:
```powershell
# Enable storage driver features
docker run --rm \
    --storage-opt overlay2.override_kernel_check=true \
    --storage-opt overlay2.mountopt=nodev \
    --storage-opt overlay2.mountopt=noexec \
    nginx:latest

# Configure storage driver performance
docker run --rm \
    --storage-opt overlay2.override_kernel_check=true \
    --storage-opt overlay2.mountopt=discard \
    --storage-opt overlay2.mountopt=async \
    nginx:latest
```

### 3. Backup and Recovery
1. Implement backup strategies:
```powershell
# Backup a volume
docker run --rm \
    -v my-volume:/source \
    -v /backup:/backup \
    alpine:latest \
    tar -czf /backup/my-volume-backup.tar.gz -C /source .

# Restore from backup
docker run --rm \
    -v my-volume:/target \
    -v /backup:/backup \
    alpine:latest \
    tar -xzf /backup/my-volume-backup.tar.gz -C /target
```

2. Configure automated backups:
```powershell
# Create backup container
docker run -d \
    --name backup-agent \
    -v my-volume:/source \
    -v /backup:/backup \
    -e BACKUP_SCHEDULE="0 0 * * *" \
    alpine:latest \
    sh -c "while true; do \
        tar -czf /backup/backup-$(date +%Y%m%d).tar.gz -C /source .; \
        sleep 86400; \
    done"
```

### 4. Performance Optimization
1. Optimize storage performance:
```powershell
# Configure volume with performance options
docker volume create \
    --driver local \
    --opt type=none \
    --opt device=/path/to/fast/storage \
    --opt o=async \
    --opt o=noatime \
    performance-volume

# Mount volume with performance options
docker run --rm \
    -v performance-volume:/data \
    --mount type=volume,source=performance-volume,target=/data,volume-opt=type=none,volume-opt=o=async \
    nginx:latest
```

2. Monitor storage performance:
```powershell
# Monitor volume I/O
docker run --rm \
    --pid=host \
    --privileged \
    nicolaka/netshoot \
    iostat -x 1

# Monitor disk usage
docker run --rm \
    -v /:/host \
    --privileged \
    nicolaka/netshoot \
    df -h /host
```

## Best Practices

### 1. Volume Management
- Use named volumes
- Implement volume labels
- Configure appropriate mount options
- Regular volume maintenance
- Monitor volume usage

### 2. Storage Drivers
- Choose appropriate driver
- Configure driver options
- Monitor driver performance
- Regular driver updates
- Backup driver data

### 3. Backup Strategy
- Regular automated backups
- Multiple backup locations
- Test recovery procedures
- Monitor backup success
- Document backup process

### 4. Performance
- Optimize mount options
- Monitor storage metrics
- Implement caching
- Regular performance testing
- Capacity planning

## Common Patterns

### 1. Volume Configuration
```yaml
version: '3.8'

volumes:
  app-data:
    driver: local
    driver_opts:
      type: none
      device: /path/to/storage
      o: bind,async
    labels:
      com.example.environment: production
      com.example.team: devops

services:
  web:
    image: nginx:latest
    volumes:
      - app-data:/var/www/html
```

### 2. Backup Configuration
```yaml
version: '3.8'

services:
  backup:
    image: alpine:latest
    volumes:
      - app-data:/source
      - backup-data:/backup
    command: >
      sh -c "
        while true; do
          tar -czf /backup/backup-$(date +%Y%m%d).tar.gz -C /source .;
          sleep 86400;
        done
      "
    deploy:
      mode: global

volumes:
  app-data:
    driver: local
  backup-data:
    driver: local
```

### 3. Performance Configuration
```yaml
version: '3.8'

volumes:
  fast-storage:
    driver: local
    driver_opts:
      type: none
      device: /path/to/fast/storage
      o: async,noatime

services:
  web:
    image: nginx:latest
    volumes:
      - fast-storage:/var/www/html
    deploy:
      resources:
        limits:
          memory: 1G
        reservations:
          memory: 512M
```

### 4. Monitoring Setup
```yaml
version: '3.8'

services:
  storage-monitor:
    image: nicolaka/netshoot
    volumes:
      - /:/host
    command: >
      sh -c "
        while true; do
          df -h /host;
          iostat -x 1;
          sleep 60;
        done
      "
    deploy:
      mode: global
```

## Challenges
1. Set up a distributed storage system
2. Implement automated backup strategy
3. Configure storage performance optimization
4. Set up storage monitoring
5. Implement disaster recovery plan

## Real-world Examples

### 1. Complete Storage Stack
```yaml
version: '3.8'

volumes:
  app-data:
    driver: local
    driver_opts:
      type: none
      device: /path/to/storage
      o: bind,async
    labels:
      com.example.environment: production
      com.example.team: devops

  backup-data:
    driver: local
    driver_opts:
      type: none
      device: /path/to/backup
      o: bind,async
    labels:
      com.example.environment: production
      com.example.team: backup

services:
  web:
    image: nginx:latest
    volumes:
      - app-data:/var/www/html
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure

  backup:
    image: alpine:latest
    volumes:
      - app-data:/source
      - backup-data:/backup
    command: >
      sh -c "
        while true; do
          tar -czf /backup/backup-$(date +%Y%m%d).tar.gz -C /source .;
          sleep 86400;
        done
      "
    deploy:
      mode: global

  monitor:
    image: nicolaka/netshoot
    volumes:
      - /:/host
    command: >
      sh -c "
        while true; do
          df -h /host;
          iostat -x 1;
          sleep 60;
        done
      "
    deploy:
      mode: global
```

### 2. Disaster Recovery Stack
```yaml
version: '3.8'

services:
  backup-agent:
    image: alpine:latest
    volumes:
      - app-data:/source
      - backup-data:/backup
    command: >
      sh -c "
        while true; do
          tar -czf /backup/backup-$(date +%Y%m%d).tar.gz -C /source .;
          sleep 86400;
        done
      "
    deploy:
      mode: global

  restore-agent:
    image: alpine:latest
    volumes:
      - app-data:/target
      - backup-data:/backup
    command: >
      sh -c "
        while true; do
          if [ -f /backup/restore-request ]; then
            latest_backup=$(ls -t /backup/backup-*.tar.gz | head -1);
            tar -xzf $latest_backup -C /target;
            rm /backup/restore-request;
          fi;
          sleep 60;
        done
      "
    deploy:
      mode: global

  health-check:
    image: nicolaka/netshoot
    volumes:
      - app-data:/data
    command: >
      sh -c "
        while true; do
          if [ ! -f /data/health ]; then
            touch /backup/restore-request;
          fi;
          sleep 30;
        done
      "
    deploy:
      mode: global

volumes:
  app-data:
    driver: local
  backup-data:
    driver: local
```

## Next Steps
- Move on to Lab 25: Docker Security Advanced
- Practice implementing advanced storage in your projects
- Explore additional storage tools and techniques

## Additional Resources
- [Docker Storage Documentation](https://docs.docker.com/storage/)
- [Volume Management Guide](https://docs.docker.com/storage/volumes/)
- [Storage Driver Guide](https://docs.docker.com/storage/storagedriver/)
- [Backup and Recovery Guide](https://docs.docker.com/storage/backups/) 