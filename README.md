# Network Traffic Analysis Lab

Hands-on packet capture and analysis lab performed with Wireshark, focused on building the foundational network visibility skills that underpin network engineering, SOC analysis, and cloud security work.

## Watch The Lab Walkthrough Here

https://www.loom.com/share/1108d67c2fc44d7d9ffc2af16457e0e6

## What This Lab Demonstrates

This project demonstrates my ability to capture, filter, and analyze network traffic using Wireshark while connecting packet-level activity to real-world networking and security concepts.

Through this lab, I demonstrated:

* Packet Capture and Analysis — Captured live network traffic and inspected packet headers, protocols, source and destination IP addresses, and ports.
* DNS Analysis — Traced DNS queries and responses to understand how domain names are resolved to IP addresses.
* TCP Connection Analysis — Identified the SYN, SYN-ACK, and ACK sequence used to establish a TCP connection.
* Protocol Filtering — Used Wireshark display filters to isolate DNS, TCP, HTTP, IP, and port-specific traffic during an investigation.
* HTTP Security Analysis — Examined unencrypted HTTP traffic and demonstrated how sensitive information can be exposed when TLS encryption is not used.
* TCP Stream Reconstruction — Reassembled individual packets into a complete client-server conversation using Follow TCP Stream.
* Packet Capture Preservation — Saved network captures in `.pcapng` format for later investigation and repeatable analysis.
* Security-Focused Investigation — Used packet-level evidence to identify communication patterns and evaluate potential security risks within network traffic.

### Tools & Technologies

`Wireshark` · `TCP/IP` · `DNS` · `HTTP` · `TCP` · `Packet Analysis` · `Network Security`


| | |
|---|---|
| **Tool** | Wireshark (free, open source) |
| **Environment** | Local machine or Azure VM |
| **Category** | Network Analysis |
| **Certification alignment** | CompTIA Network+ · Security+ · CySA+ |
| **Cost** | $0 |
| **Time invested** | ~2–4 hours across multiple sessions |
| **Career relevance** | Network Engineer · SOC Analyst · Cloud Security Engineer · Incident Responder |

---

## Overview

Every incident, outage, and intrusion eventually comes down to the same question: *what actually happened on the wire?* This lab builds that skill from the ground up — capturing live traffic, applying display filters to cut through noise, reading a TCP three-way handshake, watching DNS resolve a domain, and reconstructing a full conversation between two hosts from individual packets.

The exercises also include a deliberate demonstration of HTTP transmitting credentials in cleartext — the kind of evidence security teams use to justify enforcing HTTPS.

## Why This Matters

| Role | How this lab applies |
|---|---|
| Network Engineer | Diagnose connectivity issues by seeing exactly where packets are dropped or delayed |
| SOC Analyst | Identify malicious traffic patterns and extract indicators of compromise from packet captures |
| Cloud Security Engineer | The mental model transfers directly to reading Azure Network Watcher logs and VPC flow logs |
| Help Desk / IT Support | Prove a reported network issue is real and isolate whether it's client-side or server-side |

## Architecture

The lab architecture illustrates the path network traffic takes from external resources through the local network and network interface before being captured by Wireshark. Once captured, packets can be filtered, inspected, and reconstructed to analyze DNS resolution, TCP connections, HTTP traffic, and complete network conversations.

<img width="1000" height="560" alt="architecture-diagram" src="https://github.com/user-attachments/assets/8fea7d94-06e6-4d04-8b68-af73132d7b05" />

---
## Skills Demonstrated

- Capturing live traffic on an active network interface
- Writing and applying Wireshark display filters (`dns`, `http`, `tcp`, `icmp`, `tcp.flags.syn == 1`, `ip.addr ==`, etc.)
- Reading and validating a TCP three-way handshake (SYN → SYN-ACK → ACK)
- Identifying DNS queries/responses and matching transaction IDs
- Locating cleartext credentials in an HTTP POST to demonstrate why HTTPS is required
- Following a full TCP stream to reconstruct a client–server conversation
- Saving, exporting, and re-opening `.pcapng` capture files for later analysis
- Command-line packet capture with `tshark`

---

## Lab Walkthrough

