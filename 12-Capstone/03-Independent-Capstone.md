# Independent Capstone

## Purpose

This is the main independence test of the Wireshark workflow.

Unlike the guided capstone, this investigation does not provide a protocol-by-protocol procedure.

You are given a realistic operational problem and an authorized packet capture.

You must decide:

```text id="q6m2x8"
What to inspect
→ What question to ask
→ Which Wireshark capability to use
→ What evidence matters
→ What to investigate next
→ When the investigation is complete
```

The objective is to demonstrate independent packet-analysis judgment.

Use only an authorized capture, laboratory traffic, or a supplied training PCAP.

## Scenario

An internal application team reports:

```text id="n8v4c1"
"Users are intermittently unable to access the application.
Some requests appear to work, while others are slow or fail."
```

A packet capture was collected during the affected period.

You receive:

```text id="p3x7m5"
application-incident.pcapng
```

No protocol is identified for you.

No affected host is identified for you.

No expected root cause is provided.

No filter sequence is provided.

Your task is to determine what the capture can establish.

## Primary Objective

Determine:

```text id="m5q8x2"
What communication behavior is associated with the reported
application-access problem, and what evidence supports the finding?
```

You must decide how to decompose that question.

Possible areas may include:

```text id="v7c3n9"
DNS
TCP
TLS
HTTP
application protocols
timing
retransmissions
resets
connection failures
endpoint behavior
```

Do not assume that all of these are relevant.

## Independence Rules

During this capstone:

```text id="x2m7p4"
Do not follow a predefined command sequence.

Do not assume the reported cause is correct.

Do not inspect every packet without a purpose.

Do not classify unusual traffic without supporting evidence.

Do not treat a missing packet as proof that an event never occurred.

Do not continue investigating unrelated traffic after the primary
question has been sufficiently answered.
```

You may use the Wireshark workflow documentation in this repository as a reference.

The objective is to use the knowledge, not to copy a fixed solution.

## Phase 1: Preserve and Identify the Evidence

Before investigating, establish:

```text id="c8n4x6"
Capture filename
File format
Packet count
Capture duration
First packet
Last packet
Capture point if known
Wireshark/TShark version
```

Useful commands include:

```bash id="q5m8x3"
capinfos application-incident.pcapng
```

and:

```bash id="v2p7n4"
tshark --version
```

If evidence integrity matters, calculate and record a SHA-256 hash.

Linux:

```bash id="m6x3q9"
sha256sum application-incident.pcapng
```

PowerShell:

```powershell id="c4v8m2"
Get-FileHash .\application-incident.pcapng -Algorithm SHA256
```

## Phase 2: Orient Yourself

Determine what exists before deciding what matters.

Useful questions include:

```text id="p8n3x5"
What protocols are present?

Which endpoints communicate?

Which conversations dominate?

When is the traffic most active?

Are there obvious groups of related communication?
```

You may use:

```text id="q7m2v8"
Protocol Hierarchy
Endpoints
Conversations
I/O statistics
```

or their TShark equivalents.

Do not assume that the largest conversation is necessarily the affected application.

## Phase 3: Identify the Relevant Traffic

Use the capture evidence to determine which traffic is likely related to the reported problem.

Possible indicators include:

```text id="x4n7c2"
application ports
DNS names
server addresses
repeated connection attempts
timing
failed transactions
```

The identification should be evidence-based.

Record:

```text id="m8p3v6"
Candidate client(s):

Candidate server(s):

Candidate protocol(s):

Relevant time period:

Evidence supporting the identification:
```

If multiple candidates exist, compare them rather than selecting one arbitrarily.

## Phase 4: Define Your First Question

Do not begin deep analysis until you can state a useful first question.

Examples:

```text id="v5x2m9"
Which hosts are involved?

Did the client resolve the application server?

Did the TCP connection establish?

Where do successful and failed transactions differ?

Where does the delay occur?
```

Choose the question that best reduces uncertainty.

Record it.

```text id="q3m8p1"
First Question:
```

