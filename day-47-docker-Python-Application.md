# Day 47: Docker Python App

A python app needed to be Dockerized, and then it needs to be deployed on `App Server 3`. We have already copied a `requirements.txt` file (having the app dependencies) under `/python_app/src/` directory on `App Server 3`. Further complete this task as per details mentioned below:

1. Create a `Dockerfile` under `/python_app` directory:
   - Use any `python` image as the base image.
   - Install the dependencies using `requirements.txt` file.
   - Expose the port `8082`.
   - Run the `server.py` script using `CMD`.

2. Build an image named `nautilus/python-app` using this `Dockerfile`.

3. Once image is built, create a container named `pythonapp_nautilus`:
   - Map port `8082` of the container to the host port `8092`.

4. Once deployed, you can test the app using curl command on `App Server 3`.

```sh
curl http://localhost:8092/
```

## Specific Requirements:

1. Create a `Dockerfile` under `/python_app` directory using a Python base image, installing the dependencies from `requirements.txt`, exposing port `8082`, and running `server.py` with `CMD`.
2. Build an image named `nautilus/python-app` from this Dockerfile.
3. Create a container named `pythonapp_nautilus` and map its port `8082` to host port `8092`.
4. Test the application on App Server 1 with `curl http://localhost:8092/`.

## Solution

The supplied Flask application listens on all container interfaces at port `8082`. A single-stage Python image is sufficient because its only dependency is Flask; Docker already isolates the application's Python packages from the host. The image copies the dependency manifest first so that the dependency-installation layer can be reused when only application code changes.

---

## 🔹 Step 1: Connect to App Server 3

```bash
ssh banner@stapp03
sudo su -
```

**Purpose**: Connect to App Server 3 as the `banner` user and switch to `root` for Docker operations.

---

## 🔹 Step 2: Navigate to the Python App Directory

```bash
cd /python_app
```

**Purpose**: Navigate to the directory where the Dockerfile will be created.

**Verify Location**:
```bash
pwd
ls -la
ls -la src/
```

**Expected Structure**:
```
/python_app/
├── src/
│   ├── requirements.txt
│   └── server.py
```

---

## 🔹 Step 3: Create the Dockerfile

```bash
vi Dockerfile
```

**Purpose**: Create and edit the Dockerfile with the required configuration.

---

## 🔹 Step 4: Add Dockerfile Content

**Dockerfile**:

```dockerfile
# Use official Python base image
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Copy dependencies file
COPY src/requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application files
COPY src/ .

# Expose the application port
EXPOSE 8082

# Run the Python server
CMD ["python", "server.py"]
```

**Key Configurations**:
- **FROM**: Uses `python:3.9-slim` for a lightweight Python base image.
- **WORKDIR**: Sets `/app` as the working directory for file operations.
- **COPY**: Copies `requirements.txt` from `src/` to install dependencies.
- **RUN**: Installs dependencies with `pip`, using `--no-cache-dir` to reduce image size.
- **COPY**: Copies all `src/` files (including `server.py`) to the working directory.
- **EXPOSE**: Exposes port 8082 for the application.
- **CMD**: Runs `server.py` with the Python interpreter.

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

## 🔹 Step 5: Verify Dockerfile Syntax

```bash
cat Dockerfile
```

**Purpose**: Confirm the Dockerfile contains the correct configuration.

**Validation Checklist**:
- ✅ `FROM python:3.9-slim` as base image
- ✅ Correct `COPY` paths for `requirements.txt` and `src/`
- ✅ `EXPOSE 8082` for the application port
- ✅ `CMD ["python", "server.py"]` for running the app
- ✅ No syntax errors

---

## 🔹 Step 6: Build the Docker Image

```bash
docker build -t nautilus/python-app .
```

**Purpose**: Build the Docker image using the Dockerfile.

