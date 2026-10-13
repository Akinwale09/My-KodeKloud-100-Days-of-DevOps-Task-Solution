# Day 48: Deploy Pods in Kubernetes Cluster

The Nautilus DevOps team is diving into Kubernetes for application management. One team member has a task to create a pod according to the details below:

1. Create a pod named `pod-httpd` using the `httpd` image with the `latest` tag. Ensure to specify the tag as `httpd:latest`.
2. Set the `app` label to `httpd_app`, and name the container as `httpd-container`.

**Note**: The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

## Specific Requirements:

1. Create a pod named `pod-httpd` using the `httpd` image with the `latest` tag. Ensure to specify the tag as `httpd:latest`.
2. Set the `app` label to `httpd_app`, and name the container as `httpd-container`.

## Solution

A Pod manifest declares the required Kubernetes object, its label, and its container configuration in one versionable file. The preconfigured `kubectl` context on the jump host applies that manifest to the cluster.

---

## 🔹 Step 1: Connect to the Jump Host

```bash
ssh thor@jump_host
```

**Purpose**: Connect to the jump host where `kubectl` is configured to manage the Kubernetes cluster.

---

## 🔹 Step 2: Create the Pod YAML File

```bash
vi pod-httpd.yaml
```

**Purpose**: Create a Kubernetes YAML file to define the `pod-httpd` pod.

---

## 🔹 Step 3: Add Pod YAML Content

**Pod YAML**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: httpd_app
spec:
  containers:
    - name: httpd-container
      image: httpd:latest
```

**Key Configurations**:
- **apiVersion: v1**: Specifies the Kubernetes API version for pods.
- **kind: Pod**: Defines the resource type as a pod.
- **metadata.name: pod-httpd**: Sets the pod name to `pod-httpd`.
- **metadata.labels**: Applies the label `app: httpd_app` for identification.
- **spec.containers**: Defines a single container:
  - `name: httpd-container`: Sets the container name.
  - `image: httpd:latest`: Uses the `httpd:latest` image from Docker Hub.

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

## 🔹 Step 4: Verify Pod YAML Syntax

```bash
cat pod-httpd.yaml
```

**Purpose**: Confirm the YAML file contains the correct configuration.

**Validation Checklist**:
- ✅ `apiVersion: v1` and `kind: Pod`
- ✅ Pod name: `pod-httpd`
- ✅ Label: `app: httpd_app`
- ✅ Container name: `httpd-container`
- ✅ Image: `httpd:latest`
- ✅ Correct YAML indentation (2 spaces)

---

## 🔹 Step 5: Apply the Pod Configuration

```bash
kubectl apply -f pod-httpd.yaml
```

**Purpose**: Deploy the pod to the Kubernetes cluster.

**Expected Output**:
```
pod/pod-httpd created
```

---

## 🔹 Step 6: Verify Pod Creation

```bash
kubectl get pods
```

**Purpose**: Confirm the `pod-httpd` pod is running.

**Expected Output**:
```
NAME        READY   STATUS    RESTARTS   AGE
pod-httpd   1/1     Running   0          18s
```
**Success Indicators**:
- ✅ Pod name: `pod-httpd`
- ✅ Status: `Running`
- ✅ Ready: `1/1`
- ✅ No restarts

---

## 🔹 Step 7: Verify Pod Details

```bash
kubectl describe pod pod-httpd
```

**Purpose**: Check detailed information about the pod, including labels and container status.

**Expected Output (Excerpt)**:
```
Name:             pod-httpd
Namespace:        default
Priority:         0
Service Account:  default
Node:             jump-host/10.244.248.53
Start Time:       Tue, 22 Sep 2026 15:34:55 +0000
Labels:           app=httpd_app
Annotations:      <none>
Status:           Running
IP:               10.22.0.9
```
---

## 🔹 Step 8: Test Pod Accessibility (Optional)

```bash
kubectl port-forward pod/pod-httpd 8080:80
```

**Purpose**: Forward the pod’s port 80 to localhost:8080 for testing.

**In a new terminal**:
```bash
curl http://localhost:8080
```

**Expected Output**: Default Apache HTTPD page (e.g., `<html><body><h1>It works!</h1></body></html>`).

---

## 🔹 Step 9: Check Container Logs

```bash
kubectl logs pod-httpd
```

**Purpose**: Verify the HTTPD container started correctly.

**Expected Logs**:
```
AH00558: httpd: Could not reliably determine the server's fully qualified domain name...
[Sat Oct 25 15:42:00.000000 2025] [mpm_event:notice] [pid 1] AH00489: Apache/2.4.57 (Unix) configured
```

---

## 🔹 Step 10: Clean Up (Optional)

```bash
kubectl delete -f pod-httpd.yaml
```

**Purpose**: Remove the pod if testing is complete.

**Expected Output**:
```
pod "pod-httpd" deleted
```

---

## 📋 Quick Command Reference

For quick copy-paste, execute on **Jump Host**:

```bash
# Connect to jump host
ssh thor@jump_host

