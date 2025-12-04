# 04 — Connect Network Namespace to Root Using a Bridge

In the previous chapter, you connected a namespace directly to the host using a **veth pair**.

This works fine for one namespace, but does not scale.

A better approach is to use a **Linux bridge**, which acts like a virtual switch.

Using a bridge, you can:

- Connect multiple namespaces
- Enable communication with the host
- Forward traffic to root / outside networks
- Add NAT for egress (in the next chapter)

This is exactly how Docker’s default network (`docker0`) works.

## 🧩 What is a Linux Bridge?

A Linux bridge is a **Layer 2 virtual switch** inside the kernel.

It forwards Ethernet frames between interfaces attached to it.

```bash
          +--------------------+
          |     br0 (bridge)   |
          +---+------------+---+
              |            |
          veth-host1   veth-host2   ... N namespaces
```

## 🛠️ Step 1 — Create the Bridge

```bash
sudo ip link add br0 type bridge
sudo ip link set br0 up
```

Assign an IP to the bridge (this will act as “gateway” for namespaces):

```bash
sudo ip addr add 192.168.100.1/16 dev br0
```

## 🛠️ Step 2 — Create a Namespace

```bash
sudo ip netns add ns1
```

## 🛠️ Step 3 — Create a veth Pair

One interface stays in host (connected to bridge), the other goes to namespace.

```bash
sudo ip link add veth-ns1 type veth peer name veth-br1
```

- `veth-ns1` → inside ns1
- `veth-br1` → connected to bridge br0

## 🛠️ Step 4 — Move One End of veth to the Namespace

```bash
sudo ip link set veth-ns1 netns ns1
```

## 🛠️ Step 5 — Attach Host-Side veth to the Bridge

```bash
sudo ip link set veth-br1 master br0
sudo ip link set veth-br1 up
```

Diagram:

```bash
     Host
+-------------------+
|   br0 (switch)    |
|     192.168.100.1 |
|   veth-br1        |
+--------+----------+
         |
   [ veth cable ]
         |
+--------+----------+
|  ns1 Namespace    |
|  veth-ns1         |
|  IP: 192.168.100.2|
+-------------------+
```

## 🛠️ Step 6 — Configure the Namespace Side

Enable loopback:

```bash
sudo ip netns exec ns1 ip link set lo up
```

Assign IP:

```bash
sudo ip netns exec ns1 ip addr add 192.168.100.2/16 dev veth-ns1
sudo ip netns exec ns1 ip link set veth-ns1 up
```

## 🛠️ Step 7 — Add Default Route to Root

Inside ns1:

```bash
sudo ip netns exec ns1 ip route add default via 192.168.100.1
```

This means:

- To reach root (host) or outside, send packets to the bridge gateway.

## 🧪 Step 8 — Test Connectivity

1. **From namespace → bridge (gateway):**
    ```bash
    sudo ip netns exec ns1 ping 192.168.100.1
    ```

2. **Namespace → Host itself:**
    ```bash
    sudo ip netns exec ns1 ping $(hostname -I | awk '{print $1}')
    ```

3. **Host → Namespace:**
    ```bash
    ping 192.168.100.2
    ```

If all works, the namespace is successfully connected to the root network via a bridge.

## 📘 Visual Diagram

```bash
                          Host Root Network
+--------------------------------------------------------------+
|                                                              |
|   +-------------------+         +--------------------------+ |
|   |    br0 (bridge)   |         |   Host Network Stack     | |
|   | 192.168.100.1/16  |         |   Routes, services, etc. | |
|   +--------+----------+         +--------------------------+ |
|            |                                           ^     |
|       veth-br1                                         |     |
|            |                                      Default    |
+------------|-------------------------------------------------+
             |                                                   
     [ virtual ethernet cable ]                                  
             |                                                   
+------------|--------------------------------------------+     
|      ns1 Namespace                                      |     
|   +------------------+                                  |     
|   |   veth-ns1       | <-----> (connected to br0)       |     
|   | 192.168.100.2    |                                  |     
|   +------------------+                                  |     
|   | Default via:     |                                  |     
|   | 192.168.100.1    |                                  |     
+---------------------------------------------------------+

```

## 🧹 Cleanup Commands

```bash
sudo ip netns del ns1
sudo ip link delete br0
```

## 📝 Troubleshooting

**✔ Ping from namespace to bridge fails?**

Check if `veth-ns1` is up:

```bash
sudo ip netns exec ns1 ip link show veth-ns1
```

Check if `veth-br1` is attached to the bridge:

```bash
bridge link
```

**✔ Host cannot ping namespace?**

Make sure IPs are correctly assigned:

```bash
ip a
sudo ip netns exec ns1 ip a
```

**✔ Route missing?**

```bash
sudo ip netns exec ns1 ip route
```

Should contain:

```bash
default via 192.168.100.1 dev veth-ns1
```

## 🎯 Summary

In this chapter, you learned how to:

✔ Use a Linux bridge (br0) to connect a namespace to the host \
✔ Create a veth pair and attach one end to the bridge \
✔ Assign bridge and namespace IPs \
✔ Enable loopback and bring interfaces up \
✔ Add a default route to allow root network access \
✔ Test connectivity

This setup is essential for:

- Multi-namespace networks
- Advanced routing setups
- NAT and egress traffic (next chapter)
- Kubernetes-style bridge networking