**Expected Build Process**:
```
[+] Building 9.1s (10/10) FINISHED                                                                              docker:default
 => [internal] load build definition from Dockerfile                                                                      0.1s
 => => transferring dockerfile: 394B                                                                                      0.0s
 => [internal] load metadata for docker.io/library/python:3.9-slim                                                        1.3s
 => [internal] load .dockerignore                                                                                         0.0s
 => => transferring context: 2B                                                                                           0.0s
 => [1/5] FROM docker.io/library/python:3.9-slim@sha256:2d97f6910b16bd338d3060f261f53f144965f755599aab1acda1e13cf1731b1b  3.0s
 => => resolve docker.io/library/python:3.9-slim@sha256:2d97f6910b16bd338d3060f261f53f144965f755599aab1acda1e13cf1731b1b  0.1s
 => => sha256:085da638e1b8a449514c3fda83ff50a3bffae4418b050cfacd87e5722071f497 5.40kB / 5.40kB                            0.0s
 => => sha256:38513bd7256313495cdd83b3b0915a633cfa475dc2a07072ab2c8d191020ca5d 29.78MB / 29.78MB                          0.6s
 => => sha256:b3ec39b36ae8c03a3e09854de4ec4aa08381dfed84a9daa075048c2e3df3881d 1.29MB / 1.29MB                            0.4s
 => => sha256:fc74430849022d13b0d44b8969a953f842f59c6e9d1a0c2c83d710affa286c08 13.88MB / 13.88MB                          0.6s
 => => sha256:2d97f6910b16bd338d3060f261f53f144965f755599aab1acda1e13cf1731b1b 10.36kB / 10.36kB                          0.0s
 => => sha256:dad5b29e3506c35e0fd222736f4d4ef25d21b219acdd73f7bb41d59996ca8e0d 1.74kB / 1.74kB                            0.0s
 => => sha256:ea56f685404adf81680322f152d2cfec62115b30dda481c2c450078315beb508 251B / 251B                                0.6s
 => => extracting sha256:38513bd7256313495cdd83b3b0915a633cfa475dc2a07072ab2c8d191020ca5d                                 0.9s
 => => extracting sha256:b3ec39b36ae8c03a3e09854de4ec4aa08381dfed84a9daa075048c2e3df3881d                                 0.2s
 => => extracting sha256:fc74430849022d13b0d44b8969a953f842f59c6e9d1a0c2c83d710affa286c08                                 0.7s
 => => extracting sha256:ea56f685404adf81680322f152d2cfec62115b30dda481c2c450078315beb508                                 0.0s
 => [internal] load build context                                                                                         0.1s
 => => transferring context: 401B                                                                                         0.0s
 => [2/5] WORKDIR /app                                                                                                    0.1s
 => [3/5] COPY src/requirements.txt .                                                                                     0.1s
 => [4/5] RUN pip install --no-cache-dir -r requirements.txt                                                              3.1s
 => [5/5] COPY src/ .                                                                                                     0.1s 
 => exporting to image                                                                                                    1.1s 
 => => exporting layers                                                                                                   1.0s 
 => => writing image sha256:a663c19ec02a29eb3a332d7e877d2d2021cf487939da05b735f58e3481eb1585                              0.0s 
 => => naming to docker.io/nautilus/python-app  
```

---

## 🔹 Step 7: Verify Image Creation

```bash
docker images nautilus/python-app
```

**Purpose**: Confirm the image was created successfully.

**Expected Output**:
```
REPOSITORY            TAG       IMAGE ID       CREATED              SIZE
nautilus/python-app   latest    a663c19ec02a   About a minute ago   133MB
```

**Success Indicators**:
- ✅ Image appears in the list
- ✅ Tag is `latest`
- ✅ Recent creation timestamp
- ✅ Reasonable size (~150MB, depending on dependencies)

---

## 🔹 Step 8: Deploy the Container

```bash
docker run -d --name pythonapp_nautilus -p 8092:8082 nautilus/python-app
```

**Purpose**: Run the container in detached mode with the specified name and port mapping.

**Expected Output**:
```
9df118ae8fa270800287f2bfea2aeb91a47f1d00e16ec62836ab340bf9926391
```

---

## 🔹 Step 9: Verify Running Container

```bash
docker ps
```

**Purpose**: Confirm the `pythonapp_nautilus` container is running.

**Expected Output**:
```
CONTAINER ID   IMAGE                 COMMAND              CREATED          STATUS          PORTS                                       NAMES
9df118ae8fa2   nautilus/python-app   "python server.py"   26 seconds ago   Up 25 seconds   0.0.0.0:8092->8082/tcp, :::8092->8082/tcp   pythonapp_nautilus
```

---

## 🔹 Step 10: Test Application Accessibility

```bash
curl http://localhost:8092/
```

**Purpose**: Verify the Python application is accessible on port 8095.

**Expected Output**:
```
Welcome to xFusionCorp Industries!
```

---

## 🔹 Step 11: Check Container Logs

```bash
docker logs pythonapp_nautilus
```

**Purpose**: Ensure the application started without errors.

**Expected Logs**: Depends on `server.py` output; typically shows server startup messages (e.g., `Running on http://0.0.0.0:8082`).

 * Serving Flask app 'server'
 * Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:8082
 * Running on http://172.12.0.2:8082
Press CTRL+C to quit
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 857-762-201
172.12.0.1 - - [21/Sep/2026 22:53:48] "GET / HTTP/1.1" 200 -

---

## 🔹 Step 12: Clean Up (Optional)

```bash
docker stop pythonapp_nautilus
docker rm pythonapp_nautilus
```

**Purpose**: Stop and remove the container if testing is complete.

---

## 📋 Quick Command Reference

For quick copy-paste, execute on **App Server 3**:

```bash
# Connect and navigate
ssh banner@stapp03
sudo su -
cd /python_app

# Create Dockerfile
cat > Dockerfile << 'EOF'
# Use official Python base image
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Copy dependencies file
COPY src/requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application files
COPY src/ .

# Expose the application port
EXPOSE 8082

# Run the Python server
CMD ["python", "server.py"]
EOF

# Build and deploy
docker build -t nautilus/python-app .
docker images nautilus/python-app
docker run -d --name pythonapp_nautilus -p 8092:8082 nautilus/python-app
docker ps
curl http://localhost:8092/
```

