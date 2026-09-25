# Timing, Performance, and I/O

## Objective

This file teaches how to use packet timing and I/O analysis to investigate performance problems.

A user may report:

```text
"The website is slow."
"The connection keeps freezing."
"The download is inconsistent."
"The application sometimes takes several seconds."
```

Wireshark can help determine where the delay is observed.

The objective is not to label something "slow" from intuition.

The objective is to measure:

```text
When did the event occur?
How long did it take?
Where did the delay appear?
What happened during the delay?
Did packets retransmit?
Did the application wait?
Did the server respond?
Did traffic stop completely?
```

The core workflow is:

```text id="t8k5yb"
Performance Question
    ↓
Define Time Window
    ↓
Identify Transaction
    ↓
Measure Timing
    ↓
Inspect I/O Pattern
    ↓
Correlate TCP/Application Events
    ↓
Identify Delay Location
    ↓
Validate With Packets
    ↓
Document Evidence
```

## Why Timing Matters

Two captures can contain identical packets but very different timing.

Example:

```text id="y0w4h2"
Request
Response
```

Capture A:

```text id="v1d6r9"
Request → 20 ms → Response
```

Capture B:

```text id="w4f0sa"
Request → 5 s → Response
```

The packet content may be similar.

The user experience is completely different.

Timing makes that difference measurable.

## Time Is Evidence

A timestamp tells you when a packet was captured.

A time difference tells you how long an observed interval lasted.

For example:

```text id="f7d4n0"
Request: 10:00:00.100
Response: 10:00:01.100
```

The observed interval is approximately:

```text id="p6s2av"
1 second
```

This is an observation.

It does not automatically identify the cause.

## Relative Time

Relative timing is often useful for performance analysis.

Example:

```text id="u6j9p2"
T+0.000  TCP SYN
T+0.010  SYN/ACK
T+0.011  ACK
T+0.020  Request
T+1.520  Response
```

This makes the major delay obvious.

## Absolute Time

Absolute timestamps are useful when correlating Wireshark with:

* application logs
* server logs
* firewall events
* monitoring alerts
* user reports
* incident timelines

When correlating multiple systems, consider clock differences.

## Time Delta

A time delta represents the interval between relevant events.

Example:

```text id="3x0q8p"
Event A = 10:01:00.000
Event B = 10:01:00.250

Delta = 250 ms
```

Useful intervals include:

```text id="8f7r0e"
SYN → SYN/ACK
Request → Response
Packet → Retransmission
ClientHello → ServerHello
Application message → Reply
```

## Time to First Response

For an application request, one useful measurement is:

```text id="p0s7wk"
Request sent
      ↓
First relevant response
```

This can help distinguish:

* immediate response
* moderate delay
* significant delay

But it does not necessarily equal application processing time.

Network transit, buffering, protocol behavior, and capture location can contribute.

## Connection Establishment Time

For TCP:

```text id="8o4q4r"
SYN
↓
SYN/ACK
↓
ACK
```

You can examine the time between:

```text id="i6w1v9"
SYN
```

and:

```text id="z2n3q5"
SYN/ACK
```

This provides an observed round-trip-related interval at the capture point.

Do not treat it as a perfect measurement of end-to-end network latency in every capture architecture.

## TCP Handshake Timing

Example:

```text id="x7c5m1"
SYN       10:00:00.000
SYN/ACK   10:00:00.040
ACK       10:00:00.041
```

Observed SYN-to-SYN/ACK interval:

```text id="r4g6c2"
40 ms
```

This can be useful when comparing:

```text id="v6m0b8"
Successful session
```

with:

```text id="2k9j3n"
Slow session
```

## Application Request Timing

For HTTP or another request/response protocol:

```text id="x3z8v6"
Request
↓
Response
```

Measure:

```text id="6p1f4j"
Response timestamp - request timestamp
```

Then compare it with:

```text id="f5j2q8"
TCP establishment
TLS handshake
Retransmissions
Application behavior
```

