# Day 61: Init Containers in Kubernetes

There are some applications that need to be deployed on Kubernetes cluster and these apps have some pre-requisites where some configurations need to be changed before deploying the app container. Some of these changes cannot be made inside the images so the DevOps team has come up with a solution to use init containers to perform these tasks during deployment. Below is a sample scenario that the team is going to test first.
 
## Specific Requirements:

1. Create a `Deployment` named as `ic-deploy-devops`.
2. Configure `spec` as replicas should be `1`, labels `app` should be `ic-devops`, template's metadata lables `app` should be the same `ic-devops`.
3. The `initContainers` should be named as `ic-msg-devops`, use image `debian` with `latest` tag and use command `'/bin/bash'`, `'-c'` and `'echo Init Done - Welcome to xFusionCorp Industries > /ic/news'`. The volume mount should be named as `ic-volume-devops` and mount path should be `/ic`.
4. Main container should be named as `ic-main-devops`, use image `debian` with `latest` tag and use command `'/bin/bash'`, `'-c'` and `'while true; do cat /ic/news; sleep 5; done'`. The volume mount should be named as `ic-volume-devops` and mount path should be `/ic`.
5. Volume to be named as `ic-volume-devops` and it should be an emptyDir type.

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

## Solution

An init container performs setup work before a Pod's application containers start. It must run to completion successfully; otherwise, Kubernetes does not start the main container. In this challenge, the init container writes a message to a shared `emptyDir` volume, and the main container reads that prepared file every five seconds.

The startup sequence is:

```text
Pod starts
    ↓
ic-msg-devops writes /ic/news
    ↓
init container exits successfully
    ↓
ic-main-devops starts
    ↓
main container reads /ic/news every five seconds
```

The two containers do not run at the same time. They exchange the file through `ic-volume-devops`, which Kubernetes mounts at `/ic` in both containers.
---

## 🔹 Step 1: Connect to the Jump Host

```bash
ssh thor@jump_host
```

**Purpose**: Connect to the jump host where `kubectl` is configured to manage the Kubernetes cluster.

---

## 🔹 Step 2: Create the Deployment YAML File

```bash
vi ic-deploy-devops.yaml
```

**Purpose**: Create a Kubernetes YAML file to define the `ic-deploy-devops` deployment with an init container, main container, and shared volume.

---

## 🔹 Step 3: Add Deployment YAML Content

**Deployment YAML**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ic-deploy-devops
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ic-devops
  template:
    metadata:
      labels:
        app: ic-devops
    spec:
      initContainers:
      - name: ic-msg-devops
        image: debian:latest
        command: ["/bin/bash", "-c", "echo Init Done - Welcome to xFusionCorp Industries > /ic/beta"]
        volumeMounts:
        - name: ic-volume-devops
          mountPath: /ic
      containers:
      - name: ic-main-devops
        image: debian:latest
        command: ["/bin/bash", "-c", "while true; do cat /ic/beta; sleep 5; done"]
        volumeMounts:
        - name: ic-volume-devops
          mountPath: /ic
      volumes:
      - name: ic-volume-devops
        emptyDir: {}
