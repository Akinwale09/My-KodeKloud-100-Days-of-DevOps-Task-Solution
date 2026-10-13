# Day 39: Create a Docker Image from Container

One of the Nautilus developer was working to test new changes on a container. He wants to keep a backup of his changes to the container. A new request has been raised for the DevOps team to create a new image from this container. Below are more details about it:

a. Create an image `beta:nautilus` on `Application Server 3` from a container `ubuntu_latest` that is running on same server.

---

## 🔹 Step 1: Connect to Application Server 1

```bash
ssh banner@stapp03
```

**Purpose**: Connect to Application Server 1 as tony user for Docker operations.

**Note**: No sudo required if tony is in the docker group.

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

---

## 🔹 Step 3: List all running containers

```bash
docker ps
```

**Purpose**: Confirm the target container `ubuntu_latest` is running on the system.

**Expected Output Example**:
```
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
81e9ce3c64a0   ubuntu    "/bin/bash"   3 minutes ago   Up 3 minutes             ubuntu_latest
```

**Container Details**:
- **Name**: ubuntu_latest
- **Status**: Must be running (Up)
- **Image**: ubuntu
- **Container ID**: Unique identifier

---

## 🔹 Step 4: Inspect the running container

```bash
docker inspect ubuntu_latest
```

**Purpose**: View detailed configuration and state information of the running container.

**Key Information**:
- Current container state
- Environment variables
- Volume mounts
- Network configuration
- Image layers

---

## 🔹 Step 5: Check for changes in the container

```bash
docker diff ubuntu_latest
```

**Purpose**: Display filesystem changes made in the container compared to the original image.

**Output Indicators**:
- **A**: File or directory added
- **D**: File or directory deleted
- **C**: File or directory changed

**Example Output**:
```
C /var
A /var/log/myapp.log
C /etc
A /etc/myconfig.conf
```

---

## 🔹 Step 6: Commit the container to a new image

```bash
docker commit ubuntu_latest beta:nautilus
```

**Purpose**: Create a new Docker image from the running container's current state with specified tag.

**Expected Output**:
```
sha256:825e0ca6100abd6c22f118a7691bb825862ffae4fc2ef9b4cf34ad7b717496c9
```

**Commit Operation**:
- Captures current container state
- Creates new image layer with changes
- Assigns specified repository and tag
- Returns new image SHA256 hash

---

## 🔹 Step 7: Verify the new image was created

```bash
docker images
```

**Purpose**: Confirm the new image with tag `beta:datacenter` appears in the local image repository.

**Expected Output**:
```
REPOSITORY   TAG        IMAGE ID       CREATED          SIZE
beta         nautilus   825e0ca6100a   37 seconds ago   142MB
ubuntu       latest     af52039db3f8   3 weeks ago      100MB
```

**Image Verification**:
- ✅ Repository name: **beta**
- ✅ Tag: **nautilus**
- ✅ Recent creation timestamp
- ✅ Size includes original image + changes

---

## 🔹 Step 8: Inspect the newly created image

```bash
docker inspect beta:nautilus
```

**Purpose**: View detailed metadata of the newly committed image.

**Important Metadata**:
- Created timestamp
- Parent image
- Layer information
- Configuration settings
- Default command

---

## 🔹 Step 9: Compare image sizes

```bash
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

**Purpose**: Compare sizes to understand the overhead of committed changes.

**Size Analysis**:
- **Original ubuntu image**: ~100MB
- **New beta:datacenter image**: ~142MB
- **Difference**: Represents committed changes and layers

---

## 🔹 Step 10: Test the newly created image

```bash
docker run --rm beta:nautilus ls /
```

**Purpose**: Verify the new image works correctly and contains expected changes.

**Test Options**:
```bash
# Interactive test
docker run -it --rm beta:nautilus /bin/bash

# Check specific files
docker run --rm beta:nautilus cat /etc/myconfig.conf
```

---

## 🔹 Step 11: View image history

```bash
docker history beta:nautilus
```

**Purpose**: Display layer history showing original base image and committed changes.

**Expected Output**:
```
IMAGE          CREATED          CREATED BY                                      SIZE
00004d8ca0ec   2 minutes ago    /bin/bash                                      55MB
ce8f79aecc43   3 days ago       /bin/sh -c #(nop)  CMD ["/bin/bash"]           0B
...
```

---

## 📋 Quick Command Reference

For quick copy-paste, execute on **Application Server 1**:

```bash
# Connect to server
ssh tony@stapp01

