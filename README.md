# Wireshark Network Traffic Analysis

A practical Wireshark packet-analysis lab documenting DNS query/response behavior, DNS requests with no response visible in the reviewed evidence, a TCP three-way handshake, TLS handshake visibility, TCP conversation statistics, and a separate TCP reset observation.

The conclusions are limited to the packets and screenshots reviewed. This project does not claim that a production outage was confirmed or repaired.

## Objectives

- Inspect DNS queries and matching responses.
- Distinguish requests with no visible response from a confirmed DNS failure.
- Identify the TCP three-way handshake: SYN, SYN-ACK, ACK.
- Recognize visible TLS handshake messages.
- Review TCP conversation statistics and a separate TCP reset.
- Record observations with evidence and confidence levels rather than guessing at a root cause.

## Findings at a glance

| Finding | Evidence | Confidence / limitation |
|---|---|---|
| DNS lookup for `example.com` completed in the observed transaction | Transaction ID `0x0003`, No error, two IPv6 answers, response time about 9.0198 ms | High for this transaction only |
| DNS requests to `192.0.2.1` have no matching response visible in the reviewed evidence | Repeated requests and PTR query are visible | High for what is visible; cause unknown |
| TCP stream 27 completed connection setup | SYN, SYN-ACK, ACK are visible | High for the observed flow |
| TLS handshake-related messages are visible | Client Hello with SNI `example.com` and Server Hello-related traffic | High; full negotiation details were not established |
| TCP conversation statistics show 16 packets over about 0.155 seconds | Conversations view, local port 62100 and remote port 443 | High for the displayed statistics |
| A separate TCP reset was observed | Frame 1674: `192.168.1.8:63591 → 4.207.44.67:443 [RST, ACK]` | High for packet flags; reason is unknown |

## Display filters used in the evidence

DNS traffic:

```wireshark
dns
```

TCP stream 27:

```wireshark
tcp.stream eq 27
```

TLS handshake-related traffic:

```wireshark
tls.handshake
```

## Main findings

### DNS query and response

The capture contains an AAAA query for `example.com` with transaction ID `0x0003`. The matching response reports **No error**, contains two IPv6 answers, and has a Wireshark response time of about 9.0198 ms.

### DNS requests with no visible response

Repeated DNS requests from `192.168.1.8` to `192.0.2.1` appear in the reviewed packets, including a PTR query for `1.2.0.192.in-addr.arpa`. No matching response is visible in the evidence reviewed. This alone does not prove a resolver failure or explain why no response appears. `192.0.2.1` belongs to the documentation-only range `192.0.2.0/24`, so this should not be described as a test of a public DNS resolver.

### TCP and TLS

TCP stream 27 shows SYN, SYN-ACK, and ACK. TLS handshake-related packets include a Client Hello with SNI `example.com` and Server Hello-related traffic. The current evidence does not establish every TLS negotiation stage or the negotiated cipher suite.

### TCP conversations and reset

The TCP Conversations view shows stream 27 with local port `62100`, remote port `443`, 16 packets, and a duration of approximately `0.155 seconds`.

Separately, frame 1674 was inspected as an `RST, ACK` packet from `192.168.1.8:63591` to `4.207.44.67:443`. A reset flag alone does not establish why the connection was reset. The evidence set does not include a dedicated screenshot of frame 1674.

## Reproducing the text-based queries with TShark

If you have the original capture locally, replace `capture.pcapng` below with its actual filename. These commands were added as reproducibility guidance; they have not been run against the capture in this repository.

List DNS packets:

```bash
tshark -r capture.pcapng -Y "dns"
```

List TCP handshake packets:

```bash
tshark -r capture.pcapng -Y "tcp.stream eq 27 && (tcp.flags.syn == 1 || tcp.flags.ack == 1)"
```

List TLS handshake packets:

```bash
tshark -r capture.pcapng -Y "tls.handshake"
```

List TCP reset packets:

```bash
tshark -r capture.pcapng -Y "tcp.flags.reset == 1"
```

To export selected DNS fields:

```bash
tshark -r capture.pcapng -Y "dns" -T fields -e frame.number -e frame.time_relative -e ip.src -e ip.dst -e dns.id -e dns.qry.name -e dns.flags.rcode
```

## Evidence screenshots

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

## Methodology and limitations

The screenshots indicate that Wireshark was used on Windows and document the display filters above. The available notes do not reliably record the exact capture date, capture duration, capture interface, or the exact actions used to generate every packet. Those details are intentionally not guessed; add them after confirming them from your own environment.

The capture contains a documentation-range address (`192.0.2.1`) and a public destination (`4.207.44.67`). These should not be treated as evidence that all traffic belongs to one controlled test. The current report does not establish that addresses were anonymized or modified.

## Privacy and safe sharing

Review screenshots for IP addresses, hostnames, DNS queries, and other identifying details before sharing publicly. Raw PCAP/PCAPNG files can expose more information than screenshots; do not publish them unless reviewed and appropriately sanitized.

## Detailed report

See [Network Traffic Analysis Lab Report](reports/network-traffic-analysis-report.md) for screenshot-by-screenshot observations and interpretation.
