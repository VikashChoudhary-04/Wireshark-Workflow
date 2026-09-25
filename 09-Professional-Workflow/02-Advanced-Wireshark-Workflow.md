# Advanced Wireshark Workflow

## Objective

Advanced Wireshark analysis is not primarily about memorizing more filters.

It is about combining multiple analysis techniques to investigate complex traffic efficiently.

At this stage, the analyst should be comfortable moving between:

```text
Packet
Conversation
Stream
Endpoint
Protocol
Statistics
Timing
Graph
Timeline
```

The workflow becomes:

```text
Broad Observation
      ↓
Targeted Hypothesis
      ↓
Focused Analysis
      ↓
Cross-Validation
      ↓
Evidence Chain
      ↓
Decision
```

The objective is to reduce investigation time while increasing analytical accuracy.

## Advanced Mental Model

Use multiple perspectives on the same event:

```text
                     ┌── Packet Details
                     │
                     ├── Conversations
Traffic Question ────┼── Streams
                     │
                     ├── Statistics
                     │
                     ├── Timing
                     │
                     └── Graphs
```

No single Wireshark view should automatically become the final answer.

The strongest findings usually emerge when several views agree.

## The Advanced Investigation Loop

Use:

```text id="x9g9r3"
1. Define the question.
2. Establish the expected behavior.
3. Orient yourself in the capture.
4. Identify relevant hosts.
5. Identify relevant conversations.
6. Narrow the traffic.
7. Analyze the protocol sequence.
8. Measure timing.
9. Compare competing explanations.
10. Validate using another Wireshark view.
11. Document the evidence.
12. Decide what to investigate next.
```

## Expected Behavior First

Advanced analysis becomes easier when you know what normal behavior should look like.

For an HTTPS request:

```text id="r3y8t4"
DNS
 ↓
TCP
 ↓
TLS
 ↓
Application exchange
```

For an SSH session:

```text id="w6p4x9"
TCP
 ↓
SSH negotiation
 ↓
Encrypted session
```

For DHCP:

```text id="qv3l0h"
Discover
 ↓
Offer
 ↓
Request
 ↓
ACK
```

Expected sequences provide a baseline for identifying deviations.

## Start Broad

Before applying detailed filters, inspect:

```text id="a1f8h3"
Protocol Hierarchy
Endpoints
Conversations
Time range
Packet count
```

Ask:

```text id="0w6y6z"
What type of capture is this?

Which systems dominate it?

Which protocols dominate it?

Which time periods appear unusual?
```

## Identify Anomalies

An anomaly is something that differs from the expected pattern.

Examples:

```text id="7h5n4b"
Unexpected destination
Unexpected protocol
Unexpected port
Unexpected timing
Unexpected traffic volume
Unexpected communication direction
Unexpected retransmissions
Unexpected connection duration
```

An anomaly is a reason to investigate.

It is not automatically a security finding.

## Progressive Narrowing

Use a funnel:

```text id="3j1c6b"
Entire Capture
      ↓
Time Window
      ↓
Host
      ↓
Destination
      ↓
Protocol
      ↓
Conversation
      ↓
Stream
      ↓
Packet
```

At each stage ask:

```text id="d7p8c9"
Did this reduce noise without removing important context?
```

## Time-Window Filtering

When investigating an event, first isolate its approximate time.

For example:

```text id="c9y4ve"
10:00–10:05
```

Then investigate:

```text id="1d8qj6"
Hosts
Protocols
Connections
```

This can dramatically reduce unrelated traffic.

## Host-Centered Analysis

Choose a host and examine its behavior.

Ask:

```text id="5t8xj0"
Who does it communicate with?

Which ports?

Which protocols?

How frequently?

How much data?

For how long?
```

This produces a host communication profile.

## Destination-Centered Analysis

Sometimes the destination is more important.

Ask:

```text id="g0n4az"
Who connects to this server?

Which ports?

At what times?

How many clients?

Which protocols?

Are connections successful?
```

This is useful for:

```text id="o4m3z8"
Servers
DNS infrastructure
Web services
Authentication servers
Monitoring systems
```

## Conversation-Centered Analysis

Once the relevant endpoints are known, inspect conversations.

Record:

```text id="r7j5b1"
Source
Destination
Port
Protocol
Packets
Bytes
Duration
```

