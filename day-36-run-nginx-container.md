# Day 36: Deploy Nginx Container on Application Server

The Nautilus DevOps team is conducting application deployment tests on selected application servers. They require a nginx container deployment on `Application Server 3`. Complete the task with the following instructions:


On `Application Server 3` create a container named `nginx_3` using the `nginx` image with the `alpine` tag. Ensure container is in a running state.

**👉 Task Requirements:**
- Create a container named `nginx_3` on Application Server 2.
- Use the `nginx:alpine` image.
- Ensure the container is in a running state.

**💡 Note:** The task is performed on **Application Server 2** (assumed to be `stapp03` based on common Nautilus infrastructure naming). All commands are executed with appropriate user permissions (e.g., `banner` with `sudo` if required). The current date is September 7, 2026.

---

## 🔹 Step 1: Connect to Application Server 1

```bash
ssh banner@stapp03
```

**Purpose**: Connect to Application Server 1 (`stapp03`) as the `banner` user to perform the container deployment.

**Note**: If root access is required for Docker operations, use `sudo` or switch to root:
```bash
sudo su -
```

---

## 🔹 Step 2: Verify Docker Installation

```bash
docker --version
```

**Purpose**: Confirm that Docker is installed and accessible on Application Server 2.

**Expected Output** (example):
```
Docker version 26.1.3, build b72abbb
```

**Note**: If Docker is not installed, contact the system administrator or install it (not covered here as it’s assumed to be pre-installed).

---

## 🔹 Step 3: Pull Nginx Alpine Image (Optional)

```bash
docker pull nginx:alpine
```

**Purpose**: Ensure the `nginx:alpine` image is available locally. This step is optional as `docker run` automatically pulls the image if it’s not present.

**Expected Output** (example):
```
alpine: Pulling from library/nginx
55afa1ecc21d: Pull complete 
850bf2dcecff: Pull complete 
af7dd138f459: Pull complete 
58c524ea09ce: Pull complete 
bc98d7675616: Pull complete 
51900e10fb9c: Pull complete 
8f924cf5086c: Pull complete 
6636b9fc203c: Pull complete 
Digest: sha256:72ba65eb42c10344912a84ff42408db7d34f2feb642204570ab8fc5ffd29f1d3
Status: Downloaded newer image for nginx:alpine
docker.io/library/nginx:alpine
```
---

## 🔹 Step 4: Create and Run Nginx Container

```bash
docker run -d --name nginx_3 nginx:alpine
```

**Purpose**: Create and start a container named `nginx_3` using the `nginx:alpine` image in detached mode (`-d`) to ensure it runs in the background.

**Command Breakdown**:
- `docker run`: Creates and starts a new container.
- `-d`: Runs the container in detached mode (background).
- `--name nginx_3`: Assigns the name `nginx_3` to the container.
- `nginx:alpine`: Specifies the image and tag to use.

**Expected Output** (example):
```
9c3910c0b80fdc86f18fdbaec138ea0834e710d1b9944b585ea55c69476f520d
```

**Note**: The output is the container ID, indicating successful creation and start.

---

## 🔹 Step 5: Verify Container Status

```bash
docker ps
```

**Purpose**: Confirm that the `nginx_3` container is running.

**Expected Output** (example):
```
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
9c3910c0b80f   nginx:alpine   "/docker-entrypoint.…"   50 seconds ago   Up 49 seconds   80/tcp    nginx_3
```

**Success Indicators**:
- **Container Name**: `nginx_3` is listed.
- **Image**: `nginx:alpine` is used.
- **Status**: `Up` indicates the container is running.
- **Ports**: `80/tcp` shows Nginx’s default port is available.

**Additional Check** (optional):
```bash
docker ps -a
```

**Purpose**: List all containers (running and stopped) to ensure no naming conflicts or unexpected stopped containers.

**Expected Output** (example):
```
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS              PORTS     NAMES
9c3910c0b80f   nginx:alpine   "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp    nginx_3
```
---

## 🔹 Step 6: Inspect Container (Optional)

```bash
docker inspect nginx_3
```

**Purpose**: Retrieve detailed information about the `nginx_3` container to verify its configuration, such as image, name, and running state.

