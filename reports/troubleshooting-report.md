# Network Troubleshooting Report

## Summary

This report records observations from a Windows Wireshark lab covering DNS, TCP, and TLS. It includes one deliberately unsuccessful DNS lookup and one successful lookup through the system's default resolver.

## Environment

- Operating system: Windows
- Packet analyzer: Wireshark
- DNS test utility: nslookup
- Local address observed in the capture: 192.168.1.8
- Documentation-only DNS destination used for the controlled timeout: 192.0.2.1

## Scenario A — Successful DNS lookup

Command:

    nslookup example.com

Observed result:

The command returned IPv4 and IPv6 addresses for example.com. The configured resolver was displayed as fe80::1 and named UnKnown.

Interpretation:

The lookup succeeded. The resolver label UnKnown and the message Non-authoritative answer do not, by themselves, indicate failure. The successful result confirms name resolution for this test at that time.

Wireshark display filter:

    dns

Inspect the DNS query and its matching response, if both are present in the selected capture.

## Scenario B — Controlled DNS timeout

Command:

    nslookup example.com 192.0.2.1

Observed result:

The command reported repeated DNS request timeouts. In Wireshark, the observed packets included DNS queries from 192.168.1.8 to 192.0.2.1, including A, AAAA, and a reverse PTR query. No response from that destination was present in the packet list described during the lab.

Filter:

    dns && ip.addr == 192.0.2.1

Diagnosis:

The test deliberately targeted 192.0.2.1, which belongs to the documentation-only TEST-NET-1 range 192.0.2.0/24. It is not intended to be a publicly reachable DNS resolver. Therefore, the timeout is expected for this controlled test and does not establish a fault with the normal configured DNS service.

Verification:

A subsequent nslookup example.com using the system's default resolver returned IPv4 and IPv6 addresses. This confirms the default lookup worked during that test. It does not prove every DNS path or resolver is always healthy.

## Scenario C — TCP three-way handshake

The capture showed these packets for an HTTPS TCP flow:

1. Client to server: [SYN]
2. Server to client: [SYN, ACK]
3. Client to server: [ACK]

This is the expected TCP three-way handshake and shows that TCP connection setup completed for that flow.

Filter:

    tcp.flags.syn == 1

## Scenario D — TLS handshake visibility

The capture showed a TLS Client Hello with SNI example.com, followed by Server Hello-related traffic. This supports documenting that TLS handshake messages were visible. The screenshot or packet summary alone should not be used to claim that every TLS handshake stage was fully inspected.

Filter:

    tls.handshake

## Scenario E — TCP reset observation

One observed packet was:

    Frame 1674
    192.168.1.8 -> 4.207.44.67
    TCP 63591 -> 443 [RST, ACK]
    Seq=44432 Ack=4820 Win=0 Len=0

Nearby listed packets included TLS Application Data and FIN, ACK traffic from both endpoints.

Interpretation and limitation:

A TCP RST resets a connection, but the flag alone does not prove a network problem. Application behavior, socket state, packet-capture timing, and the complete stream may affect interpretation. The available excerpt is insufficient to identify a confirmed root cause, so this is recorded as an observation only.

Suggested filter:

    tcp.analysis.retransmission || tcp.flags.reset == 1

To examine the complete flow, use tcp.stream eq N, replacing N with the stream index from packet details.

## Troubleshooting workflow demonstrated

1. Reproduce a test with a command-line utility.
2. Capture packets on the active network interface.
3. Filter the relevant protocol and destination.
4. Compare observed packets with command output.
5. Test the default DNS resolver separately.
6. Record the evidence and clearly state what it does and does not prove.

## Conclusion

The lab demonstrates DNS query analysis, a successful DNS lookup, a controlled DNS timeout, TCP handshake recognition, TLS handshake visibility, and cautious interpretation of a TCP reset. The controlled timeout is explained by the deliberately selected documentation address. No unrelated real-world network failure is claimed.

## Evidence checklist

- [ ] Add DNS analysis screenshot.
- [ ] Add DNS query detail screenshot.
- [ ] Add DNS response detail screenshot.
- [ ] Add TCP three-way handshake screenshot.
- [ ] Add TCP packet detail screenshot.
- [ ] Add TLS handshake screenshot.
- [ ] Add TCP conversations screenshot.
- [ ] Add DNS timeout screenshot.
- [ ] Optionally add the TCP reset observation screenshot.
- [ ] Review every screenshot and capture for sensitive data before public release.
