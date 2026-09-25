# Packet Loss and TCP Problems

## Objective

TCP problems are among the most useful incidents to investigate with Wireshark because TCP exposes a detailed sequence of:

* Data transmission
* Acknowledgments
* Sequence numbers
* Retransmissions
* Duplicate acknowledgments
* Selective acknowledgments
* Window advertisements
* Connection establishment
* Connection termination

A user may report:

```text id="2a7q6m"
"The connection is slow."
"The application keeps freezing."
"The download is inconsistent."
"The connection keeps dropping."
"The server is timing out."
"The connection works, but performance is terrible."
```

These symptoms can result from many different conditions.

The objective is not to declare:

```text id="k8r3x1"
"TCP is broken."
```

The objective is to determine:

```text id="4m9p2c"
What TCP behavior is actually occurring?
Where is the first meaningful abnormality?
Which endpoint is involved?
What evidence supports the interpretation?
What remains uncertain?
```

The central workflow is:

```text id="7s5d1v"
Define the TCP flow
        ↓
Identify client/server
        ↓
Inspect handshake
        ↓
Follow sequence numbers
        ↓
Inspect acknowledgments
        ↓
Identify retransmissions
        ↓
Check duplicate ACKs
        ↓
Check out-of-order packets
        ↓
Inspect SACK behavior
        ↓
Inspect window conditions
        ↓
Check resets and termination
        ↓
Compare normal vs problematic flows
        ↓
Determine the earliest supported problem
```

## Why TCP Analysis Matters

TCP is designed to provide reliable, ordered delivery over an IP network.

It therefore contains mechanisms that reveal how endpoints perceive the communication.

Important concepts include:

```text id="u4m6z8"
Sequence numbers
Acknowledgment numbers
Receive windows
Retransmission
Duplicate acknowledgments
Selective acknowledgment
Connection state
```

Wireshark can use these fields to identify patterns that are difficult to see from raw packet lists alone.

However, TCP analysis requires discipline.

A Wireshark-generated label such as:

```text id="m2n4b6"
TCP Retransmission
```

is useful evidence, not an automatic root-cause diagnosis.

## Define the Flow

Before analyzing TCP behavior, record:

| Item             | Question                                     |
| ---------------- | -------------------------------------------- |
| Client           | Which endpoint initiated the connection?     |
| Server           | Which endpoint accepted or should accept it? |
| Source port      | Which client-side port is involved?          |
| Destination port | Which service is involved?                   |
| Time window      | When did the problem occur?                  |
| Direction        | Which direction appears affected?            |
| Application      | What application uses the connection?        |
| Capture point    | Where was the traffic captured?              |

A TCP flow should be treated as a conversation rather than a collection of unrelated packets.

## Start With the Three-Way Handshake

A normal TCP connection begins with:

```text id="f6h8j0"
Client → Server: SYN
Server → Client: SYN, ACK
Client → Server: ACK
```

This establishes the initial communication state.

When investigating a TCP problem, first determine whether this process completed.

### Successful handshake

```text id="n2p4r6"
SYN
SYN/ACK
ACK
```

### No response

```text id="t8v0x2"
SYN
SYN
SYN
```

### Reset

```text id="b4d6f8"
SYN
RST
```

These represent different starting conditions.

## TCP Connection States

At a practical level, distinguish:

```text id="j1l3n5"
Connection attempt
        ↓
Connection established
        ↓
Data transfer
        ↓
Connection termination
```

A problem during the SYN phase is different from a problem after substantial application traffic has already been exchanged.

Always identify the phase first.

## Sequence Numbers

TCP uses sequence numbers to track data.

Simplified example:

```text id="q7s9u1"
Segment A:
Sequence = 1000
Length = 500

Segment B:
Sequence = 1500
Length = 500
```

The second segment follows the first.

The receiver can acknowledge the next sequence number it expects.

For example:

```text id="w3y5a7"
ACK = 2000
```

means that data through the preceding sequence space has been received and the receiver expects sequence 2000 next.

The exact interpretation can be affected by TCP options and Wireshark's relative sequence-number display.