## Phase 5: Build the Investigation Path

Your investigation path is yours to determine.

A possible structure is:

```text id="x7c4m2"
Orientation
→ relevant endpoints
→ relevant conversation
→ transaction
→ protocol sequence
→ timing
→ anomalies
→ correlation
→ conclusion
```

You may use a different path if the evidence demands it.

The quality of the investigation depends on whether each step answers a meaningful question.

## Phase 6: Compare Successful and Failed Behavior

Because the report says that some requests work while others fail or become slow, comparison is particularly important.

Identify:

```text id="m4p8x3"
At least one successful transaction
```

and, where the capture allows:

```text id="v9c2n6"
At least one failed or delayed transaction
```

Compare:

```text id="q6m3x8"
client
server
protocol
ports
stream
timestamps
handshake
application behavior
TCP analysis events
```

The objective is to determine what differs.

## Phase 7: Investigate Timing

Construct a timeline for representative transactions.

For example:

```text id="c3x7m5"
Request start
↓
Connection establishment
↓
Protocol negotiation
↓
Application request
↓
Application response
```

Compare successful and problematic transactions.

Ask:

```text id="p8m2v4"
Where does the delay appear?

Does the delay occur before connection establishment?

During TLS?

After the application request?

During response delivery?

Is the delay associated with TCP retransmissions?

Does the same delay occur in successful transactions?
```

The capture should determine the answer.

## Phase 8: Investigate Transport Behavior

If TCP is involved, examine the affected conversations for relevant behavior.

Possible areas:

```text id="x5n8m3"
SYN/SYN-ACK/ACK
retransmissions
duplicate ACKs
out-of-order packets
zero windows
resets
connection termination
timing
```

Do not automatically treat every TCP analysis event as the root cause.

Instead ask:

```text id="m7q2v9"
Is the event in the affected transaction?

Does its timing correlate with the reported problem?

Does it differ from successful transactions?
```

## Phase 9: Investigate the Application Layer

If application traffic is visible, determine what the client actually requested and what the server returned.

For HTTP, relevant evidence may include:

```text id="v4p8x2"
method
host
URI
status code
request time
response time
stream
```

For other protocols, use the fields appropriate to the observed protocol.

If the application is encrypted:

```text id="n6c3m7"
Use visible TLS and TCP metadata.
```

Do not claim visibility into encrypted payload that the capture does not provide.

## Phase 10: Correlate Multiple Layers

Do not analyze every layer independently.

Connect the evidence.

For example:

```text id="x8m4p2"
DNS
  ↓
destination IP
  ↓
TCP stream
  ↓
TLS handshake
  ↓
application request
  ↓
application response
```

Then compare that sequence between:

```text id="q3v7n5"
successful transaction
```

and:

```text id="m8c2x6"
problematic transaction
```

The differences may reveal where the behavior changes.

## Phase 11: Investigate Alternative Explanations

Once you identify a likely explanation, challenge it.

Suppose you observe:

```text id="v5n2m8"
Retransmissions occur during a slow transaction.
```

Do not immediately conclude:

```text id="q7c3x1"
"Retransmissions caused the slowdown."
```

Ask:

```text id="m4p8v2"
Were retransmissions present in successful transactions?

Did the retransmissions occur during the delay?

Were they frequent enough to affect the transaction?

Did the application response remain delayed after transport recovery?

Could another event explain the delay?
```

This is essential independent-analysis reasoning.

## Phase 12: Determine the First Meaningful Difference

The most useful comparison may be:

```text id="x2m7c9"
Successful transaction
        vs.
Problematic transaction
```

Find the earliest meaningful difference.

Example:

```text id="p8v4n3"
Successful:
DNS → TCP → TLS → request → response

Problematic:
DNS → TCP → TLS → request → long delay → response
```

This suggests that the investigation should focus after the request.

Another example:

```text id="c5x8m2"
Successful:
DNS → TCP handshake

Problematic:
DNS → repeated SYN → no visible SYN-ACK
```

