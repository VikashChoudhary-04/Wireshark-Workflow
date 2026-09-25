# Professional Investigation Workflow

## Objective

Professional Wireshark analysis is not simply knowing where features are located.

It is the ability to take an unfamiliar network problem, establish a clear question, collect relevant evidence, analyze it systematically, and produce a conclusion that another analyst can reproduce.

This workflow combines the major capabilities developed throughout the repository:

```text
Question
  ↓
Scope
  ↓
Capture
  ↓
Initial Orientation
  ↓
Filtering
  ↓
Protocol Analysis
  ↓
Conversations
  ↓
Statistics
  ↓
Timeline
  ↓
Correlation
  ↓
Hypothesis Testing
  ↓
Evidence Validation
  ↓
Conclusion
  ↓
Documentation
```

The objective is to move from:

```text
"I can inspect packets."
```

to:

```text
"I can independently investigate network behavior."
```

## The Professional Mental Model

Use this model for unfamiliar traffic:

```text
Question
    ↓
What am I trying to determine?

Evidence
    ↓
What packets and metadata can answer it?

Action
    ↓
Which Wireshark feature should I use?

Observation
    ↓
What do I actually see?

Interpretation
    ↓
What does the observation support?

Decision
    ↓
What should I investigate next?

Next Question
    ↓
What remains unresolved?
```

This prevents random clicking through Wireshark.

## Start With the Problem

A professional investigation begins with a problem statement.

Examples:

```text
Users cannot access an application.

DNS resolution appears unreliable.

An application is slow.

A server connection repeatedly fails.

A host is communicating with unexpected systems.

A service appears unavailable.

An internal host is generating unusual traffic.

A connection succeeds but the application still fails.
```

Convert the problem into a packet-level question.

Example:

```text
Business problem:
Users report that the application is slow.

Packet question:
Where is the observed delay occurring?
```

Possible stages:

```text
DNS
↓
TCP
↓
TLS
↓
Application request
↓
Server response
```

## Define the Scope

Before opening a large capture, define:

```text
Source:
Destination:
Time window:
Protocol:
Application:
User/host:
Expected behavior:
Observed problem:
```

Example:

```text
Source:
192.0.2.25

Destination:
203.0.113.20

Time:
10:00–10:15

Protocol:
HTTPS

Problem:
Requests intermittently take several seconds.
```

A defined scope makes analysis faster and more reproducible.

## Establish What Is Known

Separate known facts from assumptions.

### Known

```text
The user reported a timeout.
The server IP is known.
The capture was taken from the client.
The capture covers 10:00–10:15.
```

### Unknown

```text
Whether DNS succeeded.
Whether TCP completed normally.
Whether TLS completed.
Whether the server responded slowly.
Whether packets were lost.
```

### Assumption

```text
The problem is probably caused by the server.
```

Do not allow assumptions to become conclusions without evidence.

## Capture Context

Before analyzing packets, understand where the capture came from.

Record:

```text
Capture source:
Interface:
Host:
Network segment:
VLAN:
SPAN/TAP:
Capture filter:
Start time:
End time:
Timezone:
Known packet loss:
```

Ask:

```text
Can this capture actually see the traffic I need?
```

## First Pass: Orient Yourself

Do not immediately apply a complex filter.

Start with the overall capture.

Inspect:

```text
Packet count
Time range
Protocol distribution
Endpoints
Conversations
```

Useful Wireshark views include:

```text
Statistics → Protocol Hierarchy
Statistics → Endpoints
Statistics → Conversations
```

The objective is to understand the shape of the capture.

## Protocol Hierarchy

Protocol Hierarchy answers:

```text
What types of traffic are present?
```

Look for:

```text
Ethernet
ARP
IPv4
IPv6
TCP
UDP
DNS
HTTP
TLS
SMB
SSH
ICMP
```

Unexpected protocols may provide useful investigation paths.

Do not assume an unfamiliar protocol is suspicious.

## Endpoints

Endpoints help answer:

```text
Which hosts are communicating?
```

Look for:

```text
Top talkers
Rare endpoints
Unexpected destinations
Internal/external relationships
```

Use endpoints to generate questions.

Do not use packet count alone to determine importance.

## Conversations

Conversations answer:

```text
Who communicated with whom?
How often?
How much traffic?
For how long?
```

