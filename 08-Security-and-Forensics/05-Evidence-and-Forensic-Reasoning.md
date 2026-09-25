# Evidence and Forensic Reasoning

## Objective

Wireshark can expose a large amount of packet-level evidence.

The difficult part is not always finding packets.

The difficult part is determining:

```text
What was actually observed?
What does the evidence support?
What remains uncertain?
What alternative explanations exist?
What additional evidence is required?
```

This file establishes a disciplined forensic reasoning workflow for packet analysis.

The central model is:

```text
Question
  ↓
Scope
  ↓
Evidence
  ↓
Observation
  ↓
Correlation
  ↓
Interpretation
  ↓
Alternative Explanations
  ↓
Confidence
  ↓
Conclusion
  ↓
Additional Evidence
```

The goal is to produce conclusions that are reproducible, defensible, and proportional to the available evidence.

## Evidence vs Interpretation

These concepts must remain separate.

### Evidence

Evidence is what the capture directly provides.

Examples:

```text
Packet 143:
192.0.2.10 → 203.0.113.20
TCP destination port: 443
Timestamp: 10:15:32.125
```

### Observation

An observation summarizes visible evidence.

Example:

```text
192.0.2.10 established a TCP connection to
203.0.113.20 on port 443 at 10:15:32.
```

### Interpretation

An interpretation explains what the observation may represent.

Example:

```text
The connection is consistent with HTTPS or another
application using TLS on TCP port 443.
```

### Conclusion

A conclusion combines evidence and context.

Example:

```text
The capture supports that 192.0.2.10 communicated with
203.0.113.20 using an encrypted TCP session.
```

Each layer should be traceable to the layer beneath it.

## Why Evidence Discipline Matters

Packet analysis can easily produce overconfident conclusions.

For example:

```text
Observed:
A host established an SSH connection.

Unsupported conclusion:
The host executed a malicious command.
```

The packet capture may not contain the evidence necessary to establish the second statement.

A professional analyst should be comfortable saying:

```text
The capture does not establish this.
```

That is a valid analytical result.

## Start With a Question

Every investigation should begin with a question.

Examples:

```text
Did the host communicate with the server?

Was DNS resolution successful?

Did the TCP connection complete?

Was the application response delayed?

Did a host contact multiple internal systems?

Was encrypted communication established?

Did the observed traffic match the expected workflow?
```

Avoid beginning with a predetermined conclusion.

## Define the Scope

Record:

```text
Capture:
Relevant hosts:
Relevant networks:
Time window:
Relevant protocols:
Known context:
Known limitations:
```

Example:

```text
Capture:
incident-2026-09-25.pcapng

Host:
192.0.2.25

Time window:
10:00–10:15

Question:
Did the host establish communication with the suspected server?
```

This prevents the investigation from expanding without reason.

## Preserve the Original Evidence

Whenever possible, keep the original capture unchanged.

Work from:

```text
Original PCAP
   ↓
Working copy
   ↓
Analysis
```

Do not overwrite the source capture with modified or filtered output.

Record:

```text
Filename
File size
Capture date/time
Source
Hash where appropriate
```

The objective is reproducibility.

## Capture Integrity

When evidence handling requires stronger integrity controls, calculate a cryptographic hash.

Examples include:

```text
SHA-256
SHA-512
```

A hash can help establish that the file being analyzed matches the recorded source file.

Example documentation:

```text
File:
incident-2026-09-25.pcapng

SHA-256:
<recorded hash>
```

Do not confuse a file hash with proof that the capture itself is complete or accurate.

A hash establishes file integrity relative to the recorded value.

## Record the Capture Context

A packet capture should be interpreted in context.

Record:

```text
Capture point:
Interface:
Host/device:
Network segment:
VLAN:
SPAN/TAP/source:
Capture start:
Capture end:
Timezone:
Timestamp configuration:
```

These details can explain missing or unexpected traffic.

