# 📘 Linux `netns` Handbook

<i>A practical handbook for learning, experimenting, and mastering Linux Network Namespaces.</i>

Linux network namespaces provide isolated network stacks — each with its own interfaces, routing tables, firewall rules, processes, and DNS settings. This repository is a structured, hands-on guide to understanding, inspecting, and building real network topologies using namespaces, veth pairs, bridges, routing, and related tools.

This project is designed for:

- DevOps & SRE engineers
- Network engineers exploring Linux virtualization
- Students learning container networking
- Anyone who wants to understand how tools like Docker, Kubernetes, Podman, etc., use namespaces under the hood

## 🚀 What You Will Learn

Through step-by-step `.md` documents and reproducible commands, you will learn:

- How to inspect and interact with network namespaces
- How to connect namespaces to the host system
- How virtual Ethernet (veth pairs) work
- How bridges enable multi-namespace networks
- How to create custom routing between namespaces
- How processes communicate across namespace boundaries
- How to build full virtual network topologies

This repository takes a <b>lab-first approach</b>, where each concept is followed by practical examples you can run on any Linux machine.

## 📂 Folder Structure

```bash

linux-netns-handbook/
│
├── README.md
│
├── docs/
│   ├── 01-introduction-to-namespaces.md
│   ├── 02-network-namespace-inspecting.md
│   ├── 03-connect-network-ns-to-host.md
│   ├── 04-connect-network-ns-to-root.md
│   ├── 05-egress-traffic.md
│   ├── 06-connect-two-custom-network-ns.md
│   ├── 07-bridge-networking-among-namespaces.md
│   ├── 08-process-communication-between-namespaces.md
│   ├── 09-fib-network-topology.md
│   ├── 10-common-tools-and-commands.md
│   ├── 11-troubleshooting-network-ns.md
│   └── extras/
│       ├── virtual-router-using-namespaces.md
│       ├── simulate-internet-topology.md
│       └── dns-inside-namespaces.md
│
└── diagrams/

```

## 📝 Included Topics

Each topic will be a dedicated `.md` file inside the `docs/` folder:

1. <b>NETWORK NAMESPACE INSPECTING</b><br>
How to inspect namespaces, interfaces, routes, and processes.

2. <b>CONNECT NETWORK NS TO HOST</b><br>
Using veth pairs to connect a namespace to the host networking.

3. <b>CONNECT NETWORK NS TO ROOT</b><br>
Connecting namespaces to default routes and enabling outbound traffic.

4. <b>EGRESS TRAFFIC</b><br>
Understanding NAT, iptables/iptables-nft, and routing egress packets.

5. <b>CONNECT TWO CUSTOM NETWORK NS</b><br>
Setting up veth pairs or bridge-based connectivity between isolated namespaces.

6. <b>BRIDGE NETWORKING AMONG NAMESPACES</b><br>
Multi-node virtual networks using Linux bridge (brctl, ip link add bridge).

7. <b>PROCESS COMMUNICATION BETWEEN NAMESPACES</b><br>
Sharing/isolating processes, using ip netns exec, and IPC considerations.

8. <b>FIB NETWORK TOPOLOGY</b><br>
Forwarding Information Base and how routing tables differ across namespaces.

## ➕ Additional Recommended Topics

To make this the definitive handbook, consider adding:

- <b>namespace lifecycle & automation</b>
    - Creating, deleting, and scripting namespaces
    - Using systemd for persistent namespace setups

- <b>firewalling inside namespaces</b>
    - Using `iptables` / `nftables` inside separate namespaces

- <b>traffic shaping & QoS</b>
    - Using `tc` inside namespaces (limit bandwidth, latency simulation)

- <b>routing daemons & virtual routers</b>
    - FRRouting / Bird inside namespaces
    - Turning namespaces into routers

- <b>simulating multi-node clusters</b>
    - Kubernetes-style networks
    - CNI plugin behavior explained via namespaces

- <b>DNS inside namespaces</b>
    - Custom resolv.conf
    - Running dnsmasq per namespace

## 🛠️ Requirements

- Linux system (Ubuntu, Debian, Fedora, Arch, etc.)
- `iproute2` tools (`ip`, `ss`, `bridge`, etc.)
- Optional: `tcpdump`, `wireshark`, `iptables-nft`, `nft`, `ethtool`

## ▶️ How to Use This Repository

1. Clone the repo:
    ```bash
      git clone https://github.com/RatulSaqibKhan/linux-netns-handbook.git
      cd linux-netnshandbook
    ```
2. Open any document in the `docs/` folder and follow the steps.
3. Run experiments directly in your terminal.
4. Check `diagrams/` for visual network topologies.

## 🤝 Contributing

Contributions, corrections, and additional example topologies are welcome!
Feel free to open issues or submit PRs.

## 📄 License

This repository is released under the [MIT License](./LICENSE).