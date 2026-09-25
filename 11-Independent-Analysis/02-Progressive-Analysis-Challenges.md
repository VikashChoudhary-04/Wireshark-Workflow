# Progressive Analysis Challenges

## Purpose

This file provides a progression from guided packet analysis to independent investigation.

The challenges are deliberately structured so that instructions decrease while decision-making increases.

The progression is:

```text
Guided
→ Partially Guided
→ Open Investigation
→ Independent Investigation
```

The objective is not to complete the largest number of exercises.

The objective is to demonstrate that you can independently:

```text
Understand the problem
→ formulate questions
→ choose evidence
→ select Wireshark capabilities
→ investigate
→ adapt
→ document
→ conclude
```

Use only authorized captures, laboratory traffic, supplied PCAPs, or traffic you are permitted to analyze.

## How to Use These Challenges

Do not immediately look for a command.

For every challenge:

```text
1. Read the problem.
2. Identify what is known.
3. Identify what is unknown.
4. Define the first question.
5. Choose the appropriate Wireshark capability.
6. Investigate.
7. Record evidence.
8. Decide what to investigate next.
9. Stop when the question is sufficiently answered.
```

For later challenges, resist the temptation to search for a predefined solution.

The goal is to train decision-making.

## Challenge Progression

The progression contains four stages.

```text
Stage 1
Guided Analysis

Stage 2
Partially Guided Analysis

Stage 3
Open Investigation

Stage 4
Independent Investigation
```

Each stage reduces the amount of information provided.

## Stage 1: Guided Analysis

These challenges reinforce the complete investigation loop.

You should already know the relevant Wireshark features.

## Challenge 1: Identify the Capture

### Scenario

You receive an unfamiliar PCAP and are told only:

```text
"Determine what kind of traffic this capture contains."
```

### Tasks

Determine:

```text
What protocols are present?

Which endpoints are present?

Which conversations exist?

What appears to be the dominant traffic?
```

### Suggested Starting Point

Begin with:

```text
Capture metadata
Protocol hierarchy
Endpoints
Conversations
```

Do not inspect every packet.

### Evidence Record

Document:

```text
Capture:

Packet count:

Time range:

Major protocols:

Major endpoints:

Major conversations:

Initial interpretation:
```

### Completion Condition

You can describe the capture's general communication landscape without relying on packet-by-packet inspection.

## Challenge 2: DNS Resolution

### Scenario

A client is reported to have experienced intermittent name-resolution problems.

You receive a PCAP containing the relevant traffic.

### Tasks

Determine:

```text
Which client generated DNS queries?

Which DNS server was contacted?

Which names were queried?

Which responses succeeded?

Which responses failed?

When did the failures occur?
```

### Investigation Direction

Use:

```text
DNS filters
DNS query fields
DNS response fields
timestamps
client/server correlation
```

### Completion Condition

You can explain the observed DNS behavior using packet evidence.

## Challenge 3: TCP Connection Establishment

### Scenario

A user reports:

```text
"The service sometimes refuses the connection."
```

### Tasks

Determine:

```text
Which host initiated the connection?

Which destination was contacted?

Did the TCP handshake complete?

Were resets observed?

Were retransmissions observed?

When did the connection attempt occur?
```

### Investigation Direction

Use:

```text
TCP flags
TCP streams
timestamps
TCP analysis fields
```

### Completion Condition

You can identify the visible TCP connection sequence and describe where it differs from the expected sequence.

## Challenge 4: HTTP Request Investigation

### Scenario

A web application is returning unexpected responses.

### Tasks

Determine:

```text
Which client made the request?

Which server received it?

What HTTP method was used?

Which host and URI were requested?

What response status was returned?

Were redirects or repeated requests observed?
```

### Investigation Direction

Use:

```text
HTTP display filters
HTTP fields
TCP streams
timestamps
```

### Completion Condition

You can reconstruct the relevant HTTP transaction from visible packet evidence.

## Challenge 5: TLS Handshake

### Scenario

A client reports that an HTTPS connection fails intermittently.

### Tasks

Determine:

```text
Did the TCP connection establish?

Did TLS negotiation begin?

Which TLS handshake messages are visible?

Were TLS alerts observed?

How long did the visible handshake take?

What information is unavailable because the application payload is encrypted?
```

### Completion Condition

You can distinguish:

```text
TCP establishment
from
TLS establishment
from
encrypted application traffic
```