## TLS Timing

For HTTPS, break the connection into stages:

```text id="m3q8s1"
TCP establishment
      ↓
TLS handshake
      ↓
Encrypted application data
```

Measure the intervals separately.

This helps determine whether the observed delay appears during:

```text id="q4d7y9"
TCP
TLS
Application exchange
```

## End-to-End Timing vs Segment Timing

A user-facing delay can contain several components.

For example:

```text id="z0r5v4"
DNS
+
TCP establishment
+
TLS handshake
+
HTTP request/response
+
Additional application requests
```

Therefore:

```text id="6j8s2n"
Total user-perceived delay
```

may be much larger than:

```text id="m4x9f0"
One individual packet interval
```

Always define what you are measuring.

## Timing a Web Transaction

A practical workflow:

```text id="6c5x2k"
1. Identify DNS resolution if relevant.
2. Identify TCP connection.
3. Measure TCP establishment.
4. Identify TLS handshake if present.
5. Measure TLS setup.
6. Identify application request.
7. Measure request-to-response interval.
8. Identify retries or retransmissions.
9. Check subsequent application requests.
10. Calculate total transaction duration.
```

## I/O Graphs

I/O Graphs visualize packet or byte activity over time.

They are useful for understanding:

```text id="5r2y6m"
Traffic bursts
Traffic gaps
Packet rates
Byte rates
Periodic behavior
Changes before and after an event
```

The graph is a map of activity over time.

It is not a diagnosis.

## I/O Graph Workflow

Use:

```text id="f9w0c3"
1. Open I/O Graphs.
2. Select an appropriate time interval.
3. Observe overall traffic.
4. Identify spikes and gaps.
5. Add focused filters.
6. Compare traffic categories.
7. Return to packet analysis.
```

## Choosing the Interval

The graph interval changes what you see.

Small interval:

```text id="v8h2k4"
Fine-grained bursts
Short delays
Packet-level changes
```

Large interval:

```text id="j2m5p7"
Overall trends
Long incidents
Sustained traffic changes
```

Choose the interval based on the problem.

## Packets Per Second

A packets-per-second view can help identify:

* bursts
* high-frequency communication
* scanning patterns
* periodic traffic
* traffic spikes

Example:

```text id="0y7k5b"
Time →      1s   2s   3s   4s
Packets →   10   12   11   200
```

The spike at 4 seconds deserves investigation.

It does not explain itself.

## Bytes Per Second

Bytes per second is useful for:

* throughput analysis
* downloads
* uploads
* bulk transfers
* traffic spikes

Example:

```text id="7r9x2v"
Time →      1s   2s   3s   4s
Bytes →     1M   1M   2M   25M
```

This indicates a large increase in traffic volume.

## Packets vs Bytes

These metrics describe different behaviors.

### High packets, low bytes

Possible pattern:

```text id="m7n1z5"
Many small messages
```

### Low packets, high bytes

Possible pattern:

```text id="k8c4q2"
Fewer large packets
```

Neither pattern is inherently good or bad.

Use protocol and context to interpret it.

## Traffic Bursts

A traffic burst can occur during:

* file transfer
* application startup
* software update
* backup
* synchronization
* web page loading
* scanning
* streaming

Investigate the burst by narrowing the time window.

## Burst Investigation

Workflow:

```text id="h2f8q6"
I/O spike
   ↓
Identify timestamp
   ↓
Filter around time
   ↓
Identify endpoints
   ↓
Identify conversations
   ↓
Identify protocol
   ↓
Inspect stream
```

Example:

```text id="a1s5k8"
14:10:00–14:10:05
```

may contain one large HTTPS transfer.

That is more useful than simply recording:

```text id="w5x9d2"
Traffic spike occurred.
```

## Traffic Gaps

A gap can indicate:

* application waiting
* server processing
* network delay
* retransmission recovery
* connection idle time
* no traffic being generated
* capture loss

