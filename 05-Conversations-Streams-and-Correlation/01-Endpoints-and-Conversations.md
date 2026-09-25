# Endpoints and Conversations

## Objective

This file teaches how to move from individual packets to communication relationships.

A packet tells you what happened at one point in time.

An endpoint and conversation view help answer:

```text
Who is communicating?
With whom?
Using which protocol?
How often?
How much traffic?
On which ports?
Is the communication expected?
Are there unusual relationships?
```

The central idea is:

```text
Packet
  ↓
Endpoint
  ↓
Conversation
  ↓
Communication pattern
  ↓
Investigation
```

This is an important transition from basic packet inspection to professional network analysis.

## Packet vs Endpoint vs Conversation

These concepts should not be confused.

### Packet

A packet is one captured network unit.

Example:

```text
10.10.10.20 → 10.10.10.50
TCP
Destination port: 443
Length: 74 bytes
```

It represents one observed event.

### Endpoint

An endpoint represents a communicating network address.

Depending on the statistics view, endpoints can include:

* IPv4 addresses
* IPv6 addresses
* Ethernet/MAC addresses
* TCP endpoints
* UDP endpoints
* other protocol-specific endpoint types

Example:

```text
10.10.10.20
```

An endpoint may participate in many conversations.

### Conversation

A conversation represents communication between two endpoints within a particular protocol context.

For example:

```text
10.10.10.20:49152
        ↕
10.10.10.50:443
```

This gives more context than a single packet.

## Why Conversations Matter

Suppose a capture contains:

```text
50,000 packets
```

Looking at every packet individually is inefficient.

Instead, ask:

```text
Which hosts are communicating?
Which conversations dominate the capture?
Which endpoints communicate repeatedly?
Which protocols are involved?
Which connections are unusual?
```

Conversation analysis reduces a large capture into manageable relationships.

## The Conversation Mental Model

Use:

```text
Endpoint A
    ↕
Conversation
    ↕
Endpoint B
```

Then add context:

```text
Who?
What?
When?
How much?
How often?
Which protocol?
Which direction?
```

This allows you to move from packet-level analysis to communication-level reasoning.

## Wireshark Endpoints Statistics

Wireshark provides endpoint statistics through its statistics functionality.

Depending on the protocol and Wireshark version, you can inspect endpoint information for protocols such as:

* Ethernet
* IPv4
* IPv6
* TCP
* UDP
* other supported protocol types

The exact menu presentation can vary between versions.

The purpose is consistent:

```text
Summarize communicating endpoints.
```

## Endpoint Analysis Workflow

Start with the broadest useful view.

```text
1. Open endpoint statistics.
2. Select the relevant protocol.
3. Review endpoint list.
4. Sort by useful columns.
5. Identify high-volume endpoints.
6. Identify unusual endpoints.
7. Select an endpoint of interest.
8. Apply or refine a display filter.
9. Investigate the associated conversations.
```

Do not immediately assume that the endpoint with the most traffic is suspicious.

High traffic can be completely normal.

## Useful Endpoint Questions

When reviewing endpoints, ask:

```text
Which hosts generated the most traffic?

Which hosts received the most traffic?

Which endpoints communicate with many other systems?

Which endpoints appear only once?

Which endpoints communicate with unusual peers?

Which endpoints use unexpected protocols?

Which endpoints appear repeatedly during the incident window?
```

These questions turn statistics into an investigation.

## Endpoint Volume

Traffic volume can help prioritize investigation.

Useful measurements can include:

* packet count
* byte count
* transmitted packets
* received packets

Interpret volume in context.

For example:

```text
High traffic
```

could represent:

* file transfer
* backup
* video
* software update
* normal server activity
* replication
* large download

It could also represent something abnormal.

Volume is evidence, not a conclusion.

## Endpoint Diversity

Another useful signal is how many peers an endpoint communicates with.

Example:

```text
Host A
  ├── Server 1
  ├── Server 2
  ├── Server 3
  └── Server 4
```

versus:

```text
Host B
  ├── External Host 1
  ├── External Host 2
  ├── External Host 3
  ├── External Host 4
  ├── External Host 5
  └── External Host 6
```

The second pattern may deserve investigation depending on the environment.

But peer count alone does not establish malicious behavior.

## Conversations Statistics

Conversation statistics provide a more detailed relationship view.

For TCP, a conversation can include:

```text
Source IP
Source Port
Destination IP
Destination Port
Packets
Bytes
```