## Stage 2: Partially Guided Analysis

At this stage, the workflow is less explicit.

You must choose more of the investigation yourself.

## Challenge 6: Connectivity Failure

### Scenario

A client cannot reach a service.

You are given:

```text
Client:
10.10.10.20

Server:
10.10.10.50

Service:
TCP-based application
```

No additional diagnosis is provided.

### Tasks

Determine:

```text
Whether the client attempted a connection

Whether the server responded

Whether the TCP handshake completed

Whether the connection was reset

Whether retransmissions occurred

Where the visible connection sequence changed
```

### Constraints

Do not begin by inspecting every packet.

Start by establishing the relevant conversation.

### Deliverable

Produce:

```text
Question:

Evidence:

Important frames:

Timeline:

Observation:

Interpretation:

Limitations:

Conclusion:
```

## Challenge 7: Slow Application

### Scenario

A user reports:

```text
"The application takes several seconds to respond."
```

The capture contains:

```text
DNS
TCP
TLS
application traffic
```

### Tasks

Determine where the delay occurs.

Investigate:

```text
DNS timing

TCP connection timing

TLS timing

Application request timing

Application response timing

TCP retransmissions

Other relevant transport behavior
```

### Important Rule

Do not assume the network is responsible for the delay.

The investigation must determine where the visible delay occurs.

### Completion Condition

You can identify the stage where the delay appears and support the finding with timestamps and packet evidence.

## Challenge 8: Repeated DNS Behavior

### Scenario

A workstation generates a large number of DNS queries.

You are asked:

```text
"Determine whether the DNS activity requires further investigation."
```

### Tasks

Determine:

```text
Which client generated the traffic?

Which names were queried?

How frequently were they queried?

Were queries repeated?

Were responses successful?

Did the behavior occur periodically?

Did other network communication correlate with the DNS activity?
```

### Important Rule

Do not classify the traffic as malicious based only on:

```text
unusual domain
long domain
high query frequency
```

Document the evidence first.

### Completion Condition

You can describe the behavior and identify whether additional investigation would be reasonable based on the observed evidence.

## Challenge 9: TCP Performance

### Scenario

A file transfer appears to stall intermittently.

### Tasks

Investigate:

```text
TCP streams

retransmissions

duplicate ACKs

out-of-order packets

window behavior

timing

connection resets

application behavior
```

### Key Question

Determine whether the observed TCP behavior correlates with the reported stall.

A TCP event elsewhere in the capture may be unrelated.

### Completion Condition

Your conclusion explicitly connects the observed transport behavior to the affected transaction or explains why the evidence does not establish that relationship.

## Challenge 10: HTTP Failure

### Scenario

A web application sometimes returns an error.

You are given a PCAP containing multiple HTTP transactions.

### Tasks

Determine:

```text
Which client experienced the error?

Which server handled the request?

Which URI was requested?

Which HTTP status was returned?

Did the same request succeed elsewhere?

Did the failure correlate with TCP or TLS problems?

When did the failure occur?
```

### Completion Condition

You can isolate the failed transaction and compare it with a successful transaction when the capture provides one.

## Stage 3: Open Investigation

At this stage, the problem description is intentionally vague.

You decide what to investigate.

## Challenge 11: Unknown Network Problem

### Scenario

You receive this report:

```text
"Something was wrong with the network around 14:32."
```

You receive:

```text
network-event.pcapng
```

No protocol or host is specified.

### Your Task

Determine what happened.

Start by establishing:

```text
capture scope
protocols
endpoints
traffic volume
important conversations
```

Then identify the most meaningful anomaly or failure.

### Deliverable

Produce an investigation record:

```text
Problem statement:

Initial observations:

First question:

Investigation path:

Important evidence:

Timeline:

Alternative explanations:

Conclusion:

Limitations:

Recommended next question:
```

The goal is not to find a predetermined answer.

The goal is to demonstrate sound investigation.

## Challenge 12: Intermittent Connectivity

### Scenario

The report says:

```text
"Connections sometimes work and sometimes fail."
```

You receive an authorized capture containing multiple connection attempts.

### Task

Determine:

```text
What distinguishes successful attempts from unsuccessful attempts?
```

Possible dimensions include:

```text
time
client
server
port
TCP handshake
retransmissions
resets
application response
```

Do not assume the cause before comparing successful and failed transactions.

### Completion Condition

You can explain the observable differences between the two groups of transactions.

