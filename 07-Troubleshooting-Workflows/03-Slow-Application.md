# Slow Application

## Objective

A slow application is not automatically a slow network.

Users may report:

```text
"The website is slow."
"The application takes forever to load."
"Login is very slow."
"The server responds slowly."
"It works, but everything takes too long."
"Sometimes it is fast and sometimes it is slow."
```

These statements describe a user experience, not the location of the delay.

Wireshark can help determine where time is being spent by correlating:

* DNS resolution
* TCP connection establishment
* TLS negotiation
* Application request/response timing
* Server response delays
* Packet retransmissions
* Duplicate acknowledgments
* Zero-window conditions
* Connection setup
* Request sequencing
* Response delivery
* Application behavior

The goal is not to declare:

```text
"The network is slow."
```

The goal is to determine:

```text
"Which stage consumed the observed time?"
```

The central workflow is:

```text
User reports slowness
        ↓
Define the transaction
        ↓
Establish a time window
        ↓
Identify client and destination
        ↓
Measure DNS
        ↓
Measure TCP establishment
        ↓
Measure TLS when applicable
        ↓
Measure application request/response
        ↓
Inspect packet loss and retransmissions
        ↓
Inspect flow control
        ↓
Compare successful and slow transactions
        ↓
Locate the dominant delay
        ↓
Document evidence and uncertainty
```

## The Slow Application Mental Model

A user-visible transaction can contain several independent delays:

```text
Application startup
        ↓
DNS lookup
        ↓
TCP connection
        ↓
TLS negotiation
        ↓
HTTP request
        ↓
Server processing
        ↓
HTTP response
        ↓
Additional requests
        ↓
Content transfer
        ↓
Application rendering
```

A delay at any stage can contribute to the user's experience.

Wireshark primarily observes network-visible stages.

It may not be able to see:

* CPU contention inside the application
* Database processing time when no network exchange exposes it
* Local application rendering
* Client-side computation
* Internal server processing between network messages

Therefore:

```text
Long packet-to-packet delay
```

may be useful evidence of server-side processing or network behavior, but it is not automatically proof of either one.

## Define What "Slow" Means

Before opening Wireshark, define the transaction.

Record:

| Item                  | Question                               |
| --------------------- | -------------------------------------- |
| Client                | Which host experienced the delay?      |
| Application           | Which application or service?          |
| Destination           | Which server or service?               |
| Protocol              | HTTP, HTTPS, SSH, SMB, etc.?           |
| Port                  | Which service port?                    |
| Start time            | When did the transaction begin?        |
| Expected time         | What is considered normal?             |
| Observed time         | How long did it take?                  |
| Successful comparison | Is there a fast transaction available? |
| Capture point         | Where was the traffic captured?        |

Without a defined transaction, "slow" is difficult to measure.

## Define the Transaction Boundary

Choose a clear beginning and end.

For example:

```text id="9w0b9v"
DNS query
        ↓
DNS response
```

measures name resolution.

For TCP:

```text id="a2yq3u"
SYN
        ↓
SYN/ACK
```

measures the observed server response to the connection attempt.

For an HTTPS transaction:

```text id="6ujy5v"
TCP SYN
        ↓
HTTP response
```

measures a much larger sequence.

Do not compare two transactions unless their boundaries are comparable.

## Build a Timing Breakdown

A useful model is:

```text id="p5m5kv"
Total observed time
=
DNS time
+
TCP setup time
+
TLS setup time
+
Application wait time
+
Data transfer time
+
Other visible delays
```

Not every transaction contains every component.

For example:

```text id="wqz3gt"
HTTP over existing TCP connection
```

may have no new TCP setup cost.

Likewise:

```text id="b8y3z1"
Cached DNS result
```

may have no visible DNS transaction during the capture.

The timing model should follow what actually occurred.

## Step 1: Confirm the Capture Is Suitable

Before analyzing performance, verify:

* Correct interface
* Correct client
* Correct destination
* Correct time window
* Complete enough traffic
* Relevant direction visible
* No obvious capture truncation
* Timestamps are meaningful

