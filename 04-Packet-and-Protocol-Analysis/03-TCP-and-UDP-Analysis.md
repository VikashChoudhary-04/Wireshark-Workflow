# Wireshark TCP and UDP Analysis

## Objective

This file teaches practical analysis of TCP and UDP traffic in Wireshark.

The goal is to move beyond identifying packets as `TCP` or `UDP` and understand:

```text
Connection establishment
Connection state
Data transfer
Acknowledgments
Sequence behavior
Retransmissions
Duplicate acknowledgments
Out-of-order delivery
Resets
Connection termination
Flow control
UDP request/response behavior
Timing
```

The central TCP workflow is:

```text
Connection
    ↓
Handshake
    ↓
Data transfer
    ↓
Acknowledgments
    ↓
Timing
    ↓
Problems or anomalies
    ↓
Termination
```

The central UDP workflow is:

```text
Endpoint
    ↓
Request/response or message flow
    ↓
Timing
    ↓
Loss/repetition indicators
    ↓
Application behavior
```

---

## TCP vs UDP

TCP and UDP solve different transport-layer problems.

### TCP

TCP provides:

```text
Connection establishment
Reliable ordered byte-stream delivery
Acknowledgments
Retransmission mechanisms
Flow control
Connection termination
```

This gives Wireshark substantial information for analyzing connection behavior.

### UDP

UDP provides:

```text
Source port
Destination port
Length
Checksum
Payload
```

It does not provide TCP-style:

```text
SYN
ACK
FIN
RST
Sequence numbers
TCP acknowledgments
TCP receive windows
```

Therefore, TCP and UDP require different analysis strategies.

---

## Start with the Conversation

Before analyzing individual TCP packets, identify the conversation.

A useful starting filter is:

```text id="f1m5w4"
tcp
```

Then identify:

```text id="i6j8g5"
Source IP
Destination IP
Source port
Destination port
```

For a specific pair of hosts:

```text id="n0g0gl"
ip.addr == 192.168.1.10 && ip.addr == 192.168.1.20 && tcp
```

For a specific service:

```text id="u6n8mm"
tcp.port == 443
```

The goal is to move from:

```text
Large capture
    ↓
TCP traffic
    ↓
Relevant connection
```

before interpreting individual packets.

---

## TCP Endpoint Model

A TCP connection can be understood using two endpoints:

```text id="jstf8e"
Endpoint A:
IP + source port

Endpoint B:
IP + destination port
```

Example:

```text id="5e4d1c"
192.168.1.10:51542
        ↓
192.168.1.20:443
```

This identifies a specific transport conversation.

Do not rely only on IP addresses when multiple services or connections exist between the same hosts.

---

## TCP Three-Way Handshake

A normal TCP connection begins with a three-way handshake:

```text id="z3b1z7"
Client → SYN
Server → SYN-ACK
Client → ACK
```

The handshake establishes initial connection state.

In Wireshark, useful filters include:

```text id="g5duj0"
tcp.flags.syn == 1
```

and:

```text id="p3c4y1"
tcp.flags.syn == 1 && tcp.flags.ack == 1
```

Then inspect the packet sequence rather than viewing the flags in isolation.

---

## SYN

The SYN begins the TCP connection attempt.

Example:

```text id="j76t8b"
Client
    ↓
SYN
    ↓
Server
```

Important evidence includes:

```text
Source IP
Destination IP
Source port
Destination port
Sequence number
TCP options
Window
Timestamp
```

The SYN tells you that one endpoint is attempting to establish a TCP connection.

It does not prove that the connection will succeed.

---

## SYN-ACK

The server or responding endpoint may return:

```text id="6z9k3c"
SYN + ACK
```

Conceptually:

```text id="6xjqax"
Client → SYN
Server → SYN-ACK
```

The SYN-ACK demonstrates that a response was received from the other endpoint.

If it is absent, investigate:

```text id="9j7m0e"
Capture visibility
Routing
Firewall behavior
Server availability
Packet loss
Wrong interface
Capture timing
```

Do not immediately conclude that the server is offline.

---

## Final ACK

The initiating endpoint normally responds with:

```text id="4o8l7c"
ACK
```

The handshake becomes:

```text id="b8c8oz"
SYN
 ↓
SYN-ACK
 ↓
ACK
```

After this, application data may begin.

When analyzing a connection, identify whether the expected transition occurs.

