# Day 57: Print Environment Variables

The Nautilus DevOps team is working on to setup some pre-requisites for an application that will send the greetings to different users. There is a sample deployment, that needs to be tested. Below is a scenario which needs to be configured on Kubernetes cluster. Please find below more details about it.

## Specific Requirements:

1. Create a `pod` named `print-envars-greeting`.
2. Configure spec as, the container name should be `print-env-container` and use `bash` image.
3. Create three environment variables:

   a. `GREETING` and its value should be `Welcome to`

   b. `COMPANY` and its value should be `xFusionCorp`

   c. `GROUP` and its value should be `Group`
4. Use command `["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']` (please use this exact command), also set its `restartPolicy` policy to `Never` to avoid crash loop back.
5. You can check the output using `kubectl logs -f print-envars-greeting` command.

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

## Solution

Kubernetes can inject environment variables directly into a container through the `env` field. References written as `$(VARIABLE)` inside a container's `command` or `args` are expanded from those configured values before the process starts. The command in this challenge prints one greeting and exits, so `restartPolicy: Never` allows the Pod to remain successfully completed instead of restarting the container.

---

## 🔹 Step 1: Connect to the Jump Host

```bash
ssh thor@jump_host
```

**Purpose**: Connect to the jump host where `kubectl` is configured to manage the Kubernetes cluster.

---

## 🔹 Step 2: Create the Pod YAML File

```bash
vi print-envars-greeting.yaml
```

**Purpose**: Create a Kubernetes YAML file to define the `print-envars-greeting` pod with environment variables and the specified command.

---

## 🔹 Step 3: Add Pod YAML Content

**Pod YAML**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: print-envars-greeting
spec:
  restartPolicy: Never
  containers:
  - name: print-env-container
    image: bash:latest
    env:
      - name: GREETING
        value: "Welcome to"
      - name: COMPANY
        value: "xFusionCorp"
      - name: GROUP
        value: "Group"
    command: ["/bin/sh", "-c", "echo \"$(GREETING) $(COMPANY) $(GROUP)\""]