Always inspect the events immediately before and after the gap.

## Application Waiting

Example:

```text id="j9k2s4"
HTTP request
        ↓
3-second gap
        ↓
HTTP response
```

This tells you there was a 3-second observed interval.

It does not tell you exactly what happened during those three seconds.

Potential causes include:

* server processing
* network delay
* packet loss
* buffering
* upstream dependency

Further evidence is required.

## Retransmission and Performance

TCP retransmissions can contribute to application delay.

Example:

```text id="v4m8q2"
Request segment
↓
No expected acknowledgement
↓
Retransmission
↓
Acknowledgement
↓
Response
```

The retransmission is observable evidence.

Check:

* when it occurred
* which segment was retransmitted
* whether duplicate ACKs preceded it
* whether application progress resumed afterward

## Retransmission Timing

Compare:

```text id="h7d2q1"
Original packet timestamp
```

with:

```text id="x8f4m6"
Retransmission timestamp
```

A large interval can contribute to visible application delay.

But do not automatically assume the retransmission was the sole cause.

## Duplicate ACKs and Performance

Duplicate ACKs can indicate that TCP is reacting to missing or reordered data.

A simplified pattern:

```text id="q4n6z9"
Data
↓
Duplicate ACK
↓
Duplicate ACK
↓
Retransmission
↓
Recovery
```

This is useful when explaining why an application exchange experienced delay.

## Out-of-Order Packets

Out-of-order delivery can appear in TCP analysis.

It may result from:

* packet reordering
* network paths
* capture artifacts
* retransmission interactions

Do not automatically equate:

```text id="f0g4m7"
Out-of-order
```

with:

```text id="x2k9p1"
Packet loss
```

Use the surrounding TCP behavior.

## Zero Window and Performance

A TCP zero-window condition can indicate that a receiver temporarily cannot accept more data.

Conceptually:

```text id="p6x3r8"
Sender → Data
Receiver → Window = 0
```

Potential implications include:

* receiver application not consuming data quickly enough
* receive-buffer pressure
* host resource constraints
* application backpressure

The capture can show the protocol condition.

It may not establish the exact host-level cause.

## Window Updates

After a zero-window condition, a window update can indicate that the receiver can accept more data.

Example:

```text id="s4m8v1"
Window = 0
    ↓
Wait
    ↓
Window Update
    ↓
Data resumes
```

This sequence can help explain pauses in data transfer.

## TCP Round-Trip Behavior

TCP acknowledgements provide useful timing information.

You may compare:

```text id="v6n2c7"
Data packet
↓
ACK
```

to understand observed acknowledgement timing.

Be careful when interpreting this as exact network RTT.

Capture location and TCP behavior matter.

## Throughput Analysis

Throughput can be approximated by:

```text id="5w1k7p"
Transferred bytes
-----------------
Elapsed time
```

For example:

```text id="6g4s2m"
10 MB transferred
over 2 seconds

≈ 5 MB/s
```

This is a simple average.

It does not describe instantaneous throughput.

## Average vs Peak Throughput

These are different.

### Average

```text id="2n8x5r"
Total data / total time
```

### Peak

Highest observed transfer rate during a selected interval.

I/O graphs can help reveal the difference.

A transfer might have:

```text id="f3x7p9"
Average: moderate
Peak: very high
```

This could represent bursty application behavior.

## Throughput Direction

Always ask:

```text id="q1z4v6"
Who is sending?
Who is receiving?
```

A download and upload can have similar byte counts but opposite direction.

For example:

```text id="2p6m8k"
Client → Server
```

versus:

```text id="6v3x9q"
Server → Client
```

Direction is critical for troubleshooting.

## Slow Download Workflow

If a user reports a slow download:

```text id="m8j2v5"
1. Identify the download conversation.
2. Identify total bytes.
3. Measure duration.
4. Examine bytes over time.
5. Check TCP retransmissions.
6. Check duplicate ACKs.
7. Check window behavior.
8. Check pauses.
9. Compare with a successful transfer if available.
```

