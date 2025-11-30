# 03 — Connect Network Namespace to Host

A fresh network namespace is **completely isolated** — it only contains a loopback (`lo`) interface that is down and has no way to communicate with the host.

To give the namespace network connectivity, we create a **veth pair** and attach one end to the namespace while keeping the other end in the host network.

This chapter teaches you how to:

- Create a veth pair
- Attach one end to a namespace
- Assign IP addresses
- Enable loopback
- Test connectivity
- Add routes

## 🧩 What is a veth pair?

A **veth pair** (Virtual Ethernet Pair) works like a cable with two ends:

```bash
[ veth-host ] <==== virtual cable ====> [ veth-ns ]
```

- Whatever enters one interface comes out the other.
- One end stays in the host network namespace.
- The other end goes inside the custom namespace.

This is how Docker and Kubernetes connect containers to the host.

## 🛠️ Step 1 — Create a Namespace

```bash
sudo ip netns add ns1
```

## 🛠️ Step 2 — Create a veth Pair

```bash
sudo ip link add veth-host type veth peer name veth-ns
```

This creates:

- `veth-host` → host side
- `veth-ns` → to be moved into namespace

## 🛠️ Step 3 — Move One End into the Namespace

```bash
sudo ip link set veth-ns netns ns1
```

Now:

- `veth-host` is visible on host
- `veth-ns` is visible only inside ns1

Verify from host:

```bash
ip link
```

Verify inside namespace:

```bash
sudo ip netns exec ns1 ip link
```

## 🛠️ Step 4 — Assign IP Addresses

### On host:

```bash
sudo ip addr add 10.0.0.1/24 dev veth-host
sudo ip link set veth-host up
```

### Inside namespace:

```bash
sudo ip netns exec ns1 ip addr add 10.0.0.2/24 dev veth-ns
sudo ip netns exec ns1 ip link set veth-ns up
```

## 🛠️ Step 5 — Enable Loopback

Inside ns1:

```bash
sudo ip netns exec ns1 ip link set lo up
```

Every namespace should always have loopback enabled.

## 🛠️ Step 6 — Add a Route in the Namespace

For namespaces to talk to networks beyond the `10.0.0.0/24` subnet, we add a **default route**:

```bash
sudo ip netns exec ns1 ip route add default via 10.0.0.1
```

This means:
- Send everything to the host's veth interface (`veth-host`).

### Why Add a Route?
Without adding the route, the system doesn't know how to reach 10.0.0.2. When we ping 10.0.0.2, the system checks its routing table to determine where to send the ping packets. If there's no specific route for 10.0.0.2, it won't know which interface to use.

By adding the route, we are explicitly telling the system that to reach 10.0.0.2, it should send the traffic through the `veth-host` interface, which is part of the veth pair connected to the `ns1` namespace.

## 🧪 Step 7 — Test Connectivity

### From host → namespace

```bash
ping 10.0.0.2
```

### From namespace → host

```bash
sudo ip netns exec ns1 ping 10.0.0.1
```

If both work, your namespace is successfully connected to the host.

## 📘 Visual Diagram

```bash
+--------------------+              +------------------------+
|     Host Network   |              |   ns1 Network Namespace|
|                    |              |                        |
| 10.0.0.1           |              |   10.0.0.2             |
|   veth-host        |<============>| veth-ns                |
|                    |   veth pair  |                        |
+--------------------+              +------------------------+
```

## 🛠️ Bonus: Clean Up

```bash
sudo ip link delete veth-host
sudo ip netns del ns1
```

## 📝 Troubleshooting

### ✔ veth-ns not found?

Ensure it was moved to ns1:

```bash
sudo ip netns exec ns1 ip link
```

### ✔ Ping fails?

Check link status on both sides:

```bash
ip link show veth-host
sudo ip netns exec ns1 ip link show veth-ns
```

Ensure routing is correct:

```bash
sudo ip netns exec ns1 ip route
```

### ✔ Forgot to enable loopback?

```bash
sudo ip netns exec ns1 ip link set lo up
```

## 🎯 Summary

By completing this chapter, you learned how to:

✔ Create a network namespace \
✔ Create a veth pair \
✔ Attach one end to the namespace \
✔ Configure host + ns IPs \
✔ Enable loopback \
✔ Add default routes \
✔ Test connectivity

This foundation is necessary for all upcoming chapters, including:

- Connecting namespaces to root
- Enabling egress traffic
- Bridging multiple namespaces
- Simulating routers and topologies