**Expected Output** (partial example):
```json
[
    {
        "Id": "9c3910c0b80fdc86f18fdbaec138ea0834e710d1b9944b585ea55c69476f520d",
        "Created": "2026-09-06T23:55:01.294476589Z",
        "Path": "/docker-entrypoint.sh",
        "Args": [
            "nginx",
            "-g",
            "daemon off;"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 35915,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2026-09-06T23:55:01.685365712Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
```

**Verification**: Confirm the container is running and using the correct image.

---

## 📋 Quick Command Reference

For quick execution on **Application Server 1 (stapp01)**:

```bash
# Connect to server
ssh tony@stapp01
sudo su -

# Verify Docker
docker --version

# Pull image (optional)
docker pull nginx:alpine

# Run container
docker run -d --name nginx_1 nginx:alpine

# Verify container status
docker ps
```

---




---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Docker Not Installed**
**Symptoms**: `docker: command not found`.
**Solution**: Verify Docker installation or contact the system administrator.
```bash
docker --version
```

### **Issue 2: Container Name Already in Use**
**Symptoms**: Error like `docker: Error response from daemon: Conflict. The container name "/nginx_1" is already in use`.
**Solution**: Remove the existing container and retry.
```bash
docker rm -f nginx_1
docker run -d --name nginx_1 nginx:alpine
```

### **Issue 3: Container Not Running**
**Symptoms**: `docker ps` shows no running container, but `docker ps -a` shows `nginx_1` as `Exited`.
**Solution**: Check container logs for errors and restart.
```bash
docker logs nginx_1
docker start nginx_1
```

### **Issue 4: Image Pull Failure**
**Symptoms**: Error like `docker: Error response from daemon: pull access denied for nginx:alpine`.
**Solution**: Ensure internet connectivity and correct image name.
```bash
docker pull nginx:alpine
```

### **Issue 5: Permission Denied**
**Symptoms**: `docker: Got permission denied while trying to connect to the Docker daemon socket`.
**Solution**: Use `sudo` or ensure the user is in the `docker` group.
```bash
sudo docker run -d --name nginx_1 nginx:alpine
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **deploying a named Nginx container** using the `nginx:alpine` image on Application Server 1 and ensuring it remains in a running state with minimal configuration.

**💡 Solution Approach:**

1. **Server Access**: Connected to Application Server 1 (`stapp01`) as the `tony` user.
2. **Docker Verification**: Confirmed Docker is installed and accessible.
3. **Container Deployment**: Used `docker run -d --name nginx_1 nginx:alpine` to create and start the container.
4. **Status Validation**: Verified the container is running using `docker ps`.
5. **Minimal Configuration**: Ensured the task requirements were met without additional configurations (e.g., port mapping), as not specified.

**🎯 Key Success Factors:**
- **Correct Image**: Used `nginx:alpine` for a lightweight deployment.
- **Named Container**: Assigned the name `nginx_1` as required.
- **Running State**: Ensured the container runs in detached mode (`-d`).
- **Simple Execution**: Avoided unnecessary configurations to meet the task’s scope.
- **Verification**: Confirmed the container’s status with `docker ps`.

**⚠️ Critical Operation Details:**
- **Container Name**: Must be exactly `nginx_1`.
- **Image Tag**: Must use `nginx:alpine` for lightweight deployment.
- **Running State**: Container must be `Up` in `docker ps` output.
- **User Permissions**: Used `tony` with `sudo` if needed, assuming standard Nautilus setup.

**🔒 Nginx Container Benefits:**
- **Lightweight Deployment**: `nginx:alpine` minimizes resource usage.
- **Quick Setup**: Single command deploys a functional web server.
- **Isolation**: Containerized environment prevents conflicts.
- **Reproducibility**: Consistent deployment across servers.

---

## ⚠️ Important Production Notes

🔧 **Minimal Configuration**: The task requires only a running container, making it suitable for quick tests.

🔐 **Security**: Using `nginx:alpine` reduces the attack surface due to its small size.

📊 **Team Collaboration**: The container can be used for further testing or integration by the DevOps team.

🛡️ **Validation**: `docker ps` ensures the container is operational, meeting deployment requirements.