Then ask:

```text id="w2s6q7"
Which conversation best matches the investigation question?
```

## Stream-Centered Analysis

A conversation may contain multiple application exchanges.

A stream lets you isolate a specific transport conversation.

Use stream following when you need:

```text id="b5q9r8"
Application sequence
Request/response relationship
Session content
Stream completeness
```

For encrypted traffic, the stream may remain encrypted.

## Packet-Centered Analysis

Return to individual packets when:

```text id="4j7v5c"
Timing matters
A flag matters
A field matters
A retransmission matters
A protocol error matters
A specific packet must be cited
```

Do not remain at the packet level when the question requires a broader pattern.

## Multi-Layer Correlation

A professional analyst moves vertically through the protocol stack.

Example:

```text id="w8c4x1"
Ethernet
 ↓
IPv4
 ↓
TCP
 ↓
TLS
 ↓
HTTP
```

Ask at each layer:

```text id="g3j6v2"
Is the expected behavior visible?
```

A failure at one layer changes what should be expected above it.

## Cross-Layer Failure Example

Suppose:

```text id="n3t9f1"
TCP handshake succeeds.

TLS handshake never completes.
```

The problem should not immediately be described as:

```text id="b7h5q2"
"HTTP failure."
```

HTTP may never have been reached.

The correct next question is:

```text id="f1p6r8"
Why did TLS negotiation fail or stop?
```

## Timing Decomposition

For performance investigations, divide total latency.

Example:

```text id="a8k3w6"
Total transaction:
2.8 seconds

DNS:
20 ms

TCP:
30 ms

TLS:
50 ms

Application:
2.7 seconds
```

The decomposition identifies where the majority of time was spent.

## Relative Timing

Absolute timestamps are useful.

Relative timestamps can be even more useful for sequence analysis.

For example:

```text id="k4j7s1"
t = 0 ms
ClientHello

t = 35 ms
ServerHello

t = 85 ms
Handshake complete

t = 110 ms
Application data
```

This makes delays easier to compare.

## Timing Comparisons

Compare multiple transactions:

```text id="u8f3q2"
Request A:
120 ms

Request B:
130 ms

Request C:
2,400 ms
```

The outlier deserves investigation.

Ask:

```text id="b1v6d9"
What differs about the outlier?
```

Compare:

```text id="k5r8x2"
Destination
Packet count
Retransmissions
Server response time
Payload size
Connection reuse
```

## Connection Reuse

Applications may reuse established TCP or TLS sessions.

Therefore:

```text id="y4q8n3"
One TCP connection
```

does not necessarily equal:

```text id="x7m2k9"
One application request
```

Investigate whether multiple transactions occur within the same connection.

This matters for:

```text id="r6j1s4"
HTTP keep-alive
HTTP/2
Persistent APIs
Long-lived sessions
```

## HTTP/2 Considerations

HTTP/2 can multiplex multiple requests over one connection.

Therefore:

```text id="n5x9k4"
One TCP/TLS connection
        ↓
Multiple logical requests
```

Do not count TCP connections as application requests.

Use protocol-level information when available.

## QUIC and HTTP/3 Considerations

QUIC uses UDP.

A single connection may carry multiple application interactions.

Therefore:

```text id="t8v3p6"
UDP packet count
```

does not directly equal:

```text id="h2k7m1"
Number of application requests
```

Use protocol dissection and available metadata.

## Conversation vs Application Transaction

Keep these concepts separate:

```text id="s6j3x8"
Network conversation:
Transport-level communication.

Application transaction:
A request and corresponding response.
```

One conversation can contain many application transactions.

## Traffic Baselines

Advanced investigations benefit from baseline comparison.

Compare:

```text id="c8f1m5"
Normal capture
vs
Problem capture
```

Metrics:

```text id="d2j7q9"
Connection count
Destination count
Average latency
Packet loss indicators
Retransmissions
Traffic volume
Protocol distribution
```

## Outlier Analysis

Look for:

```text id="p4m8v6"
Rare destinations
Rare ports
Rare protocols
Unusually long sessions
Unusually short sessions
Large traffic bursts
Unusual timing
Unexpected protocol transitions
```

Then validate the outlier at packet level.

## Traffic Volume Analysis

Use statistics to identify major traffic contributors.

Then ask:

