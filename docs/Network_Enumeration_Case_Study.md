# Network Enumeration Case Study: Investigating My MacBook and Mobile Hotspot Network

## Introduction

While learning Nmap and networking fundamentals, I decided to perform a practical investigation on my own network instead of simply memorizing commands. The goal was to understand how devices communicate, how networks are structured, how host discovery works, and how to identify services running on a machine.

This investigation was performed on a MacBook connected to a mobile hotspot network.

---

# Objective

The objectives of this investigation were:

* Identify the active network interface.
* Determine the local IP address.
* Understand subnet masks and network ranges.
* Discover devices connected to the same network.
* Perform host discovery using Nmap.
* Identify open ports.
* Map open ports to running processes.
* Determine which applications were responsible for those ports.

---

# Step 1: Identifying the Active Network Interface

The first step was determining which network interface was currently being used by the system.

Command:

```bash
ifconfig
```

Important result:

```text
en0
inet 10.49.88.34
netmask 0xffffff00
broadcast 10.49.88.255
status: active
```

### Analysis

On Linux systems, wireless interfaces are commonly named:

```text
wlan0
wlp2s0
wlp3s0
```

However, macOS uses different naming conventions:

```text
en0
en1
en2
```

The interface `en0` was active and had an IPv4 address assigned.

Therefore:

```text
Active Interface = en0
IP Address = 10.49.88.34
```

---

# Step 2: Finding the Default Gateway

Next, I wanted to identify the device providing network connectivity.

Command:

```bash
netstat -rn | grep default
```

Output:

```text
default 10.49.88.237 UGScg en0
```

### Analysis

The default gateway is the device through which all external traffic is routed.

In this case:

```text
Gateway = 10.49.88.237
```

This was my mobile phone acting as a hotspot.

---

# Step 3: Understanding the Network Range

From the earlier output:

```text
IP Address = 10.49.88.34
Netmask = 255.255.255.0
```

The netmask:

```text
255.255.255.0
```

corresponds to:

```text
/24
```

Therefore:

```text
Network = 10.49.88.0/24
```

This means the network contains:

```text
10.49.88.1
through
10.49.88.254
```

A total of 254 usable host addresses.

---

# Step 4: Discovering Nearby Devices

To determine which devices my Mac had recently communicated with, I inspected the ARP cache.

Command:

```bash
arp -a
```

Output:

```text
10.49.88.16
10.49.88.190
10.49.88.237
10.49.88.34
```

### Analysis

ARP maps:

```text
IP Address
      ↓
MAC Address
```

This revealed multiple devices present on the local network.

Observed devices:

| IP Address   | Description    |
| ------------ | -------------- |
| 10.49.88.34  | My MacBook     |
| 10.49.88.237 | Mobile Hotspot |
| 10.49.88.16  | Unknown Device |
| 10.49.88.190 | Unknown Device |

---

# Step 5: Verifying Device Availability

To confirm whether these hosts were reachable, I used ICMP.

Command:

```bash
ping 10.49.88.16
```

Result:

```text
Host responded successfully.
```

Command:

```bash
ping 10.49.88.190
```

Result:

```text
Host responded with high latency and packet loss.
```

Command:

```bash
ping 10.49.88.237
```

Result:

```text
Host responded successfully.
```

### Analysis

This confirmed that the devices discovered through ARP were alive and reachable.

---

# Step 6: Initial Nmap Host Discovery

I then attempted host discovery using Nmap.

Command:

```bash
nmap -sn 10.49.88.0/24
```

Result:

```text
Only 1 host discovered.
```

### Observation

This was unexpected because ARP and ping results showed multiple devices.

### Explanation

Many devices:

* Ignore ICMP probes
* Ignore TCP discovery probes
* Employ power-saving behavior
* Restrict responses to network scans

As a result, Nmap's default host discovery methods may miss devices.

---

# Step 7: ARP-Based Host Discovery

To improve accuracy, I used ARP discovery.

Command:

```bash
sudo nmap -sn -PR 10.49.88.0/24
```

