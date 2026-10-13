# Day 42 - Create a Docker Network

The Nautilus DevOps team needs to set up several docker environments for different applications. One of the team members has been assigned a ticket where he has been asked to create some docker networks to be used later. Complete the task based on the following ticket description:

a. Create a docker network named as `media` on `App Server 3` in `Stratos DC`.

b. Configure it to use `bridge` drivers.

c. Set it to use subnet `192.168.30.0/24` and iprange `192.168.30.0/24`.

---

## 🔹 Step 1: Connect to App Server 2

```bash
ssh banner@stapp03
```

**Purpose**: Connect to Application Server 2 as steve user for Docker network operations.

---

## 🔹 Step 2: Verify Docker service is running

```bash
sudo systemctl status docker
```

**Purpose**: Ensure Docker daemon is active before performing network operations.

**Expected Output**:
```
● docker.service - Docker Application Container Engine
   Active: active (running)
```

---

## 🔹 Step 3: Check existing Docker networks

```bash
sudo docker network ls
```

**Purpose**: View currently configured Docker networks before creating new one.

**Expected Output Example**:
```
NETWORK ID     NAME      DRIVER    SCOPE
62dd48c8a619   bridge    bridge    local
50d1428ce6ef   host      host      local
c6082dbeea2d   none      null      local
```

**Default Networks**:
- **bridge**: Default network for containers
- **host**: Uses host's network stack
- **none**: No network connectivity

---

## 🔹 Step 4: Understand Bridge network driver

**Macvlan Driver Characteristics**:
- **Direct Layer 2 access**: Containers get MAC addresses
- **Network visibility**: Containers appear as physical devices on network
- **IP allocation**: Containers receive IPs from subnet range
- **Performance**: Lower latency than bridge networks
- **Use cases**: Legacy applications, network monitoring

---

## 🔹 Step 5: Create the Bridge network

```bash
sudo docker network create -d bridge \
  --subnet=192.168.30.0/24 \
  --ip-range=192.168.30.0/24 \
  --gateway=192.168.30.1 \
  media
```

**Purpose**: Create Docker network with bridge driver and specified network configuration.

**Command Options**:
- `-d bridge`: Use bridge driver
- `--subnet`: Network address range
- `--ip-range`: Available IP allocation range
- `--gateway`: Default gateway for network
- `official`: Network name

**Expected Output**:
```
cbc63375d61538c620af41f8e1517a3ae1e3d757495e23c8d4ecb68dae0b78f1
```
**Output Explanation**: 64-character hash is the network ID.

---

## 🔹 Step 6: Verify network creation

```bash
sudo docker network ls
```

**Purpose**: Confirm the 'official' network appears in Docker network list.

**Expected Output**:
```
NETWORK ID     NAME      DRIVER    SCOPE
54564a2eb655   bridge    bridge    local
f8fd6445e9eb   host      host      local
cbc63375d615   media     bridge    local
88f896606654   none      null      local
```

**Verification Points**:
- ✅ Network name: **media**
- ✅ Driver: **bridge**
- ✅ Scope: **local**

---

## 🔹 Step 7: Inspect network details

```bash
sudo docker network inspect media
```

**Purpose**: View detailed configuration of the newly created macvlan network.

**Expected Output**:
```json
[
    {
        "Name": "media",
        "Id": "cbc63375d61538c620af41f8e1517a3ae1e3d757495e23c8d4ecb68dae0b78f1",
        "Created": "2026-09-09T15:18:30.344225246Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.30.0/24",
                    "IPRange": "192.168.30.0/24",
                    "Gateway": "192.168.30.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

**Key Configuration Points**:
- **Subnet**: 192.168.30.0/24
- **IPRange**: 192.168.30.0/24
- **Gateway**: 192.168.30.1
- **Driver**: bridge

---

## 🔹 Step 8: Verify IPAM configuration

```bash
sudo docker network inspect media --format='{{json .IPAM.Config}}'
```

**Purpose**: Extract and display IP Address Management (IPAM) configuration specifically.

**Expected Output**:
```json
[{"Subnet":"192.168.30.0/24","IPRange":"192.168.30.0/24","Gateway":"192.168.30.1"}]
```

---

## 🔹 Step 9: Test network with container (optional)

```bash
sudo docker run --rm --network=media --ip=192.168.30.10 alpine ip addr show
```

**Purpose**: Test the bridge network by running a container connected to it.

**Command Breakdown**:
- `--network=media`: Connect to bridge network
- `--ip=192.168.30.10`: Assign specific IP from range
- `alpine`: Lightweight test image
- `ip addr show`: Display container network configuration

---

## 🔹 Step 10: Check network connectivity (optional)

```bash
sudo docker run --rm --network=media --ip=192.168.30.11 alpine ping -c 3 192.168.30.1
```

**Purpose**: Test connectivity to the gateway from a container on the macvlan network.

**Expected Behavior**: Successful ping responses if gateway is reachable.

---

## 📋 Quick Command Reference

For quick copy-paste, execute on **App Server 2**:

```bash
# Connect to server
ssh steve@stapp02