A performance conclusion based on an incomplete capture can be misleading.

For example:

```text id="r5n2ry"
Client → Server request
```

may be visible while the response is captured elsewhere.

The apparent delay could then be an artifact of capture placement.

## Step 2: Identify the Slow Transaction

Do not begin with the entire capture.

First isolate the transaction.

Possible narrowing criteria include:

```text id="u4yr9q"
Client IP
Server IP
TCP port
Application protocol
Time window
Conversation
Stream
```

For example:

```text id="m4wq0q"
ip.addr == 192.0.2.10 && ip.addr == 192.0.2.50
```

Then narrow further based on the protocol.

The goal is:

```text id="wly5qb"
Large capture
    ↓
Relevant host pair
    ↓
Relevant protocol
    ↓
Relevant conversation
    ↓
Relevant transaction
```

## Step 3: Measure DNS Time

When a hostname is involved, inspect:

```text id="v4xj8m"
DNS Query
        ↓
DNS Response
```

Measure the elapsed time.

Possible outcomes:

```text id="i5xg1s"
Very fast DNS
        ↓
DNS probably does not dominate this transaction

Slow DNS
        ↓
DNS may contribute significantly

Repeated DNS
        ↓
Investigate retries, resolution behavior, and timing
```

Do not assume that DNS is responsible merely because the user typed a hostname.

## Example: DNS Dominates the Delay

Suppose:

```text id="3e2l7x"
DNS query
↓
2.8 seconds
↓
DNS response
↓
TCP handshake
↓
TLS
↓
HTTP
```

If the rest of the transaction is fast, DNS is a strong candidate for the observed delay.

The evidence supports:

```text id="v6j9d3"
Most of the observed pre-connection delay occurred during DNS resolution.
```

That is more precise than:

```text id="2v5y4f"
"The network is slow."
```

## Step 4: Measure TCP Establishment

For a new TCP connection, inspect:

```text id="g0n3m1"
SYN
SYN/ACK
ACK
```

The important interval is generally:

```text id="x7n7dd"
SYN
   ↓
SYN/ACK
```

and the completion of the handshake.

A normal-looking transaction might show:

```text id="2i0o0x"
SYN
↓ milliseconds
SYN/ACK
↓ milliseconds
ACK
```

A problematic transaction might show:

```text id="b5v0s9"
SYN
↓ long delay
SYN/ACK
```

This suggests that connection establishment contributed to the observed delay.

It does not by itself prove whether the delay was caused by:

* Network path latency
* Server processing
* Packet loss
* Filtering behavior
* Return-path problems
* Capture limitations

Additional evidence is required.

## TCP Retransmission During Connection Setup

Consider:

```text id="r4gq5t"
SYN
SYN retransmission
SYN retransmission
SYN/ACK
```

The delay may be related to packet loss or missing responses.

Investigate:

* Number of retransmissions
* Timing between attempts
* Whether retransmission occurs only during setup
* Whether application traffic also shows retransmissions

A slow connection caused by retransmission is different from a slow connection caused by application processing.

## Step 5: Measure TLS Setup

For HTTPS or other TLS-based protocols, separate:

```text id="l1y8jx"
TCP establishment
```

from:

```text id="6v6v6p"
TLS handshake
```

A transaction can have:

```text id="2sg0vo"
Fast TCP
Slow TLS
```

or:

```text id="c3m9ac"
Slow TCP
Fast TLS
```

or:

```text id="4p8v8g"
Fast TCP
Fast TLS
Slow application response
```

These represent different investigation paths.

## TLS Timing

Identify:

```text id="u9v2c1"
TLS ClientHello
        ↓
Server response
        ↓
TLS handshake progression
```

If a long delay occurs between handshake messages, determine:

* Which side transmitted the previous packet
* Which side was expected to respond
* Whether retransmissions occurred
* Whether an alert appeared
* Whether the connection eventually succeeded

