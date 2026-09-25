# Capstone Rules

## Purpose

The capstone is the final transition from structured learning to independent Wireshark analysis.

Everything before this section built individual capabilities:

```text
Capture
→ Navigation
→ Filtering
→ Protocol analysis
→ Streams
→ Statistics
→ Troubleshooting
→ Security analysis
→ TShark
→ Independent investigation
```

The capstone combines those capabilities into complete investigations.

The objective is not to demonstrate that you remember every Wireshark feature.

The objective is to demonstrate that you can take an unfamiliar packet-analysis problem and independently:

```text
Understand
→ Question
→ Investigate
→ Correlate
→ Validate
→ Document
→ Conclude
```

Use only authorized captures, laboratory traffic, supplied PCAPs, or traffic you are permitted to analyze.

## What the Capstone Tests

The capstone evaluates five major abilities.

### Investigation

Can you turn a problem statement into packet-level questions?

### Technical Analysis

Can you choose the appropriate Wireshark and TShark capabilities?

### Reasoning

Can you distinguish observations from interpretations?

### Evidence Handling

Can you preserve the relevant packets, timestamps, streams, filters, and statistics?

### Independence

Can you continue an investigation without a predefined command sequence?

## Capstone Philosophy

The capstone is intentionally different from earlier exercises.

Earlier exercises often told you:

```text id="w8y2n4"
Use this filter.
Inspect this field.
Follow this stream.
```

The capstone should increasingly tell you only:

```text id="p5c9m1"
Here is the problem.

Find out what happened.
```

You decide:

```text id="m7x3q8"
what to inspect
which tools to use
which questions to ask
when to narrow the scope
when to change direction
when enough evidence exists
```

That decision-making ability is the main capstone skill.

## Authorized Analysis Only

All capstone work must use traffic that you are authorized to analyze.

Acceptable sources include:

```text id="r4n8v2"
your own systems
your own lab
authorized organizational captures
training PCAPs
generated PCAPs
publicly supplied educational PCAPs
captures explicitly provided for analysis
```

Do not capture or analyze traffic belonging to systems or users without appropriate authorization.

The capstone evaluates packet-analysis ability, not unauthorized access.

## Preserve the Original Evidence

When working with an important capture:

```text id="z6p2k9"
Do not overwrite the original.
```

Use a working copy.

A practical structure is:

```text id="q3m7x8"
capstone/
├── evidence/
│   └── original.pcapng
├── analysis/
├── output/
└── report/
```

The exact structure is optional.

The principle is mandatory:

```text id="v9c4m2"
Original evidence
        ↓
Working analysis
        ↓
Derived evidence
        ↓
Final report
```

## Record the Capture

Before analysis, record the available metadata.

Use:

```bash id="b7n4q2"
capinfos capture.pcapng
```

Record relevant information such as:

```text id="p8x3m5"
capture filename
file format
packet count
capture duration
first packet time
last packet time
encapsulation
```

When evidence integrity matters, record a cryptographic hash of the original file.

For example:

```bash id="x5m9c1"
sha256sum capture.pcapng
```

On PowerShell:

```powershell id="k3q7v8"
Get-FileHash .\capture.pcapng -Algorithm SHA256
```

Also record the Wireshark/TShark version when reproducibility matters:

```bash id="m2p6x4"
tshark --version
```

## Define the Problem

Every capstone begins with a problem statement.

Do not immediately convert the statement into a filter.

First identify:

```text id="h8q4m2"
What is known?

What is unknown?

What is the reported symptom?

What time period matters?

Which systems may be involved?

What would a successful transaction look like?
```

Example:

```text id="f6v2p9"
Reported problem:

"The application became slow around 14:30."

Known:

A client was using the application.

Unknown:

Where the delay occurred.
```

The investigation must discover the rest.

## Define the Primary Question

Convert the problem into a question that packet evidence can answer.

Weak:

```text id="q8m4v1"
"Why is the application slow?"
```

Better:

```text id="c6p2x9"
"Which stage of the visible application transaction contains the reported delay?"
```

Possible stages:

```text id="v4m8q3"
DNS
TCP
TLS
application request
application response
transport behavior
```

The exact question depends on the capstone scenario.

## Secondary Questions

Use secondary questions to support the primary question.

Example:

```text id="n7x3m5"
Primary:
Where does the application delay occur?

Secondary:
Did DNS resolve normally?
Did TCP establish normally?
Did TLS establish normally?
Was the application request transmitted?
When did the response arrive?
Were retransmissions present?
```

Secondary questions should help answer the primary question.

They should not turn the investigation into unrelated packet exploration.

## Start With Orientation

Before deep analysis, establish the capture landscape.

Useful starting points include:

```text id="m5q8v2"
Capture metadata
Protocol hierarchy
Endpoints
Conversations
I/O statistics
```

