# Expert Information and Anomalies

## Objective

This file teaches how to use Wireshark's Expert Information and packet-level anomalies as investigation signals.

Wireshark can identify protocol conditions that may deserve attention, such as:

* retransmissions
* duplicate acknowledgements
* malformed packets
* protocol errors
* unusual sequence behavior
* connection resets
* checksum-related warnings
* missing or unexpected protocol elements
* unusual application behavior

These indicators are useful.

They are not automatic diagnoses.

The core principle is:

```text id="m6x0q8"
Expert Information
    ↓
Investigation Signal
    ↓
Packet Validation
    ↓
Context
    ↓
Interpretation
```

Do not use:

```text id="v4c2n7"
Expert warning = confirmed problem
```

Instead use:

```text id="f8y5t2"
Expert warning
    ↓
What exactly was detected?
    ↓
Where?
    ↓
When?
    ↓
Why might it occur?
    ↓
Does packet evidence support the interpretation?
```

## What Is Expert Information?

Wireshark's Expert Information system summarizes protocol conditions that may be useful during analysis.

Depending on the protocol and Wireshark version, entries can include categories such as:

```text id="0v3q9n"
Errors
Warnings
Notes
Chats
```

The exact messages and classifications depend on protocol dissectors and Wireshark's current implementation.

Expert Information is designed to help analysts find potentially important packets without manually inspecting the entire capture.

## Expert Information Is a Starting Point

Suppose Wireshark identifies:

```text id="s9m1d4"
TCP Retransmission
```

The correct workflow is:

```text id="4p8y2x"
Find retransmission
    ↓
Inspect original packet
    ↓
Inspect ACK behavior
    ↓
Inspect timing
    ↓
Inspect application context
```

Do not immediately conclude:

```text id="j6w3q8"
The network is broken.
```

## Expert Severity

Expert entries can have different severity levels.

A higher severity does not automatically mean a larger operational impact.

For example:

```text id="3q7n5m"
Protocol warning
```

may be harmless in the specific environment.

Likewise:

```text id="d9x2r5"
No warning
```

does not prove that the traffic is healthy.

Expert Information is one source of evidence among many.

## Opening Expert Information

The exact menu path can vary between Wireshark releases.

Use Wireshark's current Expert Information view to review summarized protocol conditions.

A typical workflow is:

```text id="x2v8m6"
1. Open Expert Information.
2. Review categories.
3. Identify repeated conditions.
4. Select an interesting item.
5. Locate the corresponding packet.
6. Inspect packet details.
7. Correlate with conversation and timing.
```

## Expert Information Workflow

A disciplined workflow is:

```text id="z7n1p4"
Expert entry
    ↓
Packet
    ↓
Conversation
    ↓
Stream
    ↓
Timeline
    ↓
Context
    ↓
Interpretation
```

This prevents overreliance on summarized warnings.

## TCP Retransmissions

Retransmissions are one of the most common TCP analysis signals.

A simplified pattern is:

```text id="n5y2x8"
Data
↓
Expected ACK missing
↓
Retransmission
```

Potential explanations can include:

* packet loss
* congestion
* receiver behavior
* reordering-related conditions
* capture artifacts

Do not automatically identify the root cause from one retransmission.

## Retransmission Investigation

When you find a retransmission, inspect:

```text id="s2d8m5"
Original packet
Retransmitted packet
Sequence number
Acknowledgements
Timing
Direction
Application context
```

Then ask:

```text id="u6k9p1"
Did application traffic pause?
Did the retransmission occur during a user-visible delay?
Was it repeated?
```

## Duplicate ACKs

Duplicate ACKs can indicate that the receiver is acknowledging previously received data while waiting for missing or reordered data.

A simplified pattern:

```text id="k4x7m2"
Segment N
Segment N+2
Duplicate ACK
Duplicate ACK
Retransmission of N+1
```

This is useful evidence.

It does not automatically establish the exact reason the segment was missing.

## Out-of-Order Packets

Wireshark may identify packets as out of order.

Possible causes include:

* network reordering
* parallel paths
* retransmission behavior
* capture architecture
* timing differences

Do not treat every out-of-order packet as a fault.

Investigate the surrounding sequence.

## TCP Fast Retransmissions

TCP may retransmit data before a traditional retransmission timeout when sufficient duplicate acknowledgements indicate a likely missing segment.

The important analytical question is:

```text id="j3q5w9"
What happened before the fast retransmission?
```

Inspect:

```text id="a4x8v2"
Duplicate ACKs
Sequence numbers
Timing
Following data
Application behavior
```

## TCP Resets