## Challenge 13: Unexpected Communication

### Scenario

A workstation has generated communication that the analyst did not expect.

No protocol is specified.

### Task

Determine:

```text
Which external/internal endpoints were contacted?

Which protocols were used?

When did communication occur?

Which conversations are relevant?

What application-level metadata is visible?

What additional evidence is required before drawing a security conclusion?
```

### Important Rule

An unfamiliar endpoint is not automatically malicious.

The capture should establish behavior, not unsupported attribution.

## Challenge 14: Multi-Protocol Failure

### Scenario

A user reports:

```text
"The application does not load."
```

The capture contains:

```text
DNS
TCP
TLS
application traffic
```

### Task

Determine the first stage at which expected behavior breaks.

Use:

```text
DNS
→ TCP
→ TLS
→ application
```

as a conceptual sequence.

### Completion Condition

Your report identifies:

```text
completed stages
failed/incomplete stage
evidence
limitations
```

## Challenge 15: Traffic Spike

### Scenario

A network capture contains a short period of unusually high traffic.

### Task

Determine:

```text
When the spike occurred

Which endpoints contributed

Which protocols contributed

Which conversations dominated

Whether the spike correlates with another event
```

### Useful Capabilities

Possible starting points include:

```text
I/O statistics
protocol hierarchy
endpoints
conversations
timestamps
```

You decide the final workflow.

## Stage 4: Independent Investigation

These challenges provide only a problem statement.

You decide everything else.

## Challenge 16: Independent Connectivity Investigation

### Problem

```text
A user reports that a service was unreachable during a specific period.
```

You receive:

```text
connectivity.pcapng
```

### Requirements

Determine:

```text
What service was involved

Which client was affected

Whether connection attempts occurred

What happened during those attempts

Whether the failure is visible in the capture

What evidence supports the conclusion

What the capture cannot establish
```

No filter sequence is provided.

No command sequence is provided.

No expected answer is provided.

### Required Deliverable

```text
Investigation Question

Capture Scope

Relevant Hosts

Relevant Conversations

Timeline

Evidence

Observation

Interpretation

Conclusion

Limitations
```

## Challenge 17: Independent Performance Investigation

### Problem

```text
Users report that an application became slow during part of the day.
```

You receive:

```text
application-performance.pcapng
```

### Requirements

Determine:

```text
When the performance problem occurred

Which clients were affected

Which server/service was involved

Where the visible delay occurred

Whether transport-level problems correlate with the delay

Whether the evidence supports a network-related explanation
```

Do not begin with the assumption:

```text
"The network is slow."
```

Test the problem from the packet evidence.

## Challenge 18: Independent DNS Investigation

### Problem

```text
Investigate unusual DNS activity observed from a workstation.
```

You receive:

```text
dns-investigation.pcapng
```

### Requirements

Determine:

```text
Which hosts generated DNS traffic

Which DNS servers were contacted

Which names were queried

Query frequency

Response behavior

Timing

Correlated communication

Evidence requiring additional investigation
```

Do not make a maliciousness determination solely from packet appearance.

Your task is to characterize the behavior and identify evidence-supported next steps.

## Challenge 19: Independent Security Investigation

### Problem

```text
A security team wants to understand whether a workstation
was communicating with unexpected systems.
```

You receive:

```text
security-investigation.pcapng
```

### Requirements

Determine:

```text
Major communicating endpoints

Important conversations

Protocols involved

Relevant time periods

Repeated communication

Application metadata where visible

Potentially noteworthy behavior

Evidence limitations
```

The investigation should clearly distinguish:

```text
Observed network behavior
```

from:

```text
Security interpretation
```

## Challenge 20: Final Open Investigation

### Problem

You receive an unfamiliar authorized capture.

You are told only:

```text
"Analyze this traffic and report anything important."
```

There is no predefined protocol.

There is no predefined issue.

There is no filter list.

There is no expected answer.

### Required Process

You must decide:

```text
What to inspect first

What the initial question should be

Which Wireshark capabilities to use

What evidence matters

What to investigate next

When enough evidence has been collected
```

### Required Report

```text
# Investigation Report

## Scope

Capture:

Analysis period:

Tools:

## Initial Orientation

Protocols:

Endpoints:

Major conversations:

## Investigation Questions

1.

2.

3.

## Evidence

Frame(s):

Timestamp(s):

Source(s):

Destination(s):

Protocol(s):

Stream(s):

## Timeline

## Findings

## Interpretation

## Limitations

## Conclusion

## Possible Follow-Up
```

