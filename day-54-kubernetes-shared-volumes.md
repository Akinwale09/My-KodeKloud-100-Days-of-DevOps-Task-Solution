# Day 54: Kubernetes Shared Volumes

We are working on an application that will be deployed on multiple containers within a pod on Kubernetes cluster. There is a requirement to share a volume among the containers to save some temporary data. The Nautilus DevOps team is developing a similar template to replicate the scenario. Below you can find more details about it.

## Specific Requirements:

1. Create a pod named `volume-share-nautilus`.
   
2. For the first container, use image `fedora` with `latest` tag only and remember to mention the tag i.e `fedora:latest`, container should be named as `volume-container-nautilus-1`, and run a `sleep` command for it so that it remains in running state. Volume `volume-share` should be mounted at path `/tmp/news`.
   
3. For the second container, use image `fedora` with the `latest` tag only and remember to mention the tag i.e `fedora:latest`, container should be named as `volume-container-nautilus-2`, and again run a `sleep` command for it so that it remains in running state. Volume `volume-share` should be mounted at path `/tmp/apps`.
   
4. Volume name should be `volume-share` of type `emptyDir`.
   
5. After creating the pod, exec into the first container i.e `volume-container-nautilus-1`, and just for testing create a file `news.txt` with the content `Welcome to xFusionCorp Industries` under the mounted path of first container i.e `/tmp/news`.
   
6. The file `news.txt` should be present under the mounted path `/tmp/apps` on the second container `volume-container-nautilus-2` as well, since they are using a shared volume.

**Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

## Solution

An `emptyDir` volume can be mounted by multiple containers in the same Pod. Each container may use a different `mountPath`, but the paths expose the same underlying storage. In this lab, `/tmp/beta` in the first container and `/tmp/apps` in the second container both refer to the `volume-share` volume.

---

## 🔹 Step 1: Connect to the Jump Host

```bash
ssh thor@jump_host
```

**Purpose**: Connect to the jump host where `kubectl` is configured to manage the Kubernetes cluster.

---

## 🔹 Step 2: Create the Pod YAML File

```bash
vi volume-share-nautilus.yaml
```

**Purpose**: Create a Kubernetes YAML file to define the `volume-share-nautilus` pod with a shared volume.

---

## 🔹 Step 3: Add Pod YAML Content

**Pod YAML**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-nautilus
spec:
  containers:
    - name: volume-container-nautilus-1
      image: fedora:latest
      command: ["sleep", "3600"]
      volumeMounts:
        - name: volume-share
          mountPath: /tmp/news
    - name: volume-container-nautilus-2
      image: fedora:latest
      command: ["sleep", "3600"]
      volumeMounts:
        - name: volume-share
          mountPath: /tmp/apps
  volumes:
    - name: volume-share
      emptyDir: {}