Then determine where the observable limitation occurs.

## Slow Upload Workflow

For a slow upload:

```text id="c7n3p5"
1. Identify client-to-server data.
2. Measure throughput.
3. Check retransmissions.
4. Check receiver acknowledgements.
5. Check advertised window.
6. Check zero-window behavior.
7. Check application response.
```

This can help distinguish sender-side and receiver-side behavior.

## Slow Application Workflow

For a slow application request:

```text id="z6m1q8"
DNS
 ↓
TCP
 ↓
TLS
 ↓
Request
 ↓
Wait
 ↓
Response
```

Measure each stage separately.

The goal is to answer:

```text id="p2y8x4"
Where does the time go?
```

## Comparing Fast and Slow Sessions

This is one of the most useful performance techniques.

Find:

```text id="q8f2m7"
Fast transaction
Slow transaction
```

Compare:

```text id="v1n5k3"
DNS time
TCP setup
TLS setup
Request/response delay
Retransmissions
Window behavior
Traffic volume
```

Look for the first meaningful divergence.

## Example Comparison

Fast:

```text id="g6q1v3"
TCP setup: 20 ms
TLS: 30 ms
HTTP response: 100 ms
```

Slow:

```text id="w5k8r2"
TCP setup: 20 ms
TLS: 30 ms
HTTP response: 5 s
```

This suggests the difference appears after the TLS setup and during the application exchange.

It does not automatically establish why.

## Performance Baselines

A single transaction can be misleading.

Where possible, compare:

```text id="f2n7m5"
Multiple successful requests
Multiple slow requests
```

Look for:

* consistent delay
* intermittent delay
* destination-specific delay
* time-specific delay
* protocol-specific delay

Repeated observations are stronger than one isolated event.

## Intermittent Performance Problems

An intermittent problem may look like:

```text id="e5x9p1"
Fast
Fast
Slow
Fast
Slow
Fast
```

This suggests that the problem is not necessarily constant.

Compare the slow and fast transactions.

Ask:

```text id="z2q7m4"
Same destination?
Same server?
Same TCP behavior?
Same TLS behavior?
Same request?
Same packet-loss pattern?
```

## Server Processing vs Network Delay

Wireshark can show:

```text id="h8m4p7"
Request sent
Response received later
```

But determining whether the server spent the entire interval processing is often impossible from a client-side capture alone.

The delay could include:

```text id="s7x3k2"
Network
Server processing
Upstream dependency
Queueing
Buffering
Application behavior
```

Use server logs when available and authorized.

## Capture Location and Performance

Performance measurements depend heavily on where the capture was taken.

For example:

```text id="f4q9m2"
Client-side capture
```

can observe:

```text id="a7n5c8"
Client → Server
```

but may not show:

```text id="j3w6x0"
Server → Backend
Backend → Database
```

A client-side delay therefore does not automatically identify which internal component caused it.

## Capture Loss and Performance

Dropped packets during capture can create misleading observations.

Check for:

* capture statistics
* packet loss indicators
* missing sequence ranges
* incomplete conversations
* capture limitations

Before concluding that the network dropped packets, determine whether the capture itself may have lost them.

## I/O Graph Comparison

Multiple graph series can help compare traffic categories.

For example:

```text id="u8k4p1"
All traffic
HTTPS
DNS
TCP retransmissions
```

Questions:

```text id="n2y7s6"
Does the application spike align with retransmissions?
Does DNS activity increase before the delay?
Does traffic stop during the problem?
Does traffic resume after a response?
```

This is a correlation exercise.

## Filtering Performance Traffic

Use filters to isolate the relevant traffic.

Examples:

```text id="7d3m1k"
tcp.analysis.retransmission
```

```text id="2q9v6x"
tcp.analysis.duplicate_ack
```