Do not assume that a slow TLS stage means encryption itself is computationally slow.

The delay could originate from network or endpoint behavior.

## Step 6: Measure Application Request-to-Response Time

This is often the most important stage.

For an HTTP request:

```text id="q8i4dj"
Client → Server: HTTP request
        ↓
Server → Client: HTTP response
```

Measure the time between the request and the corresponding response.

A large delay can indicate:

* Server-side processing
* Database or backend dependency
* Network delay
* Request retransmission
* Application queueing
* Proxy behavior
* Capture limitations

Wireshark may not identify which internal server operation consumed the time.

It can establish that the response was not observed until a certain point.

## Server Think Time vs Network Delay

Consider:

```text id="c7u7g3"
HTTP request
        ↓
2 seconds
        ↓
HTTP response
```

If the request arrives promptly and no packet-level loss occurs during the interval, the delay may be consistent with server-side processing.

But be precise.

The packet capture establishes:

```text id="e9w6y1"
The response was observed approximately 2 seconds after the request.
```

It does not necessarily establish:

```text id="o3z9yl"
The server spent exactly 2 seconds processing the request.
```

There may be:

* Intermediate buffering
* Proxy behavior
* Scheduling delays
* Capture-point effects
* Network path behavior

Use evidence-based language.

## Step 7: Inspect Packet Loss

Packet loss can create significant delays.

Look for:

* TCP retransmissions
* Duplicate ACKs
* Fast retransmissions
* Out-of-order packets
* Selective acknowledgments
* Repeated data segments
* Missing expected responses

A simplified pattern is:

```text id="4m7p9r"
Data
↓
ACK
↓
Data
↓
Retransmission
↓
ACK
```

The retransmission may add delay.

## Retransmission vs Slow Server

Compare two patterns.

### Possible server-response delay

```text id="j8t3v1"
Request
↓
long silence
↓
Response
```

with no obvious packet-loss indicators.

### Possible network-loss behavior

```text id="p0l2cz"
Request/data
↓
retransmission
↓
duplicate ACKs
↓
eventual response
```

These deserve different hypotheses.

Do not label both simply as "server slow."

## Step 8: Inspect Duplicate ACKs

Duplicate ACKs can indicate that a receiver is acknowledging previously received data while waiting for missing data.

Repeated duplicate acknowledgments can be associated with packet loss or reordering.

Example:

```text id="9x9xw4"
Data segment
Data segment
Missing segment
Later segment
        ↓
Duplicate ACK
Duplicate ACK
Duplicate ACK
```

Interpret this together with:

* Sequence numbers
* Packet ordering
* Retransmissions
* Timing
* SACK information

One duplicate ACK is not automatically evidence of a serious network problem.

## Step 9: Inspect Out-of-Order Traffic

Packets may arrive out of sequence.

Possible causes include:

* Packet reordering
* Multiple network paths
* Capture-point artifacts
* High-speed capture effects
* Actual network behavior

Out-of-order packets are not automatically packet loss.

Check whether the missing sequence is eventually delivered and whether retransmission occurs.

## Step 10: Inspect TCP Window Behavior

A slow application can be caused by flow control.

Look for:

```text id="0j4mkk"
Zero Window
Window Full
Window updates
```

A receiver advertising a zero window indicates that it temporarily cannot accept more data.

This can produce:

```text id="a4n8n1"
Sender transmits
        ↓
Receiver window closes
        ↓
Sender pauses
        ↓
Window update
        ↓
Transmission resumes
```

The application may appear slow even though the network path itself is functioning.

Investigate:

* Which side advertised the small or zero window?
* How long did the condition last?
* Did the condition repeat?
* Did the receiver later increase the window?

## Zero Window Example

Suppose:

```text id="v7s6jc"
Server → Client: data
Client → Server: ACK, zero window
```

The server may stop transmitting until the client advertises available receive space.

The evidence indicates receiver-side flow control.

Do not automatically classify this as network congestion.

## Step 11: Look at I/O Graphs and Timing