```

**Key Configurations**:
- **apiVersion: apps/v1**: Specifies the API version for deployments.
- **kind: Deployment**: Defines the resource type as a deployment.
- **metadata.name: ic-deploy-devops**: Sets the deployment name.
- **spec.replicas: 1**: Runs one pod instance.
- **spec.selector.matchLabels: app: ic-datacenter**: Matches pods with the label `app: ic-devops`.
- **spec.template.metadata.labels: app: ic-datacenter**: Applies the label to pods.
- **spec.template.spec.initContainers**:
  - `name: ic-msg-devops`: Sets the init container name.
  - `image: debian:latest`: Uses the `debian:latest` image.
  - `command: ["/bin/bash", "-c", "echo Init Done - Welcome to xFusionCorp Industries > /ic/beta"]`: Writes a message to `/ic/beta`.
  - `volumeMounts`: Mounts `ic-volume-devops` at `/ic`.
- **spec.template.spec.containers**:
  - `name: ic-main-devops`: Sets the main container name.
  - `image: debian:latest`: Uses the `debian:latest` image.
  - `command: ["/bin/bash", "-c", "while true; do cat /ic/beta; sleep 5; done"]`: Continuously reads `/ic/beta` every 5 seconds.
  - `volumeMounts`: Mounts `ic-volume-devops` at `/ic`.
- **spec.template.spec.volumes**:
  - `name: ic-volume-devops`: Names the volume.
  - `emptyDir: {}`: Uses an `emptyDir` volume for temporary storage.

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

## 🔹 Step 4: Verify YAML Syntax

```bash
cat ic-deploy-devops.yaml
```

**Purpose**: Confirm the YAML file contains the correct configuration.

**Validation Checklist**:
- ✅ `apiVersion: apps/v1` and `kind: Deployment`
- ✅ Deployment name: `ic-deploy-devops`
- ✅ Replicas: `1`
- ✅ Labels: `app: ic-devops` in `selector` and `template.metadata`
- ✅ Init container: `name: ic-msg-devops`, `image: debian:latest`, correct command, volume mount `/ic`
- ✅ Main container: `name: ic-main-devops`, `image: debian:latest`, correct command, volume mount `/ic`
- ✅ Volume: `name: ic-volume-devops`, `emptyDir: {}`
- ✅ Correct YAML indentation (2 spaces)

---

## 🔹 Step 5: Apply the Deployment Configuration

```bash
kubectl apply -f ic-deploy-devops.yaml
```

**Purpose**: Deploy the `ic-deploy-datacenter` deployment to the Kubernetes cluster.

**Expected Output**:
```
deployment.apps/ic-deploy-devops created
```

---

## 🔹 Step 6: Verify Deployment and Pods

```bash
kubectl get deployments
kubectl get pods
```

**Purpose**: Confirm the deployment is running and pods are operational.

**Expected Deployment Output**:
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
ic-deploy-devops   1/1     1            1           61s
```

**Expected Pod Output**:
```
NAME                                READY   STATUS    RESTARTS   AGE
ic-deploy-devops-7848d5f789-6jv5r   1/1     Running   0          88s
```

**Success Indicators**:
- ✅ Deployment name: `ic-deploy-devops`
- ✅ Pod status: `Running`
- ✅ Ready: `1/1` for both deployment and pod
- ✅ No restarts

---

## 🔹 Step 7: Verify Pod Logs

```bash
kubectl logs -f -l app=ic-devops
```

**Purpose**: Check the main container’s logs to confirm it reads the init container’s message.

**Expected Output**:
```
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
...
```

**Success Indicators**:
- ✅ Output matches: `Init Done - Welcome to xFusionCorp Industries`
- ✅ Output repeats every 5 seconds (due to `sleep 5`)

---

## 🔹 Step 8: Verify Pod Details

```bash
kubectl describe pod -l app=ic-devops
```
**Purpose**: Confirm the pod’s configuration, including init and main containers, volume mounts, and status.

**Expected Output**:
```
Name:             ic-deploy-devops-7848d5f789-6jv5r
Namespace:        default
Priority:         0
Service Account:  default
Node:             jump-host/10.244.97.147
Start Time:       Tue, 13 Oct 2026 10:56:46 +0000
Labels:           app=ic-devops
                  pod-template-hash=7848d5f789
Annotations:      <none>
Status:           Running
IP:               10.22.0.9
IPs:
  IP:           10.22.0.9
Controlled By:  ReplicaSet/ic-deploy-devops-7848d5f789
Init Containers:
  ic-msg-devops:
    Container ID:  containerd://b2181d7abe2631dfd8446d9acdf10c87a4da8951366559265f0dd283be74aecb
    Image:         debian:latest
    Image ID:      docker.io/library/debian@sha256:913f6706df59a68922d1dd08f78c2476560a8d367897200a6005b00e5f67c2d5
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/bash
      -c
      echo Init Done - Welcome to xFusionCorp Industries > /ic/beta
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 13 Oct 2026 10:56:49 +0000
      Finished:     Tue, 13 Oct 2026 10:56:49 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /ic from ic-volume-devops (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-8mzvl (ro)
Containers:
  ic-main-devops:
    Container ID:  containerd://8800c4cdceadc9611199e5139dedcc526ebcb602bee9539b964a70c78b446b03
    Image:         debian:latest
    Image ID:      docker.io/library/debian@sha256:913f6706df59a68922d1dd08f78c2476560a8d367897200a6005b00e5f67c2d5
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/bash
      -c
      while true; do cat /ic/beta; sleep 5; done
    State:          Running
      Started:      Tue, 13 Oct 2026 10:56:50 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /ic from ic-volume-devops (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-8mzvl (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  ic-volume-devops:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-8mzvl:
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
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m33s  default-scheduler  Successfully assigned default/ic-deploy-devops-7848d5f789-6jv5r to jump-host
  Normal  Pulling    3m33s  kubelet            Pulling image "debian:latest"
  Normal  Pulled     3m31s  kubelet            Successfully pulled image "debian:latest" in 2.039s (2.039s including waiting). Image size: 49403605 bytes.
  Normal  Created    3m31s  kubelet            Created container: ic-msg-devops
  Normal  Started    3m30s  kubelet            Started container ic-msg-devops
  Normal  Pulling    3m30s  kubelet            Pulling image "debian:latest"
  Normal  Pulled     3m30s  kubelet            Successfully pulled image "debian:latest" in 355ms (355ms including waiting). Image size: 49403605 bytes.
  Normal  Created    3m29s  kubelet            Created container: ic-main-devops
  Normal  Started    3m29s  kubelet            Started container ic-main-devops
```