For UDP, similar endpoint and port relationships can be observed.

The exact columns vary by protocol and Wireshark version.

## Conversation Analysis Workflow

A practical workflow is:

```text
1. Open conversation statistics.
2. Select the relevant protocol.
3. Review conversation pairs.
4. Sort by packets or bytes.
5. Identify high-volume conversations.
6. Identify unusual endpoints or ports.
7. Select an interesting conversation.
8. Filter to that conversation.
9. Inspect the packet sequence.
10. Determine whether further stream analysis is needed.
```

## Conversation 5-Tuple

For TCP and UDP analysis, think in terms of the five-tuple:

```text
Source IP
Destination IP
Source Port
Destination Port
Transport Protocol
```

Example:

```text
10.10.10.20
10.10.10.50
49152
443
TCP
```

This identifies a transport-level flow context.

A practical representation is:

```text
10.10.10.20:49152
        ↓
10.10.10.50:443
        TCP
```

The five-tuple is extremely useful when correlating traffic across a large capture.

## Direction Matters

A conversation has direction.

For example:

```text
Client → Server
Server → Client
```

Do not treat the two directions as identical.

Questions include:

```text
Who initiated the connection?
Which side sent more data?
Which side sent the error?
Which side reset the connection?
Which side stopped responding?
```

Direction can significantly change interpretation.

## Initiator and Responder

For TCP, the SYN sequence often identifies the connection initiator.

Typical establishment:

```text
Client → SYN → Server
Client ← SYN/ACK ← Server
Client → ACK → Server
```

This gives useful context when analyzing a conversation.

Do not infer application roles solely from IP address naming or port numbers.

Use the packet sequence and application evidence.

## Client vs Server

In many cases:

```text
Client
    ↓
Server
```

is obvious.

But not always.

Peer-to-peer applications, distributed systems, and bidirectional services can make roles less straightforward.

Use:

* connection initiation
* ports
* protocol behavior
* request/response patterns
* application metadata

to establish roles.

## Filtering a Conversation

Once an interesting conversation is identified, reduce the capture.

A common approach is to filter by endpoints.

For example:

```text
ip.addr == 10.10.10.20
```

This shows IPv4 traffic involving the address.

To focus on two specific IPv4 addresses:

```text
ip.addr == 10.10.10.20 && ip.addr == 10.10.10.50
```

This is useful, but it may include multiple conversations between the same hosts.

For TCP, add port information when necessary:

```text
ip.addr == 10.10.10.20 && ip.addr == 10.10.10.50 && tcp
```

Then narrow further if required.

## Host Filters vs Conversation Filters

These are different levels of investigation.

### Host-level

```text
ip.addr == 10.10.10.20
```

Question:

```text
What traffic involves this host?
```

### Host-pair level

```text
ip.addr == 10.10.10.20 && ip.addr == 10.10.10.50
```

Question:

```text
What traffic exists between these hosts?
```

### Protocol level

```text
ip.addr == 10.10.10.20 && ip.addr == 10.10.10.50 && tcp
```

Question:

```text
What TCP traffic exists between these hosts?
```

### Application level

```text
ip.addr == 10.10.10.20 && ip.addr == 10.10.10.50 && tls
```

Question:

```text
What TLS traffic exists between these hosts?
```

The important skill is progressively reducing scope.

## Endpoint and Conversation Correlation

A useful workflow is:

```text
Interesting endpoint
        ↓
Find its conversations
        ↓
Select interesting peer
        ↓
Identify protocol
        ↓
Filter conversation
        ↓
Inspect packet sequence
        ↓
Follow stream if appropriate
```

This is more efficient than randomly opening packets.

## Conversation Volume Analysis

When sorting conversations by bytes, ask:

```text
Why is this conversation large?
```

Possible explanations:

* file transfer
* backup
* web download
* streaming
* software update
* database communication
* replication

Then ask:

```text
Is the volume expected?
```

Context matters.

A 500 MB transfer to a backup server may be normal.

A 500 MB transfer to an unexpected external host may deserve investigation.

## Packet Count vs Byte Count

These metrics answer different questions.

### Packet count

Useful for understanding:

* communication frequency
* request volume
* repeated small exchanges
* scanning-like patterns

### Byte count

Useful for understanding:

* transfer volume
* large downloads/uploads
* bulk communication
* data-heavy sessions

Consider both.

Example:

```text
Conversation A
10,000 packets
2 MB

Conversation B
100 packets
200 MB
```

Conversation A has much higher packet frequency.

Conversation B carries much more data.

These are different traffic patterns.

## Short-Lived Conversations

Some conversations may contain only a few packets.

For example:

```text
SYN
SYN/ACK
ACK
```

or:

```text
DNS query
DNS response
```

Short duration does not mean unimportant.

A connection failure may exist precisely because the conversation ends immediately.

## Long-Lived Conversations

Long-lived conversations can be:

* normal persistent connections
* streaming
* tunnels
* management sessions
* database connections
* suspicious persistent communications

Investigate:

```text
Duration
Packet frequency
Data volume
Idle periods
Keepalives
Direction
Application protocol
```

Do not equate long-lived with suspicious.

## Many-to-One Communication

Example:

```text
Host A ─┐
Host B ─┤
Host C ─┼──→ Server
Host D ─┤
Host E ─┘
```

This may indicate:

* centralized service
* DNS
* authentication
* logging
* proxy
* web service
* database
* monitoring

The pattern becomes meaningful only when combined with protocol and environment context.

## One-to-Many Communication

Example:

```text
             ┌── Server A
             ├── Server B
Host ────────┼── Server C
             ├── Server D
             └── Server E
```

Possible explanations include:

* software updates
* DNS
* API calls
* service discovery
* distributed application behavior
* scanning
* monitoring

Again, the pattern is an investigation signal rather than a conclusion.

## Many-to-Many Communication

Distributed applications can generate complex communication graphs.

Example:

```text
A ↔ B
A ↔ C
B ↔ C
B ↔ D
C ↔ D
```

When a capture becomes complex, reduce the problem using:

```text
Time
Host
Protocol
Port
Conversation
```

Do not attempt to understand the entire graph at once.

## Conversation Filtering by Port

Ports can help narrow an investigation.

Examples:

```text
tcp.port == 443
```

```text
tcp.port == 22
```

```text
tcp.port == 445
```

Remember that ports are not proof of application identity.

Use them as filtering dimensions.

## Conversation Filtering by Address

Examples:

```text
ip.addr == 10.10.10.20
```

```text
ip.src == 10.10.10.20
```

```text
ip.dst == 10.10.10.20
```

These answer different questions.

### `ip.addr`

Traffic involving the address.

### `ip.src`

Traffic sent from the address.

### `ip.dst`

Traffic sent to the address.

Direction matters.

## IPv6 Conversation Analysis

The same reasoning applies to IPv6.

Useful fields include:

```text
ipv6.addr
ipv6.src
ipv6.dst
```

Example:

```text
ipv6.addr == 2001:db8::10
```

Do not assume IPv4-only traffic.

Modern environments can contain both IPv4 and IPv6 simultaneously.

## MAC Endpoint Analysis

At the Ethernet layer, MAC addresses can help answer:

```text
Which physical/interface-level endpoints appear in the capture?
```

Useful when investigating:

* local network behavior
* ARP
* switching behavior
* DHCP
* host identity at Layer 2

Be careful when interpreting MAC addresses across routed networks.

A captured MAC address represents the local Layer 2 context visible at the capture point, not necessarily the ultimate remote host.

## Conversation and VLAN Context

In environments using VLANs, capture visibility can include VLAN tags.

This can help identify:

* network segment
* VLAN membership evidence
* traffic separation

Do not assume VLAN information is always present.

It depends on:

* capture location
* interface
* hardware
* configuration
* encapsulation

## Endpoint Name Resolution

Wireshark may display names instead of raw addresses depending on name-resolution settings.

For example:

```text
10.10.10.50
```

may appear as a hostname.

Names can improve readability.

But name resolution can also introduce ambiguity.

When exact evidence matters, verify the underlying address.

## Name Resolution and Evidence

When documenting an investigation, prefer recording:

```text
IP address
```

alongside:

```text
Resolved hostname
```

when available.

Example:

```text
10.10.10.50
server01.example.internal
```

The IP address is the packet-level evidence.

The hostname is additional contextual information.

## Endpoint Anomaly Detection

Endpoint statistics can help identify candidates for investigation.

Examples:

```text
A workstation communicating with an unexpected external host
```

```text
A server communicating with an unusual number of peers
```

```text
A host generating unusually high DNS traffic
```

```text
A host making repeated connections to many ports
```

These are investigation signals.

