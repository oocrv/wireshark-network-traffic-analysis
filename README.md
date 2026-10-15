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

## Filters shown in the evidence

DNS traffic:

```
dns
```

TCP stream 27:

```
tcp.stream eq 27
```

TLS handshake-related traffic:

```
tls.handshake
```

## Main findings

### DNS query and response

The capture contains a query for `example.com` with transaction ID `0x0003`. The matching response reports **No error** and two IPv6 answers. Wireshark reports a response time of about 9.0198 ms.

### DNS requests with no visible response

Repeated DNS requests from `192.168.1.8` to `192.0.2.1` appear in the capture, including a PTR name for `192.0.2.1`. No matching response is visible in the displayed evidence. This does not by itself prove that the default DNS resolver is broken or confirm why no response appears.

### TCP and TLS

The packet list for TCP stream 27 shows SYN, SYN-ACK, and ACK. TLS handshake-related traffic includes a Client Hello with SNI `example.com` and Server Hello-related traffic. These observations do not establish that every TLS handshake stage was verified.

### TCP conversations and reset

The TCP Conversations window shows a conversation for stream 27 with local port `62100`, remote port `443`, 16 packets, and duration of approximately `0.155 seconds`.

Separately, frame 1674 was inspected as an `RST, ACK` packet from `192.168.1.8:63591` to `4.207.44.67:443`. A reset flag alone does not establish the reason for the reset. A dedicated screenshot of frame 1674 is not included.

## Evidence screenshots

Screenshots are numbered consistently so the folder, captions, and report follow the same order.

| No. | Evidence | Screenshot |
|---:|---|---|
| 01 | DNS traffic overview | [01-dns-analysis.png](screenshots/01-dns-analysis.png) |
| 02 | DNS query details | [02-dns-query-details.png](screenshots/02-dns-query-details.png) |
| 03 | DNS response details | [03-dns-response-details.png](screenshots/03-dns-response-details.png) |
| 04 | DNS requests with no visible response | [04-dns-timeout.png](screenshots/04-dns-timeout.png) |
| 05 | TCP three-way handshake | [05-tcp-three-way-handshake.png](screenshots/05-tcp-three-way-handshake.png) |
| 06 | TCP packet details | [06-tcp-packet-details.png](screenshots/06-tcp-packet-details.png) |
| 07 | TLS handshake | [07-tls-handshake.png](screenshots/07-tls-handshake.png) |
| 08 | TCP conversation statistics | [08-tcp-conversations.png](screenshots/08-tcp-conversations.png) |

## Repository structure

```
.
├── README.md
├── .gitignore
├── screenshots/
│   ├── 01-dns-analysis.png
│   ├── 02-dns-query-details.png
│   ├── 03-dns-response-details.png
│   ├── 04-dns-timeout.png
│   ├── 05-tcp-three-way-handshake.png
│   ├── 06-tcp-packet-details.png
│   ├── 07-tls-handshake.png
│   └── 08-tcp-conversations.png
└── reports/
    └── network-traffic-analysis-report.md
```

## Privacy and safe sharing

Screenshots may show IP addresses, hostnames, and DNS queries. Review them before sharing publicly. Raw PCAP/PCAPNG files can expose more sensitive details than screenshots; do not publish them unless reviewed and appropriately sanitized.

## Detailed report

See [Network Traffic Analysis Lab Report](reports/network-traffic-analysis-report.md) for the screenshot-by-screenshot observations and interpretation.
