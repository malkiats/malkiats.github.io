---
title: Docker Volumes & Storage
date: 2025-09-06 14:00:00 -0500
categories: [Docker, Storage]
tags: [docker, volumes, bind-mounts, persistence, containers]
---

# Docker Volumes & Storage

Managing persistent data in Docker is crucial when running containers. By default, data stored inside a container is **ephemeral** — it disappears when the container is removed. Docker provides multiple storage mechanisms to persist and share data between containers and the host system.

---

## 1. Anonymous Volumes

Anonymous volumes are created when a container uses a `-v /path/in/container` syntax without naming the volume.  

- Data persists across container restarts.  
- Data is deleted if the container is removed.  
- Not shareable with other containers.  
- Stored under: `/var/lib/docker/volumes/`  

**Example:**

```yaml
# docker-compose.yml
version: "3.8"
services:
  web:
    image: nginx:latest
    volumes:
      - /usr/share/nginx/html
```

---

## 2. Named Volumes

Named volumes are explicitly created and can persist data even after containers are removed.  
They are accessible by multiple containers.  

**Example:**

```bash
# Create a volume
docker volume create mydata

# Run container with volume
docker run -d -v mydata:/var/www/html nginx:latest
```

- Data persists under `/var/lib/docker/volumes/mydata/_data`  
- Can be attached to multiple containers.  

---

## 3. Bind Mounts

Bind mounts map a **host directory** into a container.  

- Provides full control over the exact directory path on the host.  
- Useful for development, where local changes should immediately reflect inside the container.  

**Example:**

```bash
docker run -d   -v /media/pi/mydrive:/var/www/html   nginx:latest
```

---

## Docker Volume Commands

### Create a volume
```bash
docker volume create test-vol
```

### Inspect a volume
```bash
docker volume inspect test-vol
```

### List all volumes
```bash
docker volume ls
```

### Remove unused volumes
```bash
docker volume prune
```

### Remove specific volumes
```bash
docker volume rm test-vol
```

---

## Practical Examples

### Create and attach a volume
```bash
docker volume create vol-busybox
docker run -d --name test_nginx --volume vol-busybox:/tmp nginx:latest
```

### Check volume details
```bash
docker volume inspect vol-busybox
```

### Attach volume to container
```bash
docker run -itd --name ubuntu1 --volume vol-ubuntu:/var/log ubuntu:latest
```

### Inspect container mounts
```bash
docker container inspect --format "{{json .Mounts}}" ubuntu1 | python -m json.tool
```

### Find volume path on host
```bash
cd /var/lib/docker/volumes/
ls
```

---

## Key Takeaways

- **Anonymous volumes**: Temporary persistence, tied to a container.  
- **Named volumes**: Persistent and shareable across containers.  
- **Bind mounts**: Direct mapping from host path → container path.  

👉 Use **named volumes** for production data, and **bind mounts** for development.