```text id="y6c9r3"
Is the high volume expected?
Which hosts generated it?
Which protocol generated it?
Was the traffic continuous or bursty?
```

High traffic volume alone is not evidence of malicious activity.

## I/O Graph Strategy

Use I/O graphs to answer:

```text id="z8p2q5"
When did traffic increase?

When did traffic stop?

Are there periodic bursts?

Does the problem coincide with a traffic spike?
```

Then click into the corresponding packet interval and investigate the actual traffic.

## Graph-to-Packet Workflow

Use:

```text id="j1r6t9"
I/O Graph
  ↓
Identify unusual interval
  ↓
Select timestamp range
  ↓
Apply focused filter
  ↓
Inspect conversations
  ↓
Inspect packets
```

This is much more useful than treating the graph as the final answer.

## Endpoint Graph Thinking

Build a mental communication graph:

```text id="q3w7k5"
        ┌── Server A
        │
Client ─┼── Server B
        │
        └── Server C
```

Then ask:

```text id="n7k2x4"
Which relationships are expected?
Which are unusual?
Which are new?
Which are high volume?
```

## Fan-Out

Fan-out means:

```text id="v5j8m3"
One source
   ↓
Many destinations
```

Possible explanations include:

```text id="u2p9d6"
Monitoring
Inventory
Service discovery
Administrative automation
Scanning
```

Context is required.

## Fan-In

Fan-in means:

```text id="e4q6s2"
Many sources
   ↓
One destination
```

Possible explanations include:

```text id="a7c3x5"
Server service
Central authentication
Logging
Monitoring
Shared application
```

Again, context matters.

## East-West Traffic

Internal host-to-host traffic can be described as:

```text id="p8w1j6"
East-West
```

Examples:

```text id="d5k9m2"
Workstation → Server
Server → Server
Workstation → Workstation
```

Use this concept when analyzing internal communication patterns.

## North-South Traffic

Traffic crossing the internal/external boundary can be described as:

```text id="h6r3q8"
North-South
```

Examples:

```text id="v9m4s1"
Internal host → Internet
Internet → Public service
```

These labels describe traffic direction.

They do not determine intent.

## Protocol Transition Analysis

Sometimes the important clue is a protocol transition.

Example:

```text id="k7m2f4"
DNS
 ↓
TCP/443
 ↓
TLS
 ↓
Application
```

Another:

```text id="p3s8n6"
SMB
 ↓
RPC
 ↓
Additional internal communication
```

Track what happens after an important connection.

## Follow-On Behavior

A connection may become more informative because of what happens afterward.

Example:

```text id="z2q6w9"
Host A
 ↓
Host B:445
 ↓
SMB session
 ↓
Host B: another internal service
```

Do not infer a specific action unless the capture supports it.

Instead document the observed sequence.

## Connection Chaining

Build chains such as:

```text id="u6x4r8"
Host A
  ↓
Server B
  ↓
Server C
  ↓
Server D
```

Then ask:

```text id="c2n7m5"
Are these independent connections?

Are they temporally related?

Does one event consistently precede the next?
```

## Advanced DNS Correlation

For a domain-based investigation:

```text id="m5r9k2"
DNS Query
 ↓
DNS Response
 ↓
Returned IP
 ↓
TCP/UDP Connection
 ↓
TLS SNI
 ↓
Certificate
 ↓
Application Traffic
```

Compare timestamps at every stage.

## DNS Caching

A missing DNS query immediately before a connection does not necessarily mean the hostname was never resolved.

The system may have used cached DNS information.

Therefore:

```text id="q4v8n6"
No visible DNS query
```

does not automatically mean:

```text id="r1s5j9"
No DNS resolution occurred.
```

## Name Resolution in Wireshark

Wireshark may display names based on its own name-resolution settings.

Distinguish:

```text id="h7c3m1"
Observed packet data
```

from:

```text id="k9x2p4"
Wireshark-generated name resolution
```

When evidence matters, record the underlying IP addresses.

## Advanced TCP Reasoning

Analyze TCP as a sequence rather than isolated flags.

Example:

```text id="s8f4j2"
SYN
 ↓
SYN/ACK
 ↓
ACK
 ↓
Data
 ↓
ACK
 ↓
FIN
 ↓
FIN/ACK
```

Compare with:

```text id="q6m3x9"
SYN
 ↓
SYN/ACK
 ↓
ACK
 ↓
Data
 ↓
Retransmission
 ↓
Duplicate ACK
 ↓
Recovery
```

The second sequence requires transport-level investigation.

## Sequence and ACK Reasoning

When investigating TCP problems, inspect:

```text id="y1v5r8"
Sequence number
Acknowledgment number
Window
Flags
Timestamp
Payload length
```

Ask:

```text id="m4q7k2"
Which packet was expected next?

Was it acknowledged?

Was it retransmitted?

Did the receiver advance the acknowledgment?
```

## Retransmission vs Out-of-Order

Do not treat all unusual packet ordering as packet loss.

Investigate:

```text id="r3n8w5"
Sequence numbers
Timestamps
ACK behavior
Packet capture ordering
```

Out-of-order delivery and retransmission can produce different evidence patterns.

## Duplicate ACKs

Duplicate ACKs can indicate that a receiver is waiting for missing sequence data.

Investigate:

```text id="u7k4m9"
Repeated acknowledgment number
Missing sequence region
Subsequent retransmission
Recovery timing
```

Then determine whether the pattern is isolated or persistent.

## Zero Window

A zero TCP window can indicate that the receiver temporarily cannot accept additional data.

Investigate:

```text id="n2p6v8"
Window announcement
Duration
Window updates
Sender behavior
Application stage
```

Do not automatically label the receiver as overloaded.

## Application Response Time

Where supported, application-level response-time analysis can reveal:

```text id="b4j9x6"
Fast requests
Slow requests
Outliers
Server-side delay patterns
```

Compare multiple transactions.

One slow request is different from a consistent slowdown.

## Advanced Encryption Analysis

When payloads are encrypted, focus on:

```text id="c5w8r2"
Handshake
SNI
Certificate
ALPN
Destination
Timing
Traffic volume
Session duration
DNS correlation
Transport behavior
```

Then document what remains unavailable.

## Encryption Does Not Remove Timing Evidence

Even when content is hidden, you can still observe:

```text id="s7n3k1"
Connection start
Handshake duration
Idle time
Burst timing
Connection duration
Termination
```

This can be valuable during performance and behavioral analysis.

## Advanced Capture Limitations

Always consider:

```text id="v6p2m9"
NAT
Proxy
Load balancer
VPN
Tunnel
SPAN
VLAN
Wireless capture
Packet loss
Truncation
Capture filters
```

The capture architecture is part of the evidence model.

## Cross-Capture Comparison

If multiple captures exist, compare them.

Example:

```text id="h3q7w1"
Client capture
vs
Server capture
```

Questions:

```text id="m8k4c2"
Did the client send the packet?

Did the server receive it?

Did the server respond?

Did the client receive the response?
```

This can help localize packet loss or visibility problems.

## Multi-Point Analysis

When captures exist at multiple locations:

```text id="p9x5d3"
Client
  ↓
Firewall
  ↓
Proxy
  ↓
Server
```

Compare packet timestamps and identifiers.

This can help determine where traffic was delayed, modified, or lost.

## Advanced Investigation of Slow Applications

Use:

```text id="w2j6s8"
1. Identify slow transactions.
2. Compare against fast transactions.
3. Decompose timing.
4. Check DNS.
5. Check TCP.
6. Check TLS.
7. Check application response.
8. Check retransmissions.
9. Check connection reuse.
10. Check server response patterns.
```

The goal is to locate the delay, not merely observe that it exists.

## Advanced Investigation of Suspicious Traffic

Use:

```text id="r8m3v7"
1. Identify the host.
2. Establish baseline behavior.
3. Identify unusual destinations.
4. Analyze ports and protocols.
5. Examine timing.
6. Follow connections.
7. Correlate DNS.
8. Analyze encryption metadata.
9. Examine internal communication.
10. Build a timeline.
11. Consider legitimate explanations.
12. Identify evidence gaps.
```

## Advanced Investigation of Scanning-Like Behavior

Use:

```text id="f6y9k2"
Source
 ↓
Destination count
 ↓
Port distribution
 ↓
Timing
 ↓
Response pattern
 ↓
Successful sessions
 ↓
Application follow-up
 ↓
Expected host role
```

Do not identify intent from packet shape alone.

## Advanced Evidence Validation

For an important finding:

```text id="x5v2n8"
1. Find the relevant packet.
2. Record the packet number.
3. Record the timestamp.
4. Record the filter.
5. Inspect protocol details.
6. Check the conversation.
7. Check related statistics.
8. Check the timeline.
9. Consider alternative explanations.
```

This creates a reproducible finding.

## Advanced Filter Construction

At this stage, filters should become compositional.

Start with a simple condition:

```text id="a6m9q4"
ip.addr == 192.0.2.25
```

Add a protocol:

```text id="j7p3w8"
ip.addr == 192.0.2.25 && tcp
```

Add a destination:

```text id="k4r8x1"
ip.addr == 192.0.2.25 && ip.dst == 203.0.113.20
```

Add a port:

```text id="z9c2m5"
ip.addr == 192.0.2.25 &&
ip.dst == 203.0.113.20 &&
tcp.dstport == 443
```

Build complexity only when it answers a specific question.

## Avoid Giant Filters

A very long filter can become difficult to understand and maintain.

Prefer:

```text id="v4n7q6"
Simple filter
↓
Observation
↓
Next filter
```

over:

```text id="d8x2k9"
One massive filter attempting to answer everything.
```

The analysis process should remain understandable.

## Advanced Use of Packet Bytes

Packet bytes are useful when:

```text id="p7w3m1"
A field is unclear
Dissection appears incomplete
You need to verify raw values
Protocol behavior is unusual
```

Use packet bytes to validate the dissector's interpretation.

Do not inspect raw bytes without a question.

## Protocol Dissection Limitations

A protocol may be:

```text id="r5c8n2"
Encrypted
Compressed
Tunneled
Fragmented
Malformed
Unsupported
Partially captured
```

If Wireshark cannot fully dissect it, document the limitation rather than forcing an interpretation.

## Advanced Profiles and Preferences

Different investigations may benefit from different Wireshark configurations.

Useful concepts include:

```text id="y6m2q8"
Profiles
Columns
Name resolution
Protocol preferences
Time display
Coloring rules
```

Avoid changing settings without understanding their effect on analysis.

For reproducible investigations, record relevant configuration changes.

## Coloring Rules

Coloring rules can help visually identify:

```text id="x8q4m7"
Retransmissions
Errors
Protocol types
Specific hosts
Specific traffic patterns
```

Remember:

```text id="n5j9k3"
Color is a visual aid.
```

It is not evidence.

## Custom Packet List Columns

Useful columns may include:

```text id="b7k3w9"
Time
Source
Destination
Protocol
Length
Info
TCP stream
TCP sequence
TCP acknowledgment
```

Choose columns based on the investigation.

Do not overload the packet list with unnecessary fields.

## Advanced Workflow Optimization

The goal is to minimize unnecessary actions.

Instead of:

```text id="v3n7m5"
Click randomly
↓
Inspect packets
↓
Try filters
↓
Open statistics
↓
Start over
```

Use:

```text id="f9k2r6"
Question
↓
Expected behavior
↓
Relevant feature
↓
Observation
↓
Next question
```

## Practical Exercise 1 — Multi-View Correlation

Choose one network conversation.

Analyze it using:

```text id="m4x8q2"
Packet List
Conversation
Follow Stream
Protocol Details
Statistics
Timeline
```

Write what each view contributes.

## Practical Exercise 2 — Timing Decomposition

Choose a slow transaction.

Measure:

```text id="r7w3n6"
DNS
TCP
TLS
Application
Termination
```

Compare it with a normal transaction.

Identify the largest difference.

## Practical Exercise 3 — Outlier Investigation

Find three similar transactions:

```text id="k6p2j9"
Normal
Normal
Outlier
```

Compare:

```text id="a8m4x7"
Endpoints
Ports
Protocols
Packet count
Timing
Retransmissions
Response behavior
```

Determine which observable difference explains the outlier best.

## Practical Exercise 4 — Host Communication Graph

Choose one internal host.

Map:

```text id="z5q8c3"
Host
 ├── DNS
 ├── Web
 ├── Authentication
 ├── File services
 └── Other destinations
```

Classify each relationship as:

```text id="y9m3v6"
Expected
Unknown
Requires investigation
```

Do not assign malicious intent without evidence.

## Practical Exercise 5 — TCP Deep Dive

Choose one problematic TCP stream.