## Capture Point Limitations

A capture may not contain every packet involved in an event.

Possible reasons include:

```text
Wrong interface
Switch visibility
VLAN boundaries
SPAN limitations
Packet loss
Capture filters
Remote capture limitations
Wireless capture limitations
VPN encryption
NAT
Proxying
Load balancing
Host-local traffic
```

Therefore:

```text
Not observed
```

does not always mean:

```text
Did not happen
```

## Evidence Scope

For every conclusion, ask:

```text
What exact packets support this statement?
```

For example:

```text
Claim:
The host connected to the web server.

Evidence:
TCP handshake followed by TLS handshake between
192.0.2.10 and 203.0.113.20:443.
```

This is stronger than:

```text
The host probably used the website.
```

The second statement requires additional application evidence.

## Packet Numbers as Evidence References

Use packet numbers to make findings reproducible.

Example:

```text
Packet 152:
TCP SYN

Packet 153:
TCP SYN/ACK

Packet 154:
TCP ACK

Packet 155:
TLS ClientHello
```

A reviewer can return to the same packets and verify the reasoning.

## Timestamps as Evidence

Use exact timestamps where timing matters.

Example:

```text
10:12:04.100
DNS query

10:12:04.125
DNS response

10:12:04.150
TCP SYN

10:12:04.180
TCP SYN/ACK
```

This can establish a sequence that would be lost in a general summary.

## Build an Event Timeline

A timeline converts packet evidence into a chronological narrative.

Example:

```text
10:00:01
DNS query

10:00:01.020
DNS response

10:00:01.050
TCP connection begins

10:00:01.090
TLS handshake begins

10:00:01.300
Encrypted application traffic

10:00:03.200
Connection terminates
```

The timeline should contain evidence-backed events.

## Correlation

One packet rarely answers an entire investigation.

Correlate multiple evidence sources inside the capture.

Useful relationships include:

```text
DNS
 ↓
TCP
 ↓
TLS
 ↓
HTTP

ARP
 ↓
IP
 ↓
TCP

TCP handshake
 ↓
Application protocol
 ↓
Connection termination
```

Correlation produces stronger evidence than isolated observations.

## Five Useful Correlations

### Endpoint Correlation

```text
Source IP
Destination IP
Source port
Destination port
```

### Protocol Correlation

```text
Ethernet
↓
IP
↓
TCP/UDP
↓
Application protocol
```

### Temporal Correlation

```text
Event A
↓
milliseconds
↓
Event B
↓
seconds
↓
Event C
```

### Behavioral Correlation

```text
Connection
↓
Authentication
↓
Application activity
```

### Infrastructure Correlation

```text
Client
↓
NAT
↓
Proxy
↓
Load balancer
↓
Server
```

## Evidence Chains

Build evidence chains rather than isolated claims.

Example:

```text
DNS query
  ↓
DNS response
  ↓
Returned IP
  ↓
TCP connection to returned IP
  ↓
TLS ClientHello
  ↓
SNI
  ↓
Certificate
  ↓
Encrypted application data
```

This chain supports a much stronger explanation of the observed communication.

## Hypothesis Building

When the evidence is incomplete, create hypotheses.

Example:

```text
Observed:
Host communicates with 203.0.113.20 every 60 seconds.
```

Possible hypotheses:

```text
H1:
Application polling

H2:
Monitoring

H3:
Background synchronization

H4:
Automated service communication
```

Do not immediately select one.

Collect evidence that distinguishes them.

## Hypothesis Testing

For each hypothesis, ask:

```text
What should I expect to see if this is true?
```

Example:

```text
Hypothesis:
Monitoring system

Expected:
Regular intervals
Multiple known infrastructure destinations
Consistent protocol
Stable destination set
```

Then compare the capture.

The purpose is not to prove a favorite explanation.

The purpose is to determine which explanations remain supported.

## Alternative Explanations

