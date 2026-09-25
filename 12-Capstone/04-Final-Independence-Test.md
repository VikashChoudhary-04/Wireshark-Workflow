# Final Independence Test

## Purpose

This is the final test of the Wireshark workflow.

The previous capstones taught and tested investigation methodology.

This final test removes nearly all guidance.

You are given an unfamiliar network-analysis problem and an authorized packet capture.

You must independently decide:

```text
What matters
→ What to investigate
→ How to investigate it
→ What evidence is sufficient
→ What remains uncertain
→ When to stop
```

The objective is not to demonstrate that you can memorize Wireshark commands.

The objective is to demonstrate that you can perform a complete packet-analysis investigation independently.

## Rules

Use only:

```text
Authorized traffic
Laboratory traffic
Training PCAPs
Supplied captures
```

Do not perform active network testing against systems that you do not own or have permission to test.

For this final test:

```text
No prescribed filter sequence is provided.
No prescribed protocol sequence is provided.
No prescribed command sequence is provided.
No expected root cause is provided.
```

You may consult the rest of this repository for reference.

You must make the investigation decisions yourself.

## Final Test Scenario

A security and infrastructure team provides the following incident report:

```text
INCIDENT REPORT

Several employees reported that an internal service behaved abnormally
during a short period of time.

Some users reported connection problems.

Others reported that the service eventually responded but appeared slow.

The infrastructure team does not know whether the problem was caused by:

- name resolution
- network connectivity
- transport behavior
- the application
- an infrastructure dependency
- unusual network activity
- or another condition

A packet capture was collected during the incident.

The analyst who collected the capture did not provide an interpretation.
```

You receive:

```text
incident-final.pcapng
```

No additional packet-analysis instructions are provided.

## Primary Question

Determine:

```text
What can this packet capture establish about the reported incident?
```

Your answer should identify the strongest evidence available and clearly distinguish:

```text
Observed
```

from:

```text
Interpreted
```

and:

```text
Not established
```

## Investigation Expectations

You are expected to independently determine whether the capture contains evidence relevant to:

```text
Host identification
Protocol behavior
Name resolution
Connection establishment
Application communication
Timing
Transport problems
Repeated failures
Unexpected communication
Anomalous behavior
```

You do not need to investigate every category.

Only investigate categories that become relevant from the evidence.

## Start With the Evidence

Before drawing conclusions, establish the capture boundaries.

Record:

```text
Capture filename:

Capture format:

Packet count:

Start time:

End time:

Capture duration:

Capture interface or location, if known:

Wireshark version:

TShark version, if used:

SHA-256, if calculated:
```

If some information is unavailable, record:

```text
Not available from the supplied evidence.
```

Do not invent missing context.

## Build Your Own Investigation Plan

Write your initial plan before performing detailed analysis.

Use:

```text
Initial Investigation Plan:

1.
2.
3.
4.
5.
```

The plan does not need to be correct from the beginning.

Professional investigation is iterative.

If evidence changes your direction, document the change.

## Establish the Network Picture

Determine what the capture tells you about:

```text
Endpoints
Protocols
Conversations
Traffic volume
Time distribution
Potentially relevant services
```

You decide which Wireshark features are appropriate.

Possible capabilities include:

```text
Protocol Hierarchy
Endpoints
Conversations
I/O Graphs
Expert Information
Packet Details
Packet Bytes
Follow Stream
Name Resolution
Display Filters
TShark
```

These are available as tools, not as a required sequence.

## Identify the Incident Traffic

Determine which traffic can reasonably be associated with the incident.

Document:

```text
Relevant host(s):

Relevant server(s):

Relevant protocol(s):

Relevant port(s):

Relevant time range:

Reason for selecting this traffic:
```

If more than one candidate exists, investigate the alternatives.

## Define Investigation Questions

Create questions based on what you observe.

For example:

```text
Question 1:

Question 2:

Question 3:

Question 4:
```

Do not create questions merely to make the report longer.

Each question should reduce meaningful uncertainty.

## Investigate the First Meaningful Deviation

When analyzing the affected communication, identify the earliest meaningful difference between:

```text
Expected behavior
```

and:

```text
Observed behavior
```

Where possible, compare:

```text
Normal transaction
```

with:

```text
Affected transaction
```

Record the evidence that establishes the difference.

## Timing Analysis

If timing appears relevant, establish a timeline.

A useful timeline may contain:

```text
Event
Timestamp
Source
Destination
Protocol
Frame
Elapsed time
Interpretation
```

