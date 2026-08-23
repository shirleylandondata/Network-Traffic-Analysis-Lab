# Lab 2 — Wireshark & Network Traffic Analysis

Hands-on packet capture and analysis lab performed with Wireshark, focused on building the foundational network visibility skills that underpin network engineering, SOC analysis, and cloud security work.

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

The diagram below traces how traffic actually reaches Wireshark: from the internet, through the gateway and local switch, into the NIC in promiscuous mode, and finally into Wireshark itself — where filtering and reassembly turn raw frames into the four analysis outcomes this lab produces.

![Architecture diagram showing traffic flow from the internet through a router and switch into a host machine's network interface in promiscuous mode, captured by Wireshark, and broken into DNS query/response, TCP handshake, cleartext HTTP credentials, and TCP stream analysis](architecture-diagram.svg)

## Skills Demonstrated

- Capturing live traffic on an active network interface
- Writing and applying Wireshark display filters (`dns`, `http`, `tcp`, `icmp`, `tcp.flags.syn == 1`, `ip.addr ==`, etc.)
- Reading and validating a TCP three-way handshake (SYN → SYN-ACK → ACK)
- Identifying DNS queries/responses and matching transaction IDs
- Locating cleartext credentials in an HTTP POST to demonstrate why HTTPS is required
- Following a full TCP stream to reconstruct a client–server conversation
- Saving, exporting, and re-opening `.pcapng` capture files for later analysis
- Command-line packet capture with `tshark`

## Lab Walkthrough

### 1. Install Wireshark
Downloaded from [wireshark.org](https://www.wireshark.org/download.html). On Windows this includes Npcap; on macOS, ChmodBPF permissions were granted; on Linux, the user was added to the `wireshark` group to allow non-root capture.

### 2. First Capture
Captured live traffic on the active interface while browsing to a website, confirming packets appear in real time and that a 30-second window alone generates hundreds of frames — motivating the need for filters.

**Interface selection screen with live activity graphs**
<img width="916" height="801" alt="01-interface-lists" src="https://github.com/user-attachments/assets/d3ff42ee-3e6a-4b32-8156-7da8f3c74a23" />

**Packets populating in real time during the first capture**
<img width="1359" height="679" alt="02-first-capture" src="https://github.com/user-attachments/assets/d201fa6d-671e-4663-b977-b70a7d4f8072" />



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

### 4. Guided Exercises

**A — DNS Lookup:** Ran `nslookup google.com` in a terminal while capturing, then filtered on `dns` to locate the A-record query and its matching response, confirming the resolved IP against the terminal output.

**DNS query response**
<img width="735" height="630" alt="03-dns-query-response" src="https://github.com/user-attachments/assets/bd324b8a-cd7c-4263-81c0-97c78ceb76dd" />

<img width="1364" height="703" alt="03 1-dns-query-filtered-response" src="https://github.com/user-attachments/assets/f0dd49de-e157-4505-a7d2-59dbfea29485" />

**B — TCP Three-Way Handshake:** Captured a connection to `http://example.com`, filtered to the target IP, and identified the SYN → SYN-ACK → ACK sequence that establishes every TCP connection.

**Three packets showing SYN, SYN-ACK, ACK flags**
<img width="1446" height="657" alt="04-tcp-handshake" src="https://github.com/user-attachments/assets/dc0078ab-8fef-4e4c-b7ad-2736be194390" />

**C — Cleartext Credentials (HTTP):** *Educational exercise, performed only against a system I own/control.* Submitted a test login over HTTP and filtered on `http.request.method == POST` to view the submitted username and password in plaintext inside the HTML Form URL Encoded layer — direct evidence of why HTTPS is mandatory for any page handling credentials.

**POST packet with HTML Form URL Encoded layer expanded (redact the actual credential value before publishing)**
<img width="1366" height="756" alt="05-http-clear-text-credentials" src="https://github.com/user-attachments/assets/a2832170-fee6-4e44-8690-0a5d4c89dbbe" />

**D — Follow TCP Stream:** Used *Follow → TCP Stream* on an HTTP conversation to reassemble the full client/server exchange into readable form, the same technique used in real incident response to reconstruct what data was transferred during an event.

**Reassembled stream window, red request / blue response**
<img width="870" height="820" alt="06-tcp-stream-follow" src="https://github.com/user-attachments/assets/6805146a-9bf3-4c7e-a736-d63d0c71fb60" />


### 5. Save & Export
Captures saved in `.pcapng` format via *File → Save As*, with filtered subsets exported via *File → Export Specified Packets → Displayed*.

**Save As dialog or the saved `.pcapng` file in your file browser**
<img width="1382" height="503" alt="07-saved-capture-file" src="https://github.com/user-attachments/assets/1ba18d68-96ef-4975-bdba-62edd778d48a" />

## Verification Checklist

- [x] DNS query and response packets identified with matching transaction IDs
- [x] SYN, SYN-ACK, and ACK packets located and explained
- [x] Traffic filtered by IP, port, and protocol from memory
- [x] Full TCP stream followed and read as a conversation
- [x] Capture saved, closed, and successfully reopened

## Evidence / Portfolio Artifacts

> ⚠️ Capture files may contain personal browsing data (IPs, hostnames, and in Exercise C, credentials from a test-only account). Review and redact before publishing any `.pcapng` files publicly.

- `captures/dns-lookup.pcapng` — DNS query/response for `google.com`
- `captures/tcp-handshake.pcapng` — Full three-way handshake to `example.com`
- `captures/tcp-stream-follow.pcapng` — Reassembled HTTP conversation
- `screenshots/` — Annotated screenshots of each exercise (see [Repository Structure](#repository-structure))
- [`video-script.md`](video-script.md) — Script for a recorded walkthrough of this lab

## Disclaimer

All packet captures in this lab were performed on networks and systems I own or have explicit permission to test. Exercise C in particular is included strictly as an educational demonstration of a known protocol weakness (unencrypted HTTP) and was not performed against any third-party or production system.

## Tools & References

- [Wireshark](https://www.wireshark.org/) — packet capture and analysis
- [Wireshark Display Filter Reference](https://www.wireshark.org/docs/dfref/)
- `tshark` — Wireshark's command-line capture utility, useful for remote/headless servers
