# 05 — Egress Traffic (Internet Access from a Network Namespace)

In the previous chapter, you connected a network namespace to the root network using a **bridge**.

Now, we take the next step: \
**✔ Enable internet access (egress traffic)** from the network namespace.

This requires:

- A bridge (`br0`)
- A veth pair per namespace
- IP assignment
- Default route inside the namespace
- **NAT (MASQUERADE)** on the host

Egress traffic is only possible when the host **NATs packets** from the namespace into the external interface — exactly how Docker does it.

## 🧩 Concept Overview

A network namespace cannot directly access the internet because:

- It uses private subnet IPs (e.g., `192.168.100.0/16`)
- These IPs are not routable on the host’s upstream network
- No NAT means packets will be dropped by your router/ISP

Therefore, we must:

1. Route traffic from the namespace → bridge → host
2. NAT the packets so they appear to come from the host’s real interface
3. Allow forwarding between host interfaces

## 🌐 Example Topology

```bash
                   Internet
                       ^
                       |
               +----------------+
               |   Host eth0    |
               |  (public IP)   |
               +-------+--------+
                       |
                 NAT & Forwarding
                       |
               +-------+--------+
               |     br0        |
               | 192.168.100.1  |
               +-------+--------+
                       |
                 veth-br1
               (host side)
                       |
               [ virtual cable ]
                       |
                 veth-ns1
               (namespace side)
                       |
           ns1 (192.168.100.2)
```

## 🛠 Step 1 — Create Namespace & Bridge Setup

*(If you already did this in Chapter 4, skip to Step 2.)*

### Create namespace

```bash
sudo ip netns add ns1
```

### Create bridge

```bash
sudo ip link add br0 type bridge
sudo ip addr add 192.168.100.1/16 dev br0
sudo ip link set br0 up
```

### Create veth pair

```bash
sudo ip link add veth-ns1 type veth peer name veth-br1
```

Move one end to namespace:

```bash
sudo ip link set veth-ns1 netns ns1
```

Attach host-side veth to bridge:

```bash
sudo ip link set veth-br1 master br0
sudo ip link set veth-br1 up
```

### Configure namespace interface:

```bash
sudo ip netns exec ns1 ip addr add 192.168.100.2/16 dev veth-ns1
sudo ip netns exec ns1 ip link set veth-ns1 up
sudo ip netns exec ns1 ip link set lo up
```

### Add default route inside ns1:

```bash
sudo ip netns exec ns1 ip route add default via 192.168.100.1
```

## 🛠 Step 2 — Enable IP Forwarding on Host

Linux must act as a router.

### Temporary:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

### Permanent:

```bash
echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/99-ns-forwarding.conf
sudo sysctl -p /etc/sysctl.d/99-ns-forwarding.conf
```

## 🛠 Step 3 — Add NAT (MASQUERADE)

We need to NAT namespace traffic through the host’s external network interface, usually:

- eth0
- wlan0
- enp3s0
- ens0
- etc.

Find your real internet interface:

```bash
ip route get 8.8.8.8
```

Output example:

```bash
8.8.8.8 dev eth0 ...
```

Assume it is `eth0`.

### Add MASQUERADE rule:

```bash
sudo iptables -t nat -A POSTROUTING -s 192.168.100.0/16 -o eth0 -j MASQUERADE
```
### Firewall Rules

```bash
sudo iptables -t nat -L -n -v
```
In case if still it not works then we may need to add some additional firewall rules to
allow forwarding:

```bash
sudo iptables -A FORWARD -i br0 -o eth0 -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o br0 -j ACCEPT
```

## 🧪 Step 4 — Test Internet Access

From the namespace:

```bash
sudo ip netns exec ns1 ping -c 3 8.8.8.8
```

If DNS doesn’t work yet, test with curl:

```bash
sudo ip netns exec ns1 curl -I https://example.com
```

#### If DNS fails, add resolv.conf inside ns1:

```bash
sudo ip netns exec ns1 bash -c "echo 'nameserver 8.8.8.8' > /etc/resolv.conf"
```

## 📘 How It Works Internally

1. **ns1 sends packet → 8.8.8.8**
  
    Default route:
    
    ```bash
    0.0.0.0/0 via 192.168.100.1
    ```
2. **Bridge forwards packet to host**

    Bridge acts like a virtual switch.

3. **Host receives packet on br0**

    Since 8.8.8.8 is not local, host routes it via `eth0`.

4. **iptables MASQUERADE rewrites source IP**

    `192.168.100.2` → public IP of host.

5. **Response packet comes back**

    Host receives the response and reverse-NATs it back to the namespace.

## 🛠 Cleanup

```bash
sudo ip netns del ns1
sudo ip link delete br0
sudo iptables -t nat -F
```

## 📝 Troubleshooting

### ❌ Namespace can’t reach internet?
Check default route inside ns1:

```bash
sudo ip netns exec ns1 ip route
```

Should be:

```bash
default via 192.168.100.1
```

### ❌ NAT rule missing?

```bash
sudo iptables -t nat -L -n -v
```

### ❌ IP forwarding disabled?

```bash
cat /proc/sys/net/ipv4/ip_forward
```
Should show:

```bash
1
```

### ❌ Wrong external interface?
Check with:

```bash
ip route get 8.8.8.8
```

## 🎯 Summary

In this chapter, you learned how to give a namespace **internet access** using:

✔ a Linux bridge \
✔ veth pair \
✔ IP addressing \
✔ default routes \
✔ IP forwarding \
✔ iptables NAT (MASQUERADE)

This setup is the foundation of:

- Docker’s default network
- Kubernetes bridge-mode CNIs
- Virtual routers
- Realistic lab topologies