For larger captures, packet-by-packet inspection may not immediately reveal the performance pattern.

Use statistics and graphs to identify:

* Traffic bursts
* Idle periods
* Retransmission periods
* Throughput changes
* Connection spikes
* Periodic delays

A useful workflow is:

```text id="a9v6a2"
I/O graph
    ↓
Find unusual time period
    ↓
Identify affected conversation
    ↓
Inspect packets
    ↓
Explain the timing
```

The graph helps locate the problem.

Packet analysis explains it.

## Step 12: Compare Fast and Slow Transactions

A known-good transaction is extremely valuable.

For example:

```text id="5b6d1m"
FAST:
DNS       20 ms
TCP       15 ms
TLS       40 ms
HTTP      80 ms
Total    155 ms
```

and:

```text id="1e5z0a"
SLOW:
DNS       20 ms
TCP       15 ms
TLS       40 ms
HTTP    2500 ms
Total   2575 ms
```

The difference strongly suggests that the additional observed delay occurred during the application stage.

The exact cause still requires additional evidence.

## Build a Timing Table

For important investigations, create a timing table.

| Stage                | Start | End | Duration | Observation       |
| -------------------- | ----: | --: | -------: | ----------------- |
| DNS                  |    T1 |  T2 |       Δ1 | Resolution        |
| TCP                  |    T2 |  T3 |       Δ2 | Handshake         |
| TLS                  |    T3 |  T4 |       Δ3 | Negotiation       |
| Application request  |    T4 |  T5 |        — | Request sent      |
| Application response |    T5 |  T6 |       Δ4 | Response observed |
| Data transfer        |    T6 |  T7 |       Δ5 | Content delivery  |

Not every transaction will have all stages.

Use only the stages that actually exist.

## Example: Slow DNS

```text id="f0d7h8"
DNS:      2200 ms
TCP:        20 ms
TLS:        50 ms
HTTP:       80 ms
```

The largest visible delay occurs during DNS.

Investigation should focus on:

* Resolver response time
* DNS retries
* Resolver selection
* Network path to DNS
* Resolver load
* Record behavior

## Example: Slow TCP Setup

```text id="g1v2s3"
DNS:       20 ms
TCP:     1800 ms
TLS:       50 ms
HTTP:      70 ms
```

The connection setup dominates the transaction.

Inspect:

* SYN timing
* SYN retransmissions
* SYN/ACK timing
* Packet loss
* Routing behavior
* Filtering
* Capture location

## Example: Slow TLS

```text id="m4n5b6"
DNS:       20 ms
TCP:       20 ms
TLS:     1400 ms
HTTP:      80 ms
```

The delay occurs after TCP establishment.

Investigate:

* TLS handshake progression
* Retransmissions
* Alerts
* Server response timing
* Certificate exchange visibility
* Protocol negotiation

## Example: Slow Application Response

```text id="q7r8s9"
DNS:       20 ms
TCP:       20 ms
TLS:       50 ms
HTTP:    2400 ms
```

The network setup is relatively fast.

The application request-to-response interval dominates the observed transaction.

Investigate:

* Request content
* Response timing
* Retransmissions
* Server behavior
* Backend dependencies
* Proxy behavior

The capture may support a server-processing hypothesis, but additional endpoint evidence may be required to prove the exact cause.

## Existing TCP Connections

Not every slow transaction begins with a new TCP connection.

For example:

```text id="v2c4x6"
Existing TCP connection
        ↓
New HTTP request
        ↓
Response
```

If the connection was already established, do not attribute the delay to TCP setup.

This is why transaction boundaries matter.

## HTTP Keep-Alive and Connection Reuse

Modern applications frequently reuse connections.

A sequence may look like:

```text id="k3l5m7"
TCP handshake
TLS handshake
HTTP request
HTTP response
HTTP request
HTTP response
HTTP request
HTTP response
```

Only the first request pays the connection setup cost.

Later requests should be analyzed separately.

A user may report that:

```text id="b9c1d3"
"The page becomes slow after clicking something."
```

The relevant transaction may be a later request on an existing connection.

## Multiple Application Requests

Modern web pages can generate many requests:

```text id="z2x4c6"
HTML
CSS
JavaScript
Images
API requests
Fonts
Analytics
Other resources
```

One slow request can delay the user-visible experience.

Do not assume that the first HTTP request is the entire transaction.

Identify which request corresponds to the observed delay.

## Application Parallelism

Applications may issue requests concurrently.

For example:

```text id="e4f6g8"
Request A ───────── Response A
Request B ─── Response B
Request C ───────────── Response C
Request D ── Response D
```

The page may wait for a particular response before completing.

A slow application therefore requires correlation between:

```text id="h6j8k0"
Network transaction
        ↓
Application dependency
        ↓
User-visible delay
```

Wireshark can reveal timing and ordering, but application knowledge may be needed to identify which response is actually blocking progress.

## HTTP Status Codes and Slowness

A slow response can still be successful:

```text id="r1t3y5"
HTTP 200
```

A fast error can also be immediately visible:

```text id="u7i9o1"
HTTP 500
```

Do not equate:

```text id="p3a5s7"
Successful status = fast
```

or:

```text id="d9f1h3"
Error status = slow
```

Measure timing independently from status.

## Large Responses

A transaction may be slow because the response is large.

Differentiate:

```text id="c5e7g9"
Slow response start
```

from:

```text id="i1k3m5"
Fast response start
        ↓
Slow content transfer
```

These represent different performance problems.

For a large response, examine:

* Time to first visible response
* Throughput
* Packet loss
* Retransmissions
* Window behavior
* Duration of content transfer

## Throughput vs Latency

These are different concepts.

### Latency

How long does it take for a communication step to receive a response?

### Throughput

How much data is transferred over time?

A connection can have:

```text id="a1b2c3"
Low latency
Low throughput
```

or:

```text id="d4e5f6"
High latency
High throughput after startup
```

A large file transfer that takes time is not automatically evidence of high latency.

## TCP Performance Clues

For a slow TCP flow, inspect:

* Retransmissions
* Duplicate ACKs
* Out-of-order packets
* SACK behavior
* Window size
* Zero-window periods
* Packet spacing
* Data volume
* ACK timing
* Connection termination

Use multiple signals together.

One warning or one retransmission rarely explains an entire performance incident.

## Network Delay vs Endpoint Delay

A useful reasoning framework is:

```text id="g7h8i9"
Where is the sender waiting?

Client sends request
        ↓
Does server respond quickly?
        |
        +-- YES → investigate transfer/network behavior
        |
        +-- NO
              ↓
        investigate server-side or path-related delay
```

But remember:

```text id="j0k1l2"
No response observed
```

can also result from capture visibility limitations.

The packet trace must be interpreted according to where it was captured.

## Capture Point Matters

Consider three possible capture locations.

### Client-side capture

You may see:

```text id="m3n4o5"
Client request
Client retransmission
Server response
```

This is useful for understanding the client-side experience.

### Server-side capture

You may see:

```text id="p6q7r8"
Server receives request
Server sends response
```

This can help distinguish network arrival from server response behavior.

### Intermediate capture

You may see only part of the transaction.

Therefore:

```text id="s9t0u1"
Capture location
```

should always be recorded during performance investigations.

## Asymmetric Routing

Traffic can take different paths in each direction.

For example:

```text id="v2w3x4"
Client → Server
       Path A

Server → Client
       Path B
```

A capture on one side may therefore provide incomplete information.

When performance behavior is difficult to explain, consider whether the capture point can observe both directions.

## VPN and Tunnel Effects

VPNs can change:

* Path
* MTU
* Encapsulation
* Latency
* Fragmentation behavior
* Visibility

A slow application over a VPN should be investigated with awareness of the tunnel.

For example:

```text id="y5z6a7"
Application traffic
        ↓
Encrypted tunnel
        ↓
Underlying network
```