Every significant interpretation should have at least one plausible alternative considered.

Example:

```text
Observation:
One host contacted 100 internal systems.

Interpretation:
Possible service discovery.

Alternative:
Vulnerability scanning.

Alternative:
Inventory collection.

Alternative:
Configuration management.
```

Then identify evidence that separates them.

## Confidence

Avoid artificial numerical scores unless the investigation specifically requires them.

Instead use descriptive language such as:

```text
Directly observed
Strongly supported
Consistent with
Possible
Uncertain
Not established
Contradicted by available evidence
```

These terms should be tied to actual evidence.

## Directly Observed

Use when the packet capture explicitly shows the event.

Example:

```text
The capture contains a DNS query for
api.example.test.
```

## Strongly Supported

Use when multiple independent observations agree.

Example:

```text
The DNS response returned 203.0.113.20,
followed immediately by a TLS connection to that address
with matching SNI.
```

## Consistent With

Use when the evidence supports an explanation but does not uniquely identify it.

Example:

```text
The repeated short-lived connections are consistent
with periodic application polling.
```

## Possible

Use when the evidence allows an explanation but is weak or incomplete.

Example:

```text
The traffic could represent automated discovery.
```

## Not Established

Use when the capture cannot prove the claim.

Example:

```text
The packet capture does not establish which user
initiated the application action.
```

## Chain of Custody Considerations

In formal forensic environments, evidence handling may require documented chain-of-custody procedures.

Relevant information can include:

```text
Evidence identifier
Source
Collector
Collection time
Transfer history
Storage location
Hash
Analyst
Analysis dates
```

The exact requirements depend on the organization's procedures and legal context.

For this repository, the objective is to understand the principle:

```text
Evidence should remain identifiable,
traceable, and reproducible.
```

## Sensitive Information

Packet captures may contain sensitive information.

Examples:

```text
Credentials
Session tokens
Cookies
Personal data
Internal hostnames
Email content
API data
File contents
Authentication metadata
```

Treat captures according to the applicable authorization and data-handling requirements.

Do not publish sensitive PCAPs or extracted secrets in public repositories.

## Redaction

When sharing evidence, consider whether sensitive information should be removed or replaced.

Possible approaches include:

```text
IP anonymization
Hostname replacement
Payload removal
Selective packet extraction
Sanitized screenshots
Metadata minimization
```

Maintain enough context for the analysis to remain understandable.

## Screenshots as Evidence

Screenshots can communicate findings, but they are weaker than preserving the original capture.

A useful screenshot should show:

```text
Packet number
Timestamp
Relevant protocol fields
Filter
Important values
```

Example:

```text
Display filter:
dns.qry.name == "example.test"
```

Then capture the relevant packet details.

Do not rely on screenshots alone when the original PCAP is available.

## Display Filters as Analytical Tools

Filters are not evidence themselves.

For example:

```text
dns
```

selects DNS packets.

The evidence is the actual packets returned by that filter.

Likewise:

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

is a method for locating TCP SYN packets.

The filter should be recorded when useful so another analyst can reproduce the observation.

## Record the Exact Filter

A professional investigation record can include:

```text
Filter:
dns && ip.addr == 192.0.2.25
```

Then:

```text
Relevant packets:
143, 151, 152, 153
```

This makes the analysis repeatable.

## Avoid Filter-Driven Reasoning

Do not begin with:

```text
"What filter can prove my theory?"
```

Instead:

```text
"What evidence do I need to answer my question?"
```

Then construct the filter.

This prevents confirmation bias.

## Negative Evidence

Negative evidence requires careful wording.

Example:

```text
No DNS response was visible in the capture.
```

This is valid.

But:

```text
DNS resolution failed.
```

may be unsupported if the capture point could have missed the response.

Always distinguish:

```text
No packet observed
```

from:

```text
Event did not occur
```

## Missing Evidence

Document important missing evidence.