Document:

```text id="j2r7k5"
Handshake
Sequence progression
ACK progression
Retransmissions
Duplicate ACKs
Window
Termination
```

Explain the sequence in plain language.

## Practical Exercise 6 — Cross-Capture Comparison

If two captures are available for the same event, compare them.

Determine:

```text id="q4n8m1"
Which packets appear in both?
Which packets appear in only one?
Where does the observed sequence diverge?
Could capture location explain the difference?
```

## Practical Exercise 7 — Encrypted Traffic

Choose one encrypted session.

Record:

```text id="w6k3p9"
SNI
Certificate
ALPN
TLS version
Timing
Traffic volume
DNS relationship
```

Then state exactly what cannot be determined.

## Practical Exercise 8 — Advanced Security Investigation

Choose unusual traffic and perform:

```text id="x7m2c5"
Baseline
↓
Outlier identification
↓
Endpoint analysis
↓
Destination analysis
↓
Protocol analysis
↓
Timing
↓
DNS
↓
Encryption
↓
Internal communication
↓
Timeline
↓
Alternative explanations
```

Produce an evidence-backed conclusion.

## Practical Exercise 9 — Analyst Handoff

Give another analyst only:

```text id="h5q9r2"
Investigation question
Capture name
Relevant time window
Known host
Known destination
Three packet references
```

Ask them to reproduce the finding.

Then compare their result with yours.

## Practical Exercise 10 — Independent Advanced Investigation

Choose an unfamiliar capture.

Do not begin with a known filter.

Perform a complete investigation using your own decisions:

```text id="p8w4n6"
1. Define the question.
2. Establish scope.
3. Understand capture context.
4. Orient using statistics.
5. Identify relevant endpoints.
6. Identify conversations.
7. Build hypotheses.
8. Select filters.
9. Analyze protocol layers.
10. Analyze timing.
11. Analyze streams.
12. Analyze anomalies.
13. Correlate events.
14. Validate findings.
15. Document limitations.
16. Produce a final conclusion.
```

The important result is not simply the conclusion.

It is whether another analyst can understand how you reached it.

## Advanced Evidence Record

Use:

```text id="c6n9q4"
Investigation:

Question:

Capture:

Capture context:

Time window:

Primary host:

Relevant destinations:

Expected behavior:

Observed behavior:

Filters:

Conversations:

Streams:

Protocol findings:

Timing findings:

Statistical findings:

Anomalies:

Evidence references:

Alternative explanations:

Capture limitations:

Conclusion:

Not established:

Additional evidence:
```

## Advanced Investigation Checklist

Before closing an advanced analysis:

* [ ] Investigation question defined
* [ ] Expected behavior established
* [ ] Scope established
* [ ] Capture context understood
* [ ] Initial overview completed
* [ ] Relevant endpoints identified
* [ ] Relevant conversations identified
* [ ] Relevant streams examined
* [ ] Protocol layers correlated
* [ ] Timing decomposed
* [ ] Outliers investigated
* [ ] TCP/UDP behavior examined
* [ ] Encryption metadata examined where relevant
* [ ] DNS correlated where relevant
* [ ] Statistics consulted
* [ ] I/O graphs used where useful
* [ ] Anomalies validated at packet level
* [ ] Alternative explanations considered
* [ ] Capture limitations documented
* [ ] Important packet references recorded
* [ ] Findings are reproducible
* [ ] Conclusion is proportional to evidence
* [ ] Unknowns documented
* [ ] Additional evidence identified

## Completion Criteria

You should be able to take a complex or unfamiliar capture and independently decide:

```text id="r1m6w8"
Where should I start?

What should normal behavior look like?

Which endpoints matter?

Which conversations matter?

Which filters should I construct?

Which protocol layers should I inspect?

Where is the important timing difference?

Which statistics help?

Which packets prove the finding?

What alternative explanations exist?

What capture limitations matter?

Can another analyst reproduce the conclusion?
```

The advanced workflow is:

```text id="n8q4v2"
Orient
  ↓
Hypothesize
  ↓
Narrow
  ↓
Analyze
  ↓
Correlate
  ↓
Measure
  ↓
Validate
  ↓
Document
```

The goal is not to use every advanced feature in every investigation.

The goal is to know **when a feature will reduce uncertainty and when it will only add noise**.
