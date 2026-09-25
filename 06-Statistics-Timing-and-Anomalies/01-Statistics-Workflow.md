# Statistics Workflow

## Objective

This file teaches how to use Wireshark's statistical views to move from individual packets toward higher-level understanding of a capture.

Packet-by-packet inspection is essential, but large captures can contain thousands or millions of packets.

Statistics help answer questions such as:

```text
Which protocols are present?
Which hosts are communicating?
Which conversations dominate the capture?
Which applications are generating traffic?
How much traffic exists?
Which communication patterns deserve investigation?
```

The core workflow is:

```text id="7mqv3e"
Question
    ↓
Choose the appropriate statistic
    ↓
Summarize the capture
    ↓
Identify interesting pattern
    ↓
Narrow the traffic
    ↓
Return to packet-level evidence
    ↓
Interpret
```

Statistics are an investigation accelerator.

They are not a replacement for packet analysis.

## Why Statistics Matter

Imagine a capture containing:

```text id="2xv4ne"
500,000 packets
```

Opening packets one at a time is inefficient.

Instead, start with questions such as:

```text id="8t5j1a"
What protocols exist?
Which protocol generates the most traffic?
Which hosts are most active?
Which conversations carry the most bytes?
Are there unusual protocols?
Is traffic concentrated around a small number of endpoints?
```

Statistics reduce a large dataset into useful summaries.

## The Statistics Mental Model

Think of statistics as different lenses over the same capture.

```text id="tqz8c7"
Capture
   ├── Protocol Hierarchy
   ├── Endpoints
   ├── Conversations
   ├── I/O Graphs
   ├── Packet Lengths
   ├── TCP-related statistics
   └── Protocol-specific statistics
```

Each lens answers different questions.

Do not expect one statistic to explain an entire incident.

## Choosing the Right Statistic

Use the question to select the view.

| Question                                   | Useful Wireshark View        |
| ------------------------------------------ | ---------------------------- |
| What protocols exist?                      | Protocol Hierarchy           |
| Which hosts are active?                    | Endpoints                    |
| Who communicates with whom?                | Conversations                |
| When does traffic occur?                   | I/O Graphs                   |
| How large are packets?                     | Packet Lengths               |
| Which TCP behavior is present?             | TCP statistics               |
| What does an application protocol contain? | Protocol-specific statistics |

The exact available statistics depend on protocol support and Wireshark version.

## Statistics Before Filtering

When investigating an unfamiliar capture, avoid immediately applying a highly specific filter.

Start broad.

A useful initial sequence is:

```text id="m9qg5n"
1. Observe capture size.
2. Review protocol hierarchy.
3. Review endpoints.
4. Review conversations.
5. Review timing.
6. Identify interesting traffic.
7. Apply focused filters.
```

This prevents premature tunnel vision.

## Protocol Hierarchy

Protocol Hierarchy Statistics provide a high-level view of protocols detected in the capture.

A simplified example:

```text id="t5f0j3"
Frame
 └── Ethernet
      └── IPv4
           └── TCP
                └── TLS
```

Another capture might show:

```text id="3a5sdy"
Frame
 └── Ethernet
      └── IPv4
           ├── UDP
           │    └── DNS
           └── TCP
                └── HTTP
```

This quickly reveals the protocol population.

## Protocol Hierarchy Questions

Ask:

```text id="jz9p7k"
Which protocols are present?
Which protocols dominate?
Are unexpected protocols present?
Is encryption common?
Are there protocols I did not expect?
```

For example:

```text id="g7j3px"
Expected:
DNS
HTTPS
SSH

Observed:
DNS
HTTPS
SSH
FTP
```

The unexpected protocol deserves investigation.

It does not automatically indicate a security problem.

## Protocol Hierarchy Interpretation

Protocol percentages represent packet/byte relationships according to the statistics being displayed.

Do not assume:

```text id="7r6tqz"
50% of packets = 50% of application usage
```

unless the specific statistic supports that interpretation.

Always inspect what the columns represent.

## Protocol Hierarchy and Encapsulation

One packet can contain several protocol layers.

For example:

```text id="q3qj0d"
Ethernet
  ↓
IPv4
  ↓
TCP
  ↓
TLS
```

