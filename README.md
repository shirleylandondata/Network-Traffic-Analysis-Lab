# Wireshark & Network Traffic Analysis Lab

Hands-on packet capture and analysis lab performed with Wireshark, focused on building the foundational network visibility skills that underpin network engineering, SOC analysis, and cloud security work.

---

## Overview

Every incident, outage, and intrusion eventually comes down to the same question: *what actually happened on the wire?* This lab builds that skill from the ground up — capturing live traffic, applying display filters to cut through noise, reading a TCP three-way handshake, watching DNS resolve a domain, and reconstructing a full conversation between two hosts from individual packets.

The exercises also include a deliberate demonstration of HTTP transmitting credentials in cleartext — the kind of evidence security teams use to justify enforcing HTTPS.

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

**B — TCP Three-Way Handshake:** Captured a connection to `http://example.com`, filtered to the target IP, and identified the SYN → SYN-ACK → ACK sequence that establishes every TCP connection.

**C — Cleartext Credentials (HTTP):** *Educational exercise, performed only against a system I own/control.* Submitted a test login over HTTP and filtered on `http.request.method == POST` to view the submitted username and password in plaintext inside the HTML Form URL Encoded layer — direct evidence of why HTTPS is mandatory for any page handling credentials.

**D — Follow TCP Stream:** Used *Follow → TCP Stream* on an HTTP conversation to reassemble the full client/server exchange into readable form, the same technique used in real incident response to reconstruct what data was transferred during an event.

### 5. Save & Export
Captures saved in `.pcapng` format via *File → Save As*, with filtered subsets exported via *File → Export Specified Packets → Displayed*.

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

## Disclaimer

All packet captures in this lab were performed on networks and systems I own or have explicit permission to test. Exercise C in particular is included strictly as an educational demonstration of a known protocol weakness (unencrypted HTTP) and was not performed against any third-party or production system.

## Tools & References

- [Wireshark](https://www.wireshark.org/) — packet capture and analysis
- [Wireshark Display Filter Reference](https://www.wireshark.org/docs/dfref/)
- `tshark` — Wireshark's command-line capture utility, useful for remote/headless servers