---

## TCP Handshake Investigation

Use:

```text id="u6w2g7"
1. Find SYN.
2. Identify source and destination.
3. Find corresponding SYN-ACK.
4. Confirm direction.
5. Find final ACK.
6. Inspect timing.
7. Determine whether application data follows.
```

This gives you a basic connection-state timeline.

---

## TCP Initial Sequence Numbers

TCP sequence numbers identify positions within the byte stream.

The initial sequence number is established during connection setup.

Do not compare raw sequence numbers from unrelated connections and assume they should match.

Instead analyze sequence numbers within the same TCP conversation.

Wireshark may display both:

```text id="j7xv6d"
Relative sequence numbers
```

and:

```text id="v7t8q8"
Raw sequence numbers
```

Relative numbering is often easier for human analysis.

---

## Relative Sequence Numbers

Wireshark commonly presents TCP sequence and acknowledgment numbers in a relative form for readability.

This makes sequences easier to follow:

```text id="y4gk7z"
1
2
3
...
```

rather than dealing with large initial sequence values.

When documenting evidence, note whether the displayed values are relative or raw if the distinction matters.

---

## TCP Acknowledgments

TCP acknowledgments communicate what data has been received.

Conceptually:

```text id="j1j2t4"
Sender
    ↓
Data
    ↓
Receiver
    ↓
ACK
```

The acknowledgment number generally indicates the next sequence number the receiver expects.

This allows Wireshark to identify:

```text
Missing data
Retransmissions
Duplicate acknowledgments
Out-of-order segments
```

---

## TCP Sequence and ACK Reasoning

Suppose one endpoint sends data with a sequence range.

The receiver acknowledges the next expected byte.

Conceptually:

```text id="cm5gyn"
Sender:
Sequence 1000
Data length 500

Receiver:
ACK 1500
```

This means the receiver is acknowledging the data through the byte preceding 1500 and expects sequence 1500 next.

The exact packet values depend on the capture.

The important skill is understanding the relationship.

---

## TCP Data Transfer

After connection establishment, packets can carry application data.

The analysis should move from:

```text id="50t7te"
Handshake
    ↓
Data
    ↓
ACKs
```

Ask:

```text
Which endpoint sends data?
How much data?
How frequently?
How quickly are ACKs returned?
Are retransmissions present?
Are there gaps?
```

This turns packet inspection into flow analysis.

---

## TCP Flags

Important TCP flags include:

```text id="xqpsgq"
SYN
ACK
FIN
RST
PSH
URG
ECE
CWR
```

For foundational analysis, focus on:

```text id="e4u9xm"
SYN
ACK
FIN
RST
```

Useful filters include:

```text id="49ktpd"
tcp.flags.syn == 1
```

```text id="a0a7b1"
tcp.flags.ack == 1
```

```text id="m3rj2g"
tcp.flags.fin == 1
```

```text id="zwc1g6"
tcp.flags.reset == 1
```

---

## TCP Reset

A TCP RST indicates that a connection is being reset.

Filter:

```text id="rj1k7b"
tcp.flags.reset == 1
```

Then inspect:

```text id="ptn8l6"
Source
Destination
Ports
Previous packets
Timing
Application state
```

Possible causes include:

```text
Application behavior
Closed service
Firewall/device behavior
Unexpected connection state
Protocol violation
Network conditions
```

Do not treat every RST as an attack or failure.

The context determines its meaning.

---

## TCP FIN

A FIN is associated with orderly TCP connection termination.

A simplified close can look like:

```text id="3m3qzq"
Endpoint A → FIN
Endpoint B → ACK
Endpoint B → FIN
Endpoint A → ACK
```

Real captures can be more complex.

When analyzing termination, identify:

```text id="b4w1xg"
Who initiated the close?
Was the connection terminated cleanly?
Was there a reset instead?
Was application data still being exchanged?
```

---

## TCP Connection Termination

Use:

```text id="4y9q4f"
tcp.flags.fin == 1
```

to locate FIN packets.

Then inspect:

```text id="g0z1kv"
Previous packets
Following packets
Direction
ACKs
RSTs
Timing
```

The final packet is not necessarily the most important packet.

The sequence of events is what explains the termination.

---

## TCP Retransmissions

A retransmission occurs when TCP sends data again.

Wireshark can identify retransmission-related TCP analysis events.

The investigative workflow is:

```text id="d5d9w6"
Find retransmission
    ↓
Identify connection
    ↓
Identify direction
    ↓
Inspect original packet
    ↓
Inspect ACK behavior
    ↓
Measure timing
    ↓
Determine likely reason
```

Possible causes include:

```text
Packet loss
Delayed acknowledgments
Congestion
Capture artifacts
Out-of-order delivery
Network path behavior
```

A retransmission is evidence of a transport event, not proof of a specific root cause.

---

## Fast Retransmissions

Wireshark may identify certain retransmissions as occurring earlier than a normal retransmission timeout would suggest.

When such an event appears:

```text id="9v3s2c"
Identify the original segment
    ↓
Inspect duplicate ACKs
    ↓
Check packet ordering
    ↓
Inspect timing
```

Do not rely solely on the label.

Inspect the underlying packets.

---

## Duplicate ACKs

Duplicate acknowledgments can indicate that a receiver is still waiting for missing data.

A simplified pattern is:

```text id="o7o6cr"
Segment 1
Segment 2
Segment 4
Segment 5
```

The receiver may repeatedly acknowledge the same expected sequence range because segment 3 has not arrived.

This can contribute to fast retransmission behavior.

However, packet capture conditions can also affect how the pattern appears.

---

## Out-of-Order Packets

Packets may arrive at the capture point in an order different from their TCP sequence order.

Wireshark can identify out-of-order behavior through TCP analysis.

When you see it:

```text id="j08jph"
Check sequence numbers
Check packet timestamps
Check direction
Check retransmissions
Check capture point
```

Do not automatically equate out-of-order delivery with packet loss.

TCP can recover from reordering.

---

## Packet Loss vs Capture Loss

A crucial distinction is:

```text id="uwp5ub"
Network packet loss
```

versus:

```text id="0h2n4w"
Packet not captured
```

A missing packet in a capture does not necessarily mean the network dropped it.

Possible capture-related causes include:

```text
Capture point
Capture filter
Dropped packets by capture process
Hardware offloading
Interface limitations
Asymmetric traffic
```

Always consider capture quality before concluding that the network lost a packet.

---

## TCP Zero Window

A receiver can advertise a zero receive window.

A useful filter is:

```text id="s0s7qn"
tcp.window_size_value == 0
```

This indicates that the advertised TCP window is zero.

Investigate:

```text id="v5g3m4"
Which endpoint advertised it?
When?
How long?
What application was running?
Did the window later reopen?
```

A zero window can contribute to application delays because the sender must wait before transmitting additional data.

---

## TCP Window Scaling

TCP window scaling allows larger effective receive windows than the basic TCP window field alone would provide.

During the handshake, inspect TCP options for:

```text id="f04s4g"
Window Scale
```

When analyzing throughput or flow control, consider:

```text id="2v6j67"
Advertised window
Window scale
Effective receive capacity
ACK behavior
```

Do not interpret the raw window field without considering scaling when it applies.

---

## TCP Selective Acknowledgment

TCP can negotiate Selective Acknowledgment support.

SACK can provide more detailed information about data received when there are gaps.

When investigating packet loss or reordering, inspect TCP options and acknowledgment behavior.

The workflow is:

```text id="ax2w8k"
Handshake
    ↓
SACK capability
    ↓
Missing sequence range
    ↓
Selective acknowledgment
    ↓
Retransmission
```

This can provide more detailed evidence than basic cumulative acknowledgments alone.

---

## TCP Timestamps

TCP timestamps can provide additional timing and sequence-related information.

When present, inspect them when investigating:

```text id="c5s5q0"
RTT behavior
Retransmissions
Connection characteristics
Timing relationships
```

Do not infer exact application performance solely from TCP timestamps.

Use them as part of the broader packet timeline.

---

## TCP RTT

Round-trip time can help characterize responsiveness.

A simplified model is:

```text id="nqj2ap"
Packet sent
    ↓
Corresponding ACK
    ↓
Time difference
```

When analyzing RTT:

```text id="gqv2pl"
Identify the correct connection
Identify the relevant packet/ACK relationship
Check direction
Compare multiple samples
Look for changes over time
```

One unusually high value is less informative than a consistent pattern.

---

## TCP Throughput

Throughput depends on more than packet count.

Consider:

```text id="cnv5gn"
Bytes transferred
Time interval
Direction
Window
ACK behavior
Retransmissions
Application behavior
```