Protocol hierarchy therefore reflects encapsulation.

It is not necessarily a simple list of independent application categories.

## Protocol Hierarchy Investigation

A practical workflow:

```text id="c9g6xq"
1. Open Protocol Hierarchy.
2. Identify major protocols.
3. Expand interesting branches.
4. Note unexpected protocols.
5. Identify encryption.
6. Identify protocol combinations.
7. Return to packet view.
8. Filter the protocol of interest.
```

## Endpoint Statistics

Endpoint statistics answer:

```text id="n6zj9m"
Which addresses are active?
```

Useful information can include:

* address
* packet count
* byte count
* direction
* protocol-specific activity

The exact columns depend on the endpoint type.

Use endpoints to identify:

```text id="1a8l4r"
High-volume hosts
High-frequency hosts
Rare hosts
Unexpected peers
```

Then move to conversations.

## Conversations Statistics

Conversations answer:

```text id="w3n0d5"
Who is communicating with whom?
```

Useful fields may include:

* source
* destination
* packets
* bytes
* duration
* direction

The exact information depends on the protocol.

Use conversations to identify:

```text id="psm6qm"
Largest transfers
Most frequent communication
Long-lived sessions
Short-lived sessions
Unexpected relationships
```

## Endpoint → Conversation Workflow

A powerful workflow is:

```text id="8x2v8y"
Protocol Hierarchy
       ↓
Endpoint
       ↓
Conversation
       ↓
Stream
       ↓
Packet analysis
```

This progressively reduces the investigation scope.

## Example: Finding a Suspicious Host

Suppose a capture contains many hosts.

Start:

```text id="qv0t2b"
Protocol Hierarchy
```

Then:

```text id="7m6r1h"
Endpoints
```

Identify an endpoint with unusual communication.

Then:

```text id="5f8v4d"
Conversations
```

Determine its peers.

Then:

```text id="h3l8w2"
Filter the conversation
```

Finally:

```text id="a8t4z0"
Inspect packets and streams
```

The statistics identify where to look.

The packets establish what happened.

## I/O Graphs

I/O Graphs help visualize traffic over time.

They can answer:

```text id="v2tq0a"
When does traffic increase?
When does traffic decrease?
Are there bursts?
Are there quiet periods?
When does the failure occur?
Does traffic change after an event?
```

Conceptually:

```text id="0a8h8c"
Traffic
  ^
  |       ███
  |      █████
  |  ██ ███████
  | ███████████
  +-----------------> Time
```

The exact graph appearance depends on the capture and configuration.

## Why I/O Graphs Matter

A packet list tells you:

```text id="8w7d1x"
Packet A
Packet B
Packet C
...
```

An I/O graph tells you:

```text id="2z6f9k"
Traffic pattern over time
```

This is especially useful for:

* performance analysis
* bursts
* outages
* scanning patterns
* periodic traffic
* traffic spikes
* incident timelines

## Basic I/O Graph Workflow

Start with:

```text id="l2n3q8"
1. Open I/O Graphs.
2. Choose an appropriate time interval.
3. Observe overall traffic.
4. Identify spikes or gaps.
5. Add focused graph filters.
6. Compare multiple traffic categories.
7. Return to packet view around interesting intervals.
```

The graph identifies time windows.

Packet analysis explains them.

## Choosing the Time Interval

A graph can look very different depending on the interval.

For example:

```text id="2o8s9k"
1 second
```

may reveal short bursts.

Whereas:

```text id="t3p0fc"
60 seconds
```

may smooth them into a broad trend.

Choose an interval based on the question.

If investigating:

```text id="n2m5tq"
A short application delay
```

use a relatively small interval.

If investigating:

```text id="j8w5sz"
A long incident
```

a larger interval may be more useful.

## Traffic Bursts

A spike in an I/O graph means traffic increased during that interval.

Possible causes include:

* download
* upload
* backup
* scan
* software update
* application activity
* attack traffic
* normal scheduled activity

The graph alone cannot identify the cause.

Click or filter into the corresponding time window.

## Traffic Gaps

A sudden traffic gap may indicate:

* idle period
* application pause
* network outage
* capture loss
* filtering
* connection termination
* waiting for server response