Example structure:

| Time | Event               | Frame | Observation         | Interpretation                 |
| ---- | ------------------- | ----: | ------------------- | ------------------------------ |
| T1   | Connection attempt  |   100 | SYN observed        | Client begins connection       |
| T2   | Response            |   101 | SYN-ACK observed    | Server response visible        |
| T3   | Application request |   110 | Request transmitted | Application transaction begins |
| T4   | Response            |   150 | Response observed   | Transaction continues          |

Use only events supported by the capture.

## Transport Analysis

If transport behavior is relevant, determine whether the capture shows evidence such as:

```text
Connection establishment
Retransmissions
Duplicate acknowledgments
Out-of-order packets
Resets
Zero-window behavior
Connection termination
Unexpected timing
```

Do not automatically interpret protocol analysis flags as root causes.

Determine whether the behavior is:

```text
Relevant
Repeated
Correlated with the incident
Different from normal traffic
```

## Application Analysis

If application-layer traffic is visible, determine what the capture establishes.

Depending on the protocol, investigate relevant information such as:

```text
Requests
Responses
Methods
Status codes
Hosts
URIs
Application errors
Streams
Protocol messages
```

If the application payload is encrypted, explicitly state what remains visible and what does not.

## Security Analysis

If unusual communication appears, determine:

```text
Who communicated?
With whom?
When?
Using which protocol?
On which port?
How frequently?
What visible metadata exists?
Is the communication isolated or repeated?
```

Do not label traffic malicious merely because it is unfamiliar.

Use evidence-based language.

For example:

```text
The capture shows repeated connections from host A to host B
over port X during the incident window.
```

Then separately state what that may warrant investigating.

## Correlation

The final test should demonstrate cross-layer reasoning.

Where relevant, correlate:

```text
DNS
↓
IP
↓
TCP/UDP
↓
TLS
↓
Application
↓
Timing
```

The exact sequence depends on the traffic.

The objective is to determine whether events across different protocol layers describe the same transaction or incident.

## TShark Requirement

Use TShark at least once if it is available in your environment.

The command should answer a real investigation question.

Examples of legitimate objectives include:

```text
Extract relevant fields
Filter a conversation
Inspect timestamps
Identify endpoints
Count protocol activity
Extract TCP stream information
Reproduce an observation made in Wireshark
```

Do not run TShark merely to satisfy the requirement.

Record:

```text
Command:

Purpose:

Relevant output:

How the output contributed to the investigation:
```

## Evidence Requirements

Every major conclusion must have traceable evidence.

Use an evidence table:

| ID | Question | Frame(s) | Observation | Interpretation | Confidence/Limitations |
| -- | -------- | -------- | ----------- | -------------- | ---------------------- |
| E1 |          |          |             |                |                        |
| E2 |          |          |             |                |                        |
| E3 |          |          |             |                |                        |

Use exact frame numbers where possible.

For streams, record the stream identifier.

For fields, record the relevant protocol field.

## Observation vs Interpretation

Maintain a strict distinction.

### Observation

An observation describes what the capture directly shows.

Examples:

```text
Frame 245 contains a TCP SYN from the client to the server.
```

```text
A DNS response is followed by a connection to the returned address.
```

```text
The application request is followed by a 2.8-second visible gap.
```

### Interpretation

An interpretation explains what the observation may mean.

Examples:

```text
The connection attempt appears to be proceeding normally.
```

```text
The DNS response appears to have supplied the destination address.
```

```text
The visible delay occurs after the application request.
```

### Conclusion

A conclusion answers the investigation question using the evidence.

Examples:

```text
The capture localizes the observed delay to the period after the
application request, but does not establish the internal server-side cause.
```

or:

```text
The capture shows repeated unsuccessful connection attempts during
the affected period, with no corresponding SYN-ACK visible at the
capture point.
```

## Alternative Explanations

For each important finding, identify at least one reasonable alternative explanation when one exists.

Use:

```text
Primary interpretation:

Alternative explanation:

Evidence supporting the primary interpretation:

Evidence that would distinguish the alternatives:
```

This prevents premature conclusions.

## Capture Limitations

Explicitly document limitations.

Consider:

```text
Capture point
Capture duration
Missing traffic
Asymmetric visibility
Packet loss
Truncation
Encryption
Timestamp accuracy
Name-resolution behavior
Protocol dissection limitations
Unknown infrastructure
```

