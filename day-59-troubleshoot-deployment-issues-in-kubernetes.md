# Day 59: Troubleshoot Deployment issues in Kubernetes

Last week, the Nautilus DevOps team deployed a redis app on Kubernetes cluster, which was working fine so far. This morning one of the team members was making some changes in this existing setup, but he made some mistakes and the app went down. We need to fix this as soon as possible. Please take a look.

## Specific Requirements:

1. The deployment name is `redis-deployment`. The pods are not in running state right now, so please look into the issue and fix the same.

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

## Solution

The Redis Pod was blocked in `ContainerCreating` because its required ConfigMap volume referenced the misspelled name `redis-cofig`. The Deployment also used the invalid image tag `redis:alpin`. Correcting both values allowed Kubernetes to create a new ReplicaSet and replace the broken Pod with a running one.

---

## 🔹 Step 1: Connect to the Jump Host

```bash
ssh thor@jump_host
```

**Purpose**: Connect to the jump host where `kubectl` is configured to manage the Kubernetes cluster.

---

## 🔹 Step 2: Investigate the Deployment Issue

```bash
kubectl get deployments
kubectl get pods
```

**Purpose**: Check the status of the `redis-deployment` and its pods to confirm the issue.

**Expected Deployment Output**:
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   0/1     1            0           55s
```

**Expected Pod Output**:
```
NAME                                READY   STATUS              RESTARTS   AGE
redis-deployment-795ffcb56c-jx692   0/1     ContainerCreating   0          72s
```

**Observations**:
- Pods are not running (e.g., `ImagePullBackOff` or `CrashLoopBackOff`).
- Likely causes: Incorrect image name (`redis:alpin`) or ConfigMap reference (`redis-cofig`).

---

## 🔹 Step 3: Inspect Deployment Details

```bash
kubectl describe deployment redis-deployment
```

**Purpose**: Identify specific errors related to the image or ConfigMap.

**Expected Output **:
```
Name:                   redis-deployment
Namespace:              default
CreationTimestamp:      Fri, 02 Oct 2026 23:19:41 +0000
Labels:                 app=redis
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=redis
Replicas:               1 desired | 1 updated | 1 total | 0 available | 1 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=redis
  Containers:
   redis-container:
    Image:      redis:alpin
    Port:       6379/TCP
    Host Port:  0/TCP
    Requests:
      cpu:        300m
    Environment:  <none>
    Mounts:
      /redis-master from config (rw)
      /redis-master-data from data (rw)
  Volumes:
   data:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
   config:
    Type:          ConfigMap (a volume populated by a ConfigMap)
    Name:          redis-cofig
    Optional:      false
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  <none>
NewReplicaSet:   redis-deployment-795ffcb56c (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  3m57s  deployment-controller  Scaled up replica set redis-deployment-795ffcb56c from 0 to 1
```

**Findings**:
- **Image Issue**: `redis:alpin` is invalid (typo; should be `redis:alpine`).
- **ConfigMap Issue**: `redis-cofig` is referenced but does not exist (should be `redis-config`).

---

## 🔹 Step 4: Edit the Deployment

```bash
kubectl edit deployment redis-deployment
```

**Purpose**: Modify the deployment to correct the image and ConfigMap names.

**Changes to Make**:
1. **Fix Image Name**:
   - Locate: `image: redis:alpin`
   - Change to: `image: redis:alpine`
2. **Fix ConfigMap Name**:
   - Locate: `configMapRef: name: redis-cofig` (in the volume configuration)
   - Change to: `configMapRef: name: redis-config`

```
Change the container image from:

```yaml
image: redis:alpin
```

to:

```yaml
image: redis:alpine
```

Change the ConfigMap reference from:

```yaml
configMap:
  name: redis-cofig
```

to:

```yaml
configMap:
  name: redis-config
```

**Reference Deployment YAML (for clarity)**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis-container
        image: redis:alpine
        ports:
        - containerPort: 6379
        volumeMounts:
        - name: redis-config-volume
          mountPath: /etc/redis
      volumes:
      - name: redis-config-volume
        configMap:
          name: redis-config
```

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

**Note**: The actual YAML may include additional fields (e.g., resource limits, labels). Ensure only the specified changes (image and ConfigMap name) are made.

---

## 🔹 Step 5: Verify ConfigMap Existence

```bash
kubectl get configmap redis-config
```

**Purpose**: Confirm the `redis-config` ConfigMap exists to avoid mount errors.

**Expected Output**:
```
NAME           DATA   AGE
redis-config   2      16m
```

**If ConfigMap is missing**:
- Create or verify the correct ConfigMap name via:
  ```bash
  kubectl describe configmap
  ```
- Contact the team for the correct ConfigMap configuration if `redis-config` does not exist.

---

## 🔹 Step 6: Verify Deployment and Pod Status

```bash
kubectl get deployments
kubectl get pods
```

**Purpose**: Confirm the deployment is updated and pods are running.

**Expected Deployment Output**:
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   1/1     1            1           16m
```

**Expected Pod Output**:
```
NAME                                READY   STATUS    RESTARTS   AGE
redis-deployment-5476b4ddd6-64lqd   1/1     Running   0          57s
```

**Success Indicators**:
- ✅ Deployment: `1/1` ready
- ✅ Pod status: `Running`
- ✅ Ready: `1/1`
- ✅ No restarts

---

## 🔹 Step 7: Verify Deployment Details

```bash
kubectl describe deployment redis-deployment
```

**Purpose**: Confirm the corrected image and ConfigMap are applied.

**Expected Output (Excerpt)**:
```
Name:                   redis-deployment
Namespace:              default
CreationTimestamp:      Fri, 02 Oct 2026 23:19:41 +0000
Labels:                 app=redis
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=redis
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=redis
  Containers:
   redis-container:
    Image:      redis:alpine
    Port:       6379/TCP
    Host Port:  0/TCP
    Requests:
      cpu:        300m
    Environment:  <none>
    Mounts:
      /redis-master from config (rw)
      /redis-master-data from data (rw)
  Volumes:
   data:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
   config:
    Type:          ConfigMap (a volume populated by a ConfigMap)
    Name:          redis-config
    Optional:      false
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  redis-deployment-795ffcb56c (0/0 replicas created)
NewReplicaSet:   redis-deployment-5476b4ddd6 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  17m   deployment-controller  Scaled up replica set redis-deployment-795ffcb56c from 0 to 1
  Normal  ScalingReplicaSet  103s  deployment-controller  Scaled up replica set redis-deployment-5476b4ddd6 from 0 to 1
  Normal  ScalingReplicaSet  99s   deployment-controller  Scaled down replica set redis-deployment-795ffcb56c from 1 to 0
```

**Success Indicators**:
- ✅ Image: `redis:alpine`
- ✅ ConfigMap: `redis-config`
- ✅ No error events (e.g., `ErrImagePull` or `FailedMount`)

---

## 🔹 Step 8: Test Redis Accessibility (Optional)

```bash
kubectl exec -it $(kubectl get pods -l app=redis -o name | head -n 1) -- redis-cli ping
```

**Purpose**: Verify the Redis container is operational.

**Expected Output**:
```
PONG
```

---

## 🔹 Step 9: Check Redis Logs (Optional)

```bash
kubectl logs -l app=redis
```

**Purpose**: Confirm the Redis container started correctly.

**Expected Logs** (example):
```
1:M 02 Oct 2026 23:35:43.478 * <ReJSON> Exported RedisJSON_V6 API
1:M 02 Oct 2026 23:35:43.478 * <ReJSON> Exported RedisJSON_V7 API
1:M 02 Oct 2026 23:35:43.478 * <ReJSON> Exported RedisJSON_V8 API
1:M 02 Oct 2026 23:35:43.478 * <ReJSON> Enabled diskless replication
1:M 02 Oct 2026 23:35:43.478 * <ReJSON> Initialized shared string cache, thread safe: true.
1:M 02 Oct 2026 23:35:43.478 * Module 'ReJSON' loaded from /usr/local/lib/redis/modules//rejson.so
1:M 02 Oct 2026 23:35:43.478 * <search> Acquired RedisJSON_V8 API
1:M 02 Oct 2026 23:35:43.495 * Server initialized
1:M 02 Oct 2026 23:35:43.495 * Ready to accept connections tcp
1:M 02 Oct 2026 23:35:43.495 # WARNING: Redis does not require authentication and is not protected by network restrictions. Redis will accept connections from any IP address on any network interface.
```

---

## 🔹 Step 10: Clean Up (Optional)

```bash
kubectl delete deployment redis-deployment
```

**Purpose**: Remove the deployment if testing is complete.

**Expected Output**:
```
deployment.apps "redis-deployment" deleted
```

---

## 📋 Quick Command Reference

For quick copy-paste, execute on **Jump Host**:

```bash
# Connect to jump host
ssh thor@jump_host

# Investigate and fix deployment
kubectl get deployments
kubectl get pods
kubectl describe deployment redis-deployment
kubectl edit deployment redis-deployment  # Change image to redis:alpine, configMap to redis-config
kubectl get configmap redis-config
kubectl get deployments
kubectl get pods
kubectl describe deployment redis-deployment
```

---

## 💡 Common Kubernetes Deployment Issues & Fixes

### **Issue 1: Image Pull Failure**
**Problem**: Pods in `ErrImagePull` or `ImagePullBackOff` due to `redis:alpin`.
**Fix**: Correct the image name to `redis:alpine`.
```bash
kubectl edit deployment redis-deployment
docker pull redis:alpine
```

### **Issue 2: ConfigMap Not Found**
**Problem**: Pods fail with `FailedMount` due to `redis-cofig`.
**Fix**: Correct the ConfigMap name to `redis-config`.
```bash
kubectl get configmap
kubectl edit deployment redis-deployment
```

### **Issue 3: Pods Not Running**
**Problem**: Pods in `CrashLoopBackOff` or `Error`.
**Fix**: Check logs and events for errors.
```bash
kubectl logs -l app=redis
kubectl describe pod -l app=redis
```

### **Issue 4: Deployment Not Updating**
**Problem**: Pods do not reflect changes after `kubectl edit`.
**Fix**: Restart the deployment.
```bash
kubectl rollout restart deployment redis-deployment
```

---

## 🔧 Troubleshooting Deployment Failures

### **Error: Pod Not Running**
**Symptoms**: `kubectl get pods` shows `Error` or `CrashLoopBackOff`.
**Solution**: Check logs and events.
```bash
kubectl logs -l app=redis
kubectl get events --field-selector involvedObject.name=redis-deployment
```

### **Error: Image Not Found**
**Symptoms**: "Failed to pull image redis:alpin".
**Solution**: Ensure correct image name and registry access.
```bash
kubectl describe pod -l app=redis
```

### **Error: ConfigMap Mount Failure**
**Symptoms**: "Unable to attach or mount volumes: configmap redis-cofig not found".
**Solution**: Verify ConfigMap exists and name is correct.
```bash
kubectl get configmap redis-config
kubectl describe deployment redis-deployment
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered**:
The primary challenge was identifying and fixing two errors in the `redis-deployment`: an incorrect image name (`redis:alpin`) and an invalid ConfigMap name (`redis-cofig`), which caused the pods to fail.

**💡 Solution Approach**:
1. **Issue Identification**: Used `kubectl describe` to confirm image and ConfigMap errors.
2. **Deployment Correction**: Edited the deployment to fix `image: redis:alpine` and `configMap: redis-config`.
3. **ConfigMap Verification**: Checked for `redis-config` existence.
4. **Deployment Update**: Applied changes via `kubectl edit`.
5. **Verification**: Confirmed pods are running with `kubectl get pods` and `kubectl describe`.

**🎯 Key Success Factors**:
- **Accurate Diagnosis**: Identified typos in image and ConfigMap names.
- **Correct Fixes**: Updated `redis:alpine` and `redis-config` in the deployment.
- **Verification**: Ensured pods reached `Running` state.
- **Testing**: Optionally validated Redis functionality with `redis-cli`.

**⚠️ Critical Fixes**:
- Correct image name from `redis:alpin` to `redis:alpine`.
- Correct ConfigMap name from `redis-cofig` to `redis-config`.
- Verify ConfigMap existence before applying changes.
- Monitor pod status post-update.

**🔒 Kubernetes Best Practices Applied**:
- **Troubleshooting**: Used `kubectl describe` and `kubectl logs` for systematic diagnosis.
- **Minimal Changes**: Modified only the necessary fields (image, ConfigMap).
- **Validation**: Confirmed ConfigMap and pod status.
- **Restart Strategy**: Leveraged `kubectl edit` for quick fixes.

**⚠️ Important Troubleshooting Concepts**:
- **Events**: Use `kubectl get events` to diagnose deployment issues.
- **Logs**: Check pod logs for runtime errors.
- **Describe**: Use `kubectl describe` for detailed configuration checks.
- **ConfigMap Verification**: Ensure referenced ConfigMaps exist.

---

## ⚠️ Important Production Notes

🔧 **Deployment Debugging**: Use `kubectl describe`, `kubectl logs`, and `kubectl exec` for thorough issue resolution.
🔐 **Image Security**: Pin specific `redis` versions (e.g., `redis:7.0-alpine`) in production to avoid unexpected updates.
📊 **Resource Limits**: Add `resources` (e.g., CPU/memory limits) to prevent overuse.
🛡️ **Service Exposure**: Configure a service (e.g., ClusterIP) for Redis access in production.

## Best Practices

- **Read Pod events before changing resources.** Events such as `FailedMount` usually identify the immediate reason a Pod cannot start.
- **Verify referenced resources by name.** ConfigMap and Secret references are exact and case-sensitive, so confirm that the resource exists before editing the workload.
- **Inspect the complete Pod template.** Fixing only the first visible error can expose another one later; review the image, volumes, commands, and other dependencies together.
- **Edit the controller instead of its Pods.** Deployment-managed Pods are replaceable. Correct the Deployment so every replacement Pod receives the valid configuration.
- **Verify both rollout and runtime state.** A successful rollout plus a `Running` and ready Pod confirms that Kubernetes accepted the update and the workload recovered.

### 📚 Official Documentation

- [Debug Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/)
- [Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [kubectl describe](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_describe/)
- [kubectl edit](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_edit/)
- [kubectl rollout status](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_rollout/kubectl_rollout_status/)