---

## 💡 Common Dockerfile Issues & Fixes

### **Issue 1: Missing requirements.txt**
**Problem**: `COPY src/requirements.txt` fails due to missing file.
**Fix**: Verify file exists in `/python_app/src/`.
```bash
ls -la src/requirements.txt
```

### **Issue 2: Dependency Installation Failure**
**Problem**: `pip install` fails due to missing packages or network issues.
**Fix**: Check `requirements.txt` and network connectivity.
```bash
docker pull python:3.9-slim
cat src/requirements.txt
```

### **Issue 3: Port Already in Use**
**Problem**: Host port 8095 is occupied.
**Fix**: Check and free the port.
```bash
netstat -tuln | grep 8092
kill -9 <pid>
```

### **Issue 4: server.py Not Found**
**Problem**: `CMD ["python", "server.py"]` fails because `server.py` is missing.
**Fix**: Ensure `server.py` is in `/python_app/src/`.
```bash
ls -la src/server.py
```

---

## 🔧 Troubleshooting Build and Deployment Failures

### **Build Error: COPY Failed**
**Symptoms**: "COPY failed: stat src/requirements.txt: no such file or directory".
**Solution**: Verify file paths relative to the build context.
```bash
ls -la src/
```

### **Build Error: pip Install Failed**
**Symptoms**: "ERROR: Could not find a version that satisfies the requirement...".
**Solution**: Validate `requirements.txt` and retry with `--no-cache`.
```bash
docker build --no-cache -t nautilus/python-app .
```

### **Container Error: Failed to Start**
**Symptoms**: `docker ps -a` shows `Exited` status.
**Solution**: Check logs for errors (e.g., missing `server.py`, dependency issues).
```bash
docker logs pythonapp_nautilus
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered**:
The primary challenge was Dockerizing a Python application by creating a Dockerfile that correctly installs dependencies, exposes the required port, and runs the application, followed by deploying a container with specific port mappings.

**💡 Solution Approach**:
1. **Directory Setup**: Navigated to `/python_app` and verified `src/requirements.txt` and `src/server.py`.
2. **Dockerfile Creation**: Defined a lean Dockerfile using `python:3.9-slim`, installing dependencies and setting up the app.
3. **Image Build**: Built the `nautilus/python-app` image with correct file paths.
4. **Container Deployment**: Ran the `pythonapp_nautilus` container with port mapping `8092:8082`.
5. **Verification**: Tested the app with `curl http://localhost:8092/` and checked container status.

**🎯 Key Success Factors**:
- **Correct Base Image**: Used `python:3.9-slim` for efficiency.
- **Dependency Management**: Installed dependencies from `requirements.txt` without caching.
- **File Path Accuracy**: Ensured `COPY src/` included all necessary files.
- **Port Mapping**: Correctly mapped `8092:8082` for external access.
- **Testing**: Validated output with `curl` and confirmed `Welcome to xFusionCorp Industries!`.

**⚠️ Critical Dockerfile Fixes**:
- Use `WORKDIR /app` for consistent file operations.
- Separate `COPY` for `requirements.txt` to leverage layer caching.
- Use `--no-cache-dir` in `pip install` to reduce image size.
- Validate `server.py` existence before running.

**🔒 Dockerfile Best Practices Applied**:
- **Minimal Base Image**: Used `python:3.9-slim` for smaller footprint.
- **Layer Optimization**: Separated dependency installation and file copying.
- **Clear Commands**: Used explicit `CMD ["python", "server.py"]` for clarity.
- **Error Prevention**: Validated file paths and dependencies.

**⚠️ Important Troubleshooting Concepts**:
- **Build Context**: Ensure all files are in the correct directory relative to the Dockerfile.
- **Layer Caching**: Use `--no-cache` for troubleshooting persistent build issues.
- **Log Analysis**: Check container logs for runtime errors.
- **Port Conflicts**: Verify host ports are free before deployment.

---

## ⚠️ Important Production Notes

🔧 **Dockerfile Optimization**: Use slim base images and minimize layers for production efficiency.
🔐 **Security**: Avoid hardcoding sensitive data in `server.py`; use environment variables or secrets.
📊 **Dependency Management**: Regularly update `requirements.txt` to patch vulnerabilities.
🛡️ **Port Security**: Restrict access to port 8095 with firewall rules in production.

### 📚 Official Documentation

- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [docker image build reference](https://docs.docker.com/reference/cli/docker/image/build/)
- [docker container run reference](https://docs.docker.com/reference/cli/docker/container/run/)
- [Docker port publishing](https://docs.docker.com/engine/network/port-publishing/)
- [Flask: Deploying to Production](https://flask.palletsprojects.com/en/stable/deploying/)
- [Flask: Debugging Application Errors](https://flask.palletsprojects.com/en/stable/debugging/)