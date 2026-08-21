---
title: "June Recap: First Steps into Cybersecurity"
date: 2026-06-21 21:00:00 +0300
tags: [kvm, network-pentest, nmap, metasploitable, dns, adguard-home, linux-administration, wireshark, packet-analysis]
---

June turned out to be packed: from building my first homelab to attending a Summer Cyber School.

## Building a Vulnerable Machine

Instead of deploying vulnerable machines directly on my home network, I designed an isolated architecture based on the Jump Box model.

**Network topology:**
1. **Client / Attacker:** laptop running Arch.
2. **Hypervisor:** a Debian 13 server acting as a bridge.
3. **Isolation:** inside Debian I set up KVM with a fully isolated virtual switch with no internet access.
4. **Victim:** a **Metasploitable 2** virtual machine.

```

🌐 HOME NETWORK (192.168.31.0/24)
│
├── 💻 Laptop (Arch)
│   └─ IP: 192.168.31.X
│
├── 📡 Xiaomi Router
│   └─ IP: 192.168.31.1
│
├── 🛡️ Debian Server (Physical PC)
│   └─ Role: Jump Box (Hypervisor)
│   └─ External IP: 192.168.31.147 (faces the home network)
│   └─ Internal IP: 192.168.100.1 (faces the isolated network)
│
==== 🧱 KVM VIRTUAL FIREWALL ====
│
🔒 KVM ISOLATED NETWORK "isolated" (192.168.100.0/24)
│
└── 🎯 Metasploitable 2 (Virtual Machine)
    └─ Role: Victim
    └─ IP: 192.168.100.240

```

### Network Reconnaissance

After sorting out a clock issue that was preventing gpg keys from updating for `apt`, I scanned Metasploitable using `nmap -sV` (service and version enumeration) from the hypervisor server.

The result was impressive (though Metasploitable is designed for exactly this) — 23 open ports, including:
* Port 21: `vsftpd 2.3.4` (the famous smiley-face backdoor).
* Port 1524: An open `root bindshell`.
* Legacy cleartext protocols (Telnet, RSH).

### Reviewing "The Fun Way To Learn CYBERSECURITY"

The day started with a video from HackersArsenal that fit perfectly with my current learning direction. The core takeaway: *"Tools don't hack systems, misunderstanding does."*

The author emphasizes the importance of understanding:
* **Operating Systems:** Especially Linux (processes, permissions, services).
* **Network Protocols:** IP, DNS, ports, firewalls. This is the foundation without which hacking is just blindly running tools.
* **Web Hacking:** Today's most vulnerable area, where understanding HTTP and sessions is critical.

This approach lined up perfectly with my session the day before, where I manually recovered `/etc/shadow` through a backdoor instead of using automated exploits.

### Homelab Evolution: My Own DNS Server

While browsing GitHub, I stumbled on the `awesome-selfhosted` repo and got curious about the `DNS` category. It seemed like it would be worthwhile to play around with this as part of my homelab work too. Out of the 4 options listed, I liked AdGuard Home the most — a modern, convenient tool with simple setup and a clear interface.

#### AdGuard Home on Debian

Instead of deploying DNS on the main home router, I decided to use my Debian server (`192.168.31.147`), which is connected to the old router.

During installation I ran into the typical issues described in AdGuard's documentation: `systemd-resolved` occupying port 53. I had to disable its internal stub listener via `/etc/systemd/resolved.conf.d/`.

I used the official installation script to set up AdGuard Home as a single Go binary.

Interestingly, during configuration AdGuard threw a `bind: address already in use` error. Using `lsof`, I found out that KVM runs its own `dnsmasq` (port 53) on the `192.168.100.1` interface to hand out addresses to the VMs.
* To resolve this, I configured AdGuard to listen on port 53 *only* on the external interface `eno1` (`192.168.31.147`), leaving KVM's `dnsmasq` alone.

---

## Summer Cyber School

Since June 10th I've been taking part in a Summer Cyber School run by the university. Over a week and a half we covered 2 blocks: Linux (Kali) and Computer Networks.

Lectures and hands-on sessions were structured to fit into 4 hours with a short coffee break.

The first half of each day was fully dedicated to the theoretical foundations of computer networks and operating systems. The amount of information was huge, but it all fit into a clear structure.

### Linux

* **OS History**
* **File Management**
* **Process Management**
* **Piping and Permissions**

I didn't find much new for myself here, since I've been using Linux for a year already. Overall, the material is well put together as a first introduction to Linux and the command line.

### Computer Networks

* **Network Architecture & TCP/IP**: We covered the basic infrastructure components: intermediate nodes (routers, switches) and transmission media.
* **Addressing & Traffic Types**: We went from the physical layer (**MAC**) up to the logical one. We revisited IP address classes, separated out private IP ranges, and covered traffic types: **unicast**, **multicast**, and **broadcast**.
* **Network Services & Security**: We covered how **NAT** works for address translation and **DHCP** for dynamic configuration. A separate, critically important block covered the concept of **AAA (Authentication, Authorization, Accounting)**.
* **Classless Addressing Logic (CIDR)**: We went into detail on masks, prefixes, and the shift from classful to classless addressing.

**Hands-on sessions:**

* **Traffic Analysis in Wireshark**: We did practical exercises capturing and analyzing packets. Through Wireshark we visually broke down **ARP** requests, data encapsulation, and headers, which made network theory click in practice.
* **Network Scanning with Nmap**: Using Nmap we ran reconnaissance on the lab environment. In practice, we learned how to identify active hosts (`-sn`) and running service versions (`-sV`), then looked up CVEs for those service versions.