The investigation then focuses on connection establishment.

## Phase 13: Validate the Finding

Once you identify an important difference, validate it in both:

```text id="m3q7v8"
Wireshark GUI
```

and, where useful:

```text id="x6p2n4"
TShark
```

For TShark, preserve the exact command.

For example:

```bash id="v8m3q5"
tshark -r application-incident.pcapng \
  -Y "RELEVANT_FILTER" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e tcp.stream
```

Replace `RELEVANT_FILTER` with the filter you actually used.

Do not invent a filter after completing the analysis.

Record the real command.

## Phase 14: Build an Evidence Chain

For every major finding, establish:

```text id="q4x8m1"
Question
↓
Relevant traffic
↓
Frame(s)
↓
Field(s)
↓
Observation
↓
Interpretation
↓
Conclusion
```

Example:

```text id="n7p3v5"
Question:
Why is this transaction slow?

Traffic:
Client → application server

Frames:
...

Observation:
Application request is followed by a long interval before response.

Interpretation:
The visible delay occurs after the request.

Conclusion:
The capture localizes the delay to the post-request portion
of the transaction, but does not establish the server's internal cause.
```

## Phase 15: Determine What the Capture Cannot Answer

Explicitly identify limitations.

Potential limitations include:

```text id="m2c7x9"
capture point
missing traffic
packet loss during capture
encryption
truncated packets
capture start/end boundaries
asymmetric visibility
unknown application internals
unsupported or incomplete protocol dissection
```

A strong independent analysis should include these without being prompted.

## Phase 16: Decide Whether the Investigation Is Complete

Use the completion test:

```text id="x5p8m3"
Can I answer the primary question?

Can I show the evidence?

Can I explain important uncertainty?

Would additional unrelated packets materially change the conclusion?
```

If:

```text id="v7n2c4"
Yes
Yes
Yes
No
```

the investigation can be closed.

## Required Investigation Report

Produce a report using this structure:

```text id="m4x8q2"
# Independent Capstone Investigation Report

## Problem Statement

## Scope

### Capture

### Time Period

### Relevant Hosts

### Relevant Protocols

## Primary Question

## Initial Orientation

## Investigation Path

## Key Evidence

## Timeline

## Successful vs Problematic Behavior

## Findings

## Interpretation

## Conclusion

## Limitations

## Remaining Uncertainty

## Follow-Up Questions
```

The exact structure may be adapted if the capture requires a different organization.

## Evidence Table

Include a table similar to:

| Finding   | Evidence      | Observation                | Interpretation | Limitation           |
| --------- | ------------- | -------------------------- | -------------- | -------------------- |
| Finding 1 | Frames/fields | What was directly observed | What it means  | What remains unknown |
| Finding 2 | Frames/fields | What was directly observed | What it means  | What remains unknown |

Every important conclusion should be traceable to evidence.

## Required Reproducibility Record

Include:

```text id="p8m3x7"
Capture:
application-incident.pcapng

Wireshark/TShark version:

Important display filters:

Important TShark commands:

Important frame numbers:

Important TCP streams:

Derived output files:
```

Only include commands you actually used.

## Independent Analysis Rules

### Rule 1: Do Not Chase Every Anomaly

Large captures contain unusual packets.

Ask:

```text id="q5x2m8"
Is this relevant to the primary problem?
```

If not, record it only if it materially affects the investigation.

### Rule 2: Do Not Treat Volume as Importance

A conversation with the largest packet count is not automatically the cause of the reported problem.

Relevance matters more than size.

### Rule 3: Do Not Treat Rarity as Maliciousness

An unusual endpoint or uncommon protocol may require investigation.

It is not automatically malicious.

### Rule 4: Do Not Treat Missing Packets as Proof

Always consider:

```text id="m8c4x2"
capture visibility
capture loss
capture filters
asymmetry
```

### Rule 5: Do Not Overstate Application Conclusions

A packet capture can often show:

```text id="x7p3m9"
when a request was sent
when a response was observed
what protocol metadata was visible
```