The key skill is understanding sequence progression rather than memorizing numbers.

## Relative Sequence Numbers

Wireshark commonly displays relative sequence numbers to make TCP conversations easier to read.

You may therefore see:

```text id="c9e1g3"
Seq = 1
Seq = 501
Seq = 1001
```

rather than the raw 32-bit TCP sequence values.

This is normal.

When comparing packets, focus on:

```text id="i5k7m9"
Sequence progression
Acknowledgment progression
Segment length
Missing sequence ranges
```

## Acknowledgment Numbers

Acknowledgments tell the sender what data the receiver has successfully received.

A simplified pattern:

```text id="o2q4s6"
Sender → Receiver: Seq 1, length 500
Receiver → Sender: ACK 501
```

If the receiver repeatedly acknowledges the same sequence number, investigate why.

For example:

```text id="u8w0y2"
ACK 1501
ACK 1501
ACK 1501
```

may indicate that the receiver is still waiting for data beginning at sequence 1501.

This can be associated with a missing segment.

## TCP Retransmissions

A retransmission occurs when previously transmitted data is sent again.

A simplified sequence:

```text id="e4g6i8"
Segment Seq 1001
        ↓
No expected acknowledgment
        ↓
Segment Seq 1001 again
```

Wireshark may identify the later packet as:

```text id="a0c2e4"
[TCP Retransmission]
```

The important questions are:

```text id="m6o8q0"
Why was the segment retransmitted?
Was an acknowledgment missing?
Was the original packet lost?
Was the acknowledgment lost?
Was the packet merely reordered?
Was the capture point unable to observe the relevant packet?
```

Do not equate retransmission with proven packet loss.

## Retransmission Patterns

### Single isolated retransmission

```text id="r2t4v6"
Data
...
Retransmission
ACK
```

This may represent an isolated loss or other TCP recovery event.

### Repeated retransmissions

```text id="x8z0b2"
Data
Retransmission
Retransmission
Retransmission
...
```

This is more significant and can contribute to substantial delay.

Investigate:

* Timing
* Direction
* ACK behavior
* Other flows
* Capture location
* Whether recovery eventually occurs

## Fast Retransmission

TCP can sometimes retransmit before a normal retransmission timer expires.

Duplicate acknowledgments can trigger faster recovery.

A simplified pattern:

```text id="d4f6h8"
Segment 1
Segment 2
Segment 4
        ↓
Duplicate ACK
Duplicate ACK
Duplicate ACK
        ↓
Retransmission of Segment 3
```

The exact recovery behavior depends on TCP implementation and negotiated mechanisms.

The practical lesson is:

```text id="j0l2n4"
Duplicate ACKs + retransmission
```

can provide evidence of missing or reordered data.

## Duplicate ACKs

A duplicate acknowledgment occurs when an endpoint acknowledges data it has already acknowledged or repeatedly acknowledges the same next expected sequence number.

For example:

```text id="p6r8t0"
ACK 5001
ACK 5001
ACK 5001
```

This can indicate that the receiver has not received the expected sequence range.

Potential explanations include:

* Packet loss
* Packet reordering
* Capture artifacts
* Network behavior

Do not treat every duplicate ACK as severe.

Look at the surrounding packet sequence.

## Out-of-Order Packets

Suppose the receiver observes:

```text id="v2x4z6"
Seq 1001
Seq 2001
Seq 1501
```

The segment with sequence 1501 arrived after a later sequence range.

This is out-of-order delivery.

Possible explanations include:

* Packet reordering
* Multiple paths
* Network behavior
* Capture-point effects
* High-speed capture limitations

Out-of-order does not automatically mean packet loss.

Check whether the missing sequence is eventually received.

## Retransmission vs Out-of-Order

This distinction matters.

### Out-of-order

```text id="b8d0f2"
Expected:
1001
1501
2001

Observed:
1001
2001
1501
```

The missing sequence arrives later.

### Possible retransmission

```text id="h4j6l8"
1001
1501
2001

Later:
1501 again
```

The same sequence range appears again.

