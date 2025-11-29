# 01 — Introduction to Linux Network Namespaces

Linux namespaces provide process-level isolation for various system resources. They are one of the foundational technologies behind containers (Docker, Kubernetes, LXC) and allow multiple isolated environments to coexist on a single host.

This document introduces <b>network namespaces</b>, explains how they work, why they matter, and how to start working with them using simple commands.

## 🌐 What Are Namespaces?

A <b>namespace</b> is a Linux kernel feature that gives a process its own view of a system resource.
Different processes can live in different namespaces and therefore <b>see different “versions” of the same system resource</b>.

Common namespace types include:

| Namespace | Purpose                                             |
| --------- | --------------------------------------------------- |
| `net`     | Isolates network interfaces, routes, firewall rules |
| `ipc`     | Isolates inter-process communication resources      |
| `mnt`     | Isolates filesystem mount points                    |
| `uts`     | Isolates hostname & domain name                     |
| `pid`     | Isolates process IDs                                |
| `user`    | Isolates UID/GID mappings                           |
| `cgroup`  | Isolates control groups                             |

In this handbook, we focus entirely on the network namespace (`net`).

## 🌍 What Is a Network Namespace?

A <b>network namespace</b> gives a process — or group of processes — its own isolated:

- Network interfaces
- IP addresses
- Routing table
- Firewall rules
- `/proc/net` contents
- ARP & neighbor table
- Ports
- Sockets

Each network namespace behaves like a separate <b>virtual Linux network stack</b>.

Imagine having multiple small "virtual routers" inside one Linux machine.

## 🧩 Why Network Namespaces Matter

Network namespaces power technologies such as:

- <b>Docker / Podman</b>
- <b>Kubernetes networking (CNI plugins)</b>
- <b>Linux Containers (LXC/LXD)</b>
- <b>Cloud virtual networking</b>
- <b>Sandboxed applications</b>
- <b>Virtual lab environments</b>
- <b>Network simulations and topologies</b>

They allow you to:

✔ Build isolated test networks <br>
✔ Simulate multi-host topologies on one machine <br>
✔ Understand how containers connect internally <br>
✔ Experiment with routing, bridges, NAT, tunneling <br>
✔ Create custom virtual networks for learning

## 📌 Key Characteristics of Network Namespaces

1. <b>Isolation</b> <br>
  Each namespace has its own:

    - `lo` (loopback) interface
    - Routing table
    - ARP table
    - `iptables` / `nftables` rules

2. <b>Communication</b><br>
  Namespaces cannot communicate with each other unless you <b>explicitly connect them</b> via:

    - veth pairs
    - bridges
    - routing
    - NAT
    - tunnels

3. <b>Process-Specific</b><br>
  A namespace is “activated” when a process is placed inside it.<br>
  Example:
    ```bash
      ip netns exec myns ip addr
    ```
    This runs `ip addr` inside the namespace named `myns`.

4. <b>All isolation is kernel-level</b><br>
  No VMs, no emulation.<br>
  Namespaces are extremely lightweight.

## 🛠️ Creating Your First Network Namespace

Linux stores namespaces under `/var/run/netns/`.

Create one:

```bash
sudo ip netns add testns
```

List namespaces:

```bash
ip netns list
```

Run commands in a namespace:

```bash
sudo ip netns exec testns ip link
```
Delete a namespace:

```bash
sudo ip netns del testns
```

## 🔍 What You Get in a Fresh Namespace

Inside a new namespace, you will see:

```bash
sudo ip netns exec testns ip a
```

Output will show only:

- `lo` (loopback) — down by default
- No `eth0` or any NIC
- No IPs
- Empty routing table

This is a completely isolated, empty network universe.

## 🧠 How Namespaces Work Internally

Every network namespace is represented as an object inside the kernel (via `struct net`).<br>
Processes reference namespaces through the `task_struct` representation.

When you run:

```bash
ip netns add testns
```

Linux creates a new network namespace and binds it to a “handle” inside:

```bash
/var/run/netns/testns
```

This file acts as a reference that tools like `iproute2` can use.

## 🔄 What’s Next?

In the next chapters of this handbook, you will learn:

1. <b>How to inspect everything inside a namespace</b>
2. <b>How to connect namespaces to the host</b>
3. <b>How to create multi-namespace topologies</b>
4. <b>Bridging, routing, NAT, and egress traffic</b>
5. <b>Simulating routers, switches, and networks</b>

## 📚 Summary

- Network namespaces isolate the Linux network stack.
- Each namespace has its own interfaces, routes, and firewall rules.
- Processes run inside namespaces, giving them unique networking views.
- You can interconnect namespaces with veth pairs, bridges, routing, and NAT.
- Namespaces are the foundation of container and cloud networking.