They are not proof of malicious behavior.

## Conversation-Based Scanning Analysis

Scanning can sometimes produce recognizable conversation patterns.

For example:

```text
One source
    ↓
Many destinations
```

or:

```text
One source
    ↓
One destination
    ↓
Many destination ports
```

Potential evidence can include:

* many short TCP conversations
* repeated SYN packets
* many refused connections
* sequential or unusual ports
* repeated connection attempts

Always distinguish:

```text
Observed scanning-like pattern
```

from:

```text
Confirmed malicious scanning
```

The latter requires additional context.

## Failed Conversation Analysis

A failed conversation may look like:

```text
SYN
SYN/ACK
ACK
Request
No response
Retransmission
Connection termination
```

Or:

```text
SYN
SYN
SYN
No SYN/ACK
```

The exact failure pattern matters.

Ask:

```text
Did the connection establish?
Did application data flow?
Did the peer respond?
Were retransmissions present?
Was a reset sent?
How long did the exchange last?
```

## Conversation Timing

Conversation statistics can identify interesting relationships, but timing requires packet-level inspection.

For an interesting conversation:

```text
Start
  ↓
Connection establishment
  ↓
Application exchange
  ↓
Idle periods
  ↓
Additional exchange
  ↓
Termination
```

Record timestamps when troubleshooting or reconstructing behavior.

## Conversation Duration

Duration can help distinguish traffic patterns.

Examples:

```text
Very short
```

may indicate:

* DNS
* failed TCP connection
* simple request/response

```text
Moderate duration
```

may indicate:

* normal web session
* API request
* authentication exchange

```text
Long duration
```

may indicate:

* persistent HTTP
* SSH
* streaming
* database connection
* tunnel

These are examples, not fixed classifications.

## Endpoint-to-Conversation Investigation

A repeatable workflow:

```text
1. Identify an endpoint of interest.
2. Review its total communication.
3. Identify major peers.
4. Identify protocols.
5. Identify high-volume conversations.
6. Identify unusual conversations.
7. Filter to a selected conversation.
8. Inspect connection establishment.
9. Inspect application behavior.
10. Follow the stream when necessary.
11. Correlate with timestamps.
12. Document findings.
```

## Practical Exercise 1: Endpoint Discovery

Open endpoint statistics for a supplied capture.

Determine:

```text
Top endpoints by packets
Top endpoints by bytes
Most active protocol
Most active host
Least common endpoints
```

Do not immediately decide whether anything is suspicious.

First understand the traffic population.

## Practical Exercise 2: Conversation Discovery

Open conversation statistics.

Find:

```text
Largest conversation by bytes
Largest conversation by packets
Shortest conversation
Longest conversation
Most common destination port
```

For each, explain what the metric tells you.

## Practical Exercise 3: Host-Centric Investigation

Choose one host.

Determine:

```text
Who does it communicate with?
Which protocols does it use?
Which peer receives the most traffic?
Which peer sends the most traffic?
Which conversation is longest?
```

Then filter the capture to that host.

## Practical Exercise 4: Two-Host Investigation

Choose two communicating hosts.

Determine:

```text
Source
Destination
Protocols
Ports
Packet count
Byte count
Connection count
```

Then inspect one conversation at packet level.

## Practical Exercise 5: Direction Analysis

Choose a TCP conversation.

Determine:

```text
Who initiated the connection?
Who sent the first application data?
Which direction carried more bytes?
Which side terminated the connection?
```

Use packet evidence rather than assumptions.

## Practical Exercise 6: High-Volume Investigation

Find the largest conversation by bytes.

Determine:

```text
Why is it large?
Which protocol is involved?
Is the traffic expected?
Is the traffic mostly one-way or bidirectional?
```

Then inspect the stream or application protocol.

## Practical Exercise 7: Many-to-One Pattern

Find an endpoint that communicates with multiple peers.

Create a small map:

```text
Endpoint
 ├── Peer A
 ├── Peer B
 ├── Peer C
 └── Peer D
```

For each peer, record:

```text
Protocol
Port
Packet count
Byte count
```

Explain the communication pattern.

## Practical Exercise 8: One-to-Many Pattern

Find a host communicating with several destinations.

Determine whether the behavior appears related to:

* DNS
* web traffic
* software updates
* service discovery
* scanning
* another application pattern

Use packet evidence to support the interpretation.

## Practical Exercise 9: Failed Conversation