Use timing and TCP analysis information to determine what Wireshark is reporting.

## Selective Acknowledgment

TCP may negotiate Selective Acknowledgment (SACK).

SACK allows a receiver to indicate that it received some later data while still missing an earlier segment.

Conceptually:

```text id="n0p2r4"
Expected:
1001 1501 2001 2501

Received:
1001 2001 2501

Missing:
1501
```

The receiver can provide information about the received blocks.

This can make TCP recovery more efficient than simply reporting one cumulative acknowledgment.

When analyzing retransmissions, inspect whether SACK information is present.

## TCP Window

The receive window controls how much unacknowledged data a receiver is prepared to accept.

Conceptually:

```text id="t6v8x0"
Sender
   ↓
Data
   ↓
Receiver
   ↓
Advertised receive window
```

A small window can limit throughput.

A zero window can temporarily stop transmission.

## Zero Window

A zero-window condition means the receiver is currently advertising no available receive space.

Example:

```text id="x2z4b6"
Receiver → Sender:
ACK + Window = 0
```

The sender may pause normal data transmission.

Investigate:

* Which endpoint advertised zero
* When the zero window began
* How long it lasted
* Whether window updates followed
* Whether the condition repeated

A zero window points toward receiver-side flow control.

It does not automatically indicate network congestion.

## Window Full

Wireshark may identify traffic associated with a sender filling the advertised receive window.

A conceptual pattern:

```text id="c8e0g2"
Sender transmits until:
available receive window ≈ exhausted
```

Investigate whether:

* The receiver expands the window
* The sender pauses
* Throughput is limited
* The condition repeats

## Window Scaling

TCP can negotiate window scaling during connection establishment.

This allows larger effective receive windows.

When analyzing high-throughput connections, consider:

```text id="i4k6m8"
Negotiated window scaling
+
Advertised window
```

rather than interpreting the raw TCP window field without context.

## TCP Keepalives

Keepalive packets can appear on long-lived connections.

They may be used to determine whether an idle peer is still reachable.

Do not confuse:

```text id="o0q2s4"
TCP keepalive
```

with:

```text id="u6w8y0"
Application data
```

Keepalive behavior may explain traffic on an otherwise idle connection.

## TCP Resets

A reset can terminate or reject a TCP connection.

Examples include:

```text id="a2c4e6"
SYN
RST
```

or:

```text id="g8i0k2"
Established connection
Application traffic
RST
```

The stage matters.

Always determine:

```text id="m4o6q8"
Who sent the RST?
When?
What happened immediately before it?
Was the connection established?
```

A reset is an event to investigate, not a complete explanation.

## TCP FIN and Graceful Termination

Normal TCP termination can involve FIN and ACK exchanges.

A simplified sequence is:

```text id="s0u2w4"
Endpoint A → Endpoint B: FIN
Endpoint B → Endpoint A: ACK
Endpoint B → Endpoint A: FIN
Endpoint A → Endpoint B: ACK
```

This differs from a reset.

When investigating "connections dropping," determine whether they ended through:

```text id="y6a8c0"
FIN
```

or:

```text id="e2g4i6"
RST
```

and what application behavior preceded the termination.

## TCP Retransmission Timing

Timing is essential.

Consider:

```text id="q8s0u2"
Packet
↓
short delay
↓
ACK
```

versus:

```text id="w4y6a8"
Packet
↓
long delay
↓
Retransmission
↓
ACK
```

The second pattern may introduce significant application delay.

For every important retransmission, record:

```text id="b0d2f4"
Original packet time
Retransmission time
Elapsed time
Acknowledgment behavior
```

## TCP Problems and Application Performance

TCP-level problems often appear as application-level slowness.

For example:

```text id="h6j8l0"
HTTP request
↓
TCP retransmission
↓
additional delay
↓
HTTP response
```

The user experiences:

```text id="n2p4r6"
"Website is slow."
```

The packet capture reveals a TCP-level event contributing to that delay.

Always connect transport behavior back to the user-visible transaction.

## Direction Matters

TCP problems may occur primarily in one direction.

