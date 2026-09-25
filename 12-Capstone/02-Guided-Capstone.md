# Guided Capstone

## Purpose

This is the first full capstone investigation.

Unlike the earlier progressive challenges, this capstone combines multiple Wireshark skills into one realistic investigation.

The scenario provides enough structure to prevent the investigation from becoming directionless, but it does not provide a packet-by-packet solution.

The objective is to demonstrate that you can independently connect:

```text
Capture orientation
→ Endpoint identification
→ DNS analysis
→ TCP analysis
→ TLS/application analysis
→ Timing
→ Correlation
→ Evidence
→ Conclusion
```

Use only an authorized capture, laboratory traffic, or a supplied training PCAP.

## Scenario

A user reports:

```text
"The internal web application became very slow,
and at some points it appeared to stop responding."
```

The network team provides a packet capture covering the period in which the problem was reported.

The capture may contain traffic from:

```text
the affected workstation
other hosts
DNS
TCP
TLS
application traffic
```

You are asked to determine:

```text
Where does the visible problem occur?

What packet evidence supports the finding?

Does the capture provide evidence of a transport-level problem?

What can and cannot be concluded from the capture?
```

No root cause should be assumed before analysis.

## Investigation Objective

The primary objective is:

```text id="h5p7c2"
Determine which stage of the visible application transaction
contains the reported delay or failure.
```

The investigation should distinguish between:

```text id="q3m8x6"
DNS resolution
TCP connection establishment
TLS negotiation
application request
application response
transport behavior
```

## Rules

Follow these rules throughout the capstone.

```text id="p8v4n2"
1. Preserve the original capture.
2. Do not modify the original evidence.
3. Record important commands and filters.
4. Start broad and narrow progressively.
5. Do not assume the reported cause is correct.
6. Separate observations from interpretations.
7. Validate important findings.
8. Document capture limitations.
9. Do not inspect unrelated traffic without a reason.
10. Stop when the primary question is sufficiently answered.
```

## Phase 1: Capture Orientation

Begin by understanding the capture.

Use:

```bash id="c4x7m9"
capinfos capture.pcapng
```

Record:

```text id="x8m2p5"
Capture format:
Packet count:
Capture duration:
First packet:
Last packet:
Encapsulation:
```

If reproducibility matters, record:

```bash id="v6q3n8"
tshark --version
```

## Phase 2: Establish the Protocol Landscape

Determine which protocols exist.

A useful starting point is:

```bash id="m7c4x2"
tshark -r capture.pcapng -q -z io,phs
```

Also inspect the capture in Wireshark using:

```text id="p3n8v5"
Statistics → Protocol Hierarchy
```

Record the protocols that appear relevant to the reported application problem.

Do not assume every protocol in the capture matters.

## Phase 3: Identify Endpoints

Determine which IP addresses communicate.

For example:

```bash id="x5m9q3"
tshark -r capture.pcapng -q -z endpoints,ip
```

Also inspect:

```text id="c7v2n4"
Statistics → Endpoints
```

Ask:

```text id="j8p4m1"
Which host appears to be the affected client?

Which host appears to be the application server?

Which host appears to provide DNS?

Are there other endpoints that may be relevant?
```

Record the evidence supporting your identification.

Do not identify the application server from an IP address alone if stronger evidence is available.

## Phase 4: Identify Relevant Conversations

Use conversation statistics:

```bash id="n4x7c2"
tshark -r capture.pcapng -q -z conv,tcp
```

Then determine:

```text id="m8p3v6"
Which TCP conversations involve the suspected client?

Which destination ports are involved?

Which conversations appear related to the application?

Which conversation contains the affected transaction?
```

At this point, narrow the investigation.

## Phase 5: Establish the DNS Stage

If the application uses DNS, determine whether name resolution completed normally.

Start with:

```bash id="q5x8m2"
tshark -r capture.pcapng -Y "dns.qry.name"
```

Then extract useful fields:

```bash id="v7m3c9"
tshark -r capture.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name
```

Investigate:

```text id="p2n6x8"
Which client sent the query?

Which DNS server received it?

When was the query sent?

Was a response observed?

How long did the visible exchange take?

Was the expected address returned?
```

Do not spend excessive time on DNS if the evidence clearly shows that resolution completed quickly.

## Phase 6: Identify the TCP Connection

Once the destination address is known, identify the corresponding TCP connection.

Use a filter based on the observed endpoints and port.

For example:

```bash id="x4m8c3"
tshark -r capture.pcapng \
  -Y "tcp && ip.addr == CLIENT_IP && ip.addr == SERVER_IP"
```

Replace:

```text id="j7p2v5"
CLIENT_IP
SERVER_IP
```

with the observed addresses.

Determine:

```text id="m9c4x7"
Which host initiated the connection?

Was SYN observed?

Was SYN-ACK observed?

Was ACK observed?

Did the handshake complete?

Was the handshake delayed?
```

## Phase 7: Identify the TCP Stream

Extract the stream identifier:

```bash id="q3v8m1"
tshark -r capture.pcapng \
  -Y "tcp && ip.addr == CLIENT_IP && ip.addr == SERVER_IP" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e tcp.srcport \
  -e tcp.dstport \
  -e tcp.stream
```

Identify the stream associated with the affected transaction.

Then isolate it:

```bash id="n6x2p4"
tshark -r capture.pcapng \
  -Y "tcp.stream == STREAM_ID"
```

Replace:

```text id="c8m4v7"
STREAM_ID
```

with the actual stream number.

From this point forward, stream-level analysis should be preferred when the question concerns that specific transaction.

## Phase 8: Analyze TCP Behavior

Inspect the relevant stream for:

```text id="p7x3m9"
retransmissions
duplicate ACKs
out-of-order packets
resets
zero-window behavior
unexpected termination
timing gaps
```

Useful filters include:

```bash id="v5m8q2"
tshark -r capture.pcapng \
  -Y "tcp.stream == STREAM_ID && tcp.analysis.retransmission"
```

```bash id="x9c4n7"
tshark -r capture.pcapng \
  -Y "tcp.stream == STREAM_ID && tcp.analysis.duplicate_ack"
```

```bash id="m2p6v8"
tshark -r capture.pcapng \
  -Y "tcp.stream == STREAM_ID && tcp.analysis.out_of_order"
```

```bash id="q8n3x5"
tshark -r capture.pcapng \
  -Y "tcp.stream == STREAM_ID && tcp.flags.reset == 1"
```

Do not interpret these events individually.

Ask:

```text id="c5v9m2"
Did they occur during the affected transaction?

When did they occur?

Which direction?

How many?

Did they coincide with the reported delay?
```

## Phase 9: Establish the TLS Stage

If the application uses TLS, determine whether the handshake completed.

Start with:

```bash id="x7m4p2"
tshark -r capture.pcapng \
  -Y "tcp.stream == STREAM_ID && tls"
```

Then investigate handshake traffic:

```bash id="n3c8v6"
tshark -r capture.pcapng \
  -Y "tcp.stream == STREAM_ID && tls.handshake"
```

Determine:

```text id="q6p2m8"
Did TLS negotiation begin?

Was the server response observed?

Did the handshake progress?

Were TLS alerts observed?

How long did the visible handshake take?
```

Remember:

```text id="v8m4x1"
TLS metadata may be visible
while application content remains encrypted.
```

Do not claim to know encrypted application content that the capture does not expose.

## Phase 10: Identify Application Traffic

If the traffic is HTTP and visible:

```bash id="c9x3m7"
tshark -r capture.pcapng \
  -Y "tcp.stream == STREAM_ID && http"
```

For requests:

```bash id="m5p8v2"
tshark -r capture.pcapng \
  -Y "tcp.stream == STREAM_ID && http.request" \
  -T fields \
  -e frame.number \
  -e frame.time \
  -e http.request.method \
  -e http.host \
  -e http.request.uri
```