```

**Key Configurations**:
- **apiVersion: v1**: Specifies the Kubernetes API version for pods.
- **kind: Pod**: Defines the resource type as a pod.
- **metadata.name: print-envars-greeting**: Sets the pod name.
- **spec.restartPolicy: Never**: Ensures the pod does not restart after the command completes.
- **spec.containers**: Defines a single container:
  - `name: print-env-container`: Sets the container name.
  - `image: bash:latest`: Uses the `bash:latest` image (explicit tag added for clarity, as task implies latest).
  - `env`: Defines three environment variables:
    - `GREETING: "Welcome to"`: First part of the greeting.
    - `COMPANY: "xFusionCorp"`: Company name.
    - `GROUP: "Group"`: Group name.
  - `command: ["/bin/sh", "-c", "echo \"$(GREETING) $(COMPANY) $(GROUP)\"]`: Prints the concatenated environment variables.
- **Note**: The command uses double quotes around the `echo` string to ensure proper variable substitution.

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

## 🔹 Step 4: Verify Pod YAML Syntax

```bash
cat print-envars-greeting.yaml
```

**Purpose**: Confirm the YAML file contains the correct configuration.

**Validation Checklist**:
- ✅ `apiVersion: v1` and `kind: Pod`
- ✅ Pod name: `print-envars-greeting`
- ✅ Container name: `print-env-container`
- ✅ Image: `bash:latest`
- ✅ Environment variables: `GREETING`, `COMPANY`, `GROUP` with correct values
- ✅ Command: `["/bin/sh", "-c", "echo \"$(GREETING) $(COMPANY) $(GROUP)\""]`
- ✅ `restartPolicy: Never`
- ✅ Correct YAML indentation (2 spaces)

---

## 🔹 Step 5: Apply the Pod Configuration

```bash
kubectl apply -f print-envars-greeting.yaml
```

**Purpose**: Deploy the pod to the Kubernetes cluster.

**Expected Output**:
```
pod/print-envars-greeting created
```

---

## 🔹 Step 6: Verify Pod Status

```bash
kubectl get pods
```

**Purpose**: Confirm the `print-envars-greeting` pod is in `Completed` state (since `restartPolicy: Never` and the command is one-time).

**Expected Output**:
```
NAME                    READY   STATUS      RESTARTS   AGE
print-envars-greeting   0/1     Completed   0          16s
```

**Success Indicators**:
- ✅ Pod name: `print-envars-greeting`
- ✅ Status: `Completed` (command executed and exited)
- ✅ Ready: `0/1` (expected, as the container has finished)
- ✅ No restarts

---

## 🔹 Step 7: Verify Pod Output

```bash
kubectl logs -f print-envars-greeting
```

**Purpose**: Check the pod’s logs to confirm the environment variables were printed correctly.

**Expected Output**:
```
Welcome to xFusionCorp Group
```

**Success Indicators**:
- ✅ Exact output matches: `Welcome to xFusionCorp Group`
- ✅ No errors in the logs

---

## 🔹 Step 8: Verify Pod Details

```bash
kubectl describe pod print-envars-greeting
```

**Purpose**: Check detailed information about the pod, including environment variables and command execution.

**Expected Output (Excerpt)**:
```
Name:             print-envars-greeting
Namespace:        default
Priority:         0
Service Account:  default
Node:             jump-host/10.244.195.209
Start Time:       Fri, 02 Oct 2026 21:52:59 +0000
Labels:           <none>
Annotations:      <none>
Status:           Succeeded
IP:               10.22.0.9
IPs:
  IP:  10.22.0.9
Containers:
  print-env-container:
    Container ID:  containerd://739400b62c14e72c20ff7af039b05f5704e325920d8bb7b5a02bddac6f284cd9
    Image:         bash:latest
    Image ID:      docker.io/library/bash@sha256:61962062d969cb46dfc2bad061d36342406fa485f64f246aa7e95693ca07df1f
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
      -c
      echo "$(GREETING) $(COMPANY) $(GROUP)"
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Fri, 02 Oct 2026 21:53:00 +0000
      Finished:     Fri, 02 Oct 2026 21:53:00 +0000
    Ready:          False
    Restart Count:  0
    Environment:
      GREETING:  Welcome to
      COMPANY:   xFusionCorp
      GROUP:     Group
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-gqrwv (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   False 
  Initialized                 True 
  Ready                       False 
  ContainersReady             False 
  PodScheduled                True 
Volumes:
  kube-api-access-gqrwv:
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
  Normal  Scheduled  75s   default-scheduler  Successfully assigned default/print-envars-greeting to jump-host
  Normal  Pulling    75s   kubelet            Pulling image "bash:latest"
  Normal  Pulled     74s   kubelet            Successfully pulled image "bash:latest" in 1.016s (1.016s including waiting). Image size: 6815946 bytes.
  Normal  Created    74s   kubelet            Created container: print-env-container
  Normal  Started    74s   kubelet            Started container print-env-container
```

**Success Indicators**:
- ✅ Correct container name and image
- ✅ Environment variables correctly set
- ✅ Command matches the specified format
- ✅ State: `Terminated` with `Reason: Completed`

---

## 🔹 Step 9: Clean Up (Optional)

```bash
kubectl delete -f print-envars-greeting.yaml
```

**Purpose**: Remove the pod if testing is complete.

**Expected Output**:
```
pod "print-envars-greeting" deleted
```

---

## 📋 Quick Command Reference

For quick copy-paste, execute on **Jump Host**:

```bash
# Connect to jump host
ssh thor@jump_host

# Create pod YAML
cat > print-envars-greeting.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: print-envars-greeting
spec:
  restartPolicy: Never
  containers:
  - name: print-env-container
    image: bash:latest
    env:
      - name: GREETING
        value: "Welcome to"
      - name: COMPANY
        value: "xFusionCorp"
      - name: GROUP
        value: "Group"
    command: ["/bin/sh", "-c", "echo \"$(GREETING) $(COMPANY) $(GROUP)\""]
EOF

# Deploy and verify
kubectl apply -f print-envars-greeting.yaml
kubectl get pods
kubectl logs -f print-envars-greeting
kubectl describe pod print-envars-greeting
```

---

## 💡 Common Kubernetes Pod Issues & Fixes

### **Issue 1: Pod in Error State**
**Problem**: `kubectl get pods` shows `Error` instead of `Completed`.
**Fix**: Check logs and events for command or image issues.
```bash
kubectl logs print-envars-greeting
kubectl describe pod print-envars-greeting
```

### **Issue 2: Image Pull Failure**
**Problem**: Pod in `ErrImagePull` or `ImagePullBackOff`.
**Fix**: Verify image name and tag.
```bash
docker pull bash:latest
kubectl delete pod print-envars-greeting
kubectl apply -f print-envars-greeting.yaml
```

### **Issue 3: Incorrect Output**
**Problem**: `kubectl logs` shows wrong or no output.
**Fix**: Verify environment variables and command syntax.
```bash
kubectl describe pod print-envars-greeting
kubectl edit pod print-envars-greeting
```

### **Issue 4: Pod Stuck in Pending**
**Problem**: `kubectl get pods` shows `Pending` status.
**Fix**: Check events for scheduling issues.
```bash
kubectl describe pod print-envars-greeting
```

---

## 🔧 Troubleshooting Pod Failures

### **Error: Command Execution Fails**
**Symptoms**: Logs show errors or no output.
**Solution**: Verify command syntax and environment variables.
```bash
kubectl logs print-envars-greeting
kubectl exec -it print-envars-greeting -- env
```

### **Error: Image Not Found**
**Symptoms**: "Failed to pull image bash:latest".
**Solution**: Ensure image exists and cluster has registry access.
```bash
kubectl describe pod print-envars-greeting
```

### **Error: YAML Syntax Error**
**Symptoms**: `kubectl apply` fails with parsing error.
**Solution**: Validate YAML syntax.
```bash
kubectl apply -f print-envars-greeting.yaml --dry-run=client
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered**:
The primary challenge was creating a pod with specific environment variables, a precise command to print them, and a `Never` restart policy to ensure the pod completes without crashing, while verifying the correct output.

**💡 Solution Approach**:
1. **YAML Creation**: Defined `print-envars-greeting.yaml` with the correct container, environment variables, command, and restart policy.
2. **Environment Variables**: Set `GREETING`, `COMPANY`, and `GROUP` as specified.
3. **Command**: Used the exact command `["/bin/sh", "-c", "echo \"$(GREETING) $(COMPANY) $(GROUP)\""]`.
4. **Pod Deployment**: Applied the pod using `kubectl apply`.
5. **Verification**: Checked pod status (`Completed`) and logs (`Welcome to xFusionCorp Group`).

**🎯 Key Success Factors**:
- **Correct YAML Structure**: Ensured proper `apiVersion: v1`, container, and environment variable specifications.
- **Accurate Environment Variables**: Used exact values (`Welcome to`, `xFusionCorp`, `Group`).
- **Command Syntax**: Maintained precise command formatting for correct output.
- **Restart Policy**: Set `Never` to avoid crash loops.
- **Verification**: Confirmed output with `kubectl logs`.

**⚠️ Critical Fixes**:
- Correct environment variable values (`COMPANY: xFusionCorp`, `GROUP: Group`).
- Use exact command with proper quoting for variable substitution.
- Ensure `restartPolicy: Never` to prevent looping.
- Validate `bash:latest` image availability.

**🔒 Kubernetes Best Practices Applied**:
- **Minimal Configuration**: Kept YAML simple with only required fields.
- **Environment Variables**: Defined variables clearly for clarity and reuse.
- **Dry Run Testing**: Validated YAML with `--dry-run=client`.
- **Log Verification**: Used `kubectl logs` to confirm output.

**⚠️ Important Troubleshooting Concepts**:
- **Logs**: Check `kubectl logs` for command output issues.
- **Events**: Use `kubectl get events` to diagnose pod issues.
- **Describe**: Use `kubectl describe` for detailed pod status.
- **Exec Debugging**: Use `kubectl exec` to inspect environment variables if needed.

---

## ⚠️ Important Production Notes

🔧 **Pod Debugging**: Use `kubectl describe` and `kubectl logs` for thorough issue resolution.
🔐 **Image Security**: Pin specific `bash` versions (e.g., `bash:5.1`) in production to avoid unexpected updates.
📊 **Resource Limits**: Add `resources` (e.g., CPU/memory limits) to prevent overuse in production.
🛡️ **Restart Policy**: Use `Never` only for one-time tasks; consider `OnFailure` for retryable jobs in production.

## Best Practices

- **Use `Never` for one-time commands.** A container that performs a finite task should not be restarted after successful completion unless retry behavior is explicitly required.
- **Do not mistake `Completed` for a failure.** For short-lived Pods, `Completed` indicates that the command exited successfully; a persistent `Running` state is not required.
- **Use ConfigMaps for reusable non-sensitive configuration.** Direct `env` values are clear for this small lab, while ConfigMaps are easier to manage when several workloads share application settings.
- **Use Secrets for confidential values.** Environment variables that contain passwords, tokens, or keys should reference Kubernetes Secrets instead of storing plaintext values in a Pod manifest.
- **Remember the Kubernetes expansion syntax.** References in `command` and `args` use `$(VARIABLE)`, which differs from the usual shell form `$VARIABLE`.

### 📚 Official Documentation

- [Define Environment Variables for a Container](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)
- [Define a Command and Arguments for a Container](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/)
- [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)