# Analysis Documentation

## Objective

A technically correct Wireshark investigation is incomplete if another analyst cannot understand, reproduce, or verify the work.

Professional documentation should answer:

```text id="v3n8k6"
What was investigated?
Why was it investigated?
What evidence was available?
What was observed?
How was it analyzed?
What was concluded?
What remains uncertain?
```

The goal is not to write a long report for every packet capture.

The goal is to create documentation that is:

```text id="h8q2m5"
Clear
Reproducible
Evidence-based
Concise
Traceable
Appropriately detailed
```

## Documentation Mental Model

Use:

```text id="c4r7w2"
Question
  ↓
Scope
  ↓
Method
  ↓
Evidence
  ↓
Analysis
  ↓
Finding
  ↓
Limitations
  ↓
Conclusion
  ↓
Next Action
```

A reader should be able to move backward from:

```text id="k5m9p3"
Conclusion
```

to:

```text id="q7v2n4"
Evidence
```

and finally:

```text id="z8c4x6"
Packet
```

## Why Documentation Matters

Good documentation provides:

* Reproducibility
* Analyst handoff
* Incident continuity
* Troubleshooting history
* Evidence traceability
* Reviewability
* Reduced duplicated work
* Better communication with technical and non-technical teams

Poor documentation creates:

```text id="w6j3r9"
"I saw something strange."
```

Good documentation creates:

```text id="b2n8q5"
"Packets 143–149 show a TCP and TLS connection from
192.0.2.25 to 203.0.113.20 beginning at 10:15:32.
The DNS response immediately before the connection
returned the same destination address."
```

## Document the Investigation Question

Start with one sentence.

Examples:

```text id="f4m7x2"
Determine whether the client completed a connection
to the application server.

Determine where latency occurs during the HTTPS transaction.

Determine whether the host communicated with unexpected
internal systems.

Determine whether repeated DNS queries correlate with
subsequent external connections.
```

Avoid vague questions such as:

```text id="p9v3k6"
"Analyze this PCAP."
```

## Document the Scope

Record:

```text id="a5q8m1"
Capture:
Time window:
Source:
Destination:
Network:
Protocol:
Application:
Known context:
```

Example:

```text id="d7n2w4"
Capture:
application-delay.pcapng

Time:
10:00–10:15

Source:
192.0.2.25

Destination:
203.0.113.20

Protocol:
HTTPS

Problem:
Intermittent response delays
```

## Document Capture Context

Include:

```text id="j6r4c8"
Capture point:
Interface:
Network segment:
VLAN:
SPAN/TAP:
Capture host:
Capture start:
Capture end:
Timezone:
Capture filter:
Known packet loss:
```

If information is unknown, write:

```text id="m3v7q9"
Unknown
```

Do not invent missing context.

## Document Evidence Integrity

When appropriate, record:

```text id="x8k2p5"
Filename:
File size:
Hash:
Source:
Collection time:
```

For formal evidence handling, follow the applicable organizational procedures.

## Document the Method

Briefly explain how the investigation was performed.

Example:

```text id="y4n8c6"
The capture was first reviewed using Protocol Hierarchy,
Endpoints, and Conversations. Traffic involving the
affected client was then isolated. DNS, TCP, TLS, and
application timing were analyzed separately. Relevant
findings were verified against packet-level evidence.
```

The methodology should explain the analytical path without documenting every click.

## Record Important Filters

Filters make an investigation reproducible.

Example:

```text id="c9m4r7"
ip.addr == 192.0.2.25
```

Then:

```text id="q2x8v5"
ip.addr == 192.0.2.25 && tcp.port == 443
```

Record only filters that materially contributed to the analysis.

## Explain Why a Filter Was Used

Do not simply list:

```text id="w5n7k3"
dns
tcp
tls
```

Explain:

```text id="r8c2m6"
dns
→ Identify hostname-resolution activity.

tcp.port == 443
→ Isolate encrypted web-related transport traffic.

ip.addr == 192.0.2.25
→ Focus the investigation on the affected host.
```

This makes the reasoning easier to follow.

## Packet References

For significant findings, record:

```text id="v6p3q8"
Packet number
Timestamp
Source
Destination
Protocol
Relevant field
```

Example:

```text id="g2m9x4"
Packet:
143

Time:
10:15:32.125

Source:
192.0.2.25

Destination:
203.0.113.20

Protocol:
TCP

Finding:
Initial SYN to destination port 443.
```

## Packet Ranges

When a sequence matters:

```text id="n4q7w1"
Packets 143–149
```

Then summarize the sequence:

```text id="j8m3c5"
TCP handshake followed by TLS handshake initiation.
```

Avoid forcing the reader to inspect every packet without explanation.

## Timeline Documentation

Use exact timestamps where timing matters.

Template:

```text id="p5x8r2"
| Time | Packet | Event | Observation |
|------|--------|-------|-------------|
|      |        |       |             |
|      |        |       |             |
|      |        |       |             |
```

Example:

```text id="y7k4m1"
| Time | Packet | Event | Observation |
|------|--------|-------|-------------|
| 10:15:32.100 | 140 | DNS Query | Host requested api.example.test |
| 10:15:32.125 | 141 | DNS Response | Address returned |
| 10:15:32.150 | 143 | TCP SYN | Connection to port 443 |
| 10:15:32.180 | 144 | TCP SYN/ACK | Server responded |
| 10:15:32.200 | 145 | TCP ACK | Handshake completed |
```

## Finding Structure

Use a consistent finding format:

```text id="c6v2n8"
Finding:

Evidence:

Interpretation:

Impact:

Limitations:

Additional evidence:
```

Not every investigation needs an impact section, but the structure is useful for significant findings.

## Observation vs Interpretation

Always separate them.

### Observation

```text id="f9k3m7"
The client sent three TCP SYN packets to the server
within 2 seconds.
```

### Interpretation

```text id="x4q8r1"
The client repeatedly attempted to establish a TCP connection.
```

### Conclusion

```text id="m2v6c9"
The capture supports repeated connection attempts, but it
does not establish why the attempts occurred.
```

This distinction is one of the most important documentation habits.

## Document What Is Not Established

Include a section such as:

```text id="a7p4n9"
Not Established
```

Examples:

```text id="j3w8q2"
The capture does not establish the initiating user.

The encrypted payload cannot be inspected.

The original client address cannot be determined because
the capture was taken after NAT.

The absence of a packet cannot establish that the event
never occurred.
```

This prevents readers from overextending the findings.

## Document Capture Limitations

Use:

```text id="r5m8c1"
Capture limitations:
```

Possible entries:

```text id="w2q7v4"
Capture was taken from a client-side interface.

Traffic inside the VPN tunnel was not visible.

Application payload was encrypted.

SPAN configuration may have limited visibility.

The capture begins after the initial connection.

Packet loss cannot be ruled out.
```

## Document Uncertainty

Use precise terms:

```text id="k6x3p8"
Observed
Supported
Consistent with
Possible
Uncertain
Not established
```

Avoid vague phrases such as:

```text id="v9m4q2"
"Definitely suspicious"
"Obviously malicious"
"Clearly the server problem"
```

unless the evidence genuinely establishes the statement and the wording is appropriate to the investigation.

## Avoid Overclaiming

Suppose the capture shows:

```text id="n8r5c2"
Host → Server:443
TLS connection
```

Do not document:

```text id="z4m7p1"
"The host downloaded a malicious file."
```

Instead:

```text id="q6w2k9"
"The host established an encrypted connection to the server.
The application payload was not visible in the capture."
```

## Evidence Chains

Document related events as a chain.

Example:

```text id="s3v7m1"
DNS query
  ↓
DNS response
  ↓
Returned IP
  ↓
TCP connection
  ↓
TLS handshake
  ↓
Encrypted application traffic
```

Then explain what the chain supports.

## Correlation Table

For larger investigations:

```text id="x7c4n8"
| Event | Time | Packet | Evidence | Related Event |
|------|------|--------|----------|---------------|
| DNS Query | | | | DNS Response |
| DNS Response | | | | TCP |
| TCP SYN | | | | TCP SYN/ACK |
| TLS ClientHello | | | | TLS Response |
```

This makes relationships easier to review.

## Compare Normal and Abnormal

When troubleshooting, document differences.

```text id="j9q5m2"
| Stage | Normal | Problem | Difference |
|------|--------|---------|------------|
| DNS | | | |
| TCP | | | |
| TLS | | | |
| Application | | | |
```