# Create pod YAML
cat > pod-httpd.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: httpd_app
spec:
  containers:
    - name: httpd-container
      image: httpd:latest
EOF

# Deploy and verify
kubectl apply -f pod-httpd.yaml
kubectl get pods
kubectl describe pod pod-httpd
```

---

## 💡 Common Kubernetes Pod Issues & Fixes

### **Issue 1: Pod Stuck in Pending**
**Problem**: `kubectl get pods` shows `Pending` status.
**Fix**: Check events for scheduling issues.
```bash
kubectl describe pod pod-httpd
```
**Common Causes**: Insufficient resources or node issues.

### **Issue 2: Image Pull Failure**
**Problem**: Pod in `ErrImagePull` or `ImagePullBackOff`.
**Fix**: Verify image name and tag.
```bash
docker pull httpd:latest
kubectl delete pod pod-httpd
kubectl apply -f pod-httpd.yaml
```

### **Issue 3: Container CrashLoopBackOff**
**Problem**: Pod restarts repeatedly; `STATUS: CrashLoopBackOff`.
**Fix**: Check logs for errors.
```bash
kubectl logs pod-httpd
kubectl describe pod pod-httpd
```

### **Issue 4: Incorrect Labels**
**Problem**: Label `app: httpd_app` not applied correctly.
**Fix**: Verify YAML and reapply.
```bash
kubectl get pods -l app=httpd_app
kubectl edit pod pod-httpd
```

---

## 🔧 Troubleshooting Pod Failures

### **Error: Pod Not Running**
**Symptoms**: `kubectl get pods` shows `Error` or `CrashLoopBackOff`.
**Solution**: Check logs and events.
```bash
kubectl logs pod-httpd
kubectl get events --field-selector involvedObject.name=pod-httpd
```

### **Error: Image Not Found**
**Symptoms**: "Failed to pull image httpd:latest".
**Solution**: Ensure image exists and cluster has registry access.
```bash
kubectl describe pod pod-httpd
```

### **Error: YAML Syntax Error**
**Symptoms**: `kubectl apply` fails with parsing error.
**Solution**: Validate YAML syntax.
```bash
kubectl apply -f pod-httpd.yaml --dry-run=client
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered**:
The primary challenge was creating a Kubernetes pod with precise specifications for the name, image, container name, and labels, ensuring it runs successfully in the cluster.

**💡 Solution Approach**:
1. **YAML Creation**: Defined `pod-httpd.yaml` with correct `apiVersion`, `kind`, `metadata`, and `spec`.
2. **Label Application**: Added `app: httpd_app` label for identification.
3. **Image Specification**: Used `httpd:latest` as required.
4. **Deployment**: Applied the YAML using `kubectl apply`.
5. **Verification**: Checked pod status with `kubectl get pods` and `kubectl describe`.

**🎯 Key Success Factors**:
- **Correct YAML Structure**: Ensured proper `apiVersion: v1` and pod specifications.
- **Accurate Naming**: Used `pod-httpd` and `httpd-container` as specified.
- **Label Accuracy**: Applied `app: httpd_app` correctly.
- **Image Validation**: Confirmed `httpd:latest` availability.
- **Testing**: Verified pod status and accessibility.

**⚠️ Critical Pod Fixes**:
- Use `apiVersion: v1` for pod resources.
- Ensure exact names for pod (`pod-httpd`) and container (`httpd-container`).
- Validate `httpd:latest` image tag.
- Check YAML indentation (2 spaces).

**🔒 Kubernetes Best Practices Applied**:
- **Clear Labeling**: Used descriptive labels for pod identification.
- **Minimal Configuration**: Kept YAML simple with only required fields.
- **Dry Run Testing**: Used `--dry-run=client` to validate YAML.
- **Log Checking**: Verified container logs for startup confirmation.

**⚠️ Important Troubleshooting Concepts**:
- **Events**: Use `kubectl get events` to diagnose scheduling issues.
- **Logs**: Check container logs for runtime errors.
- **Describe**: Use `kubectl describe` for detailed pod status.
- **Dry Run**: Validate YAML before applying.

---

## ⚠️ Important Production Notes

🔧 **Pod Debugging**: Use `kubectl describe` and `kubectl logs` to diagnose issues systematically.
🔐 **Image Security**: Verify `httpd:latest` is from a trusted source; pin specific versions in production.
📊 **Resource Limits**: Add `resources` (e.g., CPU/memory limits) for production pods.
🛡️ **Networking**: Expose pods via services (e.g., NodePort) for production access.

### 📚 Official Documentation

- [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
- [kubectl apply](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_apply/)
- [kubectl get](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_get/)
