# Wireshark Network Traffic Analysis

A practical packet-analysis lab using Wireshark on Windows. The project documents DNS query/response behavior, DNS requests with no response visible in the captured evidence, a TCP three-way handshake, TLS handshake visibility, TCP conversation statistics, and a separate TCP reset observation.

The findings are limited to what was observed in the capture. This project does not claim that a production outage was confirmed or repaired.

## Objectives

- Inspect DNS queries and matching responses.
- Review DNS requests for which no response is visible in the displayed packets.
- Identify the TCP three-way handshake: SYN, SYN-ACK, ACK.
- Recognize visible TLS handshake messages.
- Read TCP conversation statistics and inspect a TCP reset packet.
- Record observations without guessing at a root cause.

## Filters used in the screenshots

DNS packet list:

```
dns
```

TCP stream 27:

```
tcp.stream eq 27
```

TLS handshake-related packets:

```
tls.handshake
```

These are the filters used in the evidence screenshots; other filters are not listed as though they were used.

## Findings

### 1. DNS query and response

The capture contains a query for `example.com` with transaction ID `0x0003`. The matching response reports **No error** and two IPv6 answers. Wireshark reports a response time of about 9.0198 ms.

### 2. DNS requests with no visible response

Repeated DNS requests from `192.168.1.8` to `192.0.2.1` appear in the capture, including the PTR name `1.2.0.192.in-addr.arpa`. No matching response is visible in the displayed evidence. This observation alone does not prove that the default DNS resolver is broken or confirm why no response appears.

### 3. TCP three-way handshake

The packet list filtered to stream 27 shows the SYN, SYN-ACK, and ACK connection-setup exchange. Packet 1092 is shown as SYN, ACK in the packet-details screenshot.

### 4. TLS handshake visibility

The capture shows a TLS Client Hello with SNI `example.com` and Server Hello-related traffic. This demonstrates visible handshake-related packets, not that every stage was independently verified.

### 5. TCP conversation statistics and reset

The TCP Conversations window shows a conversation for stream 27 with local port `62100`, remote port `443`, 16 packets, and duration of approximately `0.155 seconds`.

Separately, frame 1674 was inspected as an `RST, ACK` packet from `192.168.1.8:63591` to `4.207.44.67:443`. A reset flag alone does not establish the reason for a connection reset. A dedicated screenshot of frame 1674 is not included.

## Evidence screenshots

| Evidence | Screenshot |
|---|---|
| DNS analysis overview | [1-dns-analysis.png](screenshots/1-dns-analysis.png) |
| DNS query details | [02-dns-query-details.png](screenshots/02-dns-query-details.png) |
| DNS response details | [3-dns-response-details.png](screenshots/3-dns-response-details.png) |
| DNS requests with no visible response | [4-dns-timeout.png](screenshots/4-dns-timeout.png) |
| TCP three-way handshake | [5-tcp-three-way-handshake.png](screenshots/5-tcp-three-way-handshake.png) |
| TCP packet details | [6-tcp-packet-details.png](screenshots/6-tcp-packet-details.png) |
| TLS handshake | [7-tls-handshake.png](screenshots/7-tls-handshake.png) |
| TCP Conversations statistics | [8-tcp-conversations.png](screenshots/8-tcp-conversations.png) |

## Repository structure

```
.
├── README.md
├── .gitignore
├── screenshots/
│   └── Eight lab evidence screenshots
└── reports/
    └── network-traffic-analysis-report.md
```

## Privacy and safe sharing

Screenshots may show IP addresses, hostnames, and DNS queries. Review them before sharing publicly. Raw PCAP/PCAPNG files can expose more sensitive details than screenshots; do not publish them unless they have been reviewed and appropriately sanitized.

## Detailed report

See [Network Traffic Analysis Lab Report](reports/network-traffic-analysis-report.md) for the screenshot-by-screenshot observations and interpretation.