This is especially useful when a known-good capture exists.

## Document Timing

Use actual measurements.

Example:

```text id="g4w8r6"
DNS:
18 ms

TCP:
32 ms

TLS:
48 ms

Application response:
2,340 ms
```

Then:

```text id="m5x2p9"
The application-response stage accounts for the majority
of the observed transaction time.
```

Do not automatically infer the root cause from timing alone.

## Document Outliers

For an unusual transaction:

```text id="c8n3v7"
Normal requests:
120–160 ms

Outlier:
2,450 ms
```

Then investigate:

```text id="r6q9m4"
Retransmissions?
Connection reuse?
Server response delay?
Packet loss?
Different destination?
Different payload size?
```

## Document Statistics

Statistics should support a finding.

Example:

```text id="y2p7k5"
Conversation statistics show that the affected host
communicated with the destination 42 times during the
investigated interval.
```

Then identify the underlying packets or conversations.

Do not use a statistical summary as a substitute for evidence.

## Document Graph Findings

For an I/O graph:

```text id="f3m8q1"
Observation:
Traffic volume increased sharply between 10:05 and 10:06.

Follow-up:
Packets in this interval were isolated and found to contain
multiple simultaneous application connections.
```

The graph identifies the interval.

Packet analysis explains it.

## Document Expert Information

If Expert Information identifies a retransmission:

```text id="w7c2m6"
Expert Information:
TCP retransmission reported.

Verification:
Packet 520 was compared with the original transmission.
```

This is stronger than documenting:

```text id="p4x8n1"
"Wireshark says there was packet loss."
```

## Analyst Notes

A useful analyst note records decisions.

Template:

```text id="q6r3v9"
Observation:
Decision:
Reason:
Next action:
Result:
```

Example:

```text id="t8m5k2"
Observation:
DNS returned the destination address.

Decision:
Inspect TCP connections to that address immediately afterward.

Reason:
Determine whether the DNS result led to the observed session.

Result:
TCP connection began 35 ms later.

Next action:
Inspect TLS handshake.
```

This creates an investigation trail.

## Reproducibility

Another analyst should be able to reproduce important findings.

Provide:

```text id="v5n2q8"
Capture
Time window
Filter
Packet numbers
Relevant fields
Method
Conclusion
```

A useful test is:

```text id="k7m4x1"
Could another analyst locate the evidence
without asking me where I found it?
```

If not, the documentation needs improvement.

## Analyst Handoff

A handoff should be concise.

Use:

```text id="b8q3w6"
Investigation:
Question:
Scope:
Key finding:
Evidence:
Timeline:
Limitations:
Open questions:
Next recommended evidence:
```

Example:

```text id="r4m9c2"
Investigation:
Application latency

Question:
Where does the observed delay occur?

Key finding:
DNS, TCP, and TLS complete quickly. The majority of delay
occurs after the application request.

Evidence:
Packets 200–230.

Limitation:
Application processing is not directly visible in the capture.

Open question:
Why did the server take approximately 2.4 seconds to respond?

Next evidence:
Server/application logs.
```

## Executive Summary vs Technical Detail

Different audiences need different levels of detail.

### Executive Summary

Focus on:

```text id="z6p2n8"
Problem
Finding
Impact
Limitations
Next action
```

### Technical Analysis

Include:

```text id="c4r7m1"
Packets
Filters
Protocol fields
Timing
Conversations
Statistics
Evidence chains
```

Do not overwhelm a non-technical reader with packet-level detail when it is unnecessary.

## Technical Appendix

For complex investigations, move detailed material into an appendix.

Possible contents:

```text id="j8w3q5"
Display filters
Packet references
Timeline
Conversation identifiers
Statistics
Screenshots
Protocol details
Capture limitations
```

This keeps the main conclusion readable while preserving reproducibility.

## Screenshots

Use screenshots when they clarify a finding.

A useful screenshot should show:

```text id="n5x9c2"
Relevant packet
Packet number
Timestamp
Important field
Filter
```

Avoid screenshots that contain:

```text id="m7q4v1"
Unrelated packets
Sensitive data
Huge amounts of unreadable text
```

## Sensitive Evidence Handling

PCAPs may contain:

```text id="p3k8r6"
Credentials
Cookies
Tokens
Personal information
Internal addresses
Email content
File contents
Authentication information
```

Before sharing documentation, determine whether sensitive material must be:

```text id="v2m6q9"
Redacted
Anonymized
Removed
Restricted
```

Never publish sensitive evidence merely to make a report look complete.

## Evidence Redaction

If a screenshot contains sensitive information, consider:

```text id="h4q7x3"
Masking credentials
Removing tokens
Anonymizing addresses
Removing personal information
Cropping unrelated fields
```

Retain enough information for the finding to remain reproducible where permitted.

## Naming Conventions

Use clear investigation names.

Example:

```text id="a8m5r2"
2026-09-25_https-latency_client25
```

For files:

```text id="k3v9q6"
capture.pcapng
analysis.md
timeline.md
screenshots/
```

Choose a consistent convention for the project.

## Documentation Versioning

When an investigation changes significantly, preserve the revision history where required.

Example:

```text id="y7c2m8"
Version:
1.0

Date:
2026-09-25

Analyst:
Analyst Name

Change:
Initial analysis completed.
```

For collaborative investigations, record substantive changes.

## Findings Table

A useful summary:

```text id="f6n3p9"
| ID | Finding | Evidence | Confidence | Status |
|----|---------|----------|------------|--------|
| F-01 | | | | |
| F-02 | | | | |
| F-03 | | | | |
```

Do not turn confidence into an arbitrary score.

Use descriptive terms.

## Open Questions

A professional report should identify unresolved issues.

Examples:

```text id="q8w4m2"
Which application component caused the delay?

Did the event continue outside the capture window?

Was the communication authorized?

What happened inside the encrypted session?

Did the same behavior occur on other hosts?
```

These questions guide additional investigation.

## Additional Evidence

Specify what would answer the open questions.

Examples:

```text id="m5r7c1"
Endpoint logs
DNS logs
Firewall logs
Proxy logs
Server logs
Application logs
Authentication logs
Additional packet captures
TLS decryption material where authorized
```

Do not request evidence simply because it exists.

Request evidence because it resolves a specific uncertainty.

## Final Conclusion

A professional conclusion should contain:

```text id="v4x9k7"
Question:
What was investigated?

Finding:
What does the evidence establish?

Evidence:
Which packets support the finding?

Interpretation:
What does the evidence mean?

Limitations:
What cannot be determined?

Next step:
What evidence or action would resolve the remaining uncertainty?
```

## Example Final Conclusion

```text id="z2m6q8"
Question:
Why did the client experience intermittent HTTPS delays?

Finding:
The capture shows normal DNS and TCP establishment. TLS
negotiation also completed normally. The primary delay occurred
between the application request and the corresponding response.

Evidence:
Packets 310–335 show the complete transaction. Comparable
transactions normally completed within approximately 150 ms,
while the investigated transaction required approximately
2.4 seconds before the response arrived.

Interpretation:
The packet evidence places the observed delay after the
application request rather than during DNS or TCP establishment.

Limitations:
The capture cannot determine the internal processing performed
by the server.

Next step:
Review application and server-side logs for the corresponding
transaction timestamp.
```

## Documentation Template

Use this as a reusable investigation template:

```text id="n7c4m9"
# Investigation Title

## Investigation Question

## Scope

## Capture Context

## Methodology

## Initial Observations

## Timeline

## Key Findings

### Finding 1

**Observation:**

**Evidence:**

**Interpretation:**

**Limitations:**

### Finding 2

**Observation:**

**Evidence:**

**Interpretation:**

**Limitations:**

## Filters Used

## Important Packet References

## Statistical Analysis

## Evidence Chains

## Alternative Explanations

## Not Established

## Open Questions

## Additional Evidence Required

## Final Conclusion
```

The template intentionally uses one H1 heading.

All subsequent sections use H2 or lower.

## Practical Exercise 1 — Document an Existing Investigation

Take one previous Wireshark exercise from this repository.

Write:

```text id="j5r8m2"
Question
Scope
Methodology
Timeline
Evidence
Finding
Limitations
Conclusion
```

Do not add information that was not actually observed.

## Practical Exercise 2 — Reproducible Finding

Choose one packet-level finding.

Document:

```text id="c9v4x7"
Filter
Packet number
Timestamp
Source
Destination
Protocol
Relevant field
Interpretation
```

Give the documentation to another analyst and ask them to reproduce it.

## Practical Exercise 3 — Timeline

Choose a complete network transaction.

Create:

```text id="p2m7n5"
Timestamp
Packet
Event
Observation
```

Then write a short narrative describing the sequence.

## Practical Exercise 4 — Evidence vs Interpretation

Take five statements from an earlier investigation.

For each, classify it as:

```text id="x6q3k8"
Observation
Interpretation
Conclusion
Not established
```

Rewrite any statement whose certainty exceeds its evidence.

## Practical Exercise 5 — Capture Limitations

Choose a capture involving encryption, NAT, VPN, proxying, or packet loss.

Document:

```text id="w8r4c2"
What is visible?
What is hidden?
How does the limitation affect the conclusion?
What additional evidence would resolve it?
```

## Practical Exercise 6 — Analyst Handoff

Create a one-page handoff containing:

```text id="m3n9v6"
Question
Scope
Key evidence
Timeline
Finding
Limitations
Open questions
Next evidence
```

Make it understandable to an analyst who has not seen your investigation.

## Practical Exercise 7 — Executive Summary

Take a technically detailed investigation and reduce it to:

```text id="f7c2m5"
Problem
Finding
Impact
Limitation
Next action
```

Keep the technical appendix separate.

## Practical Exercise 8 — Evidence Matrix

Build an evidence matrix with at least five findings.

```text id="q4x8n1"
| ID | Finding | Evidence | Alternative | Status |
|----|---------|----------|-------------|--------|
|    |         |          |             |        |
```

Check whether every finding has identifiable evidence.

## Practical Exercise 9 — Documentation Review

Review your own report and ask:

```text id="r6m2k9"
Can another analyst reproduce my findings?

Did I record packet numbers?

Did I record important filters?

Did I explain why I used them?

Did I separate observations from interpretations?

Did I document limitations?

Did I identify what remains unknown?
```

Correct any weaknesses.

## Practical Exercise 10 — Independent Investigation Report

Choose an unfamiliar capture and perform a complete investigation.

Produce a professional report containing:

```text id="y8p3v5"
1. Investigation question
2. Scope
3. Capture context
4. Methodology
5. Timeline
6. Key findings
7. Packet evidence
8. Filters
9. Statistical analysis
10. Alternative explanations
11. Limitations
12. Not established
13. Open questions
14. Additional evidence
15. Final conclusion
```

The report should be understandable without opening Wireshark, while still providing enough information for another analyst to reproduce the important findings.

## Professional Documentation Checklist

Before finalizing documentation:

* [ ] Investigation question written
* [ ] Scope defined
* [ ] Capture context documented
* [ ] Evidence integrity recorded where required
* [ ] Methodology summarized
* [ ] Important filters recorded
* [ ] Packet references recorded
* [ ] Timeline documented
* [ ] Relevant statistics documented
* [ ] Evidence chains documented
* [ ] Observations separated from interpretations
* [ ] Alternative explanations considered
* [ ] Capture limitations documented
* [ ] Sensitive information handled appropriately
* [ ] Unknowns documented
* [ ] Open questions documented
* [ ] Additional evidence identified
* [ ] Conclusion tied to evidence
* [ ] Another analyst could reproduce important findings

## Completion Criteria

You should be able to turn an independent Wireshark investigation into professional documentation that another analyst can understand and verify.

You should be able to answer:

```text id="c5m8r2"
What was investigated?

Why was it investigated?

What was the scope?

Where did the capture come from?

What methodology was used?

What happened chronologically?

Which packets support the findings?

Which filters were used?

What does the evidence establish?

What is only an interpretation?

What alternative explanations exist?

What cannot be established?

What limitations affect the analysis?

What additional evidence is required?

Can another analyst reproduce the findings?
```

The documentation workflow is:

```text id="n4q7w1"
Question
  ↓
Scope
  ↓
Method
  ↓
Evidence
  ↓
Timeline
  ↓
Analysis
  ↓
Findings
  ↓
Limitations
  ↓
Open Questions
  ↓
Conclusion
```

Professional documentation turns packet analysis into a reusable technical record rather than a collection of observations that exist only in the analyst's memory.
