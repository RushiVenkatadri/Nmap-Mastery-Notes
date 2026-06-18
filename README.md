Nmap Mastery Notes: From Zero to Practical Network Recon

Why I Started Learning Nmap

When I first opened Nmap, I thought it was just a tool to scan IP addresses. After spending time using it on my own systems and networks, I realized that Nmap is much more than a scanner. It helps answer questions like:

- What devices are connected to my network?
- Which services are running?
- What ports are exposed?
- How does a device respond to different network probes?
- What can I learn about a target before interacting with it further?

These notes are a collection of concepts, commands, observations, and lessons I learned while exploring Nmap.

---

Understanding the Basics

What is Nmap?

Nmap (Network Mapper) is an open-source network discovery and security auditing tool.

It can be used to:

- Discover hosts
- Identify open ports
- Detect running services
- Estimate operating systems
- Perform network reconnaissance
- Troubleshoot network issues

---

Understanding IP Addresses

Before using Nmap, you should understand:

Your IP Address

On macOS:

ifconfig

Example:

10.49.88.34

This identifies your device on the local network.

---

Gateway

netstat -rn | grep default

Example:

default 10.49.88.237

This is usually your router or hotspot.

---

Network Range

If your netmask is:

255.255.255.0

Then your network is:

10.49.88.0/24

This means there are 254 usable host addresses.

---

Host Discovery

Discover Live Devices

nmap -sn 10.49.88.0/24

Purpose:

- Finds devices that appear online.
- Does not scan ports.

This is usually my first command when exploring a network.

---

ARP Discovery

sudo nmap -sn -PR 10.49.88.0/24

Useful on local networks.

ARP discovery is often more reliable than normal host discovery.

---

Understanding ARP

View known devices:

arp -a

Example:

10.49.88.16
10.49.88.190
10.49.88.237

ARP helps map:

IP Address -> MAC Address

This helps identify physical devices on a network.

---

Port Scanning

Scan Top Ports

nmap 192.168.1.10

Scans common ports.

Example findings:

22/tcp open ssh
80/tcp open http
443/tcp open https

---

Scan All TCP Ports

nmap -p- 192.168.1.10

Scans all 65535 TCP ports.

Useful when performing deeper enumeration.

---

Service Detection

nmap -sV 192.168.1.10

Purpose:

Identify service versions.

Example:

OpenSSH 9.2
Apache 2.4.58

Version information can help during vulnerability research.

---

Operating System Detection

sudo nmap -O 192.168.1.10

Nmap analyzes responses and estimates the operating system.

Results are not always exact but often very useful.

---

Aggressive Scan

sudo nmap -A 192.168.1.10

Includes:

- OS detection
- Version detection
- Script scanning
- Traceroute

Good for learning.

Not always the first choice in real environments.

---

Understanding TTL

While troubleshooting networks I noticed:

TTL 128
TTL 64

Common observations:

- TTL 128 → Often Windows
- TTL 64 → Often Linux, Android, macOS

This is not a guarantee but can provide clues.

---

NSE (Nmap Scripting Engine)

One of the most powerful features in Nmap.

List available scripts:

ls /usr/share/nmap/scripts/

Examples:

nmap --script vuln target

nmap --script http-title target

nmap --script smb-os-discovery target

Scripts can automate many reconnaissance tasks.

---

Practical Learning Workflow

When investigating my own network:

1. Identify my IP address

ifconfig

2. Find the gateway

netstat -rn | grep default

3. View neighboring devices

arp -a

4. Test connectivity

ping target

5. Discover hosts

nmap -sn network

6. Enumerate services

nmap -sV target

7. Investigate further if needed

---

Lessons I Learned

- Ping and Nmap do not always behave the same way.
- Mobile hotspots can hide devices from host discovery.
- ARP tables can reveal devices Nmap initially misses.
- Network troubleshooting requires combining multiple tools.
- Enumeration is about collecting information methodically.
- Understanding networking fundamentals makes Nmap much easier to use.

---

Final Thoughts

Learning Nmap taught me that successful reconnaissance is not about running a single command. It is about understanding how networks communicate and using multiple sources of information to build an accurate picture of what is happening.

The more I practiced on my own systems and lab environments, the more I realized that networking knowledge is just as important as knowing the tool itself.

This repository documents my learning journey and serves as a practical reference for future network enumeration and cybersecurity practice.