### 1. Install Wireshark
Downloaded from [wireshark.org](https://www.wireshark.org/download.html). On Windows this includes Npcap; on macOS, ChmodBPF permissions were granted; on Linux, the user was added to the `wireshark` group to allow non-root capture.

---
### 2. First Capture
Captured live traffic on the active interface while browsing to a website, confirming packets appear in real time and that a 30-second window alone generates hundreds of frames — motivating the need for filters.

**01 — Interface Selections**

Wireshark displayed the available network interfaces and their live activity levels. I selected the active interface to begin capturing traffic, establishing the data source for the packet analysis performed throughout the lab.

<img width="916" height="801" alt="01-interface-lists" src="https://github.com/user-attachments/assets/d3ff42ee-3e6a-4b32-8156-7da8f3c74a23" />

---
**02 — First Live Packet Capture**

Live packet capture confirmed that Wireshark was successfully monitoring traffic from the selected network interface. The volume and variety of packets demonstrated how quickly network activity accumulates and why display filters are essential for isolating relevant traffic.

<img width="1359" height="679" alt="02-first-capture" src="https://github.com/user-attachments/assets/d201fa6d-671e-4663-b977-b70a7d4f8072" />

---
### 3. Display Filters
Applied and validated the core filter set used throughout the lab:

| Filter | Purpose |
|---|---|
| `dns` | Isolate DNS queries and responses |
| `http` | Isolate unencrypted HTTP traffic |
| `tcp` | Baseline for connectivity investigations |
| `tcp.flags.syn == 1` | Show connection attempts (SYN packets) |
| `tcp.flags.reset == 1` | Show refused/forcibly closed connections |
| `icmp` | Verify basic host reachability |
| `ip.addr == x.x.x.x` | Isolate all traffic to/from a specific host |
| `tcp.port == 443` | Isolate encrypted (HTTPS) traffic |
| `http.request` | Isolate HTTP GET/POST requests |

---
### 4. Guided Exercises

**03 — DNS Query and Response**

I generated DNS traffic using nslookup google.com and captured the resulting packets in Wireshark. The capture shows the DNS lookup process in which the client requests the IP address associated with a domain and receives a response from the DNS server.

<img width="735" height="630" alt="03-dns-query-response" src="https://github.com/user-attachments/assets/bd324b8a-cd7c-4263-81c0-97c78ceb76dd" />

**03.1 — Filtered DNS Analysis**

Applying the dns display filter isolated DNS traffic from the larger packet capture. This made it possible to focus on the query and response packets and verify how domain-name resolution appears at the packet level.

<img width="1364" height="703" alt="03 1-dns-query-filtered-response" src="https://github.com/user-attachments/assets/f0dd49de-e157-4505-a7d2-59dbfea29485" />

---
**04 — TCP Three-Way Handshake**

I isolated a TCP connection and identified the SYN, SYN-ACK, and ACK packets that form the TCP three-way handshake. This sequence confirms that the client and server successfully established a reliable connection before application data was exchanged.

<img width="1446" height="657" alt="04-tcp-handshake" src="https://github.com/user-attachments/assets/dc0078ab-8fef-4e4c-b7ad-2736be194390" />

---
**05 — HTTP Cleartext Credentials**

This controlled exercise demonstrates the security risk of transmitting authentication data over unencrypted HTTP. By inspecting the HTTP POST request, I was able to observe submitted form data within the captured packet, showing why sensitive web traffic should be protected with HTTPS/TLS.

<img width="1366" height="756" alt="05-http-clear-text-credentials" src="https://github.com/user-attachments/assets/a2832170-fee6-4e44-8690-0a5d4c89dbbe" />

---
**06 — Follow TCP Stream**

Using Wireshark's Follow TCP Stream feature, I reconstructed the individual TCP packets into a readable client-server conversation. This demonstrated how packet-level data can be reassembled to understand what information was exchanged during a network session.

<img width="870" height="820" alt="06-tcp-stream-follow" src="https://github.com/user-attachments/assets/6805146a-9bf3-4c7e-a736-d63d0c71fb60" />

---
**07 — Saved Packet Capture**

I saved the completed capture in .pcapng format so the network traffic could be reopened and analyzed later. Preserving packet captures supports repeatable investigations and allows analysts to apply new filters or revisit network activity without reproducing the original traffic.

<img width="1382" height="503" alt="07-saved-capture-file" src="https://github.com/user-attachments/assets/1ba18d68-96ef-4975-bdba-62edd778d48a" />

---
## Verification Checklist

- [x] DNS query and response packets identified with matching transaction IDs
- [x] SYN, SYN-ACK, and ACK packets located and explained
- [x] Traffic filtered by IP, port, and protocol from memory
- [x] Full TCP stream followed and read as a conversation
- [x] Capture saved, closed, and successfully reopened

---
## Disclaimer

All packet captures in this lab were performed on networks and systems I own or have explicit permission to test. Exercise C in particular is included strictly as an educational demonstration of a known protocol weakness (unencrypted HTTP) and was not performed against any third-party or production system.

## Tools & References

- [Wireshark](https://www.wireshark.org/) — packet capture and analysis
- [Wireshark Display Filter Reference](https://www.wireshark.org/docs/dfref/)
- `tshark` — Wireshark's command-line capture utility, useful for remote/headless servers
