---
title: Dockerfile Guide - Commands, Examples, and Best Practices
date: 2025-09-06 06:01:42 -0500
categories: [Docker, Dockerfile]
tags: [docker, dockerfile, container, build, image]
---

# Dockerfile Guide

A **Dockerfile** is a text document that contains instructions to assemble a Docker image.  
By combining multiple commands into a single Dockerfile, you can define everything needed to build and run your application.

---

## 🧩 Common Dockerfile Commands

- **FROM** – Base parent image. Alpine version is the minimal Docker image (only 5MB).
- **RUN** – Executes Linux commands (install packages, create folders, users, assign permissions, etc.).
- **ENV** – Sets environment variables. Supports multiple variables.
- **COPY** – Copies files and directories from the host to the container.
- **EXPOSE** – Exposes ports to allow external access.
- **ENTRYPOINT** – Defines the command and arguments for running the container (not overridden easily).
- **CMD** – Defines default command/arguments (overridden if command is passed at runtime). Only one `CMD` is allowed.
- **VOLUME** – Creates a directory mount point for persistent data storage.
- **WORKDIR** – Sets the working directory for following instructions.
- **LABEL** – Provides metadata (e.g., maintainer info).
- **ADD** – Copies files/dirs like `COPY`, but can also fetch via URL and auto-extract compressed files.
- **ARG** – Defines build-time variables.

---

## 🛠️ Building an Image

```bash
# Build from Dockerfile in current directory
docker build .

# Build and tag with custom name
docker build -t my-image .

# Build with Dockerfile located in another path
docker build -f /path/to/Dockerfile .
```

---

## 🔍 COPY vs. ADD

- **COPY** – Copies files/directories from host to image.
- **ADD** – Same as COPY, plus supports remote URLs and auto-extracts `.tar` files.

👉 **Best practice**: Use `COPY` unless you need `ADD`’s special features.

---

## ⚡ ENTRYPOINT vs. CMD

- **CMD**
  - Sets default command executed **only if** no other command is provided when running container.
  - Overridden when custom command is passed.

  ```bash
  docker run -d -p 5003:5000 my-image ping google.com
  ```

- **ENTRYPOINT**
  - Defines a command that always runs as the container’s executable.
  - Command-line parameters are appended, not replaced.

👉 **Best practice**: Use `ENTRYPOINT` for mandatory commands, `CMD` for defaults.

---

## 💾 Volumes

Declaring `VOLUME` in a Dockerfile indicates where data will persist.

```dockerfile
VOLUME /var/lib/mysql
```

When running a container, mount a host path with `-v`:

```bash
docker run -d -v /my/host/path:/var/lib/mysql my-image
```

---

## 📝 Example: MySQL Dockerfile

```dockerfile
FROM debian:stretch-slim

# Add user and group
RUN groupadd -r mysql && useradd -r -g mysql mysql

# Install dependencies
RUN apt-get update && apt-get install -y --no-install-recommends gnupg dirmngr     && rm -rf /var/lib/apt/lists/*

# Create init directory
RUN mkdir /docker-entrypoint-initdb.d

# Environment variables
ENV MYSQL_MAJOR 9.0
ENV MYSQL_VERSION 8.0.15-1debian9

# Data volume
VOLUME /var/lib/mysql

# Config files
COPY config/ /etc/mysql/
COPY docker-entrypoint.sh /usr/local/bin/

# Backwards compatibility symlink
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh

# Entrypoint and ports
ENTRYPOINT ["docker-entrypoint.sh"]
EXPOSE 3306 33060

# Default command
CMD ["mysqld"]
```

---

✅ With this, you now have a structured guide to writing and understanding Dockerfiles.