For example:

```text id="t8v0x2"
Client → Server:
many retransmissions

Server → Client:
normal
```

or:

```text id="z4b6d8"
Server → Client:
retransmissions

Client → Server:
normal
```

Identify the affected direction.

This can help narrow the investigation to:

* One network path
* One endpoint
* One interface
* One direction through a middlebox
* One congestion or loss condition

It does not by itself identify the root cause.

## Client-Side vs Server-Side Symptoms

Suppose a server sends data repeatedly, but the client advertises a small receive window.

The evidence may suggest a receiver-side limitation.

Conversely, if the server repeatedly retransmits because acknowledgments do not arrive, investigate the path and receiving side.

Do not assign responsibility solely from which side transmitted the retransmission.

Consider the entire exchange.

## Capture Point Limitations

A retransmission label can be misleading if the capture point cannot observe the original packet.

For example:

```text id="j0l2n4"
Original packet may have been transmitted
outside the capture point
        ↓
Later packet appears to be a retransmission
```

Possible causes include:

* Multiple capture interfaces
* SPAN behavior
* Packet duplication
* Asymmetric paths
* Capture loss
* Offloading
* Traffic visibility limitations

Wireshark's analysis is based on the packets it can see.

## TCP Offloading Considerations

Modern operating systems and network adapters can use mechanisms such as:

* TCP segmentation offload
* Generic segmentation offload
* Receive-side coalescing
* Checksum offloading

These can affect how traffic appears in host-based captures.

You may see packet sizes or checksum states that do not exactly correspond to what physically traversed the wire.

When TCP behavior appears strange, determine whether offloading could affect the capture.

Do not immediately interpret an unusual host capture as a network fault.

## Checksum Warnings

TCP checksum warnings can appear in host captures because checksum calculation may occur after packet capture.

Therefore:

```text id="p4r6t8"
Bad checksum displayed
```

does not automatically prove that the packet was transmitted with a bad checksum.

When checksum behavior matters, consider:

* Capture location
* NIC offloading
* Whether the packet was captured before checksum completion
* Whether the issue appears in an external capture

## TCP Problems With I/O Graphs

For larger incidents, use statistics to identify:

* Retransmission bursts
* Traffic spikes
* Idle periods
* Throughput changes
* Connection spikes
* Packet-rate changes

A useful workflow:

```text id="v0x2z4"
I/O graph
    ↓
Identify unusual interval
    ↓
Filter to affected conversation
    ↓
Inspect retransmissions
    ↓
Inspect ACK behavior
    ↓
Inspect window behavior
    ↓
Correlate with application timing
```

The graph helps identify when the problem occurs.

Packet analysis helps explain what happened.

## TCP Problem Pattern: Repeated Retransmissions

Example:

```text id="r8t0v2"
Data segment
Retransmission
Retransmission
Retransmission
```

Ask:

```text id="b4d6f8"
Are ACKs missing?
Are duplicate ACKs present?
Is SACK present?
Does the same direction show repeated problems?
Does the application slow down?
Does the problem affect other flows?
```

The more independent evidence points to a transport problem, the stronger the interpretation becomes.

## TCP Problem Pattern: Duplicate ACK Burst

Example:

```text id="n2p4r6"
Data
Data
Later data
Duplicate ACK
Duplicate ACK
Duplicate ACK
Retransmission
```

This can be consistent with a missing segment followed by fast recovery.

Inspect:

```text id="t8v0x2"
Sequence numbers
ACK numbers
SACK information
Timing
```

before drawing conclusions.

## TCP Problem Pattern: Out-of-Order Without Retransmission

Example:

```text id="z4b6d8"
Seq 1001
Seq 2001
Seq 1501
ACK progression continues
```

If the missing segment arrives without a retransmission, the pattern may represent reordering rather than loss.

This distinction matters because:

```text id="j0l2n4"
Reordering ≠ automatic packet loss
```

## TCP Problem Pattern: Zero Window

Example:

```text id="f6h8j0"
Server → Client: Data
Client → Server: ACK, Window = 0
Server → Client: pauses
Client → Server: Window Update
Server → Client: resumes
```

Investigate:

* Duration of zero window
* Frequency
* Which endpoint advertised it
* Whether the application slowed during the condition

## TCP Problem Pattern: Reset After Data

Example:

```text id="p2r4t6"
SYN
SYN/ACK
ACK
Application data
RST
```

Investigate:

* Direction of RST
* Application message immediately before it
* TLS/application state
* Whether similar connections succeed
* Whether a middlebox may be involved

## Compare Good and Bad TCP Flows

One of the most useful techniques is comparison.

### Normal flow

```text id="x8z0b2"
SYN
SYN/ACK
ACK
Data
ACK
Data
ACK
FIN
```

### Problematic flow

```text id="c4e6g8"
SYN
SYN/ACK
ACK
Data
Duplicate ACK
Retransmission
Duplicate ACK
Retransmission
...
```

The difference is more meaningful than any single packet.

Compare:

* Handshake
* Sequence progression
* ACK progression
* Retransmissions
* Window behavior
* Throughput
* Termination

## Practical Exercise 1 — Normal TCP Flow

Find a healthy TCP connection.

Document:

```text id="i0k2m4"
SYN:
SYN/ACK:
ACK:
Data exchange:
Termination:
```

Identify the normal sequence.

## Practical Exercise 2 — Retransmission

Find a flow containing a TCP retransmission.

Record:

```text id="o6q8s0"
Original packet:
Retransmission:
Elapsed time:
ACK behavior:
Application impact:
```

## Practical Exercise 3 — Duplicate ACKs

Find duplicate ACKs.

Determine:

```text id="u2w4y6"
Repeated ACK number:
Number of duplicate ACKs:
Missing sequence range:
Retransmission observed:
```

Then explain what evidence supports the missing-data interpretation.

## Practical Exercise 4 — Out-of-Order

Find an out-of-order sequence.

Determine:

```text id="a8c0e2"
Expected order:
Observed order:
Was the missing data later received?
Was retransmission observed?
```

Decide whether the evidence is more consistent with reordering or retransmission.

## Practical Exercise 5 — SACK

Find a TCP flow negotiating or using SACK.

Record:

```text id="g4i6k8"
SACK negotiated:
SACK information observed:
Missing sequence:
Recovery behavior:
```

## Practical Exercise 6 — Zero Window

Find a zero-window event.

Record:

```text id="m0o2q4"
Endpoint advertising zero:
Start time:
Window update:
Duration:
Application impact:
```

## Practical Exercise 7 — Reset

Find both:

```text id="s6u8w0"
SYN → RST
```

and:

```text id="y2a4c6"
Established → RST
```

Explain why they represent different connection states.

## Practical Exercise 8 — TCP Termination

Find a normally terminated connection.

Identify:

```text id="e8g0i2"
FIN:
ACK:
FIN:
ACK:
```

Then compare it with a reset-terminated connection.

## Practical Exercise 9 — Directional Problem

Find a flow where one direction shows more TCP problems than the other.

Record:

```text id="k4m6o8"
Affected direction:
Retransmissions:
Duplicate ACKs:
Window behavior:
```

Determine what additional evidence would help identify the underlying cause.

## Practical Exercise 10 — Good vs Bad Flow

Find:

```text id="q0s2u4"
One normal TCP flow
One problematic TCP flow
```

Create a comparison table covering:

```text
Handshake
Sequence progression
ACK behavior
Retransmissions
Duplicate ACKs
SACK
Window
Termination
```

Identify the first meaningful divergence.

## Professional TCP Evidence Record

Use this structure:

```text id="w6y8a0"
Incident:
Date/Time:
Client:
Server:
Source port:
Destination port:
Application:
Capture point:

User symptom:

Handshake:

Sequence behavior:

ACK behavior:

Retransmissions:

Duplicate ACKs:

Out-of-order packets:

SACK:

Window behavior:

Zero-window events:

Resets:

Termination:

Timing observations:

Successful comparison:

Earliest observed abnormality:

Evidence:

Interpretation:

Possible hypotheses:

Capture limitations:

Additional evidence required:
```