Do not turn a limitation into a conclusion.

For example:

```text
No server response is visible.
```

does not necessarily mean:

```text
The server did not respond.
```

The correct interpretation depends on capture visibility.

## Final Conclusion

Your conclusion should answer:

```text
What happened?

What evidence supports that conclusion?

What remains uncertain?

What cannot be established from the capture?
```

Keep the conclusion proportional to the evidence.

Avoid statements such as:

```text
This definitely proves...
```

unless the evidence genuinely supports that level of certainty.

Prefer precise language such as:

```text
The capture shows...

The evidence is consistent with...

The timing indicates...

The capture does not establish...

Additional evidence would be required to determine...
```

## Required Final Report

Produce a final report using this structure:

```text
# Final Independence Test Report

## Executive Summary

## Incident Statement

## Capture Scope

## Investigation Questions

## Initial Hypothesis

## Investigation Path

## Relevant Hosts and Traffic

## Timeline

## Key Evidence

## Findings

## Alternative Explanations

## Limitations

## Final Conclusion

## Unresolved Questions

## Reproducibility Record
```

The report should be understandable to another analyst who did not perform the investigation.

## Reproducibility Record

Record:

```text
Capture:

Wireshark version:

TShark version:

Important display filters:

Important TShark commands:

Important frame numbers:

Important stream identifiers:

Important statistics:

Supporting output/files:
```

Another analyst should be able to follow your evidence trail.

## Investigation Journal

During the investigation, maintain a short journal.

```text
### Investigation Entry

Time:

Question:

Action:

Observation:

Interpretation:

Next decision:
```

Repeat this for major investigation decisions.

The journal should demonstrate how you moved from uncertainty to evidence.

## Stop Condition

Do not continue indefinitely.

The investigation is complete when:

```text
The primary question has been answered as far as the evidence allows.
```

and:

```text
The key conclusion is supported by traceable evidence.
```

and:

```text
Important limitations have been documented.
```

and:

```text
Further analysis is unlikely to materially change the conclusion.
```

If the evidence cannot answer the question, that is itself a valid result.

## Final Independence Checklist

Before declaring the test complete:

```text
[ ] I preserved the original capture.
[ ] I documented the capture boundaries.
[ ] I established the relevant hosts and protocols.
[ ] I defined investigation questions.
[ ] I selected my own investigation path.
[ ] I identified the relevant traffic.
[ ] I investigated the first meaningful deviation.
[ ] I used timing where relevant.
[ ] I correlated protocol layers where appropriate.
[ ] I used TShark for a meaningful task.
[ ] I recorded important frame numbers.
[ ] I recorded important stream identifiers.
[ ] I documented important filters and commands.
[ ] I compared normal and affected behavior where possible.
[ ] I considered alternative explanations.
[ ] I separated observations from interpretations.
[ ] I documented capture limitations.
[ ] I avoided unsupported conclusions.
[ ] I stated what the capture cannot establish.
[ ] Another analyst could reproduce my important findings.
[ ] I stopped when the investigation was sufficiently answered.
```

## Competency Demonstrated

Completion of this test should demonstrate that you can independently:

```text
Orient yourself in an unfamiliar capture
        ↓
Identify relevant traffic
        ↓
Turn an operational problem into investigation questions
        ↓
Select appropriate Wireshark capabilities
        ↓
Analyze packets and protocols
        ↓
Correlate events across layers
        ↓
Reason about timing
        ↓
Validate important findings
        ↓
Challenge alternative explanations
        ↓
Build an evidence chain
        ↓
Communicate uncertainty
        ↓
Produce a reproducible conclusion
```

## What This Test Is Not

This is not:

```text
A command memorization test
A display-filter memorization test
A protocol trivia test
A packet-count competition
A requirement to explain every packet
A requirement to find a dramatic root cause
```

A concise, evidence-backed investigation is preferable to an unnecessarily large investigation.

## Final Principle

Professional packet analysis is not:

```text
Open Wireshark
→ Apply filters
→ Find something unusual
→ Declare a cause
```

It is:

```text
Question
→ Evidence
→ Investigation
→ Observation
→ Interpretation
→ Validation
→ Decision
→ Uncertainty
→ Conclusion
```

The final measure of independence is simple:

```text
Can you receive an unfamiliar packet capture,
an unfamiliar problem, and determine what the evidence
actually tells you without being told what to look for?
```

If you can do that consistently, Wireshark has become an investigation tool rather than simply a packet viewer.