A practical workflow is:

```text id="c9vrkk"
Identify conversation
    ↓
Determine direction
    ↓
Measure data volume
    ↓
Measure time
    ↓
Check retransmissions
    ↓
Check receive window
    ↓
Interpret throughput
```

Use Wireshark's statistics and I/O tools when a larger-scale view is required.

---

## TCP Keepalive

Some applications or operating systems may use TCP keepalive behavior.

Keepalive packets can help determine whether an otherwise idle connection remains active.

When analyzing them, inspect:

```text id="7e2r7x"
Sequence/ACK relationship
Timing
Direction
Application context
Connection duration
```

Do not confuse keepalive behavior with application data.

---

## TCP Port Reuse and Multiple Connections

A single client can establish multiple TCP connections to the same server.

Example:

```text id="74ry0s"
192.168.1.10:51000 → 192.168.1.20:443
192.168.1.10:51001 → 192.168.1.20:443
192.168.1.10:51002 → 192.168.1.20:443
```

Therefore, filtering only by:

```text id="8y5cxp"
ip.addr == 192.168.1.10 && ip.addr == 192.168.1.20
```

may return multiple conversations.

Use ports and stream information to distinguish them.

---

## TCP Stream Identification

Wireshark assigns TCP streams so that packets belonging to the same TCP conversation can be grouped.

A useful workflow is:

```text id="ym1y7j"
Find packet
    ↓
Identify TCP stream
    ↓
Filter the stream
    ↓
Inspect complete conversation
```

The exact stream field can be discovered through Wireshark's packet details or filter autocomplete.

This is especially useful when several connections exist between the same hosts.

---

## Follow TCP Stream

Following a TCP stream can reconstruct application-level communication where the protocol and capture permit it.

Workflow:

```text id="7fln5h"
Identify packet
    ↓
Identify TCP stream
    ↓
Follow stream
    ↓
Review reconstructed data
    ↓
Return to packet-level evidence
```

Use stream reconstruction as a way to understand the conversation.

Do not treat it as a replacement for packet analysis.

---

## UDP Analysis

UDP analysis begins with identifying endpoints.

Start:

```text id="a9qz7n"
udp
```

Then inspect:

```text id="1z5j1u"
Source IP
Destination IP
Source port
Destination port
Length
Checksum
Application protocol
```

Unlike TCP, you cannot ask:

```text id="5h8x8p"
Where is the SYN?
Where is the FIN?
Where is the TCP ACK?
```

Instead ask:

```text id="jj4m5b"
Is there a request?
Is there a response?
What is the timing?
Are messages repeated?
Are expected responses missing?
```

---

## UDP Request and Response

Many UDP protocols use request/response patterns.

Conceptually:

```text id="m6n6p9"
Client → Request
Server → Response
```

For example, DNS commonly uses UDP.

A practical workflow:

```text id="4t9wkn"
Identify request
    ↓
Identify response
    ↓
Compare identifiers
    ↓
Compare timing
    ↓
Check response status
```

The application protocol provides the state information that TCP would otherwise provide through its connection mechanisms.

---

## UDP and DNS

DNS is a common UDP analysis target.

Start:

```text id="i5k5ny"
udp && dns
```

Then inspect:

```text id="v9f0x7"
Query
Response
Source
Destination
Transaction information
Timing
Response code
```

A missing response can indicate:

```text id="3lj9z1"
Packet loss
Server issue
Routing issue
Firewall behavior
Capture limitation
Application timeout
```

Do not assume which explanation applies without additional evidence.

---

## UDP and DHCP

DHCP commonly uses UDP.

A simplified relationship is:

```text id="v46a1r"
Client UDP
    ↓
DHCP Discover
    ↓
Server UDP
    ↓
DHCP Offer
```

When troubleshooting DHCP:

```text id="g9r0st"
udp
    ↓
DHCP
    ↓
Message sequence
    ↓
Timing
    ↓
Configuration
```

Do not expect a TCP-style handshake.

---

## UDP and Application Protocols

Many protocols use UDP because they need:

```text id="1j9s2e"
Low overhead
Low latency
Application-controlled reliability
Multicast/broadcast capability
```

Examples can include:

```text id="s2t3hr"
DNS
DHCP
NTP
SNMP
Some streaming protocols
Some modern transport/application systems
```

When analyzing UDP, learn the application protocol's own message structure.