Compare:

```text
Packets
Bytes
Duration
Direction
Ports
```

This can reveal dominant communication paths.

## Build an Investigation Map

Before deep filtering, construct a simple map.

Example:

```text
Client
192.0.2.25
   |
   +---- DNS ----> DNS Server
   |
   +---- TCP/443 ----> Application
   |
   +---- TCP/53 ----> Other Service
```

Then determine which relationship matters to the investigation.

## Select the Relevant Conversation

Once the relevant conversation is identified, narrow the investigation.

Use:

```text
Conversation filters
Stream identifiers
IP addresses
Ports
Protocols
```

The objective is to reduce noise without losing context.

## Use Display Filters Strategically

Build filters from questions.

Examples:

```text
dns
```

Question:

```text
What DNS traffic exists?
```

```text
ip.addr == 192.0.2.25
```

Question:

```text
What traffic involves this host?
```

```text
tcp.port == 443
```

Question:

```text
What traffic uses TCP port 443?
```

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Question:

```text
Which initial TCP SYN packets are visible?
```

Filters should answer questions, not replace reasoning.

## Narrow Progressively

Use progressive narrowing:

```text
Entire capture
    ↓
Relevant host
    ↓
Relevant destination
    ↓
Relevant protocol
    ↓
Relevant conversation
    ↓
Relevant stream
    ↓
Relevant packet
```

Do not jump directly to a highly restrictive filter if you do not yet understand the traffic.

## Preserve Context While Narrowing

A filter can hide important context.

For example:

```text
http
```

may hide:

```text
TCP handshake
DNS resolution
TLS negotiation
Connection termination
```

Therefore, move between:

```text
Broad context
```

and:

```text
Focused evidence
```

as needed.

## Follow the Complete Communication Path

For a typical web request:

```text
DNS
 ↓
TCP
 ↓
TLS
 ↓
HTTP/application
 ↓
TCP termination
```

Investigate each stage separately.

A failure at one stage can appear as an application problem at a higher layer.

## DNS Stage

Ask:

```text
Was the hostname queried?

Did a response arrive?

What address was returned?

How long did resolution take?

Was the response successful?

Were repeated queries generated?
```

## TCP Stage

Ask:

```text
Was SYN sent?

Was SYN/ACK received?

Was ACK sent?

Were retransmissions present?

Were resets present?

Was the connection terminated normally?
```

## TLS Stage

Ask:

```text
Did the handshake begin?

Did the server respond?

Was the handshake completed?

Was an alert visible?

What version was negotiated?

Was SNI visible?
```

## Application Stage

If the application protocol is visible:

```text
Was a request sent?

Did the server respond?

What was the status?

Was there a redirect?

Was the response delayed?

Did the connection remain open?
```

If the application is encrypted:

```text
Use TLS metadata,
timing,
transport behavior,
and correlation.
```

## Transport Stage

Always return to TCP or UDP when necessary.

For TCP, inspect:

```text
Handshake
Sequence numbers
ACKs
Retransmissions
Duplicate ACKs
Window size
Zero-window events
Resets
Termination
Timing
```

For UDP, inspect:

```text
Request
Response
ICMP feedback
Timing
Repeated traffic
Application protocol
```

## Timing Analysis

Timing often answers:

```text
Where did the delay occur?
```

Break the transaction into stages:

```text
DNS request
   ↓
DNS response

TCP SYN
   ↓
TCP SYN/ACK

TLS ClientHello
   ↓
TLS response

Application request
   ↓
Application response
```

Measure each interval.

## Example Timing Analysis

Suppose:

```text
DNS:
20 ms

TCP:
30 ms

TLS:
50 ms

Application response:
2,500 ms
```

The packet evidence points toward the application-response stage as the major contributor to the observed delay.

That does not automatically establish why the application responded slowly.

The next question becomes:

```text
What was happening during those 2.5 seconds?
```

## Retransmission Analysis

Retransmissions can indicate:

```text
Packet loss
Congestion
Receiver problems
Network path issues
Capture artifacts
```

Investigate:

```text
Which packets retransmitted?
How often?
Which direction?
How long after the original?
Were duplicate ACKs present?
```

Do not automatically classify every retransmission as a network failure.

## Window Analysis

A receiver advertising a small or zero TCP window can affect throughput.

Investigate:

```text
Window size
Window updates
Zero-window events
Duration
Direction
```

Ask:

```text
Is the receiver temporarily unable to accept more data?
```

Then determine whether the behavior is actually responsible for the observed problem.

## Connection Resets

A TCP reset may indicate:

```text
Application termination
Firewall behavior
Service rejection
Closed service
Protocol problem
Endpoint behavior
```

Inspect the surrounding sequence.

Example:

```text
SYN
SYN/ACK
ACK
Application traffic
RST
```

This means something different from:

```text
SYN
RST
```

Context matters.

## Conversation-Level Reasoning

Instead of inspecting packets independently, ask:

```text
What happened during the entire conversation?
```

A conversation can reveal:

```text
Start
Handshake
Application exchange
Idle periods
Retries
Termination
```

This often provides the correct level of abstraction for troubleshooting.

## Stream Reconstruction

Use stream-following features when they provide useful application context.

Examples:

```text
Follow TCP Stream
Follow UDP Stream
Follow HTTP Stream
```

Use stream reconstruction to answer questions such as:

```text
Was an HTTP request actually sent?

What did the server return?

Was the conversation complete?

Was the stream terminated unexpectedly?
```

For encrypted protocols, stream reconstruction may only show encrypted content or protocol metadata.

## Statistics as a Second Perspective

After packet-level analysis, return to statistics.

Useful views include:

```text
Protocol Hierarchy
Endpoints
Conversations
I/O Graphs
Service Response Time
DNS statistics
HTTP statistics
Expert Information
```

The purpose is to validate whether the focused finding fits the broader capture.

## I/O Graphs

I/O graphs help visualize traffic over time.

Use them to investigate:

```text
Bursts
Traffic spikes
Quiet periods
Periodic activity
Traffic drops
```

Then correlate graph events with packet-level timestamps.

A graph shows:

```text
When traffic changed.
```

Packet analysis determines:

```text
Why traffic changed.
```

## Expert Information

Expert Information can highlight areas worth investigating.

Examples include:

```text
Retransmissions
Duplicate ACKs
Malformed packets
Protocol warnings
Connection problems
```

Use these as leads.

Always inspect the underlying packets.

## Correlation Across Views

A strong workflow verifies important findings using multiple perspectives.

Example:

```text
Endpoint view
    ↓
Conversation view
    ↓
Packet list
    ↓
Follow stream
    ↓
Timing analysis
```

If the same conclusion appears consistently across these views, the evidence is stronger.

## Build an Evidence Chain

For an application failure:

```text
User request
  ↓
DNS query
  ↓
DNS response
  ↓
TCP handshake
  ↓
TLS handshake
  ↓
Application request
  ↓
Application response
  ↓
Connection termination
```

Mark where the expected sequence breaks.

Example:

```text
DNS ✓
TCP ✓
TLS ✓
Application request ✓
Application response ✗
```

Now the investigation has a precise next question.

## Troubleshooting by Failure Stage

Classify the failure stage:

```text
DNS failure
TCP failure
TLS failure
Application failure
Transport performance problem
Capture visibility problem
```

This is more useful than saying:

```text
"The network is slow."
```

## Distinguish Network Failure From Application Failure

Suppose:

```text
TCP handshake:
Normal

TLS handshake:
Normal

Application request:
Sent

Application response:
Delayed 5 seconds
```

The capture does not automatically establish a network transport problem.

The evidence indicates that the delay occurred after the request.

Possible explanations include:

```text
Server processing
Backend dependency
Application logic
Database operation
Proxy behavior
Network response delay
```

Additional evidence may be required.

## Distinguish Capture Problems From Network Problems

Suppose a packet appears to be missing.

Before concluding packet loss, ask:

```text
Was the capture taken at the correct interface?

Was SPAN configured correctly?

Could the packet have been dropped by the capture mechanism?

Was the capture filter restrictive?

Is the traffic encrypted or tunneled?

Was the relevant direction visible?
```

Capture quality is part of the investigation.

## Baseline Comparison

If a known-good capture exists, compare:

```text
DNS timing
TCP handshake
TLS timing
Application response
Packet retransmissions
Connection duration
Traffic volume
```

Baseline comparison can be more useful than inspecting an isolated capture.

## Known-Good vs Problem Capture

Create a table:

```text
| Stage | Known Good | Problem | Difference |
|------|------------|---------|------------|
| DNS | | | |
| TCP | | | |
| TLS | | | |
| Application | | | |
| Termination | | | |
```

This turns a vague problem into measurable differences.

## Incident Investigation

For a security-related investigation:

```text
Question
  ↓
Affected host
  ↓
Time window
  ↓
Destinations
  ↓
Protocols
  ↓
Connections
  ↓
DNS
  ↓
Encrypted traffic
  ↓
Internal communication
  ↓
Timeline
  ↓
Evidence validation
```

Avoid beginning with:

```text
"Find the attack."
```

Instead determine what communication actually occurred.

## Investigating Suspicious Communication

Record:

```text
Source
Destination
Time
Port
Protocol
Frequency
Duration
DNS relationship
TLS metadata
Follow-on communication
Internal/external direction
```

Then compare against expected host behavior.

## Investigation of Scanning-Like Traffic

Use:

```text
Source
↓
Destination count
↓
Port count
↓
Timing
↓
Response pattern
↓
Successful connections
↓
Application follow-up
↓
Expected role
```

Avoid declaring intent from one packet pattern.

## Lateral Communication

Investigate:

```text
Internal source
↓
Internal destinations
↓
Administrative protocols
↓
Authentication-related traffic
↓
Successful sessions
↓
Follow-on communication
```

Possible protocols include:

```text
SMB
RDP
SSH
WinRM
LDAP
Kerberos
```

The presence of these protocols is not itself evidence of malicious behavior.

## Evidence Hierarchy

A practical hierarchy is:

```text
Packet-level evidence
      ↓
Conversation evidence
      ↓
Timeline correlation
      ↓
Statistics
      ↓
Context
      ↓
Interpretation
```

Context can be extremely valuable, but it should not override contradictory packet evidence without explanation.

## Document Uncertainty

Use precise language:

```text
Observed:
The client sent a TCP SYN.

Supported:
The server responded with SYN/ACK.

Consistent with:
A TCP connection was established.

Not established:
Which user initiated the action.

Unknown:
Why the application subsequently failed.
```

This makes the investigation defensible.

## Professional Decision Loop

At every stage ask:

```text
What do I know?
What do I not know?
What evidence supports the current conclusion?
What alternative explanation exists?
What should I inspect next?
```

This creates a repeatable investigation loop.

## Practical Exercise 1 — Full Troubleshooting Workflow

Choose a capture containing an application problem.

Perform:

```text
1. Define the problem.
2. Identify the relevant host.
3. Establish the time window.
4. Inspect endpoints.
5. Inspect conversations.
6. Identify the application protocol.
7. Trace DNS.
8. Trace TCP/UDP.
9. Trace TLS where applicable.
10. Trace application traffic.
11. Measure timing.
12. Inspect retransmissions.
13. Check statistics.
14. Build a timeline.
15. Identify the failure stage.
```

Write the final conclusion using evidence references.

## Practical Exercise 2 — Full Security Workflow

Choose a capture containing unusual communication.

Perform:

```text
1. Identify the source.
2. Identify destinations.
3. Identify ports.
4. Identify protocols.
5. Analyze timing.
6. Analyze repeated behavior.
7. Follow successful connections.
8. Correlate DNS.
9. Inspect encrypted metadata.
10. Identify internal communication.
11. Consider legitimate explanations.
12. Document unknowns.
13. Build an evidence chain.
```

Do not force a malicious classification.

## Practical Exercise 3 — Reproduce an Investigation

Complete an investigation.

Then close Wireshark.

Reopen the original capture and attempt to reproduce your conclusion using only your notes.

Record:

```text
What was easy to reproduce?
What information was missing?
Which filters should have been recorded?
Which packet numbers should have been saved?
```

This tests investigation quality.

## Practical Exercise 4 — Analyst Handoff

Write a short investigation handoff for another analyst.

Include:

```text
Question
Scope
Key evidence
Timeline
Finding
Limitations
Open questions
Recommended next evidence
```

The receiving analyst should understand the investigation without repeating your entire analysis.

## Practical Exercise 5 — Independent Workflow

Choose an unfamiliar capture.

Do not use a predefined filter.

Start with:

```text
Question:
```

Then independently decide:

```text
What should I inspect first?

Which Wireshark feature should I use?

What evidence should I collect?

What should I investigate next?
```

Document every major decision.

## Investigation Report Structure

A practical report can use:

```text
1. Investigation Question
2. Scope
3. Capture Context
4. Methodology
5. Timeline
6. Key Observations
7. Evidence
8. Analysis
9. Findings
10. Limitations
11. Open Questions
12. Additional Evidence Required
13. Conclusion
```

## Evidence Reference Format

Use a consistent format:

```text
Packet:
143

Timestamp:
10:15:32.125

Source:
192.0.2.25

Destination:
203.0.113.20

Protocol:
TCP

Observation:
TCP SYN sent to destination port 443.
```

For multiple packets:

```text
Packets:
143–149
```

## Analyst Notes

Good notes answer:

```text
Why did I inspect this?
What did I observe?
What did it change?
What should I inspect next?
```

Example:

```text
Observation:
DNS response returned 203.0.113.20.

Decision:
Inspect TCP connections to 203.0.113.20 immediately afterward.

Result:
TCP connection established 25 ms later.

Next:
Inspect TLS handshake.
```

This creates a decision trail.

## Common Mistakes

### Mistake 1: Random Exploration

Clicking through every Wireshark feature without a question wastes time.

### Mistake 2: Over-Filtering

A filter can hide the context needed to understand the event.

### Mistake 3: Ignoring the Timeline

Packet order and timing are often critical.

### Mistake 4: Looking Only at Application Protocols

Lower-layer evidence can explain application behavior.

### Mistake 5: Looking Only at TCP

DNS, TLS, UDP, ICMP, and application protocols may be equally important.

### Mistake 6: Treating Statistics as Conclusions

Statistics identify patterns; packet analysis explains them.

### Mistake 7: Ignoring Capture Limitations

Missing traffic may be a visibility problem.

### Mistake 8: Overstating Findings

Use only the level of certainty supported by the evidence.

### Mistake 9: Failing to Record Filters

Another analyst may not be able to reproduce the investigation.

### Mistake 10: Failing to Record Packet Numbers

Important evidence becomes difficult to locate.

### Mistake 11: Ignoring Alternative Explanations

A strong investigation considers plausible alternatives.

### Mistake 12: Solving the Wrong Problem

Always return to the original investigation question.

## Professional Workflow Checklist

Before considering an investigation complete:

* [ ] Problem clearly defined
* [ ] Investigation question written
* [ ] Scope established
* [ ] Capture context understood
* [ ] Capture limitations documented
* [ ] Initial protocol overview completed
* [ ] Relevant endpoints identified
* [ ] Relevant conversations identified
* [ ] Display filters constructed from questions
* [ ] Relevant streams examined
* [ ] DNS investigated where relevant
* [ ] TCP/UDP behavior investigated
* [ ] TLS/encryption investigated where relevant
* [ ] Application behavior investigated
* [ ] Timing analyzed
* [ ] Retransmissions and transport issues considered
* [ ] Statistics consulted
* [ ] Timeline constructed
* [ ] Evidence chain established
* [ ] Important findings verified at packet level
* [ ] Alternative explanations considered
* [ ] Capture limitations documented
* [ ] Unknowns documented
* [ ] Packet references recorded
* [ ] Filters recorded where useful
* [ ] Conclusion tied directly to evidence
* [ ] Additional evidence identified

## Completion Criteria

You should be able to receive an unfamiliar packet capture and independently decide:

```text
What is the problem?

What is the investigation question?

What traffic is relevant?

Which Wireshark feature should I use first?

How should I narrow the capture?

Which protocol layers should I inspect?

How do I build the timeline?

How do I determine where a failure occurred?

How do I distinguish transport behavior from application behavior?

How do I account for encryption?

How do I account for capture limitations?

How do I verify an important finding?

How do I document evidence?

How do I communicate uncertainty?

What additional evidence is needed?
```

The professional workflow is:

```text
Problem
  ↓
Question
  ↓
Scope
  ↓
Orient
  ↓
Filter
  ↓
Inspect
  ↓
Correlate
  ↓
Measure
  ↓
Validate
  ↓
Interpret
  ↓
Document
  ↓
Conclude
```

The goal is not to memorize every Wireshark feature.

The goal is to develop the ability to **choose the right analysis action for the question in front of you**.
