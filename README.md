# Lab 02 — Wireshark & Network Traffic Analysis
> **Wireshark (Free) · Local Machine or Azure VM · Network Protocol Analysis**

![Platform](https://img.shields.io/badge/Platform-Windows%20%7C%20macOS%20%7C%20Linux-0078D4?style=flat-square&logo=linux)
![Tool](https://img.shields.io/badge/Tool-Wireshark-1679A7?style=flat-square&logo=wireshark)
![Cost](https://img.shields.io/badge/Cost-%240%20Permanently%20Free-brightgreen?style=flat-square)
![Duration](https://img.shields.io/badge/Duration-2--4%20Hours-yellow?style=flat-square)
![Certs](https://img.shields.io/badge/Cert%20Alignment-Network%2B%20%7C%20Security%2B%20%7C%20CySA%2B-blueviolet?style=flat-square)

---

## Overview

This lab builds hands-on proficiency with **Wireshark** — the industry-standard open source packet analyser used by network engineers, SOC analysts, and incident responders worldwide. You will capture live traffic, apply professional-grade display filters, read TCP handshakes, inspect DNS resolution in real time, and observe cleartext credentials moving across an unencrypted HTTP connection.

The mental model you build here transfers directly to cloud-native tools: **Azure Network Watcher**, **AWS VPC Flow Logs**, and **GCP Packet Mirroring** all surface the same underlying concepts — source, destination, protocol, port, and payload.

---
## 🎬 Watch Me Build This Lab!
https://www.loom.com/share/cacc9a9f549d4e118857be49c2d3fd7d
## Architecture — How Wireshark Captures Traffic

```
                    ┌──────────────────────────────────────────────────┐
                    │               Your Local Machine                  │
                    │                                                    │
                    │   ┌──────────────┐      ┌─────────────────────┐  │
                    │   │   Browser /  │      │      Wireshark       │  │
                    │   │   Terminal   │      │                      │  │
                    │   └──────┬───────┘      │  ┌───────────────┐  │  │
                    │          │              │  │ Packet List   │  │  │
                    │          │              │  │ ──────────── │  │  │
                    │          ▼              │  │ Packet Detail │  │  │
                    │   ┌──────────────┐      │  │ ──────────── │  │  │
                    │   │  NIC (in     │─────►│  │ Packet Bytes  │  │  │
                    │   │ promiscuous  │      │  └───────────────┘  │  │
                    │   │   mode)      │      └─────────────────────┘  │
                    │   └──────┬───────┘                               │
                    └──────────┼────────────────────────────────────────┘
                               │
                    ┌──────────▼────────────────────────────────────────┐
                    │                  Network Switch                    │
                    └──────────┬───────────────────┬────────────────────┘
                               │                   │
                    ┌──────────▼───────┐  ┌────────▼──────────────────┐
                    │   DNS Server     │  │      Internet / Gateway    │
                    │  (resolves names │  │                            │
                    │   to IPs)        │  │  google.com · example.com  │
                    └──────────────────┘  └────────────────────────────┘

  What Wireshark sees at the NIC:
  ┌─────────────────────────────────────────────────────────────────────┐
  │  Frame → Ethernet Header → IP Header → TCP/UDP Header → Payload    │
  │                                                                     │
  │  Layer 2         Layer 3        Layer 4            Layer 7         │
  │  (MAC addr)     (IP addr)      (Port / Flags)    (HTTP/DNS data)  │
  └─────────────────────────────────────────────────────────────────────┘
```

```
TCP Three-Way Handshake (captured in Exercise B):

  Your Machine                              Server (example.com)
       │                                           │
       │──────── SYN ─────────────────────────────►│  "I want to connect"
       │                                           │
       │◄──────── SYN-ACK ────────────────────────│  "Request received, accepted"
       │                                           │
       │──────── ACK ─────────────────────────────►│  "Connection open, ready"
       │                                           │
       │═══════ Data Exchange Begins ══════════════│
       │                                           │

  Failure patterns to recognise:
  ┌────────────────────────────────────────────────────────────────────┐
  │  SYN → [no SYN-ACK]    = Server unreachable or port filtered      │
  │  SYN → RST             = Connection actively refused               │
  │  SYN → SYN-ACK → RST  = Connection accepted then forcibly closed  │
  └────────────────────────────────────────────────────────────────────┘
```

```
DNS Resolution Flow (captured in Exercise A):

  Your Machine          Local DNS Resolver         Authoritative DNS
       │                       │                          │
       │── Query: A google.com─►│                          │
       │                       │── Recursive query ───────►│
       │                       │◄─ Answer: 142.250.80.46 ──│
       │◄─ Response: 142.250.80.46                         │
       │                       │                          │
       │  Now connects directly to 142.250.80.46           │

  In Wireshark (dns filter):
  ┌─────────────────────────────────────────────────────────────────────┐
  │  Src           Dst         Proto  Info                             │
  │  192.168.1.5   8.8.8.8     DNS    Standard query A google.com     │
  │  8.8.8.8       192.168.1.5 DNS    Standard query response A ...   │
  └─────────────────────────────────────────────────────────────────────┘
```

```
HTTP vs HTTPS — What Wireshark can see:

  HTTP (unencrypted):
  ┌──────────────────────────────────────────────────────────┐
  │  POST /login HTTP/1.1                                    │
  │  Host: example.com                                       │
  │  Content-Type: application/x-www-form-urlencoded         │
  │                                                          │
  │  username=admin&password=Welcome123   ◄── VISIBLE        │
  └──────────────────────────────────────────────────────────┘

  HTTPS (TLS encrypted):
  ┌──────────────────────────────────────────────────────────┐
  │  TLSv1.3 Record Layer: Application Data                  │
  │  Encrypted Data: a3f8c2d1e4b7... ◄── UNREADABLE          │
  └──────────────────────────────────────────────────────────┘
```

---

## Career Relevance

| Role | How This Lab Applies |
|---|---|
| **Network Engineer** | Diagnose connectivity issues by seeing exactly where packets are dropped or delayed |
| **SOC Analyst** | Identify malicious traffic patterns, extract indicators of compromise from packet captures |
| **Cloud Security Engineer** | Mental model transfers directly to Azure Network Watcher and AWS VPC Flow Logs |
| **Help Desk** | Prove that a reported network issue is real and determine whether it is client-side or server-side |
| **Incident Responder** | Reconstruct the full conversation between two hosts to understand what data was transferred |

---

## Prerequisites

| Requirement | Details |
|---|---|
| **Wireshark** | [wireshark.org/download.html](https://www.wireshark.org/download.html) — free, no account required |
| **Npcap** (Windows only) | Bundled with the Wireshark Windows installer — accept when prompted |
| **ChmodBPF** (macOS only) | Allow when prompted during install — grants interface access permission |
| **Terminal access** | Windows: `cmd` · macOS: `Terminal` · Linux: `Ctrl+Alt+T` |
| **Active network interface** | Ethernet or Wi-Fi — you will capture your own live traffic |

---

## Installation

**Windows**
```
1. Download the Windows x64 Installer from wireshark.org
2. Run the installer — accept all defaults
3. Install Npcap when prompted (required for packet capture)
4. Launch Wireshark from the Start menu
```

**macOS**
```
1. Download the macOS .dmg (Arm or Intel) from wireshark.org
2. Run the installer — allow ChmodBPF when prompted
3. Launch Wireshark from Applications
```

**Linux (Ubuntu/Debian)**
```bash
sudo apt install wireshark

# Add yourself to the wireshark group (required to capture without root)
sudo usermod -aG wireshark $USER

# Log out and back in, then verify:
wireshark --version
```

---

## Key Concepts

| Concept | What It Means |
|---|---|
| **Packet** | A small unit of data with a header (src IP, dst IP, port) and a payload (the actual data). All network communication is broken into packets. |
| **Protocol** | A ruleset defining how data is formatted and transmitted. DNS, HTTP, TCP, and ICMP are all protocols with distinct packet structures. |
| **TCP Three-Way Handshake** | The SYN → SYN-ACK → ACK sequence that establishes every TCP connection before data flows. |
| **DNS** | Translates domain names (google.com) into IP addresses (142.250.80.46). Happens before every network action. |
| **HTTP vs HTTPS** | HTTP is unencrypted — credentials are visible in plaintext. HTTPS encrypts the payload with TLS. |
| **Promiscuous Mode** | NIC captures all packets on the segment, not just those addressed to your machine. Wireshark enables this automatically. |
| **Display Filter** | Applied after capture to limit what you see — the full capture remains intact underneath. |
| **Capture Filter** | Applied before capture to limit what gets recorded. For this lab, always use display filters instead. |

---

## Lab Steps

### Step 1 — Install Wireshark

Follow the installation instructions above for your OS. Verify with:

```bash
wireshark --version
```

---

### Step 2 — Your First Capture

1. Open Wireshark — the welcome screen shows all available network interfaces with live traffic graphs
2. Double-click your active interface (Ethernet or Wi-Fi — pick the one with the most wave activity)
3. Wireshark begins capturing immediately — packets appear in the list in real time
4. Open a browser and navigate to any website
5. After 30 seconds, click the **red square Stop** button in the toolbar

You now have a raw packet capture in memory containing every frame that crossed your interface during that window. The volume can be overwhelming — this is exactly why display filters are essential.

---

### Step 3 — Essential Display Filters

Type filters into the filter bar at the top of the Wireshark window and press **Enter**. The packet list updates instantly.

| Filter | What It Shows | When to Use |
|---|---|---|
| `dns` | All DNS queries and responses | Troubleshooting name resolution, spotting unusual domain lookups |
| `http` | Unencrypted HTTP traffic only | Finding cleartext data, debugging web apps |
| `tcp` | All TCP traffic | Starting point for any connectivity investigation |
| `tcp.flags.syn == 1` | TCP SYN packets — connection attempts | Seeing which hosts are attempting to connect to what |
| `tcp.flags.reset == 1` | TCP RST packets — connection resets | Finding refused or forcibly terminated connections |
| `icmp` | All ICMP traffic including ping | Verifying basic reachability between hosts |
| `ip.addr == 192.168.1.1` | All traffic to or from a specific IP | Isolating a single host in a busy capture |
| `ip.src == 10.0.0.5` | Traffic from a specific source IP only | Isolating outbound traffic from one host |
| `tcp.port == 443` | All HTTPS traffic by port | Identifying encrypted web traffic |
| `http.request` | HTTP GET and POST requests only | Spotting web requests and potential data exfiltration |

---

### Step 4 — Guided Exercises

#### Exercise A — Capture a DNS Lookup

> **Goal:** Watch your machine ask a DNS server for an IP address and receive the answer — in real time.

```
Workflow:
1. Start capture in Wireshark
2. Switch to a terminal window on your local machine
3. Run the DNS query command
4. Switch back to Wireshark and stop the capture
5. Apply the dns filter and analyse
```

```bash
# Windows (Command Prompt) / macOS / Linux (Terminal)
nslookup google.com
```

**In Wireshark after stopping the capture:**
1. Apply filter: `dns`
2. Find the **query** packet → Info column shows: `Standard query A google.com`
3. Find the **response** packet → Info column shows: `Standard query response A google.com`
4. Click the response packet → expand `Domain Name System (response)` → expand `Answers`
5. Confirm the IP address matches what `nslookup` returned in your terminal

> 💡 **What this means in the real world:** This invisible lookup happens before every website visit, API call, and email. Unexpected DNS queries to unusual domains in a packet capture are often the **first indicator of malware calling home** to a command-and-control server.

---

#### Exercise B — Watch the TCP Three-Way Handshake

> **Goal:** See the SYN → SYN-ACK → ACK connection sequence in captured packets.

1. Start a new capture on your active interface
2. Open a browser and navigate to `http://example.com` (HTTP — not HTTPS — makes the handshake easier to inspect)
3. Stop the capture
4. Run `nslookup example.com` in a terminal to get the server's IP address
5. Apply filter: `tcp and ip.addr == [IP from nslookup]`
6. Find the three sequential packets:

| Packet | Flags | Meaning |
|---|---|---|
| 1st | `SYN` | Your machine: *I want to connect. Here is my sequence number.* |
| 2nd | `SYN, ACK` | Server: *I received your request. Connection accepted.* |
| 3rd | `ACK` | Your machine: *Confirmed. Connection is open. Ready to send data.* |

**Failure patterns to recognise:**

| Pattern | Diagnosis |
|---|---|
| `SYN` → no `SYN-ACK` | Server unreachable or port is filtered by a firewall |
| `SYN` → `RST` | Connection actively refused by the server |
| `SYN` → `SYN-ACK` → `RST` | Connection accepted then forcibly closed |

---

#### Exercise C — Spot Cleartext Credentials in HTTP

> ⚠️ **Educational exercise only.** Only capture on networks and systems you own or have explicit written permission to analyse.

1. Start a capture on your active interface
2. Submit a login form over an **HTTP** (not HTTPS) test site with a test username and password
3. Stop the capture
4. Apply filter: `http.request.method == POST`
5. Click the POST packet → in the packet detail pane, find the **HTML Form URL Encoded** layer
6. Observe the username and password in **plaintext**

> 💡 **What this demonstrates:** Without TLS encryption, anyone on the network path — your ISP, a coffee shop router, or an attacker performing a man-in-the-middle attack — can read credentials exactly as typed. This is the hands-on proof that makes the argument for HTTPS irrefutable.

---

#### Exercise D — Follow a Full TCP Stream

> **Goal:** Reconstruct an entire client-server conversation from individual packets.

1. Capture any HTTP traffic (navigate to an HTTP website)
2. Find any HTTP packet in the capture list
3. Right-click it → **Follow → TCP Stream**
4. Wireshark reassembles all packets from that connection into a readable conversation:
   - **Red text** = your browser's request
   - **Blue text** = the server's response

> 💡 **Real-world use:** Individual packets are fragments. Stream reconstruction shows the **complete conversation** — what data was transferred, what commands were sent, and what the server returned. This is how incident responders determine the scope of a breach from a packet capture.

---

### Step 5 — Save and Export Captures

```bash
# Save the full capture
File → Save As → .pcapng format (recommended)

# Export only packets matching your current display filter
# Apply your filter first, then:
File → Export Specified Packets → Displayed

# Reopen a saved capture
File → Open → select your .pcapng file
```

**Command-line capture with tshark** (included with Wireshark — useful on remote or headless servers):
```bash
# Capture 1000 packets on eth0 and write to file
tshark -i eth0 -w capture.pcapng -c 1000

# -i  = interface name
# -w  = output file path
# -c  = stop after this many packets
```

---

## Verification Checklist

| Skill | How to Verify |
|---|---|
| **DNS capture** | Apply `dns` filter and identify a query and response packet with matching transaction IDs |
| **TCP handshake** | Find three sequential SYN → SYN-ACK → ACK packets and explain what each means without referring to notes |
| **Display filters** | Filter by IP address, port, and protocol from memory — no reference needed |
| **Stream reconstruction** | Follow a TCP stream and read the full HTTP request and response as a conversation |
| **File management** | Save a capture, close Wireshark, reopen it, load the file — confirm all packets are present |

---

## Troubleshooting

| Problem | Fix |
|---|---|
| No interfaces listed on the welcome screen | Npcap (Windows) or ChmodBPF (macOS) was not installed. Reinstall Wireshark and accept the Npcap prompt, or allow ChmodBPF on macOS |
| Linux: permission denied when capturing | Run `sudo usermod -aG wireshark $USER`, then log out and back in |
| Too many packets — can't find anything | Apply a display filter immediately. Start with `dns` or `http` to narrow the view |
| DNS packets not appearing | Confirm you ran `nslookup` **after** starting the capture, not before |
| Cannot see HTTP POST credentials | Confirm the site is `http://` not `https://`. HTTPS content is encrypted and unreadable in Wireshark |
| TCP handshake packets not found | Run `nslookup example.com` first to get the correct IP, then filter with `tcp and ip.addr == [that IP]` |
| Wireshark shows no traffic at all | Confirm you selected the correct interface — choose the one with wave activity on the welcome screen |

---

## Portfolio Artifacts — Save These

Three captures to save and include in this repository:

| Capture File | What to Include in the Description |
|---|---|
| `dns-lookup.pcapng` | The query packet, the response packet, the transaction ID that links them, and the returned IP |
| `tcp-handshake.pcapng` | The three-packet SYN/SYN-ACK/ACK sequence with an explanation of each flag |
| `http-stream.pcapng` | A followed TCP stream showing the full HTTP request and server response |

---

## Skills Demonstrated

- ✅ Installing and configuring Wireshark across Windows, macOS, and Linux
- ✅ Capturing live network traffic on active interfaces
- ✅ Applying professional display filters to isolate relevant traffic
- ✅ Reading and interpreting DNS query and response packets
- ✅ Identifying TCP three-way handshake sequences and connection failure patterns
- ✅ Extracting cleartext credentials from unencrypted HTTP traffic
- ✅ Reconstructing full client-server conversations using TCP stream follow
- ✅ Saving, exporting, and managing packet capture files (.pcapng)
- ✅ Using tshark for command-line and remote server capture

---

## Certification Alignment

| Certification | Relevant Domains |
|---|---|
| **CompTIA Network+** | Network protocols, TCP/IP model, DNS, packet analysis |
| **CompTIA Security+** | Network security, protocol vulnerabilities, traffic analysis |
| **CompTIA CySA+** | Threat and vulnerability management, network traffic analysis, incident response |

---

*Part of an ongoing home lab series documenting enterprise IT and cloud security skills.*
