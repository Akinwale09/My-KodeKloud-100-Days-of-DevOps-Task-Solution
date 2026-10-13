# Day 53: Resolve VolumeMounts Issue in Kubernetes

We encountered an issue with our Nginx and PHP-FPM setup on the Kubernetes cluster this morning, which halted its functionality. Investigate and rectify the issue:

The pod name is `nginx-phpfpm` and configmap name is `nginx-config`. Identify and fix the problem.

Once resolved, copy `/home/thor/index.php` file from the `jump host` to the `nginx-container` within the nginx document root. After this, you should be able to access the website using `Website` button on the top bar.

**Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

## Specific Requirements:

1. The pod name is `nginx-phpfpm` and configmap name is `nginx-config`. Identify and fix the problem.
2. Once resolved, copy `/home/thor/index.php` file from the `jump host` to the `nginx-container` within the nginx document root.
3. After this, the website must be accessible using the `Website` button on the top bar.

## Solution

The Pod initially reported `2/2 Running`, but this only confirmed that both container processes were healthy. The application was broken because the `shared-files` volume was mounted at `/var/www/html` in PHP-FPM and at `/usr/share/nginx/html` in Nginx, while the `nginx-config` ConfigMap defined `/var/www/html` as the document root. The fix was to mount the shared volume at `/var/www/html` in both containers and recreate the Pod.
---

## 🔹 Step 1: Connect to the Jump Host

```bash
ssh thor@jump_host
```

**Purpose**: Connect to the jump host where `kubectl` is configured to manage the Kubernetes cluster.

---

## 🔹 Step 2: Investigate the Pod Issue

```bash
kubectl get pod nginx-phpfpm
kubectl describe pod nginx-phpfpm
```

**Purpose**: Examine the pod’s configuration to identify the issue.

**Key Findings**:
- **ConfigMap (`nginx-config`)**: Specifies `root /var/www/html;` for Nginx’s document root.
- **Volume Mounts**:
  - `php-fpm-container`: Mounts the shared volume at `/var/www/html` (✅ correct).
  - `nginx-container`: Mounts the shared volume at `/usr/share/nginx/html` (❌ incorrect).
- **Issue**: Nginx serves content from `/usr/share/nginx/html`, but PHP-FPM reads/writes to `/var/www/html`, causing Nginx to fail to find PHP files (e.g., `index.php`) for processing.

**Problem Summary**: The misaligned volume mount paths prevent Nginx from accessing PHP files processed by PHP-FPM, breaking the application.

---

## 🔹 Step 3: Edit the Pod Configuration

```bash
kubectl edit pod nginx-phpfpm
```

**Purpose**: Modify the pod’s YAML to correct the `nginx-container` volume mount path.

**Change Required**:
- Update the `nginx-container`’s volume mount from:
  ```yaml
  - mountPath: /usr/share/nginx/html
    name: shared-files
  ```
  to:
  ```yaml
  - mountPath: /var/www/html
    name: shared-files
  ```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

```
error: pods "nginx-phpfpm" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-3509003944.yaml"
error: Edit cancelled, no valid changes were saved.
```

**Note**: Editing the pod directly creates a temporary YAML file (e.g., `/tmp/kubectl-edit-3509003944.yaml`) with the updated configuration.

---

## 🔹 Step 4: Delete and Recreate the Pod

```bash
kubectl delete pod nginx-phpfpm
kubectl apply -f /tmp/kubectl-edit-3509003944.yaml
```

**Purpose**: Delete the existing pod and apply the corrected YAML to recreate the pod with the proper volume mount.

**Expected Output (Delete)**:
```
pod "nginx-phpfpm" deleted
```

**Expected Output (Apply)**:
```
pod/nginx-phpfpm created
```

**Reference Pod YAML (for clarity)**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-phpfpm
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      volumeMounts:
        - mountPath: /var/www/html
          name: shared-files
    - name: php-fpm-container
      image: php:7.4-fpm
      volumeMounts:
        - mountPath: /var/www/html
          name: shared-files
  volumes:
    - name: shared-files
      emptyDir: {}
```

**Note**: The actual YAML may include additional configurations (e.g., ConfigMap mounts for `nginx-config`). Ensure the `nginx-container` mount path is corrected to `/var/www/html`.

---

## 🔹 Step 5: Verify Pod Status

```bash
kubectl get pods
```

**Purpose**: Confirm the `nginx-phpfpm` pod is running with both containers.

**Expected Output**:
```
NAME           READY   STATUS    RESTARTS   AGE
nginx-phpfpm   2/2     Running   0          3m
```

**Success Indicators**:
- ✅ Pod name: `nginx-phpfpm`
- ✅ Status: `Running`
- ✅ Ready: `2/2` (both `nginx-container` and `php-fpm-container`)
- ✅ No restarts

---

## 🔹 Step 6: Verify Pod Configuration

```bash
kubectl describe pod nginx-phpfpm
```

**Purpose**: Confirm the corrected volume mount path for `nginx-container`.

**Expected Output (Excerpt)**:
```
Containers:
  nginx-container:
    Image:          nginx:latest
    Volume Mounts:
      /var/www/html from shared-files (rw)
  php-fpm-container:
    Image:          php:7.4-fpm
    Volume Mounts:
      /var/www/html from shared-files (rw)