# Verify container is running
docker ps

# Check container changes (optional)
docker diff ubuntu_latest

# Commit container to new image
docker commit ubuntu_latest beta:nautilus

# Verify new image
docker images

# Test new image (optional)
docker run --rm beta:nautilus echo "Image test successful"
```

---

## 💡 Additional Tips

- **Commit messages**: Use `-m` flag to add commit message: `docker commit -m "Added configs" ubuntu_latest beta:nautilus`
- **Author information**: Use `-a` flag to specify author: `docker commit -a "developer@nautilus.com" ubuntu_latest beta:nautilus`
- **Pause option**: Use `--pause=false` to avoid pausing container during commit
- **Best practice**: Use Dockerfile for reproducible builds instead of commits
- **Image cleanup**: Remove unused images with `docker image prune`

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Container not found**
**Symptoms**: Error "No such container: ubuntu_latest"
**Solution**: Verify container name and check running containers
```bash
docker ps -a  # Show all containers including stopped
docker ps --filter "name=ubuntu"
```

### **Issue 2: Permission denied**
**Symptoms**: Cannot execute docker commands as tony user
**Solution**: Add user to docker group or use sudo
```bash
sudo usermod -aG docker tony
# Log out and back in
# or use sudo
sudo docker commit ubuntu_latest beta:nautilus
```

### **Issue 3: Disk space errors**
**Symptoms**: "No space left on device" during commit
**Solution**: Clean up unused Docker resources
```bash
docker system df  # Check disk usage
docker system prune  # Clean up
docker image prune  # Remove unused images
```

### **Issue 4: Container is paused**
**Symptoms**: Commit hangs or fails
**Solution**: Ensure container is running, not paused
```bash
docker ps  # Check status
docker unpause ubuntu_latest  # If paused
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **capturing the current state of a running container** including all filesystem changes and modifications into a new reusable Docker image with specific naming requirements.

**💡 Solution Approach:**

1. **Container Verification**: Confirmed target container `ubuntu_latest` was running and accessible on the system
2. **State Analysis**: Examined container for filesystem changes using `docker diff` to understand modifications
3. **Commit Operation**: Used `docker commit` to create snapshot of container's current state as new image
4. **Tag Assignment**: Applied specific tag `beta:nautilus` following repository:tag naming convention
5. **Image Validation**: Verified new image creation through image listing and metadata inspection
6. **Functionality Testing**: Confirmed new image works correctly by running test containers

**🎯 Key Success Factors:**
- **Container identification** ensuring correct target container `ubuntu_latest` for commit operation
- **Running state requirement** container must be running (not stopped) for commit to capture current state
- **Tag specification** using proper repository:tag format (`beta:nautilus`) as required
- **Layer understanding** recognizing committed image includes base layers plus container changes
- **Size awareness** understanding new image size reflects original image plus modifications
- **Testing validation** confirming committed image functions correctly with test runs

**⚠️ Critical Operation Details:**
- **Container state preservation** commit captures current filesystem state including all changes
- **Non-destructive operation** original container continues running unchanged after commit
- **Layer creation** commit creates new layer containing all container modifications
- **Tag format** must follow repository:tag convention for proper image identification
- **Image ID generation** new unique SHA256 hash identifies the committed image

**🔒 Docker Commit Benefits:**
- **State preservation** captures exact container state at commit time
- **Backup capability** creates recoverable snapshots of container work
- **Image distribution** committed images can be shared like any Docker image
- **Quick prototyping** enables rapid experimentation and state capture

**⚠️ Important Considerations**:
- **Not recommended for production** - Dockerfiles provide better reproducibility
- **Layer accumulation** - repeated commits increase image size
- **Configuration capture** - container config becomes image default config
- **Volume exclusion** - mounted volumes are not included in commit

---

## ⚠️ Important Production Notes

🔧 **Container Snapshots**: Docker commit provides quick way to capture container state for backup or testing

🔐 **Development Workflow**: Useful for prototyping and experimentation but Dockerfile preferred for production

📊 **Image Size Impact**: Committed images include all changes and can become larger than necessary

🛡️ **Best Practice Alternative**: Use Dockerfile with version control for reproducible, maintainable image builds