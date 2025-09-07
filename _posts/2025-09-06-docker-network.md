---
title: Docker Networking Deep Dive
date: 2025-09-06 15:00:00 -0500
categories: [Docker, Networking]
tags: [docker, networking, bridge, overlay, macvlan, containers]
---

# Docker Networking Deep Dive

Docker provides multiple networking drivers to allow containers to communicate with each other, the host, and the outside world. By default, Docker creates some networks automatically, and users can define custom networks as needed.

---

## Docker Default Networking (`docker0`)

- A default **bridge network** named `docker0` is created when Docker is installed.  
- Containers are automatically attached to this network unless a custom network is specified.  
- Other default networks:  
  - **host** → No isolation between host and containers. Containers share the host network stack.  
  - **none** → Containers run on their own isolated network stack with no external connectivity.  

---

## Docker Network Types

### 1. Bridge Network
- Most common and default network type.  
- Limited to containers within the same host.  
- Easy to manage and troubleshoot.  
- Requires **port mapping** for external access.  

**Example:**
```bash
# Create a custom bridge
docker network create -d bridge my-bridge-net
```

**Port mapping example:**
- Container running web service on port `80`.  
- Map host port `8000` → container port `80`:  
```bash
docker run -d -p 8000:80 --network my-bridge-net nginx
```

---

### 2. Overlay Network
- Enables multi-host networking.  
- Uses **VXLAN** technology to extend Layer 2 across Layer 3 networks.  
- Containers on different hosts can communicate securely.  
- Commonly used in **Docker Swarm** or multi-node setups.  

**Example:**
```bash
docker network create -d overlay --subnet=192.168.10.0/24 my-overlay-net
```

---

### 3. Macvlan Network
- Connects containers directly to the host network interface.  
- Assigns containers a **real IP** from the host’s LAN.  
- No NAT or port mapping required.  
- Low latency, but requires extra configuration.  

**Example:**
```bash
docker network create -d macvlan   --subnet=192.168.40.0/24   --gateway=192.168.40.1   -o parent=eth0 my-macvlan-net
```

---

## Common Network Operations

### List networks
```bash
docker network ls
```

### Filter networks
```bash
docker network ls --filter driver=bridge
```

### Inspect network
```bash
docker network inspect bridge
docker network inspect my-bridge-1
```

### Create custom bridge
```bash
docker network create --driver bridge my-bridge
```

### Create with subnet
```bash
docker network create --driver bridge   --subnet=192.168.0.0/16   --ip-range=192.168.5.0/24   my-bridge-1
```

### Connect container to network
```bash
docker network connect my-bridge-1 cont_run-env
```

### Disconnect container from network
```bash
docker network disconnect my-bridge-1 cont_run-env
```

### Run container with host network
```bash
docker container run -itd --network host --name test_nginx nginx
```

### Check container port mappings
```bash
docker container port test_nginx
```

---

## Key Takeaways

- **Bridge** → Default, single-host, needs port mapping.  
- **Overlay** → Multi-host, VXLAN-based, used in Swarm.  
- **Macvlan** → Direct L2 connectivity, containers get real IPs.  

👉 Choose the network driver based on **use case**:  
- Development → **Bridge**  
- Production multi-host → **Overlay**  
- Bare-metal style IP assignment → **Macvlan**  
