# Docker Bridge Networking: Secure Container Communication

## 📌 Objective
The goal of this exercise is to explore and demonstrate network isolation in Docker containers. We will examine how containers within the same custom bridge network can communicate, while those on different networks remain isolated. Understanding this is crucial for securing microservices and containerized applications.

---

## 🌐 Introduction to Docker Networking
Docker networking is fundamental for containerized applications, allowing containers to communicate while ensuring security and isolation. Docker provides several networking options:

### 🔹 Types of Docker Networks:
- **Bridge Network (Default)** – Allows communication between containers using internal IPs unless restricted.
- **Custom Bridge Network** – Offers better control and supports name-based resolution.
- **Host Network** – Attaches containers directly to the host’s network stack.
- **Overlay Network** – Enables communication across multiple hosts (Docker Swarm).
- **Macvlan Network** – Assigns a MAC address to each container, making them appear as separate devices.
- **None Network** – Completely disables networking.

For this demonstration, we focus on the custom bridge network, which improves control and network isolation.

---

## ⚡ Why Use a Custom Bridge Network?
- ✅ Improved Security – Containers on different networks are isolated by default.
- ✅ Better Performance – Direct communication without host networking stack overhead.
- ✅ DNS-Based Resolution – Containers communicate via names instead of IPs.
- ✅ Greater Control – Define specific subnets, IP ranges, and gateways.

To demonstrate, we create a custom bridge network called `rahul-bridge` and connect multiple containers.

---

## 🔧 1. Creating a Custom Bridge Network
```bash
docker network create --driver bridge --subnet 172.20.0.0/16 --ip-range 172.20.240.0/20 rahul-bridge
```

### 🔍 Explanation:
- `--driver bridge` → Uses the default bridge network mode.
- `--subnet 172.20.0.0/16` → Defines the network’s IP range.
- `--ip-range 172.20.240.0/20` → Allocates IPs dynamically.

### ❌ Error Faced:
```bash
Error response from daemon: cannot create network ed132135c6801a447a08b680fb304fc641903f2ce93a67a4805bdd686933c3e3 (br-ed132135c680): conflicts with network 8b5a53bffc83cf0ee9d063ef35b09e2f8b196ba8e10e4c947db44f0205d8707f (br-8b5a53bffc83): networks have overlapping IPv4
```

### ✅ Solution:
Change the subnet to avoid overlapping with an existing network:
```bash
docker network create --driver bridge --subnet 192.168.100.0/24 --ip-range 192.168.100.128/25 rahul-bridge
```

---

## 🚀 2. Running Containers in the Custom Network
### Running Redis Container (`rahul-database`)
```bash
docker run -itd --net=rahul-bridge --name=rahul-database redis
```

### Running BusyBox Container (`rahul-server-A`)
```bash
docker run -itd --net=rahul-bridge --name=rahul-server-A busybox
```

---

## 📌 3. Check Container IPs
```bash
docker network inspect rahul-bridge
```

### Expected Output:
```
rahul-database: 192.168.100.128
rahul-server-A: 192.168.100.129
```

---

## 🔄 4. Testing Communication Between Containers
### Ping from `rahul-database` to `rahul-server-A`
```bash
docker exec -it rahul-database ping 192.168.100.129
```

### Ping from `rahul-server-A` to `rahul-database`
```bash
docker exec -it rahul-server-A ping 192.168.100.128
```

✅ Both containers should successfully ping each other.

---

## 🚧 5. Demonstrating Network Isolation with a Third Container
### Add Another Container (`rahul-server-B`) on the Default Bridge Network
```bash
docker run -itd --name=rahul-server-B busybox
```

### Get IP of `rahul-server-B`
```bash
docker inspect -format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' rahul-server-B
```
(Example IP: 172.17.0.2)

---

## ❌ 6. Testing Communication Between Different Networks
### Ping from `rahul-database` to `rahul-server-B`
```bash
docker exec -it rahul-database ping 172.17.0.2
```

🚨 Expected Outcome: The ping should fail, as they are on different networks.

---

## 🔍 7. Confirming Network Isolation
```bash
docker network inspect rahul-bridge
docker network inspect bridge
```

✅ `rahul-bridge` should contain `rahul-database` & `rahul-server-A`.
✅ `bridge` should contain `rahul-server-B`.

---

## 🏆 Conclusion
- Containers in the same network can communicate.
- Containers in different networks are isolated by default.
- Docker’s networking model ensures security and separation unless explicitly connected.

🚀 Now you have mastered Docker Bridge Networking! 🎯