A gap is an observation.

Investigate before assigning a cause.

## Comparing Traffic Categories

I/O Graphs become more useful when comparing categories.

For example:

```text id="0p7n4q"
All traffic
HTTPS traffic
DNS traffic
TCP retransmissions
```

You can then ask:

```text id="7e6q2s"
Does the retransmission spike align with the application delay?
```

This is much stronger than looking at one metric alone.

## Statistics and Time Correlation

A useful investigation pattern is:

```text id="f7c1e0"
I/O spike
   ↓
Identify time window
   ↓
Filter packets in that interval
   ↓
Identify endpoints
   ↓
Identify conversations
   ↓
Identify protocol
   ↓
Inspect stream
```

This combines several Wireshark capabilities.

## Packet Length Statistics

Packet-length analysis can reveal traffic characteristics.

Questions include:

```text id="0r9y5p"
Are packets mostly small?
Are there many maximum-sized packets?
Are packet sizes highly variable?
Did packet sizes change during an event?
```

Packet size alone rarely identifies an application.

But it can support traffic-pattern analysis.

## Packet Length Distribution

A capture may contain:

```text id="g4s6yt"
Many small packets
Few large packets
```

or:

```text id="z5v1qa"
Mostly large packets
```

These patterns can represent different workloads.

Examples:

```text id="2x4l7f"
Small frequent packets
→ interactive communication

Large packets
→ bulk transfer
```

These are general patterns, not guaranteed classifications.

## Statistics and Security Analysis

Statistics can help identify:

* unusual destinations
* unexpected protocols
* high-volume transfers
* scanning patterns
* periodic communication
* sudden traffic spikes
* unusual endpoint relationships

But statistics should be used for:

```text id="9v4c1e"
Prioritization
```

rather than:

```text id="k1z6mw"
Automatic verdicts
```

## High-Volume Traffic Investigation

Suppose one conversation dominates the capture.

Workflow:

```text id="r6y8r1"
1. Identify conversation.
2. Identify endpoints.
3. Identify protocol.
4. Check duration.
5. Check direction.
6. Follow stream if appropriate.
7. Determine whether volume is expected.
```

Possible legitimate explanations include:

* backup
* streaming
* software update
* file transfer
* database replication

Context determines significance.

## High-Packet-Count Investigation

A conversation with a very high packet count but modest byte volume may involve:

* frequent small messages
* interactive traffic
* polling
* control traffic
* repeated connection attempts
* scanning

Investigate the protocol and timing pattern.

## Long-Duration Investigation

For a long-lived conversation:

```text id="a6x3j4"
Duration
+
Packets
+
Bytes
+
Direction
+
Protocol
```

can provide useful context.

Then inspect:

```text id="x3c1i7"
Idle periods
Keepalives
Application messages
Termination
```

## Short-Duration Investigation

A very short conversation may represent:

```text id="g8v0f4"
Successful request
Failed connection
DNS exchange
Port probe
Application rejection
```

Look at the actual packets.

## Protocol Statistics

Wireshark provides protocol-specific statistics for supported protocols.

Depending on the capture, these can provide useful information about:

* requests
* responses
* errors
* message types
* endpoints
* transactions

The exact available statistics depend on the protocol and Wireshark version.

Use them when the investigation question is protocol-specific.

## Protocol Statistics Workflow

```text id="j4w1o2"
1. Identify protocol.
2. Open its available statistics.
3. Review aggregate behavior.
4. Identify unusual or important entries.
5. Filter to relevant traffic.
6. Inspect packets.
```

Do not use protocol statistics without understanding what they count.

## Statistics as a Filtering Strategy

Statistics can help generate better filters.

Example:

```text id="7d1b4m"
Endpoint statistics
      ↓
Find 10.10.10.50
      ↓
Conversation statistics
      ↓
Find TCP/443
      ↓
Filter
      ↓
Inspect TLS
```

The statistics become a navigation tool.

## Statistics as an Investigation Accelerator

Instead of:

```text id="m0r5y6"
Read 100,000 packets
```

use:

```text id="w9f2z8"
1. Protocol hierarchy
2. Endpoint statistics
3. Conversation statistics
4. I/O graph
5. Focused filter
6. Stream
7. Packet evidence
```

