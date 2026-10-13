# Day 43: Docker Ports Mapping

The Nautilus DevOps team is planning to host an application on a nginx-based container. There are number of tickets already been created for similar tasks. One of the tickets has been assigned to set up a nginx container on `Application Server 1` in `Stratos Datacenter`. Please perform the task as per details mentioned below:

a. Pull `nginx:alpine-perl` docker image on `Application Server 2`.

b. Create a container named `beta` using the image you pulled.

c. Map host port `8088` to container port `80`. Please keep the container in running state.
---

## 🔹 Step 1: Connect to App Server 3

```bash
ssh steve@stapp02
sudo su -
```

**Purpose**: Connect to Application Server 3 as banner user and switch to root for Docker operations.

---

## 🔹 Step 2: Verify Docker service is running

```bash
systemctl status docker
```

**Purpose**: Ensure Docker daemon is active before performing container operations.

**Expected Output**:
```
● docker.service - Docker Application Container Engine
   Active: active (running)
```

**If not running**:
```bash
systemctl start docker
```

---

## 🔹 Step 3: Check for existing containers

```bash
docker ps -a
```

**Purpose**: View all containers (running and stopped) to check if 'official' name is already in use.

**Expected Behavior**: Should not show any container named 'official' if this is first deployment.

---

## 🔹 Step 4: Pull the nginx:alpine image

```bash
docker pull nginx:alpine-perl
```

**Purpose**: Download the NGINX Alpine image from Docker Hub registry.

**Expected Output**:
```
alpine-perl: Pulling from library/nginx
55afa1ecc21d: Pull complete 
850bf2dcecff: Pull complete 
af7dd138f459: Pull complete 
58c524ea09ce: Pull complete 
bc98d7675616: Pull complete 
51900e10fb9c: Pull complete 
8f924cf5086c: Pull complete 
6636b9fc203c: Pull complete 
e152362fea62: Pull complete 
Digest: sha256:9c0cd54a2f03e1c7d8e9973ac60f112dd988d04049a6fee49fbd994ce9f39057
Status: Downloaded newer image for nginx:alpine-perl
docker.io/library/nginx:alpine-perl
```

**Image Details**:
- **Base**: Alpine Linux (minimal footprint)
- **Size**: ~109MB (much smaller than standard nginx)
- **Benefits**: Faster pulls, smaller disk usage

---

## 🔹 Step 5: Verify image download

```bash
docker images nginx
```

**Purpose**: Confirm the nginx:alpine image is available locally.

**Expected Output**:
```
REPOSITORY   TAG           IMAGE ID       CREATED      SIZE
nginx        alpine-perl   deae8d68086f   6 days ago   109MB
```

---

## 🔹 Step 6: Check if port 3000 is available

```bash
ss -tlnp | grep :8088
```

**Purpose**: Verify host port 6300 is not already in use by another service.

**Expected Result**: No output means port is available.

**If port is in use**:
```bash
# Find what's using the port
lsof -i :8088
# Stop the conflicting service if needed
```

---

## 🔹 Step 7: Run the NGINX container

```bash
docker run -d --name beta -p 8088:80 nginx:alpine-perl
```

**Purpose**: Start NGINX container with specified name and port mapping in detached mode.

**Command Options**:
- `-d`: Detached mode (run in background)
- `--name beta`: Container name
- `-p 8088:80`: Port mapping (host:container)
- `nginx:alpine-perl`: Image to use

**Expected Output**: Container ID (64-character hash)

---

## 🔹 Step 8: Verify container is running

```bash
docker ps
```

**Purpose**: Confirm the 'official' container is in running state.

**Expected Output**:
```
CONTAINER ID   IMAGE               COMMAND                  CREATED          STATUS          PORTS                                   NAMES
2ff7b2e50551   nginx:alpine-perl   "/docker-entrypoint.…"   38 seconds ago   Up 37 seconds   0.0.0.0:8088->80/tcp, :::8088->80/tcp   beta
```

**Verification Points**:
- ✅ Container name: **beta**
- ✅ Status: **Up**
- ✅ Port mapping: **0.0.0.0:8088->80/tcp**

---

## 🔹 Step 9: Check container logs

```bash
docker logs beta
```

**Purpose**: View NGINX startup logs to ensure service started correctly.

**Expected Output**:
```
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
...
/docker-entrypoint.sh: Configuration complete; ready for start up
```

---

## 🔹 Step 10: Test NGINX from host

```bash
curl http://localhost:8088
```

**Purpose**: Test NGINX web server is serving content on mapped port.

**Expected Output**: HTML content of NGINX default welcome page.

**Success Indicators**:
- ✅ HTTP 200 OK response
- ✅ HTML content displayed
- ✅ "Welcome to nginx!" message

---

## 🔹 Step 11: Test with HTTP headers

```bash
curl -I http://localhost:8088
```

**Purpose**: View HTTP response headers to confirm NGINX is serving requests.

**Expected Output**:
```
HTTP/1.1 200 OK
Server: nginx/1.25.x
Date: [timestamp]
Content-Type: text/html
...
```

---

## 🔹 Step 12: Inspect container details

```bash
docker inspect beta
```

