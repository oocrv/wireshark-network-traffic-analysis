# Wireshark Network Traffic Analysis — Lab Report

## 1. Project overview

**Objective:** inspect DNS, TCP, and TLS traffic in Wireshark and document what the captured packets show.

**Tool:** Wireshark on Windows.

**Evidence:** eight screenshots from the analysis. Conclusions are limited to what is visible in the capture; packet behavior alone does not establish a root cause.

## 2. DNS traffic analysis

### 2.1 DNS packet-list overview

This is the broad DNS packet-list view. The `dns` display filter was used to focus on DNS requests and responses.

![01 — DNS traffic overview](../screenshots/01-dns-analysis.png)

### 2.2 DNS query details

The query-details view shows an **AAAA query** for `example.com`, requesting IPv6 address information. The packet details reference response frame 1341.

**Display filter:** `dns`

![02 — DNS query details](../screenshots/02-dns-query-details.png)

### 2.3 Matching DNS response

The matching response uses transaction ID `0x0003`, reports **No error**, and contains two IPv6 answers. Wireshark reports a response time of approximately **9.0198 ms** for this transaction.

This supports the conclusion that this observed DNS lookup completed successfully; it does not mean every DNS request in the capture succeeded.

![03 — DNS response details](../screenshots/03-dns-response-details.png)

### 2.4 DNS requests with no response visible

The capture shows repeated DNS requests from `192.168.1.8` to `192.0.2.1`. The visible traffic includes the PTR name `1.2.0.192.in-addr.arpa`, a reverse-lookup name for `192.0.2.1`. No matching response is visible in the displayed evidence.

**Display filter:** `dns`

![04 — DNS requests with no visible response](../screenshots/04-dns-timeout.png)

**Interpretation:** the screenshot shows requests without a visible response in the displayed packets. It does not by itself prove that the default DNS resolver was faulty or establish which command produced the traffic. The address `192.0.2.1` is within the documentation-only range `192.0.2.0/24`, so this should not be described as a test of a public DNS resolver.

## 3. TCP three-way handshake

The packet list for TCP stream 27 shows the three connection-setup messages:

1. **SYN** — the client requests to start a TCP connection.
2. **SYN, ACK** — the other endpoint acknowledges the request and responds.
3. **ACK** — the client acknowledges the response.

This sequence indicates that TCP connection setup completed for the observed flow.

**Display filter:** `tcp.stream eq 27`

![05 — TCP three-way handshake](../screenshots/05-tcp-three-way-handshake.png)

The packet-details screenshot identifies packet 1092 as **SYN, ACK** and shows TCP stream index 27.

![06 — TCP packet details](../screenshots/06-tcp-packet-details.png)

## 4. TLS handshake visibility

The capture includes a TLS Client Hello with SNI `example.com`, followed by Server Hello-related traffic.

**Display filter:** `tls.handshake`

![07 — TLS handshake](../screenshots/07-tls-handshake.png)

This evidence shows TLS handshake-related messages were visible. It does not establish that every stage of the negotiation was fully verified, nor does it mean encrypted application contents were readable.

## 5. TCP conversation statistics

The Wireshark **Statistics → Conversations** window, on the TCP tab, shows a conversation associated with stream 27. The visible statistics include:

- Local port: `62100`
- Remote port: `443`
- Total packets: `16`
- Duration: approximately `0.155 seconds`

This is a statistics view rather than a packet-list view, so there is no display-filter bar in this screenshot.

![08 — TCP conversation statistics](../screenshots/08-tcp-conversations.png)

## 6. Separate TCP reset observation

A separate packet inspected in the capture showed:

```
Frame 1674
192.168.1.8 -> 4.207.44.67
TCP 63591 -> 443 [RST, ACK]
Seq=44432 Ack=4820 Win=0 Len=0
```

Nearby traffic included TLS Application Data and FIN, ACK packets. The TCP Conversations screenshot above does not display frame 1674, and this report does not include a dedicated screenshot of that frame.

**Interpretation:** `RST, ACK` indicates a TCP reset packet. This flag alone does not reveal why the connection was reset and does not prove that a network fault occurred. The surrounding stream and application context would be needed before assigning a root cause.

## 7. Analysis workflow

The evidence documents these analysis steps:

1. Use `dns` to locate DNS packets.
2. Compare a DNS query with its matching response and inspect the response status and answers.
3. Review DNS requests for which no response is visible in the displayed packets.
4. Use `tcp.stream eq 27` to inspect the TCP handshake.
5. Use `tls.handshake` to view TLS handshake-related traffic.
6. Review TCP conversation statistics and a separately inspected TCP reset.
7. Distinguish packet observations from hypotheses about root cause.

## 8. Conclusion

The screenshots document a DNS query and matching response for `example.com`, separate DNS requests to `192.0.2.1` with no response visible in the displayed evidence, a TCP three-way handshake, TLS handshake visibility, TCP conversation statistics, and a separately observed TCP reset.

These are packet-analysis findings, not proof of a production outage or a confirmed root cause. The report does not claim that a fault was repaired or that a specific command caused the DNS requests to `192.0.2.1`.

## 9. Privacy and safe sharing

Review screenshots for IP addresses, hostnames, DNS queries, and other identifying details before sharing them publicly. Raw PCAP/PCAPNG files can expose substantially more information than screenshots, so they should not be published without review and sanitization.