```

**Key Configurations**:
- **apiVersion: v1**: Specifies the Kubernetes API version for pods.
- **kind: Pod**: Defines the resource type as a pod.
- **metadata.name: volume-share-nautilus**: Sets the pod name.
- **spec.containers**: Defines two containers:
  - **First Container**:
    - `name: volume-container-nautilus-1`: Sets the container name.
    - `image: fedora:latest`: Uses the `fedora:latest` image.
    - `command: ["sleep", "3600"]`: Keeps the container running for 1 hour.
    - `volumeMounts`: Mounts `volume-share` at `/tmp/news`.
  - **Second Container**:
    - `name: volume-container-nautilus-2`: Sets the container name.
    - `image: fedora:latest`: Uses the `fedora:latest` image.
    - `command: ["sleep", "3600"]`: Keeps the container running for 1 hour.
    - `volumeMounts`: Mounts `volume-share` at `/tmp/apps`.
- **spec.volumes**: Defines a shared volume:
  - `name: volume-share`: Names the volume.
  - `emptyDir: {}`: Uses an `emptyDir` volume for temporary storage.

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

## 🔹 Step 4: Verify Pod YAML Syntax

```bash
cat volume-share-nautilus.yaml
```

**Purpose**: Confirm the YAML file contains the correct configuration.

**Validation Checklist**:
- ✅ `apiVersion: v1` and `kind: Pod`
- ✅ Pod name: `volume-share-nautilus`
- ✅ Container names: `volume-container-nautilus-1` and `volume-container-nautilus-2`
- ✅ Image: `fedora:latest` for both containers
- ✅ Command: `["sleep", "3600"]` for both containers
- ✅ Volume mounts: `/tmp/news` and `/tmp/apps` for `volume-share`
- ✅ Volume: `volume-share` with `emptyDir: {}`
- ✅ Correct YAML indentation (2 spaces)

---

## 🔹 Step 5: Apply the Pod Configuration

```bash
kubectl apply -f volume-share-nautilus.yaml
```

**Purpose**: Deploy the pod to the Kubernetes cluster.

**Expected Output**:
```
pod/volume-share-nautilus created
```

---

## 🔹 Step 6: Verify Pod Creation

```bash
kubectl get pods
```

**Purpose**: Confirm the `volume-share-nautilus` pod is running with both containers.

**Expected Output**:
```
NAME                    READY   STATUS    RESTARTS   AGE
volume-share-nautilus   2/2     Running   0          18s
```

**Success Indicators**:
- ✅ Pod name: `volume-share-nautilus`
- ✅ Status: `Running`
- ✅ Ready: `2/2` (both containers)
- ✅ No restarts

---

## 🔹 Step 7: Verify Pod Details

```bash
kubectl describe pod volume-share-nautilus
```

**Purpose**: Check detailed information about the pod, including volume mounts and container status.

**Expected Output (Excerpt)**:
```
Name:             volume-share-nautilus
Namespace:        default
Priority:         0
Service Account:  default
Node:             jump-host/10.244.164.51
Start Time:       Sat, 26 Sep 2026 21:42:48 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.22.0.9
IPs:
  IP:  10.22.0.9