---

## UDP "Loss" Requires Caution

UDP itself does not provide a TCP-style retransmission mechanism.

If you observe:

```text id="h3e3pr"
Request
(no visible response)
```

you cannot automatically conclude that the packet was lost.

Possible explanations include:

```text id="p7t8q6"
No response was expected
Response was filtered
Response took another path
Server did not respond
Packet was not captured
Application behavior
```

Use protocol-specific evidence and capture context.

---

## TCP and UDP Comparison

| Feature                  | TCP   | UDP                   |
| ------------------------ | ----- | --------------------- |
| Connection setup         | Yes   | No                    |
| SYN                      | Yes   | No                    |
| ACK mechanism            | Yes   | No TCP ACK            |
| Sequence numbers         | Yes   | No                    |
| Retransmission mechanism | Yes   | No TCP retransmission |
| Flow control             | Yes   | No TCP flow control   |
| Ordered byte stream      | Yes   | No                    |
| FIN/RST                  | Yes   | No                    |
| Ports                    | Yes   | Yes                   |
| Application protocol     | Often | Often                 |

This table describes transport behavior, not whether an application itself implements reliability.

An application using UDP can implement its own reliability mechanisms.

---

## TCP Troubleshooting Workflow

Use this sequence:

```text id="t3h8n0"
1. Identify endpoints.
2. Identify the TCP stream.
3. Locate SYN.
4. Locate SYN-ACK.
5. Locate final ACK.
6. Identify application traffic.
7. Inspect sequence and acknowledgment behavior.
8. Check retransmissions.
9. Check duplicate ACKs.
10. Check out-of-order packets.
11. Check receive windows.
12. Check resets or termination.
13. Analyze timing.
14. Determine where behavior diverges from expectation.
```

This should become a standard troubleshooting pattern.

---

## UDP Troubleshooting Workflow

Use:

```text id="k8x1x5"
1. Identify endpoints.
2. Identify application protocol.
3. Identify request.
4. Identify expected response.
5. Compare message identifiers.
6. Measure timing.
7. Check repeated requests.
8. Check response status.
9. Check routing/addressing.
10. Consider capture visibility.
11. Determine whether the application or network behavior explains the result.
```

---

## Connectivity Failure Workflow

Suppose:

```text id="d9s5jr"
Client cannot connect to server.
```

Use:

```text id="xqg7g0"
Client
    ↓
SYN?
    ↓
SYN-ACK?
    ↓
ACK?
    ↓
Application data?
    ↓
RST?
    ↓
Retransmission?
    ↓
Termination?
```

If the connection never gets past SYN:

```text id="n2v2o9"
Focus on network reachability, filtering, routing, server availability, and capture visibility.
```

If TCP establishes but the application fails:

```text id="k7f0t7"
Move upward into the application protocol.
```

---

## Slow Application Workflow

Suppose:

```text id="v5v0g8"
Application feels slow.
```

Break it into stages:

```text id="3ny7oc"
DNS
 ↓
TCP setup
 ↓
TLS setup
 ↓
Application request
 ↓
Server processing/response
 ↓
Data transfer
```

Measure where the delay occurs.

Do not assume:

```text id="3v0y1q"
Slow application = packet loss
```

The delay could be caused by:

```text id="t4r5m0"
DNS
TCP establishment
TLS negotiation
Server response time
Network latency
Retransmissions
Flow control
Application behavior
```

---

## TCP Analysis with Time

TCP problems become much easier to understand when timing is included.

For an event, record:

```text id="1c7s0q"
Packet number
Timestamp
Direction
Sequence number
Acknowledgment number
Flags
Relevant field
```

Then build:

```text id="f8r6m7"
Event A
    ↓
Delay
    ↓
Event B
    ↓
Response
    ↓
Delay
```

Timing transforms individual packets into a sequence.

---

## TCP Analysis with Packet Numbers

Packet numbers help you return to evidence.

Example:

```text id="r1a9e3"
Packet 120:
SYN

Packet 121:
SYN-ACK

Packet 122:
ACK

Packet 135:
Application request

Packet 141:
RST
```

The packet numbers are references.

They do not explain the cause by themselves.

Use them together with packet details and timing.

---

## TCP Analysis with Display Filters

Common starting filters include:

```text id="k8qgk6"
tcp
```

```text id="b9b3mx"
tcp.flags.syn == 1
```