For TShark:

```bash id="r3x7c9"
tshark -r capture.pcapng -q -z io,phs
```

```bash id="p6m2w8"
tshark -r capture.pcapng -q -z endpoints,ip
```

```bash id="v8q4n1"
tshark -r capture.pcapng -q -z conv,tcp
```

Do not assume these are always the complete starting sequence.

Choose the orientation steps that fit the problem.

## Establish the Relevant Scope

Once you understand the capture, narrow it.

Potential scope dimensions:

```text id="k4x9m7"
time
host
protocol
port
conversation
stream
packet
field
```

The investigation should progressively move from:

```text id="a8m2q5"
Entire capture
```

to:

```text id="x7c3v9"
Relevant transaction
```

Avoid prematurely filtering away context that may later prove important.

## Expected Protocol Sequence

When appropriate, establish the expected sequence before analyzing the actual traffic.

For an HTTPS application:

```text id="n2p7x4"
DNS
→ TCP
→ TLS
→ application
```

For a plain HTTP application:

```text id="q5m8c3"
DNS
→ TCP
→ HTTP
```

For a simple connectivity test:

```text id="v3x6p9"
ARP/neighbor resolution where relevant
→ IP
→ TCP or UDP
→ application
```

The expected sequence provides a reference point.

## Find the First Meaningful Deviation

Do not simply look for the most unusual packet.

Ask:

```text id="j8q4m1"
Where does the expected sequence stop behaving normally?
```

Example:

```text id="z6c2v7"
DNS
✓

TCP
✓

TLS
✓

Application request
✓

Application response
delayed
```

The meaningful investigation point is the application-response stage.

Another example:

```text id="m4x8p2"
DNS query
✓

DNS response
✗

TCP
not observed
```

The investigation may stop at DNS if the primary question is whether the application could resolve its server.

## Evidence Standards

Every important capstone finding should have supporting evidence.

Useful evidence includes:

```text id="c7n3x5"
frame number
timestamp
source
destination
protocol
source port
destination port
stream
relevant protocol field
packet sequence
statistics
```

Not every finding needs every field.

Use the fields that establish the finding.

## Evidence Record

For each important finding, use:

```text id="p4x8m2"
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

This should be sufficient for another analyst to understand how the finding was established.

## Observation vs Interpretation

The capstone requires a strict distinction.

### Observation

Directly supported by packet evidence.

Example:

```text id="v5m9q2"
Frames 200–202 contain a SYN, SYN-ACK, and ACK.
```

### Interpretation

What the observation means.

```text id="x3c7p8"
The TCP three-way handshake completed.
```

### Conclusion

A broader statement that answers the investigation question.

```text id="n8m4v6"
TCP connection establishment was not the source of the
observed application delay.
```

Do not combine these levels without showing the evidence.

## Handle Negative Evidence Carefully

Never make a stronger claim than the capture supports.

Prefer:

```text id="q2m7x4"
"No SYN-ACK is visible in the capture."
```

over:

```text id="c8p3n5"
"The server never sent a SYN-ACK."
```

unless the capture location and visibility justify the stronger statement.

Similarly:

```text id="v4x9m2"
"No HTTP response is visible."
```

does not automatically prove:

```text id="p7n3q8"
"The server did not generate an HTTP response."
```

Document capture limitations.

## Capture Visibility

The capstone must account for where the traffic was captured.

Possible capture points include:

```text id="m8c4x1"
client
server
network tap
switch mirror
gateway
firewall
wireless capture point
remote capture system
```

Ask:

```text id="q6p2v9"
What traffic should this capture be able to see?

What traffic might be outside its visibility?
```

This is especially important when packets appear to be missing.

## Do Not Assume the Reported Cause

If the problem statement says:

```text id="h3m8q5"
"The network is slow."
```

do not treat that as an established fact.

The packet analysis should determine:

```text id="r7x2c4"
Whether delay is visible
Where it occurs
What evidence correlates with it
What the capture cannot establish
```

Similarly:

```text id="k9m4p7"
"The server is down."
```

is a report, not a packet-analysis conclusion.

## Test Alternative Explanations

When you identify a likely explanation, ask:

```text id="x5c8n2"
What else could produce the same observation?
```

Example:

```text id="m2q7v4"
Observation:
No response is visible.
```

Possible explanations:

```text id="p8n3x6"
server did not respond
firewall dropped traffic
capture point did not see response
packet was lost during capture
capture filter excluded traffic
```

Investigate alternatives when they materially affect the conclusion.

## Use the Minimum Necessary Analysis

The capstone is not a contest to use every Wireshark feature.

You may not need:

```text id="n7x4c2"
Expert Information
I/O graphs
TShark
packet bytes
stream following
```

unless they help answer the question.

The correct tool is the one that provides the evidence you need.

## GUI and TShark

Use both when appropriate.

Wireshark GUI is particularly useful for:

```text id="p4m8x7"
interactive exploration
packet-tree inspection
stream following
bytes
visual timing
rapid filtering
```

TShark is particularly useful for:

```text id="x3q6n9"
repeatable extraction
structured output
large captures
saved evidence
command-line analysis
```

A capstone investigation may use either or both.

## When to Use Statistics

Statistics are useful for answering questions such as:

```text id="v7m2c5"
Which endpoints exist?