Volumes:
  shared-files:
    Type:       EmptyDir
```

**Success Indicators**:
- ✅ `nginx-container` mounts `shared-files` at `/var/www/html`
- ✅ `php-fpm-container` mounts `shared-files` at `/var/www/html`
- ✅ Both containers share the same volume path

---

## 🔹 Step 7: Copy index.php to the Nginx Container

```bash
kubectl cp /home/thor/index.php nginx-phpfpm:/var/www/html/index.php -c nginx-container
```

**Purpose**: Copy the `index.php` file from the jump host to the `nginx-container`’s document root (`/var/www/html`).

**Expected Output**: No output on success (silent operation).

**Verify File Copy**:
```bash
kubectl exec nginx-phpfpm -c nginx-container -- ls -l /var/www/html/index.php
```

**Expected Output**:
```
-rw-r--r-- 1 1000 1000 19 Sep 25 11:16 /var/www/html/index.php
```

---

## 🔹 Step 8: Test Application Accessibility

**Action**: In the KodeKloud lab interface, click the **Website** button to access the Nginx-PHP-FPM application.

**Purpose**: Verify the application is functional and displays the PHP page.

**Expected Output** (via browser):
- A PHP page rendered correctly (content depends on `index.php`).

**Alternative CLI Test** (if no website button is available):
```bash
kubectl port-forward pod/nginx-phpfpm 8080:80
```

**In a new terminal**:
```bash
curl http://localhost:8080/index.php
```

**Expected Output**: PHP page content (depends on `index.php`).

---

## 🔹 Step 9: Check Container Logs

```bash
kubectl logs nginx-phpfpm -c nginx-container
kubectl logs nginx-phpfpm -c php-fpm-container
```

**Purpose**: Verify both containers started correctly and are processing requests.

**Expected Logs (nginx-container)**:
```
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
10.22.0.1 - - [25/Sep/2026:11:19:08 +0000] "GET / HTTP/1.1" 200 126574 "https://sx5at77d3hiqcvgb.labs.kodekloud.com/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/152.0.0.0 Safari/537.36"
```

**Expected Logs (php-fpm-container)**:
```
[25-Sep-2026 11:11:20] NOTICE: fpm is running, pid 1
[25-Sep-2026 11:11:20] NOTICE: ready to handle connections
127.0.0.1 -  25/Sep/2026:11:19:08 +0000 "GET /index.php" 200
```
---

## 🔹 Step 10: Clean Up (Optional)

```bash
kubectl delete pod nginx-phpfpm
```

**Purpose**: Remove the pod if testing is complete.

**Expected Output**:
```
pod "nginx-phpfpm" deleted
```

---

## 📋 Quick Command Reference

For quick copy-paste, execute on **Jump Host**:

```bash
# Connect to jump host
ssh thor@jump_host

# Investigate and fix pod
kubectl describe pod nginx-phpfpm
kubectl edit pod nginx-phpfpm  # Change nginx-container mountPath to /var/www/html
kubectl delete pod nginx-phpfpm
kubectl apply -f /tmp/kubectl-edit-2234947867.yaml

