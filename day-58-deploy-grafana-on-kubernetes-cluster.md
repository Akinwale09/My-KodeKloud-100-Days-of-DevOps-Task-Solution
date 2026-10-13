# Day 58: Deploy Grafana on Kubernetes Cluster

The Nautilus DevOps teams is planning to set up a Grafana tool to collect and analyze analytics from some applications. They are planning to deploy it on Kubernetes cluster. Below you can find more details.

## Specific Requirements:

1. Create a deployment named `grafana-deployment-datacenter` using any grafana image for Grafana app. Set other parameters as per your choice.
   
2. Create `NodePort` type service with nodePort `32000` to expose the app.

`You do not need to make any configuration changes inside the Grafana app once deployed; just make sure you can access the Grafana login page.`

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

## Solution

Grafana listens for HTTP traffic on container port `3000` by default. A Kubernetes NodePort Service makes that internal port reachable through port `32000` on the cluster node. The Deployment, Pod template, and Service must use matching labels so that the Service can discover the Grafana Pod.

---

## 🔹 Step 1: Connect to the Jump Host

```bash
ssh thor@jump_host
```

**Purpose**: Connect to the jump host where `kubectl` is configured to manage the Kubernetes cluster.

---

## 🔹 Step 2: Create the Deployment YAML File

```bash
vi grafana-deployment.yaml
```

**Purpose**: Create a Kubernetes YAML file to define the `grafana-deployment-nautilus` deployment.

---

## 🔹 Step 3: Add Deployment YAML Content

**Deployment YAML**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana-deployment-datacenter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana-container
        image: grafana/grafana:latest
        ports:
        - containerPort: 3000
```

**Key Configurations**:
- **apiVersion: apps/v1**: Specifies the API version for deployments.
- **kind: Deployment**: Defines the resource type as a deployment.
- **metadata.name: grafana-deployment-datacenter**: Sets the deployment name.
- **spec.replicas: 1**: Runs one pod instance.
- **spec.selector.matchLabels: app: grafana**: Matches pods with the label `app: grafana`.
- **spec.template.metadata.labels: app: grafana**: Applies the label to pods.
- **spec.template.spec.containers**: Defines a single container:
  - `name: grafana-container`: Sets the container name.
  - `image: grafana/grafana:latest`: Uses the official Grafana image with the latest tag.
  - `ports.containerPort: 3000`: Exposes Grafana’s default port (3000).

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

## 🔹 Step 4: Create the Service YAML File

```bash
vi grafana-service.yaml
```

**Purpose**: Create a Kubernetes YAML file to define a NodePort service to expose the Grafana application.

---

## 🔹 Step 5: Add Service YAML Content

**Service YAML**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: grafana-service
spec:
  type: NodePort
  selector:
    app: grafana
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
      nodePort: 32000
```

**Key Configurations**:
- **apiVersion: v1**: Specifies the API version for services.
- **kind: Service**: Defines the resource type as a service.
- **metadata.name: grafana-service**: Sets the service name.
- **spec.type: NodePort**: Exposes the service on a specific port of each node.
- **spec.selector: app: grafana**: Targets pods with the label `app: grafana`.
- **spec.ports**: Defines port mappings:
  - `protocol: TCP`: Uses TCP protocol.
  - `port: 3000`: Service port for internal cluster access.
  - `targetPort: 3000`: Maps to the container’s port 3000.
  - `nodePort: 32000`: Exposes the service on port 32000 of each node.

**Save and Exit**: Press `Esc`, type `:wq`, press `Enter`.

---

## 🔹 Step 6: Verify YAML Syntax

```bash
cat grafana-deployment.yaml
cat grafana-service.yaml
```

**Purpose**: Confirm both YAML files contain the correct configurations.

**Deployment Validation Checklist**:
- ✅ `apiVersion: apps/v1` and `kind: Deployment`
- ✅ Deployment name: `grafana-deployment-datacenter`
- ✅ Label: `app: grafana` in both `selector` and `template`
- ✅ Container name: `grafana-container`
- ✅ Image: `grafana/grafana:latest`
- ✅ Container port: `3000`
- ✅ Correct YAML indentation (2 spaces)

**Service Validation Checklist**:
- ✅ `apiVersion: v1` and `kind: Service`
- ✅ Service name: `grafana-service`
- ✅ Type: `NodePort`
- ✅ Selector: `app: grafana`
- ✅ Ports: `port: 3000`, `targetPort: 3000`, `nodePort: 32000`
- ✅ Correct YAML indentation (2 spaces)

---

## 🔹 Step 7: Apply the Configurations

```bash
kubectl apply -f grafana-deployment.yaml
kubectl apply -f grafana-service.yaml
```

**Purpose**: Deploy the Grafana deployment and NodePort service to the Kubernetes cluster.