This is significantly more scalable.

## Statistics and Capture Scope

Statistics only describe what the capture contains.

If the capture begins after the event:

```text id="5u3r2m"
Statistics may omit the earlier behavior.
```

If a capture filter excluded traffic:

```text id="9v5x8a"
Statistics cannot recover excluded packets.
```

Always consider capture scope.

## Statistics and Missing Traffic

An apparent absence of traffic can have multiple explanations:

```text id="a0m4x1"
No traffic occurred
```

or:

```text id="8z6p7r"
Traffic was not captured
```

or:

```text id="2k7v4m"
Traffic was filtered out
```

or:

```text id="h7y1q5"
Traffic used another interface/path
```

Never treat absence as proof without considering capture limitations.

## Statistics and Display Filters

Display filters can affect some statistical views depending on how the statistic is generated and the current Wireshark behavior.

Before interpreting a filtered statistic, understand:

```text id="m7c0e4"
Is this statistic based on the complete capture?
Or the currently filtered packet set?
```

When uncertain, verify the view's behavior in your Wireshark version.

## Practical Exercise 1: Protocol Overview

Open Protocol Hierarchy for a supplied capture.

Record:

```text id="7s3f9a"
Top protocols
Encrypted protocols
Unexpected protocols
Major protocol combinations
```

Then explain what the capture appears to contain at a high level.

## Practical Exercise 2: Endpoint Statistics

Open endpoint statistics.

Identify:

```text id="0t1z9k"
Top endpoint by packets
Top endpoint by bytes
Least active endpoint
Unexpected endpoint
```

Investigate one endpoint further.

## Practical Exercise 3: Conversation Statistics

Find:

```text id="j4r3n6"
Largest conversation
Highest packet-count conversation
Longest conversation
Shortest conversation
```

For each, determine the likely reason for its traffic pattern.

## Practical Exercise 4: Build an I/O Graph

Create an I/O graph for the entire capture.

Identify:

```text id="v5p2x6"
Traffic spikes
Traffic gaps
Quiet periods
Major changes
```

Choose one interesting time window.

Return to packet analysis.

## Practical Exercise 5: Compare Two Protocols

Create two graph series, such as:

```text id="n7z4x1"
DNS
TCP
```

Compare their timing.

Ask:

```text id="2k9q0b"
Does DNS activity precede application connections?
Are there spikes?
Are there periods where one continues while the other stops?
```

## Practical Exercise 6: High-Volume Conversation

Find the highest-byte conversation.

Determine:

```text id="4r1x8v"
Endpoints
Protocol
Duration
Packet count
Byte count
Direction
```

Then follow the stream if appropriate.

## Practical Exercise 7: High-Frequency Communication

Find a conversation with unusually high packet count.

Determine:

```text id="7f5m2a"
Packet rate
Protocol
Message pattern
Duration
```

Determine whether the pattern is consistent with:

* polling
* interactive traffic
* control traffic
* scanning
* another behavior

Support the interpretation with evidence.

## Practical Exercise 8: Traffic Spike Investigation

Find the largest I/O spike.

Determine:

```text id="9h2y6p"
Time
Source
Destination
Protocol
Conversation
Packet count
Byte count
```

Then explain what caused the spike based on packet evidence.

## Practical Exercise 9: Traffic Gap Investigation

Find a significant traffic gap.

Determine:

```text id="k0w6n5"
What happened before the gap?
What happened after?
Was a connection waiting?
Was there a retransmission?
Did an application response arrive later?
```

Do not assume the gap means a network outage.

## Practical Exercise 10: Statistics-to-Packet Workflow

Start with no specific host in mind.

Use:

```text id="5c7x9q"
Protocol Hierarchy
→ Endpoint
→ Conversation
→ I/O Graph
→ Filter
→ Stream
→ Packet analysis
```

Document how each step reduced the investigation scope.

## Practical Exercise 11: Security-Oriented Statistics

Using an authorized capture, identify:

```text id="n0s8x4"
Unexpected protocols
Unexpected endpoints
High-volume external communication
Unusual communication frequency
Unusual destination diversity
```

For each candidate, perform packet-level validation.