The inner application traffic may be visible at one capture point and hidden at another.

## MTU and Fragmentation Considerations

Large packets traversing a path with a smaller MTU can create performance problems.

Investigate when there are signs such as:

* Fragmentation
* ICMP fragmentation-related messages
* Repeated retransmissions around larger segments
* VPN/tunnel deployment
* Connection works for small transactions but fails or slows with larger data

Do not conclude "MTU problem" from retransmissions alone.

Look for supporting evidence.

## Practical Exercise 1 — Measure a Fast Transaction

Find a normal application transaction.

Record:

```text id="b2c4d6"
DNS time:
TCP time:
TLS time:
Application response time:
Data transfer time:
Total visible time:
```

Identify the dominant stage.

## Practical Exercise 2 — Find a Slow DNS Transaction

Locate a transaction with unusually slow DNS.

Determine:

```text id="e8f0g2"
Query:
Response:
Delay:
Retries:
Resolver:
```

Compare it with a normal DNS transaction.

## Practical Exercise 3 — Slow TCP Setup

Find a connection with a long TCP setup.

Record:

```text id="h4j6l8"
SYN time:
SYN/ACK time:
Handshake completion:
Retransmissions:
```

Determine whether packet loss or retransmission contributes to the delay.

## Practical Exercise 4 — Slow TLS

Find a TLS transaction with a noticeable handshake delay.

Record:

```text id="n0p2r4"
TCP completion:
TLS start:
TLS progression:
TLS completion:
Retransmissions:
Alerts:
```

Determine which TLS interval contains the largest delay.

## Practical Exercise 5 — Slow Application Response

Find:

```text id="t6v8x0"
Application request
        ↓
Long delay
        ↓
Application response
```

Record the elapsed time.

Determine whether packet-loss indicators appear during the waiting period.

## Practical Exercise 6 — Zero Window

Find a TCP conversation containing a zero-window condition.

Determine:

```text id="z2b4d6"
Which host advertised zero window?
How long did it last?
Did transmission resume?
Was the condition repeated?
```

## Practical Exercise 7 — Retransmission-Driven Delay

Find a flow containing retransmissions.

Determine:

```text id="f8h0j2"
Original packet:
Retransmission:
Time difference:
Duplicate ACKs:
Did the application experience a delay?
```

Do not automatically conclude that the retransmission was the only cause.

## Practical Exercise 8 — Fast vs Slow Comparison

Find:

```text id="l4n6p8"
One fast transaction
One slow transaction
```

Create a timing table.

Identify the largest difference.

## Practical Exercise 9 — Existing Connection

Find an application transaction that occurs on an already-established TCP connection.

Determine:

```text id="r0t2v4"
When was TCP established?
When did the application request begin?
How long did the application response take?
```

This prevents incorrectly attributing the delay to connection setup.

## Practical Exercise 10 — Capture Limitation

Find a slow transaction where the capture cannot establish the complete cause.

Document:

```text id="x6z8b0"
What is visible?
What is missing?
What timing can be measured?
What cannot be measured?
What additional capture point would help?
```

## Professional Performance Evidence Record

Use this structure:

```text id="c2e4g6"
Incident:
Date/Time:
Client:
Application:
Destination:
Protocol:
Capture point:

User-visible symptom:

Expected behavior:

Observed duration:

Transaction boundary:

DNS duration:

TCP setup duration:

TLS duration:

Application request/response duration:

Data transfer duration:

Retransmissions:

Duplicate ACKs:

Out-of-order packets:

Window conditions:

Zero-window periods:

Relevant packet range:

Successful comparison:

Dominant observed delay:

Evidence:

Interpretation:

Possible hypotheses:

Capture limitations:

Additional evidence required:
```

## Common Mistakes

### Mistake 1: Calling the Network Slow

"The application is slow" does not establish that the network is responsible.

Measure each visible stage.

### Mistake 2: Looking Only at Retransmissions

Retransmissions can contribute to delay, but not every slow transaction is caused by packet loss.

