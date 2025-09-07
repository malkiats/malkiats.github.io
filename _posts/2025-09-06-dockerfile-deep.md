---
title: Dockerfile Deep Dive - Layers, CMD vs ENTRYPOINT, and Best Practices
date: 2025-09-06 15:00:00 -0500
categories: [Docker, Dockerfile]
tags: [docker, container, dockerfile, CMD, ENTRYPOINT]
---

## Dockerfile Best Practices and Key Concepts

### Command Chaining
- `\` → Start another line in Dockerfile commands.
- `&&` → Append another command in the same layer (better for reducing image size).

### Layer Creation
- **RUN, COPY, ADD** → Each creates a **new image layer**.  
  ⚡ Fewer layers = smaller, faster images.

### Common Commands
- **WORKDIR** → Sets the default working directory inside the container.  
- **ADD** → Copies files from local or **remote URLs** (can also extract compressed files).  
- **COPY** → Copies files **only from the local context**.  
- **USER** → Defines the user ID to run commands inside the container.  

---

## ENTRYPOINT vs CMD

### ENTRYPOINT
Specifies the **binary or executable** the container will always run.  
Example:
```dockerfile
ENTRYPOINT ["nginx"]
```

- `docker run <image>` → runs `nginx`.  
- `docker run <image> -g 'daemon off;'` → runs `nginx -g 'daemon off;'`.

---

### CMD
Provides **default arguments** for the `ENTRYPOINT`.  
Example:
```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

- Equivalent to `/bin/sh -c 'nginx -g "daemon off;"'`.  
- Ensures Nginx runs **in the foreground** (important so Docker doesn’t stop the container).

---

### ENTRYPOINT + CMD Together
```dockerfile
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```
- Default execution: `nginx -g daemon off;`  
- If overridden: `docker run <image> -g 'daemon on;'` → runs `nginx -g 'daemon on;'`.

---

### Default Behavior
- If **only CMD is specified** → container runs `/bin/sh -c <CMD>`.  
- If **only ENTRYPOINT is specified** → container runs `<ENTRYPOINT>`.  
- If **neither is specified** → container defaults to `/bin/sh -c`.

---

## Practical Example

```dockerfile
FROM nginx:alpine

WORKDIR /usr/share/nginx/html
COPY index.html .

ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```

✅ This ensures:
- The container **always runs Nginx** (`ENTRYPOINT`).  
- It **stays in the foreground** with `-g daemon off;` (`CMD`).  