Do not classify a host as malicious based only on statistics.

## Practical Exercise 12: Build a Statistics Investigation Record

For one investigation, record:

```text id="g4m7q2"
Question:
Capture scope:
Protocol hierarchy findings:
Endpoint findings:
Conversation findings:
Timing findings:
Interesting traffic:
Focused filter:
Packet-level validation:
Interpretation:
Unknowns:
```

This creates a repeatable analysis process.

## Common Mistakes

### Mistake 1: Treating Statistics as Conclusions

Statistics identify patterns.

They do not automatically explain them.

### Mistake 2: Ignoring Capture Scope

A statistic cannot show packets that were never captured.

### Mistake 3: Using the Wrong Statistic

Choose the view based on the question.

### Mistake 4: Looking Only at Bytes

Packet count and timing can reveal different behavior.

### Mistake 5: Looking Only at Packet Count

Large transfers can involve relatively few packets compared with high-frequency traffic.

### Mistake 6: Ignoring Time

Traffic volume without timing loses important context.

### Mistake 7: Treating a Spike as Malicious

Spikes can be completely normal.

### Mistake 8: Ignoring Protocol Hierarchy

Unexpected protocols may be important even when they do not generate much traffic.

### Mistake 9: Stopping at Statistics

Always return to packet evidence for important conclusions.

### Mistake 10: Overinterpreting Absence

No observed traffic does not necessarily mean no traffic existed.

## Professional Statistics Workflow

A mature investigation often follows:

```text id="0x8qvf"
Question
    ↓
Protocol Hierarchy
    ↓
Endpoints
    ↓
Conversations
    ↓
Timing / I/O Graph
    ↓
Interesting pattern
    ↓
Focused filter
    ↓
Stream
    ↓
Packet validation
    ↓
Interpretation
    ↓
Document limitations
```

Example:

```text id="z9p1mc"
Question:
Why did traffic increase sharply around 14:32?

Protocol hierarchy:
HTTPS dominates the capture.

I/O graph:
Large traffic spike at 14:32.

Conversations:
One client/server conversation accounts for most bytes.

Stream:
Large application transfer observed.

Interpretation:
The spike corresponds to a high-volume encrypted application transfer.

Limitation:
The encrypted application payload cannot be interpreted from the available capture.
```

This is a complete statistics-driven investigation.

## Statistics Checklist

### Initial Review

* [ ] Check capture scope.
* [ ] Review Protocol Hierarchy.
* [ ] Review Endpoints.
* [ ] Review Conversations.
* [ ] Review traffic over time.

### Investigation

* [ ] Identify unusual protocols.
* [ ] Identify important endpoints.
* [ ] Identify important conversations.
* [ ] Check packet count.
* [ ] Check byte count.
* [ ] Check duration.
* [ ] Check direction.
* [ ] Check timing.
* [ ] Use I/O graphs where useful.

### Validation

* [ ] Apply focused filters.
* [ ] Inspect packets.
* [ ] Follow streams where appropriate.
* [ ] Correlate with protocol behavior.
* [ ] Validate statistical observations.

### Reasoning

* [ ] Separate pattern from explanation.
* [ ] Avoid treating unusual as malicious automatically.
* [ ] Consider missing traffic.
* [ ] Consider capture scope.
* [ ] Document unknowns.

## Completion Criteria

You are ready to continue when you can independently:

* explain why statistics are useful
* choose an appropriate statistic for an investigative question
* use Protocol Hierarchy Statistics
* use endpoint statistics
* use conversation statistics
* compare packet counts and byte counts
* use I/O Graphs
* investigate traffic spikes
* investigate traffic gaps
* analyze traffic patterns over time
* use statistics to generate focused filters
* move from statistics back to packet-level evidence
* account for capture limitations
* distinguish unusual behavior from proven malicious behavior
* document statistics-based findings professionally

The core skill is:

```text id="7h4w8c"
Large Capture
    ↓
Statistical Summary
    ↓
Interesting Pattern
    ↓
Focused Traffic
    ↓
Packet Evidence
    ↓
Evidence-Based Interpretation
```

Statistics make Wireshark scalable.

The next file will build on this by focusing specifically on **timing, performance, and I/O analysis**.