# Copy file and verify
kubectl cp /home/thor/index.php nginx-phpfpm:/var/www/html/index.php -c nginx-container
kubectl exec nginx-phpfpm -c nginx-container -- ls -l /var/www/html/index.php
kubectl get pods
kubectl describe pod nginx-phpfpm
```

---

## 💡 Common Kubernetes Pod Issues & Fixes

### **Issue 1: Pod Stuck in Pending**
**Problem**: `kubectl get pods` shows `Pending` status.
**Fix**: Check events for scheduling issues.
```bash
kubectl describe pod nginx-phpfpm
```
**Common Causes**: Insufficient resources or node issues.

### **Issue 2: Nginx Cannot Find PHP Files**
**Problem**: Nginx returns 404 errors for PHP files.
**Fix**: Verify volume mount paths align for both containers.
```bash
kubectl describe pod nginx-phpfpm
```

### **Issue 3: File Copy Failure**
**Problem**: `kubectl cp` fails with "no such file or directory".
**Fix**: Verify source and destination paths.
```bash
ls -l /home/thor/index.php
kubectl exec nginx-phpfpm -c nginx-container -- ls -l /var/www/html
```

### **Issue 4: PHP-FPM Not Processing Requests**
**Problem**: PHP pages fail to render.
**Fix**: Check PHP-FPM logs and Nginx configuration.
```bash
kubectl logs nginx-phpfpm -c php-fpm-container
kubectl exec nginx-phpfpm -c nginx-container -- cat /etc/nginx/conf.d/default.conf
```

---

## 🔧 Troubleshooting Pod Failures

### **Error: Pod Not Running**
**Symptoms**: `kubectl get pods` shows `Error` or `CrashLoopBackOff`.
**Solution**: Check logs and events for both containers.
```bash
kubectl logs nginx-phpfpm -c nginx-container
kubectl logs nginx-phpfpm -c php-fpm-container
kubectl get events --field-selector involvedObject.name=nginx-phpfpm
```

### **Error: Volume Mount Misconfiguration**
**Symptoms**: Nginx cannot access PHP files.
**Solution**: Ensure both containers mount the shared volume at `/var/www/html`.
```bash
kubectl describe pod nginx-phpfpm
```

### **Error: kubectl cp Fails**
**Symptoms**: "error: tar: unexpected EOF" or similar.
**Solution**: Verify file exists and container is running.
```bash
kubectl get pods
ls -l /home/thor/index.php
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered**:
The primary challenge was identifying and fixing the misaligned volume mount paths in the `nginx-phpfpm` pod, where `nginx-container` used `/usr/share/nginx/html` while `php-fpm-container` used `/var/www/html`, causing Nginx to fail to find PHP files. Additionally, copying `index.php` and verifying application functionality was required.

**💡 Solution Approach**:
1. **Issue Identification**: Used `kubectl describe` to find the mismatched volume mount paths.
2. **Pod Correction**: Edited the pod to align `nginx-container`’s mount path to `/var/www/html`.
3. **Pod Recreation**: Deleted and reapplied the corrected pod configuration.
4. **File Copy**: Copied `index.php` to the Nginx document root using `kubectl cp`.
5. **Verification**: Confirmed pod status and application accessibility via the website button.

**🎯 Key Success Factors**:
- **Accurate Diagnosis**: Identified the volume mount mismatch using `kubectl describe`.
- **Correct Mount Path**: Aligned `nginx-container` to `/var/www/html`.
- **File Copy**: Successfully copied `index.php` to the correct location.
- **Testing**: Validated PHP page rendering via the website button.
- **Verification**: Confirmed pod and container status post-fix.

**⚠️ Critical Fixes**:
- Correct `nginx-container` volume mount to `/var/www/html`.
- Ensure `shared-files` volume is used consistently across containers.
- Verify `index.php` exists on the jump host before copying.
- Validate Nginx configuration in `nginx-config` ConfigMap.

**🔒 Kubernetes Best Practices Applied**:
- **Shared Volumes**: Ensured consistent mount paths for multi-container pods.
- **Troubleshooting**: Used `kubectl describe` and `kubectl logs` for systematic diagnosis.
- **File Management**: Verified file paths before `kubectl cp`.
- **Validation**: Confirmed functionality with direct testing.

**⚠️ Important Troubleshooting Concepts**:
- **Volume Mounts**: Verify mount paths align with application configuration.
- **Logs**: Check both containers’ logs for errors.
- **Events**: Use `kubectl get events` to diagnose pod issues.
- **ConfigMap**: Ensure `nginx-config` aligns with volume mounts.

---

## ⚠️ Important Production Notes

🔧 **Pod Debugging**: Use `kubectl describe`, `kubectl logs`, and `kubectl exec` for thorough issue resolution.
🔐 **Security**: Restrict access to the PHP application with proper Nginx configuration and network policies.
📊 **Monitoring**: Use `kubectl top` to monitor resource usage for Nginx and PHP-FPM containers.
🛡️ **Service Exposure**: Configure a Kubernetes service (e.g., ClusterIP or NodePort) for production access.

## Best Practices

- **Align application paths with volume mounts.** A shared volume may be mounted at different paths in different containers, but each path must match what that container's application configuration expects.
- **Do not equate Pod health with application health.** `Running` and `Ready` show that processes passed their container checks; they do not prove that Nginx can locate and execute the expected PHP file.
- **Edit the controller instead of its Pods.** Pods created by a Deployment should not be edited directly. Update the Deployment's Pod template so that the controller performs the replacement and preserves the declared state.
- **Use a controller for production workloads.** A Deployment normally manages Pod replacement and recovery. Force-replacing a standalone Pod causes downtime and discards data stored in its `emptyDir` volume.
- **Treat `emptyDir` data as temporary.** Its contents survive container restarts but are deleted when the Pod is removed, which is why `index.php` had to be copied after recreating the Pod.

### 📚 Official Documentation

- [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
- [Communicate Between Containers in the Same Pod Using a Shared Volume](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/)
- [kubectl replace](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#replace)
- [kubectl cp](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_cp/)