Find a failed TCP conversation.

Document:

```text
SYN behavior
SYN/ACK behavior
ACK behavior
Application data
Retransmissions
RST
Termination
```

Then identify the earliest observable failure.

## Practical Exercise 10: Endpoint-Based Security Investigation

Using an authorized capture, select one endpoint that deserves investigation.

Analyze:

```text
Peers
Protocols
Ports
Volume
Frequency
Timing
Connection count
```

Separate:

```text
Observation
```

from:

```text
Interpretation
```

Do not label the endpoint malicious solely from statistics.

## Practical Exercise 11: IPv4 and IPv6 Comparison

If the capture contains both IPv4 and IPv6:

```text
Compare endpoint usage.
Compare major conversations.
Compare application protocols.
```

Determine whether the same service appears over both protocols.

## Practical Exercise 12: Conversation Evidence Record

For one important conversation, create an evidence record:

```text
Source:
Destination:
Transport:
Source port:
Destination port:
Protocol:
Start time:
End time:
Duration:
Packets:
Bytes:
Initiator:
Important observations:
```

This prepares you for professional documentation later in the repository.

## Common Mistakes

### Mistake 1: Treating High Volume as Malicious

High traffic can be completely normal.

### Mistake 2: Looking Only at Packets

Statistics provide a higher-level view that can dramatically reduce investigation time.

### Mistake 3: Ignoring Direction

Source and destination matter.

### Mistake 4: Assuming Ports Prove Applications

Ports provide clues, not absolute proof.

### Mistake 5: Ignoring IPv6

IPv6 can carry the same application traffic as IPv4.

### Mistake 6: Confusing Endpoint With Conversation

An endpoint can participate in many conversations.

### Mistake 7: Stopping at Statistics

Statistics identify interesting relationships.

Packet-level analysis explains what happened.

### Mistake 8: Assuming Unusual Means Malicious

An unusual communication pattern should trigger investigation, not an automatic conclusion.

## Professional Investigation Pattern

A strong endpoint investigation looks like:

```text
Question:
Which hosts communicated with the affected workstation?

Evidence:
Endpoint statistics show communication with six peers.

Next:
Conversation statistics identify three major TCP conversations.

Next:
One conversation involves TCP/443 and carries most of the bytes.

Next:
Packet analysis shows a successful TCP handshake followed by TLS.

Interpretation:
The workstation established encrypted communication with the identified server.

Limitation:
The application payload cannot be interpreted from the available encrypted capture.
```

This is a structured investigation.

## Endpoint and Conversation Checklist

### Endpoints

* [ ] Identify relevant endpoints.
* [ ] Review packet counts.
* [ ] Review byte counts.
* [ ] Identify major peers.
* [ ] Check IPv4 and IPv6.
* [ ] Consider MAC-level information where relevant.
* [ ] Verify names against addresses.

### Conversations

* [ ] Identify source and destination.
* [ ] Identify transport.
* [ ] Identify ports.
* [ ] Identify protocol.
* [ ] Determine initiator.
* [ ] Review packet count.
* [ ] Review byte count.
* [ ] Review direction.
* [ ] Review timing.
* [ ] Inspect interesting conversations at packet level.

### Investigation

* [ ] Start broad.
* [ ] Identify an interesting endpoint.
* [ ] Narrow to conversations.
* [ ] Narrow to a specific flow.
* [ ] Follow the stream when useful.
* [ ] Correlate application and transport behavior.
* [ ] Separate observations from conclusions.
* [ ] Document limitations.

## Completion Criteria

You are ready to continue when you can independently:

* explain the difference between packets, endpoints, and conversations
* use endpoint statistics to understand a capture
* use conversation statistics to identify communication relationships
* distinguish packet count from byte count
* analyze communication direction
* identify likely initiators
* filter traffic around a host
* filter traffic between two hosts
* narrow an investigation by protocol and port
* recognize many-to-one and one-to-many communication patterns
* investigate short-lived and long-lived conversations
* analyze failed conversations
* use endpoint statistics to prioritize packet-level investigation
* avoid treating unusual traffic as automatically malicious
* create a structured conversation evidence record

The core skill is:

```text
Many packets
    ↓
Endpoints
    ↓
Conversations
    ↓
Interesting relationship
    ↓
Focused packet analysis
    ↓
Evidence-based conclusion
```

That workflow is the foundation for the next step: reconstructing the actual traffic exchanged within a conversation.