A TCP reset can indicate an abrupt termination.

Example:

```text id="8m3c7v"
Client → Request
Server → RST
```

Possible explanations include:

* application rejection
* closed service
* firewall behavior
* connection policy
* protocol mismatch
* abnormal termination

The RST itself does not reveal which explanation is correct.

## RST Investigation

Record:

```text id="p7n4x2"
Who sent the RST?
When?
What happened immediately before it?
Was application data exchanged?
Was the connection newly established?
Was a response expected?
```

Then correlate with:

```text id="m5q8z1"
TCP flags
Application protocol
Timing
Other connections
```

## TCP Zero Window

A zero-window event indicates that the receiver advertised no available receive window.

Conceptually:

```text id="0d7k3m"
Sender → Data
Receiver → Window = 0
```

This may indicate receiver-side backpressure.

Possible causes include:

* application not consuming data quickly enough
* receive-buffer pressure
* host resource conditions
* application-level processing delay

The capture demonstrates the TCP condition.

It may not establish the host-level cause.

## TCP Window Full

A sender may transmit enough data to fill the receiver's advertised window.

This can be relevant when investigating throughput or receiver-side limitations.

Do not interpret a single window-related event in isolation.

Look for:

```text id="s8m2c5"
Repeated window pressure
Zero-window events
Window updates
Application timing
```

## TCP Keepalive

Keepalive traffic may appear in long-lived TCP sessions.

It can be normal for:

* persistent connections
* idle sessions
* application infrastructure

Do not classify keepalive traffic as anomalous simply because it is periodic.

## Checksum Warnings

Wireshark may report checksum-related problems.

Be careful.

A common issue is checksum offloading.

The packet can appear to contain an invalid checksum at the capture point because the operating system or network interface has not yet inserted the final checksum when the packet is captured.

Therefore:

```text id="h8y4v1"
Checksum warning
```

does not automatically mean:

```text id="w2f9s6"
Corrupted packet on the network
```

## Checksum Investigation

When you encounter a checksum warning, consider:

```text id="q3m7x8"
Is checksum offloading enabled?
Where was the capture taken?
Is the packet locally generated?
Does the remote side acknowledge it?
Does the problem repeat?
```

If the capture is from a host itself, offloading can be especially relevant.

## Malformed Packets

Wireshark may identify malformed or unexpected protocol structures.

Possible explanations include:

* genuinely malformed traffic
* unsupported protocol behavior
* truncated capture
* dissector limitations
* unusual but valid protocol extensions
* capture corruption

Do not immediately classify malformed traffic as malicious.

Inspect the actual packet.

## Malformed Packet Workflow

```text id="v4j7n1"
Malformed indication
    ↓
Inspect packet bytes
    ↓
Inspect protocol tree
    ↓
Check frame length
    ↓
Check capture truncation
    ↓
Check surrounding packets
    ↓
Check protocol expectations
```

Then decide whether the evidence supports a protocol problem.

## Truncated Packets

A packet may be captured with less data than was actually transmitted.

This can happen because of capture settings or capture architecture.

A truncated packet may cause:

```text id="n8p4q5"
Incomplete protocol dissection
```

or:

```text id="z1x6m3"
Malformed-looking packet
```

Therefore, always consider capture completeness.

## Capture Truncation vs Network Malformation

These are different possibilities.

### Capture limitation

```text id="m4k7c2"
The capture did not contain the complete packet.
```

### Network/protocol issue

```text id="y8n3v5"
The captured packet itself violates expected protocol structure.
```

Determine which explanation is supported before reporting a malformed packet as a network fault.

## Expert Information and DNS

DNS-related expert information can help identify conditions such as:

* malformed responses
* unusual response behavior
* protocol errors
* repeated failures

When investigating DNS anomalies, correlate with:

```text id="p6z2r8"
Query
Response
Response code
Timing
Retries
Destination
```

A DNS warning should lead to packet inspection.

## Expert Information and HTTP

HTTP-related warnings or unusual behavior may point toward:

* malformed requests
* malformed responses
* unexpected fields
* protocol violations

Use:

```text id="q7m3x1"
HTTP packet
↓
Request/response
↓
TCP stream
↓
Application context
```

to determine significance.

## Expert Information and TLS

TLS-related expert information can be useful for:

* handshake failures
* alerts
* malformed messages
* unexpected negotiation behavior

Investigate:

```text id="r9w2k6"
ClientHello
ServerHello
Certificate
TLS alerts
TCP behavior
```

Do not rely on the expert message alone.

## Expert Information and ICMP

ICMP messages can provide useful diagnostic information.

For example:

```text id="3n5v8p"
Destination Unreachable
Time Exceeded
```

These should be correlated with the traffic that triggered them.

An ICMP error is often most useful when you identify:

```text id="c7m1z4"
Which original packet or flow was affected?
```

## Expert Information and Application Protocols

Protocol-specific expert messages may identify unusual behavior.

Examples can include:

* invalid fields
* unexpected message types
* protocol errors
* failed exchanges

The workflow remains:

```text id="t2p8q6"
Warning
↓
Packet
↓
Conversation
↓
Application context
↓
Interpretation
```

## Anomaly Mental Model

An anomaly is something that differs from the expected or established baseline.

The baseline matters.

For example:

```text id="5h2m8x"
Normal:
Workstation makes occasional DNS requests.

Observed:
Workstation makes hundreds of DNS requests per minute.
```

The difference is the anomaly.

Without a baseline:

```text id="c8v1n4"
"Unusual"
```

can be subjective.

## Types of Network Anomalies

Useful categories include:

```text id="9r3x7k"
Protocol anomalies
Timing anomalies
Volume anomalies
Endpoint anomalies
Conversation anomalies
Sequence anomalies
Application anomalies
Behavioral anomalies
```

Each requires different evidence.

## Protocol Anomalies

Examples:

* malformed protocol fields
* unexpected message types
* invalid sequences
* unexpected encapsulation

Start with the protocol specification and Wireshark dissection.

## Timing Anomalies

Examples:

```text id="m5w8q2"
Normally 100 ms
Observed 5 seconds
```

Timing anomalies are strongest when compared against a baseline.

## Volume Anomalies

Examples:

```text id="x4j6p1"
Normal:
10 MB/hour

Observed:
2 GB in 10 minutes
```

Investigate:

* destination
* protocol
* application
* direction
* context

## Endpoint Anomalies

Examples:

```text id="r8q2n6"
Internal workstation
communicating with
unexpected external destination
```

This deserves investigation.

It does not automatically prove compromise.

## Conversation Anomalies

Examples:

* unexpected peer
* unusual port
* unusually long connection
* unusually high packet count
* repeated failed connections
* unusual connection frequency

Always compare with expected behavior.

## Sequence Anomalies

Examples:

* unexpected TCP flags
* retransmission patterns
* repeated resets
* unusual sequence behavior

Sequence analysis requires packet-level inspection.

## Application Anomalies

Examples:

* repeated authentication failures
* unexpected HTTP methods
* unusual URI patterns
* repeated application errors
* unusual request frequency
* unexpected protocol usage

Application context is essential.

## Baseline-Based Analysis

A useful approach is:

```text id="u8r4c7"
Establish normal
    ↓
Observe deviation
    ↓
Measure deviation
    ↓
Investigate cause
    ↓
Validate against packet evidence
```

The stronger the baseline, the more meaningful the anomaly.

## Repeated Behavior

One unusual packet may be noise.

Repeated behavior is often more informative.

For example:

```text id="a6w3q9"
One failed connection
```

versus:

```text id="n1v7p4"
500 failed connections in one minute
```

The second pattern provides much stronger evidence that something systematic is happening.

Still, the correct interpretation depends on the environment.

## Frequency Analysis

Measure:

```text id="5y8m3p"
Connections per second
Requests per minute
DNS queries per minute
Resets per minute
```

Frequency can reveal:

* polling
* retries
* scanning
* application loops
* monitoring
* automated traffic

Frequency alone does not establish malicious intent.

## Destination Diversity

A source communicating with many destinations can be normal.

For example:

```text id="h7m2x8"
Browser
→ many CDNs and services
```

But a sudden increase in destination diversity may deserve investigation.

Compare:

```text id="q4v9n1"
Normal destination count
```

with:

```text id="s3k6m5"
Observed destination count
```

## Port Diversity

A host contacting many destination ports may indicate:

* service discovery
* legitimate application behavior
* monitoring
* scanning

Analyze:

```text id="x8p1m7"
Port range
Connection success/failure
Timing
Destination
Protocol
```

Do not classify scanning based only on port diversity.

## Repeated Failed Connections

A pattern such as:

```text id="g5m8c2"
SYN
RST
SYN
RST
SYN
RST
...
```

may indicate:

* service unavailable
* application behavior
* scanning
* incorrect configuration
* blocked services

Investigate source, destination, timing, and expected environment.

## Periodic Connections

Example:

```text id="j7w4n9"
Connection every 60 seconds
```

Possible explanations include:

* monitoring
* keepalive
* scheduled synchronization
* polling
* application heartbeat
* automated software