```text id="z3l3u7"
tcp.flags.syn == 1 && tcp.flags.ack == 1
```

```text id="4tqf4r"
tcp.flags.reset == 1
```

```text id="1w8n3w"
tcp.flags.fin == 1
```

```text id="7j2s7k"
tcp.window_size_value == 0
```

For a host:

```text id="q7q0cf"
tcp && ip.addr == 192.168.1.10
```

For a service:

```text id="7u8x6n"
tcp.port == 443
```

Use these as building blocks rather than a memorized checklist.

---

## UDP Analysis with Display Filters

Common starting filters include:

```text id="6t4b8m"
udp
```

For a host:

```text id="y7d2a8"
udp && ip.addr == 192.168.1.10
```

For DNS:

```text id="2x3g4p"
udp && dns
```

For a specific port:

```text id="8w4f2n"
udp.port == 53
```

Again, use packet dissection to determine the actual application protocol.

---

## Common TCP Mistakes

### Mistake 1 — Treating SYN as a Successful Connection

A SYN only begins the connection attempt.

### Mistake 2 — Treating RST as an Attack

RST has many legitimate causes.

### Mistake 3 — Calling Every Retransmission Packet Loss

Capture conditions and reordering can complicate interpretation.

### Mistake 4 — Ignoring Direction

TCP sequence and acknowledgment behavior is directional.

### Mistake 5 — Ignoring Stream Identity

Multiple connections can exist between the same hosts.

### Mistake 6 — Reading Sequence Numbers Across Connections

Sequence spaces are connection-specific.

### Mistake 7 — Ignoring TCP Options

Window scaling and SACK can affect analysis.

---

## Common UDP Mistakes

### Mistake 1 — Expecting a Handshake

UDP does not have a TCP-style connection handshake.

### Mistake 2 — Calling a Missing Response Packet Loss

The capture may not contain the response for many reasons.

### Mistake 3 — Ignoring Application Protocol

UDP provides little state by itself.

### Mistake 4 — Treating UDP as Unreliable by Definition

The transport itself does not guarantee delivery, but the application may implement reliability.

### Mistake 5 — Assuming a Port Proves the Application

Confirm the protocol through packet dissection and context.

---

## Practical Exercise 1 — Three-Way Handshake

Find a TCP connection and identify:

```text id="6zzn9r"
SYN
SYN-ACK
ACK
```

Record:

```text
Packet number
Timestamp
Source
Destination
Source port
Destination port
Sequence number
Acknowledgment number
```

Explain what each packet contributes to connection establishment.

---

## Practical Exercise 2 — Failed Handshake

Find a TCP SYN that does not appear to receive a normal SYN-ACK.

Investigate:

```text id="1i7x6w"
Is another SYN sent?
Does a RST appear?
Is there a response from another host?
Is the capture complete?
Could the response be outside the capture?
```

Document the evidence without assuming the root cause.

---

## Practical Exercise 3 — TCP Retransmission

Find a retransmission.

Record:

```text id="g3o9w8"
Original packet
Retransmitted packet
Sequence number
Direction
Time difference
ACK behavior
Surrounding packets
```

Then explain what the capture supports.

---

## Practical Exercise 4 — Duplicate ACK

Find duplicate ACK behavior if available.

Identify:

```text id="2x1f9h"
Acknowledgment number
Direction
Packets before the duplicate ACKs
Missing or delayed sequence
Subsequent retransmission
```

Explain the observed sequence.

---

## Practical Exercise 5 — Out-of-Order Traffic

Find an out-of-order TCP event.

Compare:

```text id="m8l3u2"
Capture packet number
Timestamp
TCP sequence number
Expected sequence
Following packets
```

Determine whether retransmission or reordering appears more consistent with the evidence.

---

## Practical Exercise 6 — TCP Reset

Find a TCP RST.

Determine:

```text id="h2d5w7"
Who sent it?
Which connection?
What happened immediately before it?
Was application data flowing?
Was the connection being established or closed?
```

Write an observation and an interpretation separately.

---

## Practical Exercise 7 — TCP Termination

Find a clean TCP termination.

Identify:

```text id="c4p7v1"
FIN
ACK
FIN
ACK
```

if the complete sequence is visible.

Then compare it with a connection terminated by RST if one exists.

---

## Practical Exercise 8 — Zero Window

Find a zero-window packet.