**Expected Output**:
```
deployment.apps/grafana-deployment-nautilus created
service/grafana-service created
```

---

## 🔹 Step 8: Verify Deployment and Pods

```bash
kubectl get deployments
kubectl get pods
```

**Purpose**: Confirm the deployment is running and pods are operational.

**Expected Deployment Output**:
```
NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
grafana-deployment-datacenter   1/1     1            1           34s
```

**Expected Pod Output**:
```
NAME                                             READY   STATUS    RESTARTS   AGE
grafana-deployment-datacenter-7c9f688d84-bgtwr   1/1     Running   0          72s
```

**Success Indicators**:
- ✅ Deployment name: `grafana-deployment-nautilus`
- ✅ Pod status: `Running`
- ✅ Ready: `1/1` for both deployment and pod
- ✅ No restarts

---

## 🔹 Step 9: Verify Service

```bash
kubectl get svc
```

**Purpose**: Confirm the NodePort service is created and exposes port 32000.

**Expected Output**:
```
NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
grafana-service   NodePort    10.43.24.215   <none>        3000:32000/TCP   83s
kubernetes        ClusterIP   10.43.0.1      <none>        443/TCP          97m
```

**Success Indicators**:
- ✅ Service name: `grafana-service`
- ✅ Type: `NodePort`
- ✅ Port mapping: `3000:32000/TCP`

---

## 🔹 Step 10: Access Grafana Login Page

**Action**: In the KodeKloud lab interface, click the **Grafana** button to access the Grafana login page.

**Purpose**: Verify the Grafana application is accessible via the NodePort (32000).

**Expected Output** (via browser):
- The Grafana login page appears, typically prompting for a username and password (default: `admin/admin`).

**Alternative CLI Test** (if no Grafana button is available):
```bash
kubectl get nodes -o wide
```

**Note the node’s external IP** (e.g., `192.168.1.100`), then:
```bash
curl http://<node-ip>:32000
```

**Expected Output**:
- HTML content of the Grafana login page or a redirect to `/login`.

---

## 🔹 Step 11: Verify Pod Details (Optional)

```bash
kubectl describe pod -l app=grafana
```

**Purpose**: Check detailed information about the pod, including container status and image.

**Expected Output (Excerpt)**:
```
Name:         grafana-deployment-nautilus-xyz123456-abc789
Namespace:    default
Containers:
  grafana-container:
    Image:        grafana/grafana:latest
    Port:         3000/TCP
    State:        Running
```

---

## 🔹 Step 12: Check Grafana Logs (Optional)

```bash
kubectl logs -l app=grafana
```

**Purpose**: Verify the Grafana container started correctly.

**Expected Logs** (example):
```
t=2025-10-25T15:50:00+0000 lvl=info msg="Starting Grafana" version=10.0.0
t=2025-10-25T15:50:00+0000 lvl=info msg="HTTP Server Listen" address=0.0.0.0:3000
```

---

## 🔹 Step 13: Clean Up (Optional)

```bash
kubectl delete -f grafana-deployment.yaml
kubectl delete -f grafana-service.yaml
```

**Purpose**: Remove the deployment and service if testing is complete.

**Expected Output**:
```
deployment.apps "grafana-deployment-nautilus" deleted
service "grafana-service" deleted
```

---

## 📋 Quick Command Reference

For quick copy-paste, execute on **Jump Host**:

```bash
# Connect to jump host
ssh thor@jump_host

# Create deployment YAML
cat > grafana-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana-deployment-datacenter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana-container
        image: grafana/grafana:latest
        ports:
        - containerPort: 3000
EOF

# Create service YAML
cat > grafana-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: grafana-service
spec:
  type: NodePort
  selector:
    app: grafana
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
      nodePort: 32000
EOF

# Deploy and verify
kubectl apply -f grafana-deployment.yaml
kubectl apply -f grafana-service.yaml
kubectl get deployments
kubectl get pods
kubectl get svc
```

---

## 💡 Common Kubernetes Deployment/Service Issues & Fixes

### **Issue 1: Pods Not Running**
**Problem**: `kubectl get pods` shows `Error` or `CrashLoopBackOff`.
**Fix**: Check logs and events.
```bash
kubectl logs -l app=grafana
kubectl describe pod -l app=grafana
```

### **Issue 2: Image Pull Failure**
**Problem**: Pods in `ErrImagePull` or `ImagePullBackOff`.
**Fix**: Verify image name and tag.
```bash
docker pull grafana/grafana:latest
kubectl rollout restart deployment grafana-deployment-nautilus
```

### **Issue 3: Service Not Accessible**
**Problem**: Grafana login page not accessible via nodePort 32000.
**Fix**: Verify service configuration and node IP.
```bash
kubectl describe svc grafana-service
kubectl get nodes -o wide
curl http://<node-ip>:32000
```

