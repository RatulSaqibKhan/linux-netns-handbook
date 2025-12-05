# 06 — Connect Two Custom Network Namespaces

In this chapter, you will learn how to create **two separate network namespaces** and connect them together so that they can communicate with each other — using a **veth pair** or via a **Linux bridge**. This setup is useful for simulating two “containers” on the same machine.

We will cover:

- **Method A:** Direct veth cable between two namespaces
- **Method B:** Bridge-based connection

## 🔧 Prerequisites

- Root (or `sudo`) access to a Linux machine
- `iproute2` installed (`ip`, `bridge`, `netns` commands)

## ⚙️ Method A: Direct Veth Pair Between Two Namespaces

This is the simplest: connect two namespaces directly with a veth pair. Traffic from one namespace goes directly to the other.

### Step 1 — Create the namespaces

```bash
sudo ip netns add ns1
sudo ip netns add ns2
```

### Step 2 — Create a veth pair

```bash
sudo ip link add veth1 type veth peer name veth2
```

- `veth1` will go into **ns1**
- `veth2` will go into **ns2**

### Step 3 — Move the veth ends into the namespaces

```bash
sudo ip link set veth1 netns ns1
sudo ip link set veth2 netns ns2
```

### Step 4 — Assign IP addresses

Choose a private subnet, for example `10.200.10.0/16`.

```bash
sudo ip netns exec ns1 ip addr add 10.200.10.1/16 dev veth1
sudo ip netns exec ns2 ip addr add 10.200.10.2/16 dev veth2
```

### Step 5 — Bring up the interfaces & loopback

```bash
sudo ip netns exec ns1 ip link set veth1 up  
sudo ip netns exec ns2 ip link set veth2 up  

sudo ip netns exec ns1 ip link set lo up  
sudo ip netns exec ns2 ip link set lo up  
```

### Step 6 — Test connectivity

From **ns1**:

```bash
sudo ip netns exec ns1 ping 10.200.10.2 -c 3
```

From **ns2**:

```bash
sudo ip netns exec ns2 ping 10.200.10.1 -c 3
```

You should see the pings succeed, which means the two namespaces are now directly connected.

## 🌉 Method B: Bridge-Based Connection

Using a bridge gives more flexibility and is more scalable (especially if you want more than two namespaces).

### Step 1 — Create a bridge on the host

```bash
sudo ip link add br0 type bridge  
sudo ip link set br0 up  
```

Optionally, assign an IP to the bridge (this can act like a gateway):

```
sudo ip addr add 192.168.50.1/16 dev br0
```

### Step 2 — Create veth pairs for each namespace

```bash
sudo ip link add veth-ns1 type veth peer name veth-br1  
sudo ip link add veth-ns2 type veth peer name veth-br2  
```

- `veth-ns1` → to namespace `ns1`
- `veth-br1` → to bridge `br0`
- `veth-ns2` → to namespace `ns2`
- `veth-br2` → to bridge `br0`

### Step 3 — Move namespace-side veths into namespaces

```bash
sudo ip link set veth-ns1 netns ns1  
sudo ip link set veth-ns2 netns ns2  
```

### Step 4 — Attach the bridge-side veths to the bridge

```bash
sudo ip link set veth-br1 master br0  
sudo ip link set veth-br2 master br0  

sudo ip link set veth-br1 up  
sudo ip link set veth-br2 up  
```

### Step 5 — Configure namespace interfaces

Assign IPs in the same subnet as the bridge (if the bridge has an IP):

```bash
sudo ip netns exec ns1 ip addr add 192.168.50.2/16 dev veth-ns1  
sudo ip netns exec ns2 ip addr add 192.168.50.3/16 dev veth-ns2  
```

Bring them up:

```bash
sudo ip netns exec ns1 ip link set veth-ns1 up  
sudo ip netns exec ns2 ip link set veth-ns2 up  

sudo ip netns exec ns1 ip link set lo up  
sudo ip netns exec ns2 ip link set lo up  
```

### Step 6 — Set default route (optional)

If you want the namespaces to route through the bridge:

```bsah
sudo ip netns exec ns1 ip route add default via 192.168.50.1  
sudo ip netns exec ns2 ip route add default via 192.168.50.1  
```

(This only makes sense if the bridge IP is a gateway or if you run NAT / forwarding on the host.)

### Step 7 — Test connectivity

From `ns1` → `ns2`:

```bash
sudo ip netns exec ns1 ping 192.168.50.3 -c 3
```

From `ns2` → `ns1`:

```bash
sudo ip netns exec ns2 ping 192.168.50.2 -c 3
```

If pings work, then the bridge is correctly connecting the two namespaces.

## 📝 Why Use Bridge vs Direct veth?

| Feature              | Direct veth                          | Bridge-based                                     |
| -------------------- | ------------------------------------ | ------------------------------------------------ |
| Simplicity           | ✅ Very simple (just 2 veth ends)     | Slightly more setup                              |
| Scaling              | ❌ Not ideal for > 2 namespaces       | ✅ Easily scale to many namespaces                |
| Intermediate control | ⚠ Less control over “middle” network | ✅ Can configure bridge IP, NAT, forwarding, etc. |
| Isolation            | ✅ Simple point-to-point isolation    | ✅ Full L2 network between namespaces             |

## ⚠️ Cleanup

When you’re done, clean up:

```bash
sudo ip netns del ns1  
sudo ip netns del ns2  
sudo ip link delete br0  
```

If bridge-side veths still exist, you may need to delete them manually.

## 🎯 Summary

- You can connect **two custom network namespaces** either directly (via a veth pair) or via a Linux bridge.
- Direct veth is simple but not flexible for more than two namespaces.
- A bridge-based setup is more powerful: you can add more namespaces later, assign a gateway IP, run NAT, simulate more realistic networks.
- These same techniques are commonly described in Linux networking guides