Examples:

```text
No endpoint logs
No DNS response
No decrypted application data
No traffic from the suspected network segment
No packet after the proxy
No visibility inside VPN tunnel
```

Missing evidence can explain why a conclusion remains uncertain.

## Capture Loss

If the capture indicates dropped packets or incomplete collection, record it.

Potential effects include:

```text
Missing handshake packets
Incorrect timing
Incomplete TCP streams
Missing application data
Misleading retransmission patterns
```

Capture quality is part of evidence quality.

## Malformed Packets

A malformed or unusual packet should not automatically be treated as an attack.

Possible causes include:

```text
Corruption
Capture problems
Unsupported protocol features
Incorrect dissection
Truncated packets
Implementation bugs
Intentional malformed traffic
```

Investigate the surrounding traffic.

## Expert Information

Wireshark's Expert Information can highlight:

```text
Warnings
Errors
Notes
Retransmissions
Protocol problems
Conversation anomalies
```

Use it as an investigative lead.

Do not treat every expert message as an incident finding.

Always return to the actual packet evidence.

## Statistics as Supporting Evidence

Statistics can help establish patterns.

Useful views include:

```text
Protocol Hierarchy
Endpoints
Conversations
I/O Graphs
Service Response Time
Packet Lengths
DNS statistics
HTTP statistics
```

Statistics summarize packet evidence.

They should support, not replace, packet-level verification.

## Cross-Checking a Finding

For important conclusions, verify using multiple Wireshark views.

Example:

```text
Finding:
Host communicated with server.

Check:
Packet list
Conversation view
Endpoint statistics
Follow stream
```

If all agree, confidence increases.

## Reconstructing an Incident Timeline

A useful incident timeline may look like:

```text
09:45:10
Host obtains DNS response.

09:45:11
TCP connection established.

09:45:11.100
TLS handshake begins.

09:45:12
Encrypted application traffic begins.

09:45:15
Connection terminates.

09:46:15
Similar connection repeats.
```

Each event should have:

```text
Timestamp
Packet number
Source
Destination
Protocol
Description
Evidence reference
```

## Event Timeline Template

```text
| Time | Packet | Source | Destination | Protocol | Observation |
|------|--------|--------|-------------|----------|-------------|
|      |        |        |             |          |             |
|      |        |        |             |          |             |
|      |        |        |             |          |             |
```

Keep descriptions factual.

## Evidence Matrix

For larger investigations, use an evidence matrix.

```text
| Finding | Evidence | Supporting Packets | Alternative Explanation | Status |
|---------|----------|--------------------|-------------------------|--------|
|         |          |                    |                         |         |
```

Possible status values:

```text
Observed
Supported
Uncertain
Not established
Contradicted
```

## Example Evidence Matrix

```text
| Finding | Evidence | Packets | Alternative | Status |
|---------|----------|---------|-------------|--------|
| DNS query occurred | Query visible | 120 | None needed | Observed |
| TCP connection followed DNS | Matching IP/timing | 124-126 | Shared destination | Supported |
| Periodic communication | Repeated 60s intervals | 124-500 | Monitoring | Supported |
| Malicious intent | No direct evidence | — | Legitimate automation | Not established |
```

This prevents the final report from overstating what the capture proves.

## Practical Exercise 1 — Observation Extraction

Choose a small packet sequence.

Write only observations.

For example:

```text
Packet 100:
TCP SYN from A to B.

Packet 101:
TCP SYN/ACK from B to A.

Packet 102:
TCP ACK from A to B.

Packet 103:
TLS ClientHello from A to B.
```

Do not interpret the traffic yet.

## Practical Exercise 2 — Interpretation

Take the observations from Exercise 1.

Write:

```text
Interpretation:
```

Then explain what the sequence supports.

Finally write:

```text
Not established:
```

List what cannot be concluded.

## Practical Exercise 3 — Build an Evidence Chain