**Success Indicators**:
- ✅ Init container: `ic-msg-devops`, `debian:latest`, correct command, `State: Terminated (Completed)`
- ✅ Main container: `ic-main-devops`, `debian:latest`, correct command, `State: Running`
- ✅ Volume mounts: `/ic` for both containers
- ✅ Volume: `ic-volume-devops` as `EmptyDir`

---

## 🔹 Step 9: Verify File in Main Container (Optional)

```bash
kubectl exec -it $(kubectl get pods -l app=ic-devops -o name | head -n 1) -- cat /ic/beta
```

**Purpose**: Confirm the init container created the file `/ic/beta` in the shared volume.

**Expected Output**:
```
Init Done - Welcome to xFusionCorp Industries
```

---

## 🔹 Step 10: Clean Up (Optional)

```bash
kubectl delete -f ic-deploy-devops.yaml
```

**Purpose**: Remove the deployment if testing is complete.

**Expected Output**:
```
deployment.apps "ic-deploy-devops" deleted
```

---

## 📋 Quick Command Reference

For quick copy-paste, execute on **Jump Host**:

```bash
# Connect to jump host
ssh thor@jump_host

# Create deployment YAML
cat > ic-deploy-devops.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ic-deploy-devops
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ic-devops
  template:
    metadata:
      labels:
        app: ic-devops
    spec:
      initContainers:
      - name: ic-msg-devops
        image: debian:latest
        command: ["/bin/bash", "-c", "echo Init Done - Welcome to xFusionCorp Industries > /ic/beta"]
        volumeMounts:
        - name: ic-volume-devops
          mountPath: /ic
      containers:
      - name: ic-main-devops
        image: debian:latest
        command: ["/bin/bash", "-c", "while true; do cat /ic/beta; sleep 5; done"]
        volumeMounts:
        - name: ic-volume-devops
          mountPath: /ic
      volumes:
      - name: ic-volume-devops
        emptyDir: {}
EOF

# Deploy and verify
kubectl apply -f ic-deploy-devops.yaml
kubectl get devops
kubectl get pods
kubectl logs -f -l app=ic-devops
kubectl describe pod -l app=ic-devops
```

---

## 💡 Common Kubernetes Deployment Issues & Fixes

### **Issue 1: Init Container Failure**
**Problem**: Init container fails, preventing main container from starting.
**Fix**: Check init container logs and command syntax.
```bash
kubectl logs -l app=ic-devops --container=ic-msg-datacenter
kubectl describe pod -l app=ic-devops
```

### **Issue 2: Image Pull Failure**
**Problem**: Pods in `ErrImagePull` or `ImagePullBackOff`.
**Fix**: Verify image name and tag.
```bash
docker pull debian:latest
kubectl rollout restart deployment ic-deploy-devops
```

### **Issue 3: Main Container Cannot Read File**
**Problem**: Main container logs show errors or no output for `/ic/beta`.
**Fix**: Verify volume mount and init container output.
```bash
kubectl exec -it $(kubectl get pods -l app=ic-devops -o name | head -n 1) -- ls -l /ic
kubectl describe pod -l app=ic-devops
```

### **Issue 4: Deployment Not Running**
**Problem**: `kubectl get deployments` shows `0/1` ready.
**Fix**: Check pod status and events.
```bash
kubectl get pods
kubectl get events --field-selector involvedObject.name=ic-deploy-devops
```

