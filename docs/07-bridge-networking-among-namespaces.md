# 07 — Bridge Networking Among Namespaces (3 Namespaces)

In this chapter, you will build a **multi-namespace Layer-2 network** using a **Linux bridge**, allowing **three network namespaces** to communicate with each other as if they were connected to the same switch.

This setup closely resembles:

- A physical Ethernet switch
- Docker bridge networking (`docker0`)
- Kubernetes bridge-based CNIs
- Virtual lab environments

## 🎯 Goals

By the end of this chapter, you will be able to:

- Connect multiple namespaces to a single bridge
- Enable inter-namespace communication
- Understand Layer-2 forwarding inside Linux bridges
- Scale from 1 → N namespaces without redesign

## 🌐 Network Topology

```bash
                     Linux Host
+--------------------------------------------------+
|                                                  |
|              br0 (Linux Bridge)                  |
|              172.18.0.1/24                       |
|      +-----------+-----------+-----------+       |
|      |           |           |           |       |
|  veth-br1    veth-br2    veth-br3                |
|      |           |           |                   |
+------+-----------+-----------+-------------------+
       |           |           |
   [ veth ]    [ veth ]    [ veth ]
       |           |           |
   ns1            ns2            ns3
 172.18.0.2    172.18.0.3    172.18.0.4

```
All namespaces are on the same subnet and can talk to each other.

## 🛠 Step 1 — Create the Bridge

```bash
sudo ip link add br0 type bridge
sudo ip addr add 172.18.0.1/24 dev br0
sudo ip link set br0 up
```

## 🛠 Step 2 — Create Three Network Namespaces

```bash
sudo ip netns add ns1
sudo ip netns add ns2
sudo ip netns add ns3
```

Verify:

```bash
ip netns list
```

## 🛠 Step 3 — Create veth Pairs

Each namespace gets its own veth pair.

```bash
sudo ip link add veth-ns1 type veth peer name veth-br1
sudo ip link add veth-ns2 type veth peer name veth-br2
sudo ip link add veth-ns3 type veth peer name veth-br3
```

## 🛠 Step 4 — Move Namespace Interfaces

```bash
sudo ip link set veth-ns1 netns ns1
sudo ip link set veth-ns2 netns ns2
sudo ip link set veth-ns3 netns ns3
```

## 🛠 Step 5 — Attach Host Interfaces to the Bridge

```bash
sudo ip link set veth-br1 master br0
sudo ip link set veth-br2 master br0
sudo ip link set veth-br3 master br0

sudo ip link set veth-br1 up
sudo ip link set veth-br2 up
sudo ip link set veth-br3 up
```

Verify bridge membership:

```bash
bridge link
```

## 🛠 Step 6 — Configure Namespace Networking

### Enable loopback:

```bash
sudo ip netns exec ns1 ip link set lo up
sudo ip netns exec ns2 ip link set lo up
sudo ip netns exec ns3 ip link set lo up
```
### Assign IP addresses:

```bash
sudo ip netns exec ns1 ip addr add 172.18.0.2/24 dev veth-ns1
sudo ip netns exec ns2 ip addr add 172.18.0.3/24 dev veth-ns2
sudo ip netns exec ns3 ip addr add 172.18.0.4/24 dev veth-ns3
```

### Bring interfaces up:

```bash
sudo ip netns exec ns1 ip link set veth-ns1 up
sudo ip netns exec ns2 ip link set veth-ns2 up
sudo ip netns exec ns3 ip link set veth-ns3 up
```

## 🧪 Step 7 — Test Inter-Namespace Connectivity

### ns1 → ns2

```bash
sudo ip netns exec ns1 ping 172.18.0.3 -c 3
```

### ns2 → ns3

```bash
sudo ip netns exec ns2 ping 172.18.0.4 -c 3
```

### ns3 → ns1

```bash
sudo ip netns exec ns3 ping 172.18.0.2 -c 3
```

All pings should succeed.

## 🔍 Inspect Bridge Behavior

### Show bridge interfaces:

```bash
bridge link
```

### Show bridge MAC table:

```bash
bridge fdb show br0
```

You’ll see MAC addresses learned dynamically from the namespaces — exactly how a real switch behaves.

## 🧠 How This Works (Internals)

- The Linux bridge operates at **Layer-2** (Ethernet)
- It forwards frames based on **MAC address learning**
- Each namespace believes it is on a real Ethernet LAN
- No routing or NAT is required for namespace-to-namespace communication
- Broadcasts (ARP) work exactly like physical networks

## 🛠 Cleanup

```bash
sudo ip netns del ns1
sudo ip netns del ns2
sudo ip netns del ns3
sudo ip link delete br0
```

## 🎯 Summary

In this chapter, you learned how to:

✔ Create a Linux bridge \
✔ Attach multiple namespaces to the same bridge \
✔ Build a shared Layer-2 network \
✔ Enable full inter-namespace communication \
✔ Inspect bridge forwarding and MAC learning

This setup is the foundation for:

- Multi-container networking
- Virtual switches and routers
- Complex lab topologies
- Kubernetes-style pod networks