For responses:

```bash id="q7n4x9"
tshark -r capture.pcapng \
  -Y "tcp.stream == STREAM_ID && http.response" \
  -T fields \
  -e frame.number \
  -e frame.time \
  -e http.response.code
```

If the application traffic is encrypted, use the visible TLS and TCP metadata instead.

## Phase 11: Build a Transaction Timeline

Construct a timeline containing the important stages.

Example:

```text id="x4m7c2"
14:30:01.100
DNS query

14:30:01.115
DNS response

14:30:01.120
TCP SYN

14:30:01.135
TCP SYN-ACK

14:30:01.140
TCP ACK

14:30:01.160
TLS handshake begins

14:30:01.250
TLS handshake completes

14:30:01.300
Application request

14:30:04.800
Application response
```

The exact values will depend on the capture.

Your objective is to identify where the large gap occurs.

## Phase 12: Correlate the Delay

Do not simply identify a large timestamp gap.

Ask:

```text id="n8p3v5"
What packet begins the delay?

What packet ends it?

Which protocol stage does the gap belong to?

Was the client waiting?

Was the server transmitting?

Were retransmissions occurring?

Was the connection idle?

Did the application request precede the delay?
```

This is the core of the guided capstone.

## Phase 13: Investigate Transport Correlation

If a large delay appears, check whether TCP behavior explains it.

Look for:

```text id="p5x2m8"
retransmissions
duplicate ACKs
out-of-order packets
zero windows
window changes
resets
```

Then compare their timestamps with the delayed transaction.

For example:

```text id="c8m4v7"
Application request
        ↓
long delay
        ↓
retransmission
        ↓
response
```

is more relevant than:

```text id="x3n7p9"
Unrelated retransmission
        ↓
several minutes later
        ↓
application transaction
```

Correlation requires temporal and conversational relevance.

## Phase 14: Use I/O Statistics if Useful

If the capture contains a broader performance issue, inspect traffic over time.

```bash id="m6p2x8"
tshark -r capture.pcapng -q -z io,stat,1
```

Use this to identify:

```text id="v9c4n5"
traffic spikes
quiet periods
bursts
changes in packet rate
```

Then correlate those periods with the affected transaction.

Do not treat a traffic spike as the cause without further evidence.

## Phase 15: Use Expert Information as a Lead

If appropriate, inspect Wireshark's Expert Information.

Use it to identify possible:

```text id="q3x8m1"
warnings
errors
TCP analysis events
protocol anomalies
```

Then validate important events in the actual packet sequence.

The rule is:

```text id="n7m4p2"
Expert Information
→ lead
→ packet evidence
→ interpretation
```

not:

```text id="c5v9x8"
Expert Information
→ automatic root cause
```

## Phase 16: Investigate the First Meaningful Deviation

At this point, identify the first stage that does not behave as expected.

Possible outcomes:

```text id="m8x3p5"
DNS delay
```

or:

```text id="q4n7v2"
TCP connection delay
```

or:

```text id="x6m2c8"
TLS handshake delay
```

or:

```text id="p9v4n1"
Application response delay
```

or:

```text id="c3m8x5"
Transport behavior correlating with the delay
```

or:

```text id="n7p2v6"
No meaningful network-level deviation visible.
```

The last outcome is important.

A packet capture may show that the reported delay is not explained by the visible network behavior.

## Phase 17: Challenge Your Initial Interpretation

Before writing the conclusion, ask:

```text id="m5x8q2"
What else could explain this behavior?
```

For example:

```text id="v3n7c9"
Observed:
Large delay after application request.

Possible explanations:
server-side processing
network delay
packet loss
capture limitation
application-layer waiting
```

Investigate enough evidence to distinguish the explanations where the capture permits.

Do not claim a server-side processing problem merely because the response was delayed unless the packet evidence supports that interpretation.

## Phase 18: Validate the Key Finding

A strong finding should be independently validated.