---

## 🔧 Troubleshooting Deployment Failures

### **Error: Init Container Not Completing**
**Symptoms**: Pod stuck in `Init:Error` or `Init:CrashLoopBackOff`.
**Solution**: Check init container logs and command.
```bash
kubectl logs -l app=ic-datacenter --container=ic-msg-devops
kubectl describe pod -l app=ic-datacenter
```

### **Error: Volume Not Shared**
**Symptoms**: Main container cannot access `/ic/beta`.
**Solution**: Verify volume mounts and `emptyDir` configuration.
```bash
kubectl describe pod -l app=ic-devops
```

### **Error: YAML Syntax Error**
**Symptoms**: `kubectl apply` fails with parsing error.
**Solution**: Validate YAML syntax.
```bash
kubectl apply -f ic-deploy-devops.yaml --dry-run=client
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered**:
The primary challenge was creating a deployment with an init container to write a configuration file to a shared `emptyDir` volume and a main container to read it continuously, ensuring proper volume mounts and command execution.

**💡 Solution Approach**:
1. **Deployment Creation**: Defined `ic-deploy-devops` with one replica and correct labels.
2. **Init Container**: Configured `ic-msg-devops` to write a message to `/ic/beta`.
3. **Main Container**: Configured `ic-main-devops` to read `/ic/beta` every 5 seconds.
4. **Volume Configuration**: Used `emptyDir` for `ic-volume-datacenter`, mounted at `/ic` in both containers.
5. **Verification**: Checked logs to confirm the expected output.

**🎯 Key Success Factors**:
- **Correct YAML**: Ensured proper labels, volume, and container configurations.
- **Init Container**: Successfully wrote the message to the shared volume.
- **Main Container**: Continuously read the message with correct output.
- **Verification**: Confirmed logs show `Init Done - Welcome to xFusionCorp Industries`.
- **Testing**: Validated pod status and volume mounts.

**⚠️ Critical Fixes**:
- Ensure exact command syntax for init and main containers.
- Use `debian:latest` for both containers.
- Mount `ic-volume-devops` at `/ic` in both containers.
- Verify `emptyDir` volume configuration.

**🔒 Kubernetes Best Practices Applied**:
- **Init Containers**: Used for pre-configuration tasks.
- **Shared Volumes**: Leveraged `emptyDir` for temporary data sharing.
- **Minimal Configuration**: Kept YAML simple with required fields.
- **Log Verification**: Used `kubectl logs -f` to confirm functionality.

**⚠️ Important Troubleshooting Concepts**:
- **Init Container Logs**: Check logs for init container failures.
- **Volume Mounts**: Verify mount paths align with volume configuration.
- **Events**: Use `kubectl get events` to diagnose deployment issues.
- **Exec Debugging**: Use `kubectl exec` to inspect files in the volume.

---

## ⚠️ Important Production Notes

🔧 **Deployment Debugging**: Use `kubectl describe`, `kubectl logs`, and `kubectl exec` for thorough issue resolution.
🔐 **Image Security**: Pin specific `debian` versions (e.g., `debian:11`) in production to avoid unexpected updates.
📊 **Resource Limits**: Add `resources` (e.g., CPU/memory limits) to prevent overuse.
🛡️ **Volume Management**: Use persistent volumes (e.g., PVCs) instead of `emptyDir` for production data.

## Best Practices

- **Use init containers for finite prerequisites.** They are suitable for generating configuration, waiting for dependencies, setting permissions, or downloading files before an application starts.
- **Keep initialization separate from the application image.** A dedicated init image can contain setup tools that the runtime image does not need, reducing coupling and the application's attack surface.
- **Make initialization idempotent.** Kubernetes may rerun an init container when a Pod restarts, so its command should remain safe if files or directories already exist.
- **Share initialization results through volumes.** Containers in a Pod do not share their image filesystems, but they can exchange generated files through a commonly mounted volume.
- **Understand `emptyDir` lifetime.** Its contents survive a container restart within the same Pod but disappear when that Pod is deleted or replaced.
- **Do not confuse init containers with sidecars.** A regular init container completes before the application starts; a sidecar continues running alongside the main application.
- **Pin image versions in production.** The challenge requires `fedora:latest`, but immutable or fixed tags provide more predictable production deployments.

### 📚 Official Documentation

- [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
- [Volumes and emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [kubectl logs](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/)