Result:

```text
10.49.88.237
10.49.88.34
```

### Why ARP Works Better

Inside a local network:

```text
ARP is Layer 2
```

and devices generally must respond to ARP requests.

This often makes ARP scanning more reliable than ICMP-based discovery on local networks.

---

# Step 8: Port Enumeration

After identifying active hosts, I performed a port scan.

Command:

```bash
sudo nmap -Pn 10.49.88.0/24
```

Results:

```text
10.49.88.237
Port 53 open

10.49.88.34
Port 5000 open
Port 7000 open
```

---

# Step 9: Understanding Open Ports

## Mobile Hotspot

Host:

```text
10.49.88.237
```

Open Port:

```text
53/tcp
```

### Analysis

Port 53 corresponds to:

```text
DNS
```

The hotspot was functioning as the network gateway and DNS provider for connected devices.

---

## MacBook

Host:

```text
10.49.88.34
```

Open Ports:

```text
5000/tcp
7000/tcp
```

These ports required further investigation.

---

# Step 10: Mapping Ports to Processes

Command:

```bash
lsof -iTCP -sTCP:LISTEN -n -P
```

Result:

```text
ControlCenter
TCP *:5000
TCP *:7000
```

### Analysis

The process responsible for both ports was:

```text
ControlCenter
```

However, I still needed to determine exactly what executable this process represented.

---

# Step 11: Identifying the Executable

Command:

```bash
ps -p 717 -o pid,comm
```

Output:

```text
/System/Library/CoreServices/ControlCenter.app/Contents/MacOS/ControlCenter
```

### Conclusion

The process belonged to:

```text
Apple Control Center
```

This is a legitimate macOS system component.

---

# Step 12: Investigating Running Network Services

To obtain a complete view of active network activity:

Command:

```bash
sudo lsof -i -P -n
```

Interesting findings included:

### ControlCenter

```text
Ports 5000 and 7000
```

Associated with:

* AirPlay
* Screen Mirroring
* Apple ecosystem services

---

### rapportd

```text
Port 50501
```

Associated with:

* AirDrop
* Continuity
* Handoff

---

### mDNSResponder

```text
UDP 5353
```

Associated with:

* Bonjour
* Service Discovery
* AirPlay Discovery

---

### netbiosd

```text
UDP 137
UDP 138
```

Associated with:

* SMB
* Windows network discovery

---

### Notes Application

Observed active encrypted communication:

```text
TCP -> Port 993
```

Port 993 corresponds to:

```text
IMAPS
```

indicating secure synchronization activity.

---

# Lessons Learned

This investigation demonstrated how multiple networking tools complement one another.

Workflow:

```text
ifconfig
    ↓
Find IP Address

netstat
    ↓
Find Gateway

arp -a
    ↓
Find Nearby Devices

ping
    ↓
Confirm Reachability

nmap
    ↓
Discover Hosts & Ports

lsof
    ↓
Identify Processes

ps
    ↓
Identify Executables
```

Each tool answered a different question:

| Tool     | Purpose                   |
| -------- | ------------------------- |
| ifconfig | Network configuration     |
| netstat  | Routing information       |
| arp      | IP-to-MAC mapping         |
| ping     | Reachability testing      |
| nmap     | Host and port discovery   |
| lsof     | Process-to-port mapping   |
| ps       | Executable identification |

---

# Final Conclusion

This exercise transformed a simple Nmap scan into a complete network investigation. Rather than merely executing commands, the focus was on understanding how devices communicate, how networks are structured, how services expose ports, and how to trace those ports back to real applications.

The investigation confirmed:

* The active interface was `en0`.
* The local IP address was `10.49.88.34`.
* The hotspot gateway was `10.49.88.237`.
* Multiple devices existed on the network.
* The hotspot provided DNS services on port 53.
* macOS Control Center exposed ports 5000 and 7000.
* Additional Apple networking services were operating as expected.

This exercise provided a practical foundation in network enumeration, host discovery, service identification, and system-level investigation using Nmap and native macOS networking tools.
