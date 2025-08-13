# Docker Volumes in docker-compose.yml Guide

## Volume Types Overview

Docker Compose supports several volume types for data persistence and sharing between containers.

## 1. Named Volumes

Named volumes are managed by Docker and stored in Docker's volume directory.

```yaml
volumes:
  my_data:
    driver: local
  
services:
  app:
    volumes:
      - my_data:/app/data
```

**Advantages:**
- Docker manages storage location
- Easy backup/restore with Docker commands
- Portable between environments
- Automatic cleanup when volume removed

**Use Cases:**
- Database storage
- Application state that needs to persist

## 2. Bind Mounts

Bind mounts link a specific host path to a container path.

```yaml
services:
  app:
    volumes:
      - ./host/path:/container/path
      - /absolute/host/path:/container/path
```

**Advantages:**
- Direct access to files from host
- Easy development workflows
- Can edit files with host tools

**Disadvantages:**
- Host-dependent paths
- Permission issues across different systems

**Use Cases:**
- Development code mounting
- Configuration files
- Log file access

## 3. Anonymous Volumes

Volumes without names, managed by Docker but harder to reference.

```yaml
services:
  app:
    volumes:
      - /container/path  # Anonymous volume
```

**Use Cases:**
- Temporary storage
- Cache directories
- When you don't need to reference the volume elsewhere

## 4. Volume Drivers

Specify custom storage drivers for volumes.

```yaml
volumes:
  my_data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=192.168.1.100,rw
      device: ":/path/to/dir"
```

**Common Drivers:**
- `local`: Default, stores on Docker host
- `nfs`: Network File System
- `cifs`: Windows shares
- Cloud storage drivers (AWS EFS, Azure Files)

## 5. External Volumes

Reference volumes created outside of docker-compose.

```yaml
volumes:
  external_volume:
    external: true
    name: actual_volume_name
```

**Use Cases:**
- Volumes shared between multiple compose projects
- Pre-existing volumes
- Production environments with pre-configured storage

## Volume Configuration Examples

### Basic Named Volume
```yaml
version: '3.8'
services:
  database:
    image: postgres:15
    volumes:
      - db_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=myapp

volumes:
  db_data:
```

### Bind Mount with Relative Path
```yaml
version: '3.8'
services:
  web:
    image: nginx
    volumes:
      - ./html:/usr/share/nginx/html:ro  # :ro = read-only
      - ./config/nginx.conf:/etc/nginx/nginx.conf:ro
```

### Complex Volume Configuration
```yaml
version: '3.8'
services:
  app:
    image: myapp
    volumes:
      - app_data:/app/data
      - logs:/app/logs
      - ./config:/app/config:ro
      - shared_storage:/shared

volumes:
  app_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: ./volumes/app_data
  
  logs:
    driver: local
  
  shared_storage:
    external: true
    name: company_shared_volume
```

### Volume with Initialization
```yaml
version: '3.8'
services:
  database:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d:ro
    environment:
      - POSTGRES_DB=myapp

volumes:
  postgres_data:
    driver: local
```

## Volume Mount Options

### Read-Only Mounts
```yaml
volumes:
  - ./config:/app/config:ro
```

### Bind Propagation
```yaml
volumes:
  - ./data:/app/data:shared  # Bind propagation
```

### SELinux Labels
```yaml
volumes:
  - ./data:/app/data:Z   # Relabel for container access
  - ./data:/app/data:z   # Shared label
```

## Best Practices

### Development Environment
```yaml
services:
  app:
    volumes:
      # Source code for live editing
      - .:/app
      # Node modules as named volume (better performance)
      - node_modules:/app/node_modules
      # Config files
      - ./config:/app/config:ro

volumes:
  node_modules:
```

### Production Environment
```yaml
services:
  app:
    volumes:
      # Only essential data volumes
      - app_data:/app/data
      - logs:/var/log/app

  database:
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  app_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /opt/docker/volumes/app_data
  
  db_data:
    driver: local
    driver_opts:
      type: none  
      o: bind
      device: /opt/docker/volumes/db_data
  
  logs:
    driver: local
```

## Volume Sharing Between Services

```yaml
services:
  producer:
    image: producer-app
    volumes:
      - shared_data:/data

  consumer:
    image: consumer-app
    volumes:
      - shared_data:/input:ro  # Read-only access

volumes:
  shared_data:
```

## Backup and Restore

### Backup Named Volume
```bash
# Create backup
docker run --rm -v my_volume:/data -v $(pwd):/backup alpine tar czf /backup/backup.tar.gz -C /data .

# Restore backup
docker run --rm -v my_volume:/data -v $(pwd):/backup alpine tar xzf /backup/backup.tar.gz -C /data
```

### Backup Bind Mount
```bash
# Simply copy the host directory
cp -r ./volumes/my_data ./backups/my_data_$(date +%Y%m%d)
```

## Common Patterns

### Shared Configuration
```yaml
x-common-volumes: &common-volumes
  - ./config:/app/config:ro
  - logs:/var/log

services:
  app1:
    volumes: *common-volumes
  
  app2:
    volumes: *common-volumes
```

### Environment-Specific Volumes
```yaml
services:
  app:
    volumes:
      - ${DATA_PATH:-./data}:/app/data
      - ${CONFIG_PATH:-./config}:/app/config:ro
```

## Troubleshooting

### Permission Issues
```yaml
# Set user/group for container
services:
  app:
    user: "1000:1000"  # host user:group
    volumes:
      - ./data:/app/data
```

### Performance on macOS/Windows
```yaml
# Use cached or delegated for better performance
volumes:
  - ./src:/app/src:cached     # Host -> Container optimized
  - ./dist:/app/dist:delegated # Container -> Host optimized
```

## Volume Commands

```bash
# List volumes
docker volume ls

# Inspect volume
docker volume inspect volume_name

# Remove unused volumes
docker volume prune

# Remove specific volume
docker volume rm volume_name

# Create volume manually
docker volume create my_volume
```