## Common Mistakes

### Mistake 1: Retransmission Equals Packet Loss

A retransmission is evidence of TCP recovery behavior.

Investigate why it occurred.

### Mistake 2: Duplicate ACK Equals Congestion

Duplicate ACKs can be associated with loss or reordering.

Inspect the sequence.

### Mistake 3: Out-of-Order Equals Packet Loss

A later-arriving segment may simply have been reordered.

Determine whether retransmission occurred.

### Mistake 4: Zero Window Equals Network Problem

A zero window indicates receiver-side flow control.

Identify which endpoint advertised it.

### Mistake 5: Every RST Is a Firewall

A reset can have many causes.

Analyze its context.

### Mistake 6: Ignoring Direction

TCP problems often affect one direction differently from the other.

Always identify the affected direction.

### Mistake 7: Ignoring Capture Location

Host captures and network captures can produce different visibility.

### Mistake 8: Ignoring Offloading

Checksum and segmentation behavior can be affected by NIC or operating-system offloading.

### Mistake 9: Looking at Only One Packet

TCP behavior is sequential.

Analyze surrounding packets.

### Mistake 10: Assuming Wireshark's Label Is the Root Cause

Wireshark identifies packet patterns.

The analyst determines their meaning in context.

## Professional TCP Troubleshooting Workflow

Use this workflow:

```text id="c2e4g6"
1. Define the affected application transaction.
2. Identify the TCP conversation.
3. Identify both endpoints.
4. Confirm the capture point.
5. Inspect the three-way handshake.
6. Establish the normal sequence progression.
7. Inspect acknowledgments.
8. Identify retransmissions.
9. Investigate duplicate ACKs.
10. Check for out-of-order delivery.
11. Inspect SACK behavior.
12. Inspect receive-window behavior.
13. Check zero-window events.
14. Check resets and termination.
15. Review timing.
16. Compare with a healthy TCP flow.
17. Correlate transport behavior with application performance.
18. Consider capture and offloading limitations.
19. Identify the earliest meaningful abnormality.
20. Document evidence and remaining uncertainty.
```

## TCP Troubleshooting Checklist

Before closing the investigation:

* [ ] Client identified
* [ ] Server identified
* [ ] Ports identified
* [ ] Application identified
* [ ] Capture point documented
* [ ] TCP handshake checked
* [ ] Sequence progression checked
* [ ] ACK progression checked
* [ ] Retransmissions checked
* [ ] Duplicate ACKs checked
* [ ] Out-of-order traffic checked
* [ ] SACK checked
* [ ] Window scaling considered
* [ ] Zero-window conditions checked
* [ ] Window-full behavior considered
* [ ] Resets checked
* [ ] FIN-based termination checked
* [ ] Timing examined
* [ ] Directionality examined
* [ ] Successful comparison performed
* [ ] Offloading considered
* [ ] Capture limitations documented
* [ ] Evidence separated from hypotheses
* [ ] Additional evidence identified

## Completion Criteria

You should be able to independently investigate a TCP performance or reliability problem and answer:

```text id="g8i0k2"
Did the TCP handshake succeed?

Was data transferred?

How did sequence numbers progress?

How did acknowledgments progress?

Were retransmissions observed?

Were duplicate ACKs observed?

Was the traffic out of order?

Was SACK involved?

Did either endpoint advertise a small or zero window?

Was the connection reset?

Was the connection terminated normally?

Which direction showed the problem?

When did the abnormal behavior begin?

Did it affect application performance?

What does the capture prove?

What remains uncertain?
```

The core mental model is:

```text id="m4o6q8"
TCP Flow
   ↓
Sequence
   ↓
Acknowledgments
   ↓
Loss / Reordering / Recovery
   ↓
Window Behavior
   ↓
Connection State
   ↓
Application Impact
```

The objective is not to memorize every TCP flag or Wireshark expert label.

The objective is to reconstruct the conversation and determine where the observed behavior diverges from the expected flow.
