# 02 — Network Namespace Inspecting

Before building any network topology, you must understand how to inspect the contents of a Linux network namespace.
This includes checking:

- Interfaces
- IP addresses
- Routing tables
- ARP/Neighbor tables
- Open sockets
- Firewall rules
- Processes inside the namespace

This chapter teaches you every essential inspection command you’ll need throughout the handbook.

## 🧪 Prerequisite

Create a sample network namespace for testing:

```bash
sudo ip netns add ns1
```

Most examples below will use this namespace:

```bash
sudo ip netns exec ns1 <command>
```

## 🕵️‍♂️ 1. Inspecting Network Interfaces

### List all interfaces

```bash
sudo ip netns exec ns1 ip link
```

You will typically see:

```bash
1: lo: <LOOPBACK,DOWN> ...
```

💡 A fresh namespace only contains `lo` (loopback), and it is <b>down</b> by default.

## 🧩 2. Inspecting IP Addresses

### Show all assigned IPs

```bash
sudo ip netns exec ns1 ip address
```

(or shortcut)

```bash
sudo ip netns exec ns1 ip a
```

This shows:

- Assigned IPv4/IPv6 addresses
- Interface statuses
- MAC addresses

## 🛣️ 3. Inspecting Routing Table

Each namespace has its own routing table.

### List routes

```bash
sudo ip netns exec ns1 ip route
```

A new namespace usually shows:

```bash
(no output)
```

Because:

- No network interfaces exist
- No default route
- No egress possible

## 🔍 4. Neighbor Table (ARP / NDP)

Useful to check Layer-2 neighbor discovery.

### IPv4 ARP table:

```bash
sudo ip netns exec ns1 ip neigh
```

### IPv6 neighbor table:

```bash
sudo ip netns exec ns1 ip -6 neigh
```

## 🔌 5. Checking Sockets and Ports

### List open sockets

```bash
sudo ip netns exec ns1 ss -tulnp
```

This shows:

- TCP/UDP listening ports
- Associated processes
- States
- Sockets

💡 Very useful when debugging service connectivity inside namespaces.

## 🔥 6. Firewall Rules (iptables / nftables)

Network namespaces <b>also isolate firewall rules</b>.

### nftables inspection

```bash
sudo ip netns exec ns1 nft list ruleset
```

### iptables rules

```bash
sudo ip netns exec ns1 iptables -L -n -v
```


A fresh namespace will have <b>empty rules</b>.

## 🧵 7. Processes Running Inside a Namespace

To verify which processes belong to a namespace:

### Option 1: Run a shell in the namespace

```bash
sudo ip netns exec ns1 bash
```

Inside it, run:

```bash
ps aux
```

### Option 2: Check from the host

```bash
lsns -t net
```

This shows:

- All network namespaces
- The PID of the process attached
- Namespace identifiers

Example:

```bash
4026532673 net  1234 bash
```

### Detailed process namespace info

```bash
sudo ls -l /proc/<pid>/ns/net
```

## 🛠️ 8. General Namespace Information

## List all namespaces

```bash
ip netns list
```

This shows all named network namespaces under:

```bash
/var/run/netns/
```

### Namespace IDs

```bash
sudo lsns -t net
```

To list namespaces with more detail:

```bash
sudo lsns
```

## 🧹 9. Deleting a Namespace

When you're done:

```bash
sudo ip netns del ns1
```

Verify:

```bash
ip netns list
```

## 📦 Bonus: Quick Summary of Useful Inspection Commands

| Purpose                | Command                              |
| ---------------------- | ------------------------------------ |
| List interfaces        | `ip netns exec ns1 ip link`          |
| Show IPs               | `ip netns exec ns1 ip a`             |
| View routing table     | `ip netns exec ns1 ip route`         |
| ARP table              | `ip netns exec ns1 ip neigh`         |
| Open sockets           | `ip netns exec ns1 ss -tulnp`        |
| Firewall rules         | `ip netns exec ns1 iptables -L`      |
| nftables rules         | `ip netns exec ns1 nft list ruleset` |
| Processes by namespace | `lsns -t net`                        |
| All namespaces         | `ip netns list`                      |
| Delete namespace       | `ip netns del <name>`                |


## 🎯 Summary

In this chapter, you learned how to inspect everything inside a network namespace:

- Interfaces
- IP addresses
- Routing tables
- ARP/neighbor table
- Sockets
- Firewall rules
- Processes

These inspection techniques will be essential as you begin connecting namespaces, building bridges, creating routes, simulating networks, and analyzing traffic in later chapters.