The important question is:

```text id="p6c2x8"
Is the behavior expected?
```

## Beacon-Like Patterns

Repeated communication at highly regular intervals can be an investigation signal.

For example:

```text id="k3v7q1"
10:00:00
10:01:00
10:02:00
10:03:00
```

But periodicity alone is not proof of command-and-control or malicious activity.

Investigate:

* destination
* protocol
* payload visibility
* endpoint role
* interval
* volume
* environmental baseline

## Anomaly Investigation Workflow

Use:

```text id="w6n2r4"
1. Define normal behavior.
2. Identify deviation.
3. Measure deviation.
4. Identify affected endpoint.
5. Identify conversations.
6. Identify protocol.
7. Examine timing.
8. Inspect packet sequence.
9. Correlate related events.
10. Determine plausible explanations.
11. Validate against additional evidence.
12. Document confidence and limitations.
```

## Expert Information + Statistics

These capabilities work well together.

Example:

```text id="a4q8x2"
Statistics
    ↓
Identify unusual host
    ↓
Expert Information
    ↓
Identify retransmissions/errors
    ↓
Conversation
    ↓
Stream
    ↓
Timeline
```

This creates a strong investigation path.

## Expert Information + I/O Graphs

Example:

```text id="p7m3v8"
I/O spike
    ↓
Expert Information
    ↓
Retransmissions increase
    ↓
Focused packet analysis
```

This may help explain a performance event.

Again, correlation is not automatically causation.

## Expert Information + Security Investigation

A security investigation may combine:

```text id="z5x8c3"
Unexpected endpoint
+
Unusual protocol
+
Repeated connections
+
Expert warnings
+
Abnormal timing
```

The combination may justify deeper investigation.

It still does not automatically establish malicious activity.

## Practical Exercise 1: Expert Information Review

Open Expert Information for a supplied capture.

Record:

```text id="q8m2v5"
Most common expert category
Most repeated warning
Most severe-looking entry
Affected protocol
Affected packets
```

Then inspect at least one packet for each.

## Practical Exercise 2: Retransmission Investigation

Find a retransmission.

Determine:

```text id="f6x3p9"
Original packet
Retransmitted packet
Sequence number
Timing
ACK behavior
Application context
```

Explain the event without overclaiming.

## Practical Exercise 3: TCP Reset Investigation

Find a TCP reset.

Determine:

```text id="r2m7k4"
Sender
Receiver
Time
Previous packet
Application state
Connection duration
```

Write an evidence-based interpretation.

## Practical Exercise 4: Checksum Warning Investigation

Find a checksum warning if available.

Determine:

```text id="v9n4q1"
Protocol
Packet source
Capture location
Whether offloading could explain it
Whether the remote endpoint responded
```

Do not label the packet corrupted without sufficient evidence.

## Practical Exercise 5: Malformed Packet Investigation

Find a malformed packet.

Determine:

```text id="x5c8m2"
What protocol reported the issue?
Which field appears problematic?
Is the packet truncated?
Does the surrounding traffic support the warning?
```

Document whether the issue appears to be:

```text id="m7p1r6"
Capture-related
Protocol-related
Dissector-related
Uncertain
```

## Practical Exercise 6: Baseline Anomaly

Choose an endpoint.

Establish its normal communication pattern within the capture.

Then identify one deviation.

Record:

```text id="n4q8s2"
Baseline
Deviation
Evidence
Possible explanations
Further validation
```

## Practical Exercise 7: Frequency Anomaly

Find a host with repeated connections.

Measure:

```text id="c7m2x9"
Number of connections
Time window
Approximate frequency
Destination
Protocol
```

Determine whether the pattern is expected or requires investigation.

## Practical Exercise 8: Destination Diversity

Choose one endpoint.

Determine:

```text id="w3p8k5"
Number of destinations
Most common destination
Least common destination
Protocol distribution
```

Investigate one unusual destination.

## Practical Exercise 9: Port Diversity

Choose one source.

Determine:

```text id="y6n1r4"
Destination ports
Connection success/failure
Destination count
Timing
```

Determine whether the traffic resembles:

* normal application behavior
* service discovery
* monitoring
* scanning
* another pattern

Support the conclusion with evidence.

## Practical Exercise 10: Periodic Communication

Find a repeated communication pattern.

Measure:

```text id="p2x7m5"
Destination
Protocol
Interval
Duration
Bytes
Packets
```

Determine possible explanations.

Do not label it malicious from periodicity alone.

## Practical Exercise 11: Combine Statistics and Expert Information

Use:

```text id="h8m3q6"
Endpoint statistics
→ Conversation statistics
→ Expert Information
→ Packet analysis
```

Identify one endpoint with unusual behavior.

Explain how each stage narrowed the investigation.

## Practical Exercise 12: Build an Anomaly Investigation Record

Document:

```text id="z4c8v1"
Anomaly:
Baseline:
Affected endpoint:
Time window:
Protocol:
Conversation:
Expert information:
Packet evidence:
Possible explanations:
Evidence supporting each:
Unknowns:
Confidence:
Next investigation step:
```

## Common Mistakes

### Mistake 1: Treating Expert Information as Ground Truth

Expert Information identifies protocol conditions.

You still need context.

### Mistake 2: Treating Every Warning as Serious

Some warnings are benign or environment-dependent.

### Mistake 3: Treating Checksum Warnings as Network Corruption

Checksum offloading can produce misleading local observations.

### Mistake 4: Treating Retransmission as Proof of Network Failure

Retransmission has multiple possible explanations.

### Mistake 5: Treating Out-of-Order as Packet Loss

Reordering and capture behavior can produce out-of-order observations.

### Mistake 6: Treating Periodicity as Malicious

Legitimate systems generate periodic traffic constantly.

### Mistake 7: Ignoring the Baseline

Anomaly requires a meaningful comparison point.

### Mistake 8: Overlooking Capture Limitations

A malformed or incomplete packet may result from the capture itself.

### Mistake 9: Making Conclusions From One Packet

Repeated patterns and correlated evidence are stronger.

### Mistake 10: Confusing Suspicion With Proof

An investigation signal is not a final verdict.

## Professional Anomaly Investigation

A strong investigation might look like:

```text id="n5r8x2"
Observation:
One workstation communicates with an unexpected external host.

Statistics:
The host establishes 120 connections to that destination in ten minutes.

Timing:
Connections occur approximately every five seconds.

Expert Information:
Several TCP retransmissions occur during the communication.

Packet analysis:
The connections use TLS.

Interpretation:
The workstation is repeatedly communicating with the external destination over encrypted connections.

Security relevance:
The regular communication pattern and unexpected destination justify further investigation.

Limitation:
The encrypted application payload is not visible, so the capture alone does not establish the purpose or maliciousness of the communication.
```

This is the correct professional approach.

## Expert Information Checklist

### Expert Review

* [ ] Open Expert Information.
* [ ] Review severity categories.
* [ ] Identify repeated conditions.
* [ ] Identify affected protocols.
* [ ] Select important packets.

### TCP

* [ ] Investigate retransmissions.
* [ ] Investigate duplicate ACKs.
* [ ] Investigate out-of-order packets.
* [ ] Investigate resets.
* [ ] Investigate zero-window events.
* [ ] Check termination behavior.

### Packet Integrity

* [ ] Investigate checksum warnings.
* [ ] Consider checksum offloading.
* [ ] Investigate malformed packets.
* [ ] Check for truncation.
* [ ] Consider dissector limitations.

### Anomalies

* [ ] Establish a baseline.
* [ ] Identify deviations.
* [ ] Measure frequency.
* [ ] Measure volume.
* [ ] Check destination diversity.
* [ ] Check port diversity.
* [ ] Check periodicity.
* [ ] Correlate timing.

### Reasoning

* [ ] Validate expert warnings.
* [ ] Separate observation from interpretation.
* [ ] Avoid unsupported security conclusions.
* [ ] Consider benign explanations.
* [ ] Document uncertainty.
* [ ] Document capture limitations.

## Completion Criteria

You are ready to continue when you can independently:

* use Expert Information as an investigation starting point
* investigate TCP retransmissions
* investigate duplicate acknowledgements
* investigate out-of-order behavior
* investigate TCP resets
* investigate zero-window conditions
* understand checksum-offloading caveats
* investigate malformed packets
* recognize capture truncation
* establish a baseline
* identify timing anomalies
* identify volume anomalies
* identify endpoint anomalies
* identify conversation anomalies
* analyze periodic communication
* analyze port and destination diversity
* combine statistics with Expert Information
* validate anomalies at packet level
* distinguish investigation signals from conclusions
* document uncertainty professionally

The core workflow is:

```text id="w2x7m5"
Statistics
    ↓
Expert Information
    ↓
Anomaly
    ↓
Packet Validation
    ↓
Conversation
    ↓
Timeline
    ↓
Context
    ↓
Evidence-Based Interpretation
```

This completes **06-Statistics-Timing-and-Anomalies**.

The next section moves into practical troubleshooting workflows, beginning with diagnosing **connectivity failures** from the packet capture.