### Mistake 3: Ignoring Server Response Timing

A long request-to-response interval may be important even when packet delivery itself appears clean.

### Mistake 4: Ignoring Existing Connections

A later request may reuse an established TCP/TLS session.

Do not charge every transaction with the initial handshake time.

### Mistake 5: Confusing Large Transfers With High Latency

A large response can take time simply because there is substantial data to transfer.

### Mistake 6: Treating Zero Window as Network Congestion

A zero window is receiver-side flow control evidence.

Investigate which endpoint advertised it.

### Mistake 7: Ignoring Capture Location

A missing response or unusual delay may be impossible to interpret correctly without knowing where the capture was taken.

### Mistake 8: Ignoring Application Parallelism

A page may make many concurrent requests.

The slowest or blocking request may not be the first request.

### Mistake 9: Measuring Without a Transaction Boundary

If you do not define when the transaction starts and ends, timing measurements can be meaningless.

### Mistake 10: Assuming One Packet Explains the Incident

Performance analysis is usually about sequences and timing, not one packet.

## Professional Slow-Application Workflow

Use this workflow consistently:

```text
1. Define what "slow" means.
2. Identify the exact transaction.
3. Establish a reliable time window.
4. Confirm the capture point.
5. Identify client and destination.
6. Measure DNS when applicable.
7. Measure TCP setup when a new connection exists.
8. Measure TLS when applicable.
9. Measure application request-to-response time.
10. Measure content-transfer duration.
11. Inspect retransmissions and duplicate ACKs.
12. Inspect out-of-order behavior.
13. Inspect TCP window conditions.
14. Use statistics or graphs to locate larger patterns.
15. Compare against a successful transaction.
16. Identify the dominant observed delay.
17. Separate packet evidence from hypotheses.
18. Document capture limitations.
19. Identify what additional evidence would confirm the root cause.
```

The professional question is not:

```text id="q2w4e6"
"Why is the application slow?"
```

It is:

```text id="s8u0y2"
"Where does the observed time accumulate,
what packet evidence demonstrates that delay,
and which explanations remain possible?"
```

## Performance Investigation Checklist

Before closing the investigation:

* [ ] User-visible symptom defined
* [ ] Exact transaction identified
* [ ] Client identified
* [ ] Destination identified
* [ ] Protocol identified
* [ ] Capture point documented
* [ ] DNS timing checked when relevant
* [ ] TCP setup timing checked when relevant
* [ ] TLS timing checked when relevant
* [ ] Application request/response timing checked
* [ ] Data-transfer duration considered
* [ ] Retransmissions checked
* [ ] Duplicate ACKs checked
* [ ] Out-of-order traffic checked
* [ ] Window behavior checked
* [ ] Zero-window conditions checked
* [ ] I/O or statistics used when useful
* [ ] Successful comparison performed
* [ ] Dominant observed delay identified
* [ ] Evidence separated from hypotheses
* [ ] Capture limitations documented
* [ ] Additional evidence identified

## Completion Criteria

You should be able to take a complaint such as:

```text id="a1c3e5"
"The application is slow."
```

and independently determine:

```text id="g7i9k1"
What transaction was slow?

Where did the transaction begin?

Where did it end?

How long did each visible stage take?

Did DNS contribute?

Did TCP setup contribute?

Did TLS contribute?

Did application response time contribute?

Did retransmissions contribute?

Did flow control contribute?

Was the transfer itself slow?

Was there a successful transaction for comparison?

What does the packet capture prove?

What remains uncertain?
```

The core mental model is:

```text id="m3o5q7"
Slow User Experience
        ↓
Break Into Stages
        ↓
Measure Each Stage
        ↓
Find the Largest Observable Delay
        ↓
Inspect Packets Around That Delay
        ↓
Compare With a Normal Transaction
        ↓
Separate Evidence From Hypothesis
```

Once this becomes intuitive, Wireshark stops being merely a packet viewer and becomes a timing and evidence-analysis tool.