It may not show:

```text id="v4n8q2"
what the server internally did between those events
```

unless additional evidence exists.

## Independent Performance Reasoning

If the investigation becomes a performance investigation, separate:

```text id="q3m7x5"
Network timing
```

from:

```text id="n8c2v4"
Application/server processing time
```

For example:

```text id="m6p4x9"
Request leaves client
        ↓
No packets visible for 3 seconds
        ↓
Server response arrives
```

The capture establishes a three-second visible gap.

It does not automatically establish what happened inside the server during that period.

Use precise language.

## Independent Security Reasoning

If the investigation reveals unexpected communication, record:

```text id="x5m2c8"
endpoint
timestamp
protocol
port
frequency
direction
application metadata
correlated activity
```

Then state:

```text id="p7v3n9"
what is observed
```

and separately:

```text id="m4x8q2"
what requires additional investigation
```

Avoid unsupported attribution.

## Independent Troubleshooting Reasoning

If the capture reveals a failure, identify:

```text id="c8p2m5"
Expected sequence
        ↓
Observed sequence
        ↓
First meaningful deviation
        ↓
Supporting evidence
        ↓
Possible explanations
        ↓
Conclusion
```

This is more useful than simply saying:

```text id="v9m3x7"
"The connection failed."
```

## Self-Review Before Submission

Before considering the capstone complete, ask:

```text id="q2x8m4"
[ ] Did I define the primary question?
[ ] Did I identify the relevant traffic?
[ ] Did I justify why that traffic was relevant?
[ ] Did I compare successful and problematic behavior where possible?
[ ] Did I build a meaningful timeline?
[ ] Did I investigate the relevant protocol layers?
[ ] Did I validate the key finding?
[ ] Did I test alternative explanations?
[ ] Did I distinguish observation from interpretation?
[ ] Did I account for capture visibility?
[ ] Did I avoid unsupported claims?
[ ] Did I document the exact filters/commands used?
[ ] Did I preserve important frame and stream references?
[ ] Did I identify what remains unknown?
[ ] Did I stop once the primary question was sufficiently answered?
```

## Independence Standard

You should be able to complete this capstone without being told:

```text id="m7x3p8"
which filter to use
which packet to inspect
which protocol to investigate first
which statistics to run
which stream to follow
which explanation to believe
```

You may consult the repository documentation for syntax or capability reference.

The investigation decisions should be yours.

## What Counts as Success

Success does not require finding a dramatic root cause.

A successful investigation could conclude:

```text id="x4p8m2"
The capture shows that DNS, TCP, and TLS establishment completed normally.
The affected transaction contains a significant delay after the application
request. The capture does not expose the server's internal processing,
so the underlying server-side cause cannot be established from this
capture alone.
```

That is a valid professional conclusion if supported by the evidence.

Another valid conclusion could be:

```text id="n6c3v9"
The affected connection attempts contain repeated SYN retransmissions
without a visible SYN-ACK at the capture point. The capture therefore
shows unsuccessful connection establishment from the client's visible
perspective, but it does not establish whether the server received
the original SYN packets.
```

The quality lies in the evidence and precision of the conclusion.

## Final Deliverable

Your completed independent capstone should contain:

```text id="p5m8x3"
A clear problem statement
A defined primary question
A bounded scope
An investigation path
Relevant evidence
A timeline where useful
A comparison of important transactions where possible
Evidence-backed findings
A careful interpretation
A defensible conclusion
Documented limitations
Reproducibility information
Follow-up questions where appropriate
```

## Final Principle

The independent capstone is successful when you can take:

```text id="c7m2x9"
An unfamiliar operational problem
+
An unfamiliar packet capture
```

and independently transform them into:

```text id="v4n8p3"
A structured investigation
+
reproducible evidence
+
careful reasoning
+
a defensible conclusion
```

The goal is not to prove that you know every Wireshark feature.

The goal is to demonstrate that you know **how to investigate when nobody tells you what to click next.**