Find a sequence containing:

```text
DNS
↓
TCP
↓
TLS
```

Record:

```text
DNS packet:
TCP packets:
TLS packet:
Timestamps:
Destination:
```

Explain how the events correlate.

## Practical Exercise 4 — Competing Hypotheses

Find repeated communication.

Create:

```text
Hypothesis A:
Hypothesis B:
Hypothesis C:
```

For each:

```text
Supporting evidence:
Contradicting evidence:
Missing evidence:
```

Do not force a single explanation.

## Practical Exercise 5 — Negative Evidence

Find an investigation where an expected packet is missing.

Write:

```text
Observed:
```

Then:

```text
Possible explanations:
```

Then:

```text
What would be required to conclude that the event truly did not occur?
```

This exercise develops disciplined reasoning about absence.

## Practical Exercise 6 — Capture Limitation

Choose a capture affected by one of:

```text
NAT
VPN
Proxy
VLAN
SPAN
Packet loss
Encryption
```

Document:

```text
What is visible?
What is hidden?
How does the limitation affect the conclusion?
```

## Practical Exercise 7 — Reproducible Finding

Choose one important finding.

Record:

```text
Filter:
Packet numbers:
Timestamps:
Endpoints:
Relevant fields:
Interpretation:
Limitations:
```

Then clear the filter and reproduce the finding from the original capture.

## Practical Exercise 8 — Evidence Matrix

Choose a small investigation and build an evidence matrix containing at least five findings.

For each finding, include:

```text
Evidence
Packet references
Alternative explanation
Status
```

Review whether any finding is stronger than its evidence permits.

## Practical Exercise 9 — Timeline Reconstruction

Build a complete timeline for an event.

Include:

```text
First observed event
DNS
Connection establishment
Application negotiation
Application traffic
Repeated activity
Termination
Follow-on activity
```

Use exact timestamps and packet numbers.

## Practical Exercise 10 — Independent Forensic Analysis

Select an unfamiliar PCAP.

Start with only a question.

Then independently perform:

```text
1. Scope definition
2. Evidence preservation
3. Capture-context review
4. Endpoint identification
5. Protocol identification
6. Timeline construction
7. Conversation analysis
8. Statistical analysis
9. Packet-level verification
10. Evidence correlation
11. Hypothesis construction
12. Alternative explanation analysis
13. Limitation analysis
14. Evidence matrix creation
15. Final conclusion
```

Your final conclusion must clearly distinguish:

```text
What happened
What probably happened
What may have happened
What cannot be established
```

## Professional Evidence Record

Use this structure for significant investigations:

```text
Investigation ID:

Analyst:

Date:

Capture:

Capture hash:

Capture source:

Capture point:

Time zone:

Investigation question:

Scope:

Known context:

Relevant hosts:

Relevant protocols:

Timeline:

Key observations:

Packet references:

Display filters used:

Statistics consulted:

Evidence chains:

Interpretations:

Alternative explanations:

Capture limitations:

Missing evidence:

Findings:

Not established:

Additional evidence required:

Final conclusion:
```

## Final Conclusion Structure

A strong conclusion should answer:

```text
Question:
What was investigated?

Finding:
What does the evidence establish?

Evidence:
Which packets or observations support it?

Context:
What surrounding information affects interpretation?

Limitations:
What cannot be determined?

Next evidence:
What would resolve remaining uncertainty?
```

Example:

```text
Question:
Did the host establish communication with the external service?

Finding:
The capture establishes that the host completed a TCP
handshake and TLS handshake with the destination.

Evidence:
Packets 143–149 show the TCP and TLS exchange.

Context:
A DNS response immediately before the connection returned
the same destination IP.

Limitation:
Application payload remained encrypted.

Not established:
The specific application content exchanged cannot be
determined from this capture alone.

Additional evidence:
Endpoint logs or appropriate TLS decryption material
would be required for deeper content analysis.
```