```text id="c4m7p8"
tcp.analysis.out_of_order
```

```text id="8r5w2n"
tcp.analysis.zero_window
```

You can combine these with protocol or host filters.

For example:

```text id="g1p6s4"
ip.addr == 10.10.10.20 && tcp.analysis.retransmission
```

The exact analysis fields available can depend on Wireshark's dissection and version.

## Performance Investigation With I/O Graphs

A practical sequence:

```text id="v4k8m1"
I/O Graph
    ↓
Identify abnormal interval
    ↓
Filter time window
    ↓
Identify endpoint
    ↓
Identify conversation
    ↓
Inspect TCP analysis
    ↓
Inspect application timing
    ↓
Determine evidence-supported explanation
```

## Practical Exercise 1: Measure TCP Setup

Find several TCP connections.

For each, record:

```text id="q3f7v8"
SYN time
SYN/ACK time
Observed interval
```

Compare the sessions.

## Practical Exercise 2: Measure Application Response

Find an HTTP or similar request/response pair.

Record:

```text id="m7z2q5"
Request time
Response time
Observed delay
```

Repeat for several transactions.

Determine whether the delay is consistent.

## Practical Exercise 3: TLS Timing

Find a TLS session.

Measure:

```text id="f5x8n2"
TCP establishment
TLS handshake
Encrypted application start
```

Determine which stage consumes the most observed time.

## Practical Exercise 4: Build an I/O Graph

Create a graph for the capture.

Identify:

```text id="b3m9r7"
Highest traffic interval
Lowest traffic interval
Major burst
Major gap
```

Investigate one spike at packet level.

## Practical Exercise 5: Retransmission Impact

Find a conversation containing retransmissions.

Determine:

```text id="p9c2x6"
Original packet
Retransmission
Time difference
Application event occurring around it
```

Explain the observed relationship.

## Practical Exercise 6: Zero-Window Investigation

Find a TCP conversation containing a zero-window event.

Determine:

```text id="t4q7n1"
Which endpoint advertised zero window?
What data was being transferred?
How long did the condition last?
When did the window reopen?
Did data resume?
```

Document what the capture proves and what it does not.

## Practical Exercise 7: Fast vs Slow Comparison

Find:

```text id="k8m3v5"
One fast transaction
One slow transaction
```

Compare:

```text id="q1f6s9"
TCP
TLS
Application timing
Retransmissions
Packet volume
```

Identify the first meaningful difference.

## Practical Exercise 8: Throughput Calculation

Choose a bulk transfer.

Determine:

```text id="m6p2z4"
Total bytes
Start time
End time
Duration
Average throughput
```

Then compare the average with the I/O graph.

## Practical Exercise 9: Burst Investigation

Find a major traffic burst.

Determine:

```text id="c5r8y2"
Time
Endpoints
Protocol
Conversation
Packet count
Byte count
```

Explain the burst using packet evidence.

## Practical Exercise 10: Intermittent Delay

Find multiple requests to the same service.

Separate:

```text id="v2n7m5"
Fast requests
Slow requests
```

Compare their:

* connection behavior
* timing
* retransmissions
* response codes
* destinations

Determine whether the delay is consistent or intermittent.

## Practical Exercise 11: Multi-Layer Performance Analysis

Choose a slow web transaction.

Measure:

```text id="r7m2x9"
DNS
TCP
TLS
HTTP
TCP recovery events
```

Create a timeline.

Identify where most of the observed time occurs.

## Practical Exercise 12: Performance Evidence Record

For one investigation, document:

```text id="n4c8q2"
Question:
Affected host:
Destination:
Protocol:
Time window:
Connection setup:
TLS setup:
Application delay:
Retransmissions:
Window behavior:
Throughput:
Observed bottleneck:
Unknowns:
```

## Common Mistakes

### Mistake 1: Calling a Delay a Root Cause

A delay is an observation.

Find the evidence behind it.

### Mistake 2: Treating RTT as Perfect Network Latency