For example, if you identify a retransmission:

```bash id="x8m4p2"
tshark -r capture.pcapng \
  -Y "tcp.stream == STREAM_ID && tcp.analysis.retransmission" \
  -T fields \
  -e frame.number \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e tcp.stream
```

Then inspect the same packets in Wireshark.

Confirm:

```text id="q6v9m3"
frame
timestamp
direction
stream
surrounding packets
```

## Phase 19: Record the Evidence

For each important finding, record:

```text id="n4x7c2"
Question:

Frame(s):

Timestamp(s):

Source:

Destination:

Protocol:

Port(s):

Stream:

Filter:

Observation:

Interpretation:

Limitation:
```

Example:

```text id="p8m3v5"
Question:
Where does the application transaction experience the delay?

Frame(s):
...

Timestamp(s):
...

Stream:
...

Observation:
A significant gap occurs between the application request
and the visible response.

Interpretation:
The delay occurs after the request was transmitted.

Limitation:
The capture does not expose the server's internal processing.
```

## Phase 20: Determine Whether the Primary Question Is Answered

Ask:

```text id="c7x2m9"
Can I identify where the visible delay occurs?

Can I support it with packet evidence?

Can I explain relevant transport behavior?

Can I explain what remains unknown?
```

If yes, stop the investigation.

If no, define the missing evidence and continue.

## Final Investigation Report

Use this structure:

```text id="m3p8x5"
# Guided Capstone Investigation Report

## Problem

## Scope

### Capture

### Time Period

### Client

### Server

### Protocols

## Primary Question

## Initial Orientation

## Investigation Path

### DNS

### TCP

### TLS

### Application

### Timing

## Key Evidence

## Timeline

## Findings

## Interpretation

## Conclusion

## Limitations

## Follow-Up Questions
```

You do not need to include a section if it is genuinely irrelevant.

## Example Evidence Table

| Stage        | Evidence         | Observation                  | Interpretation                                    |
| ------------ | ---------------- | ---------------------------- | ------------------------------------------------- |
| DNS          | Query/response   | Resolution completed quickly | DNS does not explain the visible delay            |
| TCP          | SYN/SYN-ACK/ACK  | Handshake completed          | Connection establishment succeeded                |
| TLS          | Handshake        | Negotiation completed        | TLS establishment is visible                      |
| Application  | Request/response | Large response delay         | Delay occurs after application request            |
| TCP analysis | Relevant events  | Correlated or absent         | Determines whether transport behavior contributes |

Fill this table with actual capture evidence.

Do not invent results.

## Required Deliverables

The guided capstone is complete only when you have:

```text id="v5n2x8"
[ ] Identified the relevant client
[ ] Identified the relevant server
[ ] Identified the relevant transaction
[ ] Established the DNS stage where applicable
[ ] Established the TCP stage
[ ] Established the TLS stage where applicable
[ ] Established the application stage where visible
[ ] Built a timeline
[ ] Investigated relevant TCP behavior
[ ] Validated important findings
[ ] Distinguished observations from interpretations
[ ] Documented limitations
[ ] Answered the primary question
[ ] Produced a reproducible investigation record
```

## Completion Standard

You have successfully completed the guided capstone when you can explain the investigation without saying:

```text id="q8m3x7"
"I just followed the instructions."
```

Instead, you should be able to explain:

```text id="c4p9n2"
I started with this question.

I identified these systems.

I narrowed the investigation to this transaction.

I examined these protocol stages.

I found this deviation.

I validated it using these packets.

This evidence supports this interpretation.

The capture cannot establish this additional point.

Therefore, the primary question is answered as follows.
```

## Final Principle

The guided capstone is still guided by the scenario.

Your investigation path should increasingly be your own.

The goal is to demonstrate that you can take:

```text id="m7x2c5"
A vague operational symptom
```

and transform it into:

```text id="p4n8v3"
A bounded packet-analysis investigation
with reproducible evidence and a defensible conclusion.
```
