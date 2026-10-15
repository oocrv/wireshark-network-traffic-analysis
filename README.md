# Wireshark Network Troubleshooting

A hands-on packet-analysis project using Wireshark on Windows. This repository documents DNS query/response behavior, TCP connection setup and teardown, TLS handshake visibility, and a controlled DNS timeout test.

**Status:** Lab documentation and eight screenshots are published. Findings describe observed evidence and do not claim an unverified production outage or repair.

## Objectives

- Inspect DNS queries and responses using Wireshark display filters.
- Identify the TCP three-way handshake: SYN, SYN-ACK, ACK.
- Inspect visible TLS handshake messages without assuming encrypted application contents are readable.
- Review TCP conversations and selected packet details.
- Compare a deliberately unsuccessful DNS query with a successful query using the system's default resolver.
- Practice evidence-based troubleshooting and document limitations.

## Tools

- Wireshark
- Windows Command Prompt and `nslookup`
- The active Windows network interface

## Lab scenarios

### 1. DNS query and response

A successful `nslookup example.com` query using the system's default DNS resolver returned IPv4 and IPv6 addresses. The resolver was displayed as `fe80::1` with the label `UnKnown`; that label alone does not indicate a DNS failure.

Display filter:

```
dns
```

### 2. TCP connection setup

A captured TCP flow showed the expected three-way handshake:

1. Client to server: SYN
2. Server to client: SYN, ACK
3. Client to server: ACK

Display filter shown in the handshake screenshot:

```
tcp.stream eq 27
```

The screenshot shows the three handshake packets from TCP stream 27. `tcp.flags.syn == 1` is another valid filter for SYN packets, but it is not the filter shown in that screenshot.

### 3. TLS handshake visibility

The capture showed a TLS Client Hello with SNI `example.com` and Server Hello-related traffic. This demonstrates visible handshake messages, not that every TLS handshake stage was independently verified from the screenshot alone.

Display filter:

```
tls.handshake
```

### 4. Controlled DNS timeout

The command below was used to intentionally query a documentation-only address:

```
nslookup example.com 192.0.2.1
```

The command timed out, and the capture showed DNS requests from the local host to `192.0.2.1` without a corresponding response in the observed packets. The address is in the documentation range `192.0.2.0/24`; this is a controlled test, not evidence that the user's normal DNS service is broken.

Display filter shown in the timeout screenshot:

```
dns
```

Optional narrower filter for a separate follow-up inspection (not the filter shown in the screenshot):

```
dns && ip.addr == 192.0.2.1
```

### 5. TCP connection teardown review

One captured flow showed an RST, ACK after FIN-related traffic. A reset flag alone does not prove a network fault. The surrounding stream and application context would be needed before assigning a root cause, so this is documented as an observation rather than a confirmed incident.

Suggested filter for a separate packet-list inspection (not shown in the TCP Conversations screenshot):

```
tcp.flags.reset == 1
```

To inspect retransmissions and resets together in a separate packet-list view:

```
tcp.analysis.retransmission || tcp.flags.reset == 1
```

The TCP Conversations screenshot is a Statistics window, not a packet-list view, and does not show a display-filter bar. It also does not depict frame 1674; the reset is documented as a separate observation.

## Evidence screenshots

The screenshots below are from the lab capture. Select any image to open its full-size version.

| Evidence | Screenshot |
|---|---|
| DNS analysis overview | [dns-analysis.pcapng.png](screenshots/dns-analysis.pcapng.png) |
| DNS query details | [02-dns-query-details.png](screenshots/02-dns-query-details.png) |
| DNS response details | [03-dns-response-details.png](screenshots/03-dns-response-details.png) |
| TCP three-way handshake | [1-tcp-three-way-handshake.png](screenshots/1-tcp-three-way-handshake.png) |
| TCP packet details | [2-tcp-packet-details.png](screenshots/2-tcp-packet-details.png) |
| TLS handshake | [3-tls-handshake.png](screenshots/3-tls-handshake.png) |
| TCP conversations | [4-tcp-conversations.png](screenshots/4-tcp-conversations.png) |
| Controlled DNS timeout | [dns-timeout.png](screenshots/dns-timeout.png) |

## Repository structure

```
.
├── README.md
├── .gitignore
├── screenshots/
│   └── Eight lab evidence screenshots
├── captures/
│   └── Reserved for reviewed, sanitized captures
└── reports/
    └── troubleshooting-report.md
```

## Privacy and safe sharing

Before publishing screenshots or PCAP/PCAPNG files, review them for public IP addresses, hostnames, DNS queries, usernames, internal network details, cookies, tokens, and other identifying information. Packet captures can reveal more than screenshots. Do not publish raw captures until they have been reviewed and sanitized. The captures directory is intentionally empty for now.

## Key takeaway

This lab practices a repeatable workflow: observe packets, apply display filters, compare command-line results, record evidence, and distinguish confirmed observations from hypotheses. No real network outage or root cause is claimed without sufficient supporting evidence.

## Detailed report

See [reports/troubleshooting-report.md](reports/troubleshooting-report.md) for the scenario-by-scenario findings and limitations.