Capture location and protocol behavior affect timing measurements.

### Mistake 3: Blaming the Network Automatically

Application processing can introduce large delays.

### Mistake 4: Ignoring Retransmissions

TCP recovery can materially affect application timing.

### Mistake 5: Ignoring Window Behavior

Zero-window conditions can pause data transfer.

### Mistake 6: Looking at Average Throughput Only

Bursts and pauses can be hidden by averages.

### Mistake 7: Ignoring Capture Loss

A missing packet can create an apparent delay or retransmission pattern.

### Mistake 8: Using an Inappropriate Time Interval

A graph can hide important behavior if the interval is too large.

### Mistake 9: Looking at One Transaction

Performance problems are often intermittent.

### Mistake 10: Confusing Correlation With Causation

A retransmission occurring during a delay does not automatically prove it caused the entire delay.

## Professional Performance Investigation

A strong investigation might look like:

```text id="w8q3p1"
Question:
Why does the web application sometimes take several seconds?

Scope:
Client 10.10.10.20 communicating with the application server.

Observation:
TCP establishment is consistently fast.

Observation:
TLS establishment is also consistent.

Observation:
Fast requests receive responses within approximately 100 ms.

Observation:
Slow requests show a multi-second request-to-response interval.

Observation:
One slow exchange contains TCP retransmission activity.

Interpretation:
The observed delay occurs primarily after the application request is sent.

Additional evidence:
The retransmission occurs during one slow exchange.

Limitation:
The capture does not establish whether the delay originated from packet loss, server processing, an upstream dependency, or another factor.
```

This is a professional performance conclusion because it identifies what the capture supports and what it cannot establish.

## Performance Checklist

### Timing

* [ ] Define the event being measured.
* [ ] Record timestamps.
* [ ] Calculate relevant intervals.
* [ ] Compare multiple transactions.
* [ ] Consider clock and capture limitations.

### TCP

* [ ] Measure connection establishment.
* [ ] Check retransmissions.
* [ ] Check duplicate ACKs.
* [ ] Check out-of-order behavior.
* [ ] Check zero-window conditions.
* [ ] Check window updates.
* [ ] Check termination.

### Application

* [ ] Measure request/response time.
* [ ] Check response status.
* [ ] Check retries.
* [ ] Check redirects.
* [ ] Compare successful and slow transactions.

### I/O

* [ ] Build an I/O graph.
* [ ] Choose an appropriate interval.
* [ ] Identify spikes.
* [ ] Identify gaps.
* [ ] Compare packet and byte rates.
* [ ] Investigate interesting intervals.

### Reasoning

* [ ] Distinguish delay from cause.
* [ ] Distinguish correlation from causation.
* [ ] Consider capture location.
* [ ] Consider capture loss.
* [ ] Document uncertainty.

## Completion Criteria

You are ready to continue when you can independently:

* measure useful packet and transaction timing
* work with relative and absolute time
* analyze TCP connection establishment timing
* analyze application request/response timing
* analyze TLS timing
* use I/O Graphs
* identify traffic spikes and gaps
* compare packet rate and byte rate
* investigate retransmissions
* investigate duplicate ACKs
* investigate out-of-order packets
* investigate zero-window behavior
* calculate approximate throughput
* compare fast and slow transactions
* identify where an observed delay occurs
* avoid assigning unsupported root causes
* account for capture location and capture loss
* document a professional performance investigation

The core skill is:

```text id="j6v3n9"
User-Reported Slowness
    ↓
Measure Time
    ↓
Break Into Layers
    ↓
Inspect I/O
    ↓
Inspect TCP
    ↓
Inspect Application
    ↓
Find Observable Delay
    ↓
Validate
    ↓
Document Evidence and Limitations
```

The next file will add another important layer: **Expert Information and anomaly-oriented analysis**, where Wireshark's protocol warnings and unusual packet conditions become part of the investigation workflow.