Determine:

```text id="s8z0t3"
Which endpoint advertised it?
What connection?
What was happening immediately before?
Did the window later reopen?
Was application data waiting?
```

---

## Practical Exercise 9 — UDP Request/Response

Find a UDP request/response protocol such as DNS.

Identify:

```text id="e9n2h4"
Request
Response
Endpoints
Ports
Application identifier
Timing
```

Then determine whether the response arrived normally.

---

## Practical Exercise 10 — Compare TCP and UDP

Select one TCP exchange and one UDP exchange.

For each, document:

```text id="v5p4x8"
Endpoint A
Endpoint B
Ports
Application protocol
Connection state
Request/response behavior
Timing
Error indicators
```

Then explain why the analysis methods differ.

---

## Practical Exercise 11 — Connectivity Investigation

Investigate:

```text id="m7j5w9"
Client cannot reach server.
```

Determine:

```text
Was a SYN sent?
Was a SYN-ACK returned?
Was the final ACK sent?
Did application traffic begin?
Were retransmissions present?
Was there an RST?
Where does the expected sequence stop?
```

Your conclusion must identify the observed point of failure or uncertainty.

---

## Practical Exercise 12 — Slow TCP Application

Find a TCP exchange that appears delayed.

Determine:

```text id="n8x6c2"
Connection setup time
Request timing
Response timing
ACK behavior
Retransmissions
Window behavior
Data transfer duration
```

Identify where the largest observable delay occurs.

Do not automatically attribute that delay to packet loss.

---

## TCP Analysis Checklist

Before concluding a TCP investigation:

```text id="8t9f3a"
[ ] Did I identify the correct TCP stream?
[ ] Did I identify both endpoints?
[ ] Did I inspect the handshake?
[ ] Did I inspect direction?
[ ] Did I inspect sequence numbers?
[ ] Did I inspect acknowledgment behavior?
[ ] Did I inspect TCP flags?
[ ] Did I check retransmissions?
[ ] Did I check duplicate ACKs?
[ ] Did I check out-of-order packets?
[ ] Did I check receive-window behavior?
[ ] Did I check TCP options where relevant?
[ ] Did I inspect timing?
[ ] Did I inspect termination?
[ ] Did I consider capture limitations?
[ ] Did I separate observation from interpretation?
```

---

## UDP Analysis Checklist

Before concluding a UDP investigation:

```text id="w2m7z1"
[ ] Did I identify both endpoints?
[ ] Did I identify the application protocol?
[ ] Did I identify the request?
[ ] Did I identify the expected response?
[ ] Did I compare message identifiers?
[ ] Did I inspect timing?
[ ] Did I check repeated requests?
[ ] Did I inspect response status?
[ ] Did I consider routing and addressing?
[ ] Did I consider capture visibility?
[ ] Did I avoid assuming missing response means packet loss?
```

---

## Professional TCP/UDP Workflow

Use this mental model:

```text id="j8y5r2"
TCP:
Endpoint
    ↓
Handshake
    ↓
Stream
    ↓
Sequence/ACK
    ↓
Timing
    ↓
Loss/reordering/flow control
    ↓
Termination

UDP:
Endpoint
    ↓
Application protocol
    ↓
Request/response
    ↓
Timing
    ↓
Repetition/errors
    ↓
Application behavior
```

The transport layer should become a bridge between addressing and application behavior.

---

## Completion Criteria

You are ready to continue when you can independently:

* Identify TCP and UDP conversations.
* Identify TCP endpoints and streams.
* Explain the three-way handshake.
* Read SYN, SYN-ACK, and ACK behavior.
* Understand TCP sequence and acknowledgment numbers.
* Interpret major TCP flags.
* Analyze connection termination.
* Investigate TCP resets.
* Identify retransmissions.
* Investigate duplicate acknowledgments.
* Recognize out-of-order behavior.
* Analyze TCP receive-window conditions.
* Understand the purpose of window scaling and SACK.
* Use timing to investigate TCP behavior.
* Analyze UDP request/response patterns.
* Distinguish transport-level evidence from application-level evidence.
* Investigate connectivity failures.
* Investigate application slowness.
* Avoid confusing capture limitations with network failures.
* Document observations without overstating conclusions.

The next stage will build on this transport-layer foundation by analyzing DNS, DHCP, ICMP, HTTP, and TLS as complete protocol workflows.