**Purpose**: View detailed container configuration and state information.

**Key Information**:
- Network settings
- Port mappings
- Environment variables
- Mount points
- Container state

---

## 🔹 Step 13: Check container resource usage

```bash
docker stats beta --no-stream
```

**Purpose**: Display current resource usage (CPU, memory, network) of the container.

**Expected Output**:
```
CONTAINER ID   NAME       CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O     PIDS
7c18c2ac5545   official   0.00%     12.85MiB / 61.92GiB   0.02%     1.95kB / 1.56kB   0B / 39.9kB   17
```

---

## 🔹 Step 14: Test from external host (optional)

**From Jump Host or another server**:

```bash
curl http://stapp03:8088
```

**Purpose**: Verify NGINX is accessible from external hosts.

**Note**: Requires firewall rules to allow port 3000 if firewall is active.

---

## 🔹 Step 15: Verify container auto-restart policy

```bash
docker inspect beta --format='{{.HostConfig.RestartPolicy.Name}}'
```

**Purpose**: Check if container will restart automatically.

**Current State**: "no" (default, no auto-restart)

**To add restart policy** (optional):
```bash
docker update --restart=unless-stopped beta
```

---

## 📋 Quick Command Reference

For quick copy-paste, execute on **App Server 3**:

```bash
# Connect to server
ssh steve@stapp02
sudo su -

# Pull and verify image
docker pull nginx:alpine-perl
docker images nginx

# Run container
docker run -d --name official -p 6300:80 nginx:alpine-perl

# Verify and test
docker ps
docker logs beta
curl http://localhost:8088
```

---

## 💡 Additional Tips

- **Alpine vs standard**: Alpine images are ~10x smaller than standard images
- **Container persistence**: Use volumes for persistent data storage
- **Environment variables**: Pass config using `-e` flag
- **Custom nginx.conf**: Mount custom configuration with `-v`
- **HTTPS support**: Add port 443 mapping and SSL certificates

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Port 8088 already in use**
**Symptoms**: Error "port is already allocated"
**Solution**: Stop conflicting service or use different port
```bash
ss -tlnp | grep :8088
docker run -d --name official -p 8088:80 nginx:alpine-perl
```

### **Issue 2: Container name already exists**
**Symptoms**: Error "name is already in use"
**Solution**: Remove existing container or use different name
```bash
docker rm -f beta
docker run -d --name beta -p 8088:80 nginx:alpine-perl
```

### **Issue 3: Container exits immediately**
**Symptoms**: Container status shows "Exited"
**Solution**: Check logs for errors
```bash
docker logs beta
docker ps -a  # Show all containers including stopped
```

### **Issue 4: Cannot access from external hosts**
**Symptoms**: Curl works locally but not remotely
**Solution**: Check firewall rules and network configuration
```bash
firewall-cmd --list-ports
firewall-cmd --permanent --add-port=6300/tcp
firewall-cmd --reload
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **deploying an NGINX web server container with specific naming and port mapping requirements** while ensuring the container remains in running state for service availability.

**💡 Solution Approach:**

1. **Image Selection**: Used nginx:alpine for lightweight, efficient deployment
2. **Image Acquisition**: Pulled image from Docker Hub to ensure latest stable version
3. **Port Mapping**: Configured host port 8088 to map to container's default HTTP port 80
4. **Container Naming**: Used specific name "official" for easy identification and management
5. **Detached Mode**: Ran container in background to keep it running persistently
6. **Service Verification**: Tested NGINX functionality through HTTP requests to mapped port

**🎯 Key Success Factors:**
- **Alpine image usage** providing minimal footprint while maintaining full NGINX functionality
- **Port mapping syntax** using `-p 8088:80` to expose container port 80 on host port 3000
- **Detached mode** using `-d` flag to run container in background
- **Container naming** using `--name beta` for consistent identification
- **Service testing** validating NGINX serves content correctly on mapped port
- **State verification** confirming container remains in "Up" status

**⚠️ Critical Configuration Details:**
- **Port mapping format** uses `host_port:container_port` notation
- **Detached mode** essential for keeping container running in background
- **Container naming** must be unique across all containers on host
- **Image tag** nginx:alpine specifies both repository and tag
- **Default behavior** NGINX listens on port 80 inside container by default

**🔒 Container Deployment Benefits:**
- **Rapid deployment** container starts in seconds vs traditional installation
- **Consistency** same image runs identically across environments
- **Isolation** NGINX runs in isolated namespace with minimal dependencies
- **Resource efficiency** Alpine base provides minimal overhead

**⚠️ Important Container Concepts**:
- **Port publishing** `-p` makes container port accessible from host
- **Detached vs attached** detached runs in background, attached shows logs
- **Container lifecycle** container runs as long as PID 1 process runs
- **Image layers** nginx:alpine built on Alpine Linux base layers

---

## ⚠️ Important Production Notes

🔧 **Container Management**: NGINX container provides quick, isolated web server deployment

🔐 **Port Configuration**: Port mapping enables external access while maintaining container isolation

📊 **Resource Efficiency**: Alpine-based images provide full functionality with minimal footprint

🛡️ **Service Availability**: Detached mode ensures container continues running independently.