## Common Mistakes

### Mistake 1: Starting With a Conclusion

Do not search the capture only for evidence supporting an assumption.

### Mistake 2: Treating Every Packet as Equally Important

Focus on evidence relevant to the investigation question.

### Mistake 3: Confusing Filters With Evidence

A filter locates packets. The packets provide the evidence.

### Mistake 4: Ignoring Capture Context

Capture location and visibility can fundamentally change interpretation.

### Mistake 5: Treating Missing Packets as Proof of Absence

A missing packet may result from capture limitations.

### Mistake 6: Ignoring Alternative Explanations

A strong interpretation should survive reasonable alternatives.

### Mistake 7: Overstating Encrypted Traffic

Do not infer application content that is not visible.

### Mistake 8: Ignoring Timestamps

Timing often provides critical correlation.

### Mistake 9: Failing to Record Packet References

A finding that cannot be reproduced is difficult to validate.

### Mistake 10: Treating Wireshark Expert Messages as Conclusions

Expert information identifies areas worth investigating.

### Mistake 11: Mixing Facts and Interpretation

Keep:

```text
Observed
```

separate from:

```text
Interpreted
```

### Mistake 12: Hiding Uncertainty

Uncertainty is part of professional analysis.

## Professional Forensic Workflow

Use this workflow for authorized packet investigations:

```text
Question
  ↓
Scope
  ↓
Preserve Evidence
  ↓
Capture Context
  ↓
Initial Overview
  ↓
Endpoints + Conversations
  ↓
Protocol Analysis
  ↓
Timeline
  ↓
Statistics
  ↓
Packet-Level Verification
  ↓
Evidence Correlation
  ↓
Hypotheses
  ↓
Alternative Explanations
  ↓
Limitations
  ↓
Evidence Matrix
  ↓
Conclusion
  ↓
Additional Evidence
```

The analyst should always be able to move backward:

```text
Conclusion
   ↓
Interpretation
   ↓
Observation
   ↓
Packet
```

If that chain breaks, the conclusion needs to be reconsidered.

## Investigation Checklist

Before closing a forensic packet investigation:

* [ ] Question defined
* [ ] Scope defined
* [ ] Original evidence preserved
* [ ] Capture context recorded
* [ ] Capture integrity recorded where required
* [ ] Relevant endpoints identified
* [ ] Relevant protocols identified
* [ ] Timeline constructed
* [ ] Packet references recorded
* [ ] Display filters recorded where useful
* [ ] Conversations examined
* [ ] Statistics used where useful
* [ ] Important findings verified at packet level
* [ ] Evidence chains documented
* [ ] Observations separated from interpretations
* [ ] Alternative explanations considered
* [ ] Capture limitations documented
* [ ] Negative evidence handled carefully
* [ ] Missing evidence identified
* [ ] Sensitive information handled appropriately
* [ ] Evidence matrix completed where appropriate
* [ ] Final conclusion tied to evidence
* [ ] Uncertainty explicitly documented
* [ ] Additional evidence identified

## Completion Criteria

You should be able to independently analyze a PCAP and produce a defensible evidence-based investigation.

You should be able to answer:

```text
What was the investigation question?

What evidence was available?

Where did the evidence come from?

What did the packets directly show?

How were the observations correlated?

What interpretations are supported?

What alternative explanations exist?

What limitations affect the analysis?

What was not established?

Which packets support the conclusion?

Can another analyst reproduce the finding?

What additional evidence would resolve remaining uncertainty?
```

The final mental model is:

```text
Question
   ↓
Evidence
   ↓
Observation
   ↓
Correlation
   ↓
Interpretation
   ↓
Alternative Explanations
   ↓
Limitations
   ↓
Conclusion
   ↓
Additional Evidence
```

Professional packet analysis is not about making the strongest claim possible.

It is about making the **strongest claim the evidence can actually support**.