Containers:
  volume-container-nautilus-1:
    Container ID:  containerd://e5eace1e85b45541f37946a0cf3b29c73f6b16392d854c9de9520e1847e6a48c
    Image:         fedora:latest
    Image ID:      docker.io/library/fedora@sha256:43b29f65a41eb9c35e1cd5323e3bdf3b655c2357a9f4f1ff2f9c2798e5045d80
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      3600
    State:          Running
      Started:      Sat, 26 Sep 2026 21:42:52 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /tmp/news from volume-share (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-rnt26 (ro)
  volume-container-nautilus-2:
    Container ID:  containerd://cc244a87305b838643f5450d3211074364b7bdd3fdef074569b4eb5463ee3c9f
    Image:         fedora:latest
    Image ID:      docker.io/library/fedora@sha256:43b29f65a41eb9c35e1cd5323e3bdf3b655c2357a9f4f1ff2f9c2798e5045d80
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      3600
    State:          Running
      Started:      Sat, 26 Sep 2026 21:42:53 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /tmp/apps from volume-share (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-rnt26 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  volume-share:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-rnt26:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  46s   default-scheduler  Successfully assigned default/volume-share-nautilus to jump-host
  Normal  Pulling    46s   kubelet            Pulling image "fedora:latest"
  Normal  Pulled     43s   kubelet            Successfully pulled image "fedora:latest" in 2.845s (2.845s including waiting). Image size: 69403471 bytes.
  Normal  Created    43s   kubelet            Created container: volume-container-nautilus-1
  Normal  Started    43s   kubelet            Started container volume-container-nautilus-1
  Normal  Pulling    43s   kubelet            Pulling image "fedora:latest"
  Normal  Pulled     43s   kubelet            Successfully pulled image "fedora:latest" in 338ms (338ms including waiting). Image size: 69403471 bytes.
  Normal  Created    43s   kubelet            Created container: volume-container-nautilus-2
  Normal  Started    42s   kubelet            Started container volume-container-nautilus-2
```

**Success Indicators**:
- ✅ Correct container names and images
- ✅ Volume mounts at `/tmp/news` and `/tmp/apps`
- ✅ `volume-share` as `EmptyDir`
- ✅ Both containers in `Running` state

---

## 🔹 Step 8: Exec into First Container and Create File

```bash
kubectl exec -it volume-share-nautilus -c volume-container-nautilus-1 -- bash
```

**Inside the container**:
```bash
cd /tmp/news
echo "Welcome to xFusionCorp Industries" > news.txt
exit
```

**Purpose**: Create a `news.txt` file in the shared volume at `/tmp/news` in the first container.

---

## 🔹 Step 9: Verify File in Second Container

```bash
kubectl exec -it volume-share-nautilus -c volume-container-nautilus-2 -- bash
```

**Inside the container**:
```bash
cat /tmp/apps/news.txt
```

**Expected Output**:
```
Welcome to xFusionCorp Industries
```

**Purpose**: Confirm the `news.txt` file is accessible in the second container at `/tmp/apps`, verifying the shared volume.

**Exit the container**:
```bash
exit
```

---

## 🔹 Step 10: Check Container Logs (Optional)

```bash
kubectl logs volume-share-nautilus -c volume-container-nautilus-1
kubectl logs volume-share-nautilus -c volume-container-nautilus-2
```

**Purpose**: Verify both containers are running without errors.

**Expected Output**: Likely empty, as `sleep 3600` produces no logs unless interrupted.

---

## 🔹 Step 11: Clean Up (Optional)

```bash
kubectl delete -f volume-share-nautilus.yaml
```

**Purpose**: Remove the pod if testing is complete.

**Expected Output**:
```
pod "volume-share-nautilus" deleted
```

---

## 📋 Quick Command Reference

For quick copy-paste, execute on **Jump Host**:

```bash
# Connect to jump host
ssh thor@jump_host

# Create pod YAML
cat > volume-share-nautilus.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-nautilus
spec:
  containers:
    - name: volume-container-nautilus-1
      image: fedora:latest
      command: ["sleep", "3600"]
      volumeMounts:
        - name: volume-share
          mountPath: /tmp/news
    - name: volume-container-nautilus-2
      image: fedora:latest
      command: ["sleep", "3600"]
      volumeMounts:
        - name: volume-share
          mountPath: /tmp/apps
  volumes:
    - name: volume-share
      emptyDir: {}
EOF

# Deploy and verify
kubectl apply -f volume-share-nautilus.yaml
kubectl get pods
kubectl describe pod volume-share-nautilus

# Create and verify file in shared volume
kubectl exec -it volume-share-nautilus -c volume-container-nautilus-1 -- bash -c 'cd /tmp/blog && echo "Welcome to xFusionCorp Industries" > news.txt'
kubectl exec -it volume-share-nautilus -c volume-container-nautilus-2 -- cat /tmp/games/news.txt
```

---

## 💡 Common Kubernetes Pod Issues & Fixes

### **Issue 1: Pod Stuck in Pending**
**Problem**: `kubectl get pods` shows `Pending` status.
**Fix**: Check events for scheduling issues.
```bash
kubectl describe pod volume-share-nautilus
```
**Common Causes**: Insufficient cluster resources or node issues.

### **Issue 2: Image Pull Failure**
**Problem**: Pods in `ErrImagePull` or `ImagePullBackOff`.
**Fix**: Verify image name and tag.
```bash
docker pull ubuntu:latest
kubectl delete pod volume-share-nautilus
kubectl apply -f volume-share-nautilus.yaml
```

### **Issue 3: File Not Shared**
**Problem**: `news.txt` not visible in `volume-container-nautilus-2`.
**Fix**: Verify volume mount paths and `emptyDir` configuration.
```bash
kubectl describe pod volume-share-nautilus
kubectl exec -it volume-share-nautilus -c volume-container-nautilus-1 -- ls -l /tmp/news
```

### **Issue 4: Exec Command Fails**
**Problem**: `kubectl exec` fails with "container not found" or "no shell".
**Fix**: Ensure container is running and has `bash`.
```bash
kubectl get pods
kubectl exec -it volume-share-nautilus -c volume-container-nautilus-1 -- bash
```

---

## 🔧 Troubleshooting Pod Failures

### **Error: Pod Not Running**
**Symptoms**: `kubectl get pods` shows `Error` or `CrashLoopBackOff`.
**Solution**: Check logs and events for both containers.
```bash
kubectl logs volume-share-nautilus -c volume-container-nautilus-1
kubectl logs volume-share-nautilus -c volume-container-nautilus-2
kubectl get events --field-selector involvedObject.name=volume-share-nautilus
```

### **Error: Volume Not Shared**
**Symptoms**: `news.txt` not visible in `/tmp/apps`.
**Solution**: Verify `volume-share` is mounted correctly in both containers.
```bash
kubectl describe pod volume-share-nautilus
```

### **Error: YAML Syntax Error**
**Symptoms**: `kubectl apply` fails with parsing error.
**Solution**: Validate YAML syntax.
```bash
kubectl apply -f volume-share-nautilus.yaml --dry-run=client
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered**:
The primary challenge was creating a pod with two containers sharing an `emptyDir` volume, ensuring the volume is mounted at different paths (`/tmp/news` and `/tmp/apps`), and verifying that a file created in one container is accessible in the other.

**💡 Solution Approach**:
1. **YAML Creation**: Defined `volume-share-nautilus.yaml` with two containers and a shared `emptyDir` volume.
2. **Volume Configuration**: Mounted `volume-share` at `/tmp/news` and `/tmp/apps` for the respective containers.
3. **Pod Deployment**: Applied the pod using `kubectl apply`.
4. **File Creation**: Created `news.txt` in `volume-container-nautilus-1` at `/tmp/news`.
5. **Verification**: Confirmed `news.txt` is accessible in `volume-container-nautilus-2` at `/tmp/apps`.

**🎯 Key Success Factors**:
- **Correct YAML Structure**: Ensured proper `apiVersion: v1`, container, and volume specifications.
- **Shared Volume**: Configured `emptyDir` volume correctly with consistent naming.
- **Mount Paths**: Used distinct mount paths (`/tmp/news`, `/tmp/apps`) for the shared volume.
- **Testing**: Validated file sharing by creating and reading `news.txt`.
- **Verification**: Confirmed pod status and volume mounts with `kubectl describe`.

**⚠️ Critical Fixes**:
- Ensure `volume-share` is defined as `emptyDir` and mounted correctly.
- Use `["sleep", "3600"]` to keep containers running.
- Validate `ubuntu:latest` image availability.
- Confirm file creation and access in both containers.

**🔒 Kubernetes Best Practices Applied**:
- **Shared Volumes**: Used `emptyDir` for temporary data sharing between containers.
- **Minimal Configuration**: Kept YAML simple with only required fields.
- **Dry Run Testing**: Validated YAML with `--dry-run=client`.
- **Exec Verification**: Used `kubectl exec` to test volume sharing interactively.

**⚠️ Important Troubleshooting Concepts**:
- **Volume Mounts**: Verify mount paths align with volume configuration.
- **Logs**: Check container logs for runtime issues.
- **Events**: Use `kubectl get events` to diagnose pod issues.
- **Exec Debugging**: Use `kubectl exec` to inspect container filesystems.

---

## ⚠️ Important Production Notes

🔧 **Pod Debugging**: Use `kubectl describe`, `kubectl logs`, and `kubectl exec` for thorough issue resolution.
🔐 **Image Security**: Pin specific `fedora` versions (e.g., `fedora:20.04`) in production to avoid unexpected updates.
📊 **Resource Limits**: Add `resources` (e.g., CPU/memory limits) to prevent overuse in production.
🛡️ **Volume Management**: Use persistent volumes (e.g., PVCs) instead of `emptyDir` for production data.

## Best Practices

- **Use shared volumes for data, not container layers.** Containers have separate root filesystems, so files that must be available to more than one container should be written to a shared volume.
- **Treat `emptyDir` data as temporary.** The volume survives individual container restarts but is deleted permanently when the Pod is removed from its node.
- **Use descriptive volume and container names.** Clear names make multi-container Pod manifests and troubleshooting output easier to understand.
- **Choose persistent storage for durable data.** Use a PersistentVolume rather than `emptyDir` when data must survive Pod replacement or deletion.

### 📚 Official Documentation

- [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
- [Communicate Between Containers in the Same Pod Using a Shared Volume](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/)
- [kubectl exec](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_exec/)