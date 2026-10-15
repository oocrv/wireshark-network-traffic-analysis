# Wireshark Network Troubleshooting — Lab Report

## 1. Project overview

**Goal:** practise packet-level troubleshooting with Wireshark on Windows by comparing command-line DNS tests with captured traffic and identifying common TCP/TLS behavior.

**Tools:** Wireshark, Windows Command Prompt, `nslookup`.

**Important scope note:** this is a small lab using real traffic plus one intentionally unsuccessful DNS query. It does not claim that a production network fault was found or repaired.

## 2. Successful DNS lookup

**Command used**

```cmd
nslookup example.com
```

**What we observed:** the lookup returned IPv4 and IPv6 addresses. The resolver appeared as `fe80::1` with the label `UnKnown`.

**What it means:** name resolution succeeded for this test. The label `UnKnown` and the phrase “Non-authoritative answer” do not, by themselves, mean DNS is broken.

**Wireshark filter**

```
dns
```

**Evidence — DNS overview**

![DNS analysis overview](../screenshots/dns-analysis.pcapng.png)

**Evidence — query packet details**

![DNS query details](../screenshots/02-dns-query-details.png)

**Evidence — response packet details**

![DNS response details](../screenshots/03-dns-response-details.png)

## 3. Controlled DNS timeout test

**Command used**

```cmd
nslookup example.com 192.0.2.1
```

**What we observed:** the command timed out. The capture showed DNS requests from local host `192.168.1.8` to `192.0.2.1`, without a matching response in the packet list reviewed.

**Wireshark filter**

```
dns && ip.addr == 192.0.2.1
```

**Interpretation:** `192.0.2.1` belongs to the documentation-only range `192.0.2.0/24`; this test deliberately queried an address that is not intended to be a public DNS resolver. The timeout is therefore a controlled demonstration, not proof that the normal DNS service is faulty.

**Evidence**

![Controlled DNS timeout](../screenshots/dns-timeout.png)

## 4. TCP three-way handshake

An HTTPS TCP flow showed the expected connection setup:

1. **SYN** — the client starts the connection.
2. **SYN, ACK** — the server acknowledges the request and responds.
3. **ACK** — the client acknowledges the server response.

This indicates TCP setup completed for the observed flow.

**Wireshark filter**

```
tcp.flags.syn == 1
```

**Evidence**

![TCP three-way handshake](../screenshots/1-tcp-three-way-handshake.png)

![TCP packet details](../screenshots/2-tcp-packet-details.png)

## 5. TLS handshake visibility

The capture showed a TLS Client Hello with SNI `example.com`, followed by Server Hello-related traffic. This is evidence that TLS handshake messages were visible in the capture. It does **not** establish that every stage of the TLS negotiation was fully verified.

**Wireshark filter**

```
tls.handshake
```

**Evidence**

![TLS handshake](../screenshots/3-tls-handshake.png)

## 6. TCP conversations and reset observation

The TCP Conversations view was used to review network conversations. In one flow, a packet was observed with these details:

```
Frame 1674
192.168.1.8 -> 4.207.44.67
TCP 63591 -> 443 [RST, ACK]
Seq=44432 Ack=4820 Win=0 Len=0
```

Nearby packets included TLS Application Data and FIN, ACK traffic.

**Interpretation:** RST indicates a TCP reset, but this flag alone does not identify the cause or prove a network fault. Application behavior, socket state, and the complete stream can affect interpretation. We therefore record this as an observation only, not a confirmed incident.

**Filter used to review possible retransmissions and resets**

```
tcp.analysis.retransmission || tcp.flags.reset == 1
```

**Evidence — TCP conversations**

![TCP conversations](../screenshots/4-tcp-conversations.png)

## 7. Troubleshooting method practised

1. Run a reproducible command-line test.
2. Capture traffic on the active network interface.
3. Apply a protocol-specific Wireshark display filter.
4. Compare captured packets with the command output.
5. Separate a deliberately failed test from normal network behavior.
6. Document evidence, limitations, and conclusions without guessing at root cause.

## 8. Conclusion

This lab demonstrates basic DNS analysis, a controlled DNS timeout, TCP three-way handshake recognition, TLS handshake visibility, and cautious review of a TCP reset. The evidence supports these observations, but does not prove a real production outage or a definitive root cause.

## 9. Privacy note

Screenshots may expose IP addresses, hostnames, and DNS queries. Review them before sharing widely. Raw PCAP/PCAPNG files are intentionally not included in this public repository because they can contain more sensitive detail than screenshots.