The report should not attempt to explain every packet.

It should focus on the most meaningful evidence.

## Progressive Difficulty Model

The difficulty should increase through decision-making, not through arbitrary packet count.

```text
Stage 1
Known problem
Known protocol
Explicit tasks

↓

Stage 2
Known problem
Multiple possible paths
Reduced guidance

↓

Stage 3
Vague problem
Unknown protocol
Analyst chooses investigation path

↓

Stage 4
Open problem
No predefined workflow
Analyst owns the investigation
```

The final stage should feel different from the first.

If you still require a command-by-command walkthrough, continue practicing the earlier stages.

## How to Evaluate Yourself

Do not judge success only by whether you reached the expected answer.

Evaluate the process.

### Question Quality

Did you ask questions that could actually be answered from the capture?

### Tool Selection

Did you choose an appropriate Wireshark capability?

### Investigation Efficiency

Did you narrow the problem progressively?

### Evidence Quality

Did you preserve the relevant frames, timestamps, streams, and fields?

### Reasoning

Did you distinguish observation from interpretation?

### Adaptability

Did you change direction when the evidence contradicted your initial hypothesis?

### Uncertainty

Did you identify what the capture could not establish?

### Reproducibility

Could another analyst follow your investigation?

## Self-Assessment Matrix

Use the following categories:

| Area               | Needs Practice           | Developing                 | Independent                                            |
| ------------------ | ------------------------ | -------------------------- | ------------------------------------------------------ |
| Question formation | Needs explicit questions | Can refine vague questions | Creates precise questions independently                |
| Tool selection     | Requires instructions    | Usually selects correctly  | Selects efficiently                                    |
| Filtering          | Copies known filters     | Modifies filters           | Builds filters from questions                          |
| Correlation        | Focuses on packets       | Uses endpoints/streams     | Builds timelines across protocols                      |
| Reasoning          | Jumps to conclusions     | Usually separates evidence | Consistently distinguishes evidence and interpretation |
| Troubleshooting    | Follows procedures       | Adapts procedures          | Creates investigation paths                            |
| Documentation      | Records results          | Records useful evidence    | Produces reproducible investigations                   |
| Uncertainty        | Often overlooks limits   | Identifies obvious limits  | Explicitly accounts for capture visibility             |
| Independence       | Needs walkthroughs       | Needs occasional guidance  | Can investigate unfamiliar captures                    |

Use this matrix to identify areas for additional practice.

It is not a score.

## Rules for Independent Challenges

During the later challenges:

```text
Do not:
```

```text
- Search for a predetermined answer.
- Assume the reported cause is correct.
- Treat unusual traffic as automatically malicious.
- Treat missing packets as proof that an event never happened.
- Collect every available field without purpose.
- Continue investigating after the question is adequately answered.
```

Instead:

```text
Do:
```

```text
- Start with orientation.
- Define the question.
- Test hypotheses.
- Follow evidence.
- Narrow progressively.
- Preserve important context.
- Document uncertainty.
- Reproduce important findings.
```

## Challenge Completion Criteria

The challenge set is complete when you can independently perform the following:

```text
[ ] Orient yourself in an unfamiliar capture
[ ] Identify useful protocols
[ ] Identify relevant endpoints
[ ] Identify conversations
[ ] Define packet-level questions from vague problems
[ ] Select appropriate Wireshark capabilities
[ ] Build filters without a predefined list
[ ] Isolate relevant streams
[ ] Build timelines
[ ] Correlate multiple protocols
[ ] Investigate connectivity failures
[ ] Investigate DNS behavior
[ ] Investigate TCP behavior
[ ] Investigate application traffic
[ ] Investigate encrypted traffic metadata
[ ] Investigate performance problems
[ ] Investigate unusual communication
[ ] Test alternative explanations
[ ] Document limitations
[ ] Produce reproducible evidence
[ ] Decide when an investigation is complete
```

## Final Principle

The progression is successful when you stop thinking:

```text
"What command should I run?"
```

and start thinking:

```text
"What do I need to know,
what evidence would answer it,
and what is the most efficient way to obtain that evidence?"
```

The ultimate skill is not command recall.

It is independent packet-analysis judgment:

```text
Problem
→ Question
→ Evidence
→ Decision
→ Investigation
→ Adaptation
→ Conclusion
```