# Verify Docker is running
sudo systemctl status docker

# Create macvlan network
sudo docker network create -d bridge \
  --subnet=192.168.30.0/24 \
  --ip-range=192.168.30.0/24 \
  --gateway=192.168.30.1 \
  official

# Verify network creation
sudo docker network ls
sudo docker network inspect official
```

---

## 💡 Additional Tips

- **Parent interface**: Add `--parent=eth0` to specify host network interface
- **VLAN tagging**: Use `--parent=eth0.100` for VLAN-tagged interfaces
- **IPv6 support**: Add `--ipv6` flag for dual-stack configuration
- **Custom IPAM**: Use `--aux-address` for reserving specific IPs
- **Network removal**: Use `docker network rm official` to delete network

---

## 🔧 Troubleshooting Common Issues

### **Issue 1: Network already exists**
**Symptoms**: Error "network with name official already exists"
**Solution**: Remove existing network or use different name
```bash
sudo docker network ls
sudo docker network rm official
sudo docker network create -d bridge --subnet=192.168.30.0/24 --gateway=192.168.30.1 media
```

### **Issue 2: Parent interface not specified**
**Symptoms**: Warning about missing parent interface
**Solution**: Specify parent network interface explicitly
```bash
sudo docker network create -d macvlan \
  --subnet=192.168.30.0/24 \
  --gateway=192.168.30.1 \
  --parent=eth0 \
  media
```

### **Issue 3: IP range conflicts**
**Symptoms**: Cannot allocate IPs or network conflicts
**Solution**: Verify subnet doesn't conflict with existing networks
```bash
ip addr show
sudo docker network inspect bridge
```

### **Issue 4: Container cannot reach gateway**
**Symptoms**: No connectivity from macvlan containers
**Solution**: Check host firewall and network configuration
```bash
sudo iptables -L
sudo ip route show
```

---

## 🚨 Task-Specific Challenge & Solution

**🔍 Main Challenge Encountered:**

The primary challenge was **creating a Docker macvlan network with specific network parameters** including subnet, IP range, and gateway configuration for advanced container networking scenarios.

**💡 Solution Approach:**

1. **Network Driver Selection**: Used macvlan driver to provide containers with direct Layer 2 network access
2. **Subnet Configuration**: Specified 192.168.30.0/24 subnet for network address space
3. **IP Range Allocation**: Defined same IP range for container IP assignment
4. **Gateway Configuration**: Set 192.168.30.1 as default gateway for network traffic
5. **Network Naming**: Created network with name "official" for easy identification
6. **Verification Process**: Confirmed network creation and inspected detailed configuration

**🎯 Key Success Factors:**
- **Driver specification** using `-d bridge` for advanced networking capabilities
- **Network parameters** properly configuring subnet, IP range, and gateway
- **CIDR notation** using /24 notation for subnet mask (255.255.255.0)
- **Gateway IP** setting appropriate gateway within subnet range
- **Network inspection** validating configuration through docker network inspect
- **IPAM verification** confirming IP Address Management settings

**⚠️ Critical Configuration Details:**
- **Subnet specification** must use CIDR notation (192.168.30.0/24)
- **IP range** typically matches subnet but can be subset for IP allocation control
- **Gateway IP** must be within subnet range and typically host's interface IP
- **Macvlan driver** provides Layer 2 access but requires compatible network infrastructure
- **Parent interface** optional but recommended for explicit interface binding

**🔒 Macvlan Network Benefits:**
- **Direct network access** containers appear as physical devices on network
- **Performance** lower latency compared to bridge networking
- **Legacy compatibility** supports applications requiring Layer 2 access
- **IP allocation** containers receive real IPs from network subnet

**⚠️ Important Macvlan Considerations**:
- **Host communication** macvlan containers cannot communicate with host by default
- **Network requirements** requires promiscuous mode on parent interface
- **Subnet planning** must not conflict with existing network ranges
- **IP management** careful planning needed to avoid IP conflicts

---

## ⚠️ Important Production Notes

🔧 **Advanced Networking**: bridge provides Layer 2 network access for containers requiring direct network visibility

🔐 **Network Isolation**: Containers on bridge network appear as separate devices on physical network

📊 **IP Management**: Careful subnet and IP range planning required to avoid conflicts with existing networks

🛡️ **Use Cases**: Ideal for legacy applications, network monitoring tools, and scenarios requiring direct Layer 2 access