Which conversations dominate?

Which protocols dominate?

When did traffic increase?

Which hosts exchanged the most data?
```

Examples:

```bash id="q8n4x1"
tshark -r capture.pcapng -q -z endpoints,ip
```

```bash id="m3p7c9"
tshark -r capture.pcapng -q -z conv,tcp
```

```bash id="x6v2n8"
tshark -r capture.pcapng -q -z io,stat,1
```

Use statistics to orient and correlate, not to replace packet-level validation when a precise finding is required.

## When to Follow a Stream

Follow or isolate a stream when the investigation concerns one conversation.

Use:

```text id="c5m9x2"
TCP stream
```

to move from:

```text id="n7q3v8"
many connections
```

to:

```text id="p2m6c4"
one conversation
```

This is especially useful for:

```text id="x8v4q1"
HTTP
TLS
application protocols
TCP troubleshooting
conversation reconstruction
```

## When to Inspect Packet Bytes

Packet bytes are useful when:

```text id="m5x2n7"
the decoded fields are insufficient
you need to validate protocol content
you need to inspect raw data
you suspect malformed or unexpected data
```

Do not inspect bytes simply because the packet contains them.

Use byte-level analysis when it answers a question.

## When to Use Expert Information

Expert Information can help identify:

```text id="q4v8m1"
warnings
errors
unusual protocol behavior
analysis-generated events
```

Treat these as investigation leads.

An expert-information entry is not automatically the root cause of a problem.

Validate important findings with packet evidence.

## Capstone Evidence Hierarchy

Prioritize evidence approximately as follows:

```text id="x6m3p9"
Relevant packet
↓
Relevant protocol fields
↓
Packet relationships
↓
Conversation behavior
↓
Timing
↓
Statistics
↓
Interpretation
```

The exact order can vary.

The important principle is that conclusions should remain traceable to actual packet evidence.

## Capstone Documentation Requirements

Every completed capstone should contain:

```text id="v9c4x2"
1. Problem statement
2. Primary investigation question
3. Scope
4. Initial observations
5. Investigation path
6. Important evidence
7. Timeline where relevant
8. Findings
9. Interpretation
10. Conclusion
11. Limitations
12. Follow-up questions if applicable
```

The report should be concise enough to understand but detailed enough to reproduce important findings.

## Recommended Report Structure

```text id="m2x7q5"
# Capstone Investigation Report

## Problem

## Scope

### Capture

### Time Period

### Hosts

### Protocols

## Primary Question

## Investigation Approach

## Initial Observations

## Investigation Timeline

## Evidence

## Findings

## Interpretation

## Conclusion

## Limitations

## Follow-Up Questions
```

Use only the sections that are relevant to the investigation.

## Completion Criteria

A capstone is complete when:

```text id="p7n3x8"
[ ] The problem was clearly defined
[ ] The primary question was stated
[ ] The capture scope was established
[ ] Relevant traffic was identified
[ ] The investigation followed a logical path
[ ] Important evidence was preserved
[ ] Relevant packets were validated
[ ] Observations were separated from interpretations
[ ] Alternative explanations were considered where necessary
[ ] Capture visibility was considered
[ ] The conclusion answered the primary question
[ ] The conclusion did not exceed the evidence
[ ] Important limitations were documented
[ ] Important findings are reproducible
```

## What the Capstone Is Not

The capstone is not:

```text id="x8m4q2"
A filter memorization test.

A command memorization test.

A packet-count competition.

A test of whether you can explain every packet.

A test of whether you can identify something suspicious
and immediately label it malicious.
```

The capstone is:

```text id="c3p7n9"
A test of independent packet-analysis reasoning.
```

## Professional Standard

A professional-quality analysis should allow another analyst to ask:

```text id="q5x2m8"
What was the question?

What did you examine?

Why did you examine it?

What did you observe?

What does the evidence support?

What remains uncertain?

How can I reproduce the finding?
```

Your report should answer all of these.

## Final Capstone Principle

The strongest capstone result is not:

```text id="v4n8x1"
"I found the problem."
```

It is:

```text id="m7c2p5"
"I can show exactly what I investigated,
what the packets demonstrate,
what they do not demonstrate,
and why my conclusion follows from the evidence."
```

That is the standard for independent Wireshark analysis.