### **Issue 4: Incorrect Selector Labels**
**Problem**: Service does not route to pods.
**Fix**: Ensure deployment and service selectors match.
```bash
kubectl get deployment grafana-deployment-nautilus -o yaml | grep selector
kubectl get svc grafana-service -o yaml | grep selector
```

---

## 🔧 Troubleshooting Deployment/Service Failures

### **Error: Pod Not Running**
**Symptoms**: `kubectl get pods` shows `Error` or `CrashLoopBackOff`.
**Solution**: Check logs and events.
```bash
kubectl logs -l app=grafana
kubectl get events --field-selector involvedObject.name=grafana-deployment-nautilus
```

### **Error: Service Not Routing**
**Symptoms**: Accessing `<node-ip>:32000` fails.
**Solution**: Verify service port mappings and pod selector.
```bash
kubectl describe svc grafana-service
kubectl get endpoints grafana-service
```

### **Error: YAML Syntax Error**
**Symptoms**: `kubectl apply` fails with parsing error.
**Solution**: Validate YAML syntax.
```bash
kubectl apply -f grafana-deployment.yaml --dry-run=client
kubectl apply -f grafana-service.yaml --dry-run=client
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered**:
The primary challenge was deploying Grafana with a NodePort service to expose the login page on port 32000, ensuring proper label matching and accessibility via the KodeKloud lab’s Grafana button.

**💡 Solution Approach**:
1. **Deployment Creation**: Defined `grafana-deployment-nautilus` with `grafana/grafana:latest` and a single replica.
2. **Service Creation**: Configured a NodePort service with `nodePort: 32000` and selector `app: grafana`.
3. **Label Matching**: Ensured consistent `app: grafana` labels between deployment and service.
4. **Deployment**: Applied both YAML files using `kubectl apply`.
5. **Verification**: Confirmed deployment, pod, and service status, and tested accessibility via the Grafana button.

**🎯 Key Success Factors**:
- **Correct Deployment**: Used `grafana/grafana:latest` with proper labels.
- **NodePort Service**: Configured port 32000 correctly.
- **Label Consistency**: Ensured `app: grafana` for pod-service routing.
- **Verification**: Confirmed pod status and service accessibility.
- **Testing**: Validated Grafana login page access.

**⚠️ Critical Fixes**:
- Ensure `app: grafana` labels match in deployment and service.
- Use `grafana/grafana:latest` for the official Grafana image.
- Validate `nodePort: 32000` is within the valid range (30000-32767).
- Confirm cluster node accessibility for NodePort.

**🔒 Kubernetes Best Practices Applied**:
- **Label Management**: Used consistent labels for deployment-service matching.
- **Minimal Configuration**: Kept YAMLs simple with required fields only.
- **Dry Run Testing**: Validated YAMLs with `--dry-run=client`.
- **Service Exposure**: Used NodePort for external access in a lab environment.

**⚠️ Important Troubleshooting Concepts**:
- **Service Endpoints**: Check `kubectl get endpoints` to ensure service routes to pods.
- **Logs**: Use `kubectl logs` to diagnose Grafana startup issues.
- **Events**: Use `kubectl get events` to identify deployment or service issues.
- **NodePort Access**: Verify node IP and port accessibility.

---

## ⚠️ Important Production Notes

🔧 **Deployment Debugging**: Use `kubectl describe`, `kubectl logs`, and `kubectl get endpoints` for issue resolution.
🔐 **Security**: Secure Grafana with proper authentication and network policies in production.
📊 **Resource Limits**: Add `resources` (e.g., CPU/memory limits) to prevent overuse.
🛡️ **Service Exposure**: Use Ingress or LoadBalancer for production instead of NodePort.

## Best Practices

- **Verify Service endpoints before testing external access.** A non-empty endpoint confirms that the Service selector matches a ready Pod and isolates routing problems from application startup problems.
- **Wait for the rollout to complete.** A newly created Pod can be running while its application is still initializing, so wait for the Deployment before testing the application.
- **Keep labels consistent.** Deployment selectors, Pod labels, and Service selectors must agree or the Service will have no backend endpoints.
- **Pin image versions in production.** The `latest` tag is acceptable for this lab, but a fixed Grafana version makes production deployments repeatable and prevents unexpected upgrades.
- **Use health probes for production workloads.** Readiness and liveness probes help Kubernetes distinguish between a running container and a Grafana instance that is actually ready to receive traffic.

### 📚 Official Documentation

- [Run Grafana Docker Image](https://grafana.com/docs/grafana/latest/setup-grafana/installation/docker/)
- [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Kubernetes Service and NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport)
- [kubectl describe](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_describe/)
