# Independent Analysis Decision Framework

## Purpose

This file begins the transition from guided Wireshark learning to independent packet analysis.

Up to this point, the workflow has been taught through explicit procedures:

```text
Question
→ Filter
→ Inspect
→ Extract
→ Correlate
→ Interpret
```

Independent analysis requires a different skill:

```text
Question
→ Decide what evidence is needed
→ Choose the right Wireshark capability
→ Investigate
→ Adapt based on observations
→ Reach a defensible conclusion
```

The objective is no longer to remember which command comes next.

The objective is to know **how to decide what to do next**.

Use only authorized captures, laboratory traffic, supplied PCAPs, or traffic you are permitted to analyze.

## The Independent Analyst Mental Model

An independent analyst should be able to begin with a vague technical problem and progressively turn it into observable packet-level questions.

For example:

```text
"The application is slow."
```

should become:

```text
When did the problem occur?
        ↓
Which client was affected?
        ↓
Which server was contacted?
        ↓
Did DNS resolution complete?
        ↓
Did TCP establish normally?
        ↓
Did TLS establish?
        ↓
Was application traffic exchanged?
        ↓
Where did delay appear?
        ↓
What evidence supports that conclusion?
```

This is the central transition:

```text
Tool knowledge
        ↓
Workflow knowledge
        ↓
Decision-making
        ↓
Independent investigation
```

## The Seven-Step Decision Loop

Use this loop whenever you do not know what to investigate next.

```text
1. Define the question
2. Identify the observable evidence
3. Choose the least expensive useful analysis
4. Inspect the result
5. Update the hypothesis
6. Narrow or change the investigation
7. Record the evidence
```

Then repeat as necessary.

### Step 1: Define the Question

Write the question in observable terms.

Weak:

```text
"Why is the network broken?"
```

Better:

```text
"Does the client successfully establish a TCP connection with the server?"
```

Better still:

```text
"Does 10.10.10.20 complete the TCP handshake with 10.10.10.50:443 during the reported failure window?"
```

The more precisely the question can be answered from packets, the easier the investigation becomes.

## Step 2: Identify Observable Evidence

Ask:

```text
What would I expect to see if the event occurred?
```

For TCP connection establishment:

```text
SYN
SYN-ACK
ACK
```

For DNS resolution:

```text
DNS query
DNS response
response status
```

For HTTP:

```text
request
response
status code
```

For TLS:

```text
handshake
server response
alerts if present
```

For packet loss or delivery problems:

```text
retransmissions
duplicate ACKs
out-of-order packets
timing changes
```

The goal is to connect:

```text
Question
→ Observable packet evidence
```

## Step 3: Choose the Least Expensive Useful Analysis

Do not immediately inspect every packet in detail.

Choose the smallest analysis that can answer the current question.

For example:

```text
Question:
Which hosts communicated?

Use:
Endpoint statistics
```

```text
Question:
Which TCP conversations exist?

Use:
Conversation statistics
```

```text
Question:
Which packets were retransmitted?

Use:
Display filter
```

```text
Question:
What exactly happened in one connection?

Use:
TCP stream isolation
```

```text
Question:
What bytes were actually transmitted?

Use:
Packet bytes
```

The principle is:

```text
Start broad enough to orient yourself.
Then become progressively narrower.
```

## Step 4: Inspect the Result

Never treat the first command result as the final answer.

Ask:

```text
What did I actually observe?

What surprised me?

What is missing?

What assumptions did I make?

What new question does this result create?
```

Example:

```text
Filter:
tcp.analysis.retransmission
```

Observation:

```text
Several retransmissions occurred.
```

That does not immediately answer:

```text
Why did they occur?
```

The next investigation might require:

```text
stream
→ timing
→ sequence numbers
→ acknowledgments
→ direction
→ surrounding packets
```

## Step 5: Update the Hypothesis

Packet analysis is iterative.

Start with a working hypothesis:

```text
"The application may be experiencing network delay."
```

Then gather evidence.

Suppose you observe:

```text
DNS completes quickly.
TCP handshake completes quickly.
TLS handshake completes quickly.
Application response is delayed.
```

The investigation should change.

The evidence does not support spending all your time on DNS.

Instead:

```text
Move toward application-level timing.
```

The hypothesis should adapt to the evidence.

## Step 6: Narrow or Change the Investigation

A good analyst knows when to stop following an unproductive path.

Example:

```text
Question:
Is DNS causing the application delay?

Observation:
DNS response arrives immediately.

Decision:
DNS is unlikely to explain the observed delay.

Next question:
Where does delay appear after connection establishment?
```

This is more valuable than collecting additional unrelated DNS packets.

## Step 7: Record the Evidence

Important findings should be reproducible.

Record:

```text
Capture
Frame
Timestamp
Source
Destination
Protocol
Port
Stream
Filter
Observation
Interpretation
Limitation
```

The exact fields depend on the investigation.

## The Evidence Hierarchy

Not every observation has the same evidentiary value.

A useful hierarchy is:

```text
Raw packet bytes
        ↓
Decoded protocol fields
        ↓
Packet relationships
        ↓
Conversation behavior
        ↓
Timing patterns
        ↓
Interpretation
        ↓
Conclusion
```

The further down the chain you go, the more reasoning is involved.

For example:

```text
Observed:
TCP RST packet.

Interpretation:
The connection was reset at that point.

Conclusion:
Requires additional context.
```

Do not silently transform an observation into a broader conclusion.

## Observation vs Interpretation

This distinction is fundamental.

### Observation

Something directly visible in the capture.

Examples:

```text
A DNS query was observed.

A TCP SYN was observed.

A SYN-ACK was not observed in the visible capture.

A retransmission was identified.

An HTTP 500 response was observed.
```

### Interpretation

What the observation may mean.

Examples:

```text
The client attempted to establish a connection.

The server response was not visible.

The TCP exchange experienced retransmission.

The application returned an error response.
```

### Conclusion

A broader statement supported by multiple observations.

Example:

```text
The capture provides evidence that the client experienced
application-level failures after the TCP connection was established.
```

Conclusions should be based on correlated evidence rather than one isolated packet.

## Negative Evidence

One of the hardest concepts in packet analysis is understanding absence.

Suppose you observe:

```text
SYN
```

but no:

```text
SYN-ACK
```

You can state:

```text
"No SYN-ACK is visible in this capture."
```

You should not automatically state:

```text
"The server never sent a SYN-ACK."
```

The capture may be:

* taken from the wrong location
* incomplete
* asymmetric
* filtered
* affected by packet loss
* limited to one network segment

Use precise language:

```text
visible in the capture
```

rather than making claims about events outside the capture's visibility.

## The Capture-Visibility Question

Before making a strong conclusion, ask:

```text
Where was the packet captured?
```

Possible locations include:

```text
client host
server host
network tap
switch mirror/SPAN port
gateway
firewall
wireless capture point
remote capture system
```

The same event can look different from different capture points.

Example:

```text
Client
  ↓
Switch
  ↓
Firewall
  ↓
Server
```

A capture taken at the client may show:

```text
SYN
```

without showing what happened beyond the client's visibility.

A capture taken at the server may provide different evidence.

Capture location affects what can be concluded.

## Decide What You Know

For every investigation, divide information into three categories.

### Known

Directly supported by the capture.

```text
TCP SYN was observed.
DNS query was observed.
HTTP 404 response was observed.
```

### Probable

Supported by multiple observations but not directly proven.

```text
The client likely reached the server.
```

### Unknown

The capture cannot establish the answer.

```text
Whether a firewall dropped a packet outside the capture point.
```

This prevents overconfidence.

## Build an Evidence Table

A simple table can organize an investigation.

| Question                  | Evidence           | Observation                             | Interpretation                            | Confidence       |
| ------------------------- | ------------------ | --------------------------------------- | ----------------------------------------- | ---------------- |
| Did DNS occur?            | DNS query/response | Query and response visible              | Name resolution occurred                  | High             |
| Did TCP establish?        | SYN/SYN-ACK/ACK    | Complete handshake visible              | Connection established                    | High             |
| Did TLS start?            | TLS handshake      | ClientHello and server response visible | TLS negotiation began                     | High             |
| Was the application slow? | timestamps         | Large response delay observed           | Application transaction experienced delay | Requires context |

Confidence should describe the evidence supporting the statement, not become a score or ranking of competing explanations.

## The Question Ladder

When analyzing an unfamiliar capture, progress through increasingly specific questions.

### Level 1: What Exists?

```text
What protocols are present?
What hosts are present?
What conversations exist?
```

Useful tools:

```text
Protocol Hierarchy
Endpoints
Conversations
```

### Level 2: What Happened?

```text
Which hosts communicated?
Which protocols were used?
Which connections were established?
```

### Level 3: When Did It Happen?

```text
What happened first?
When did the event begin?
How long did it last?
```

### Level 4: Where Did It Change?

```text
Where did the expected sequence break?
```

### Level 5: Why Might It Have Happened?

```text
What evidence explains the observed behavior?
```

### Level 6: What Can Be Concluded?

```text
What does the evidence actually support?
```

Do not jump directly from Level 1 to Level 6.

## The Investigation Funnel

A practical independent-analysis funnel is:

```text
Entire capture
      ↓
Protocol
      ↓
Endpoint
      ↓
Conversation
      ↓
Stream
      ↓
Packet
      ↓
Field
      ↓
Bytes
```

Use the narrowest level necessary.

Example:

```text
Entire capture
→ TCP
→ client/server pair
→ TCP stream 8
→ retransmission
→ sequence/ACK relationship
```

You do not need to inspect every byte of every packet.

## Choosing the Right Wireshark Capability

When unsure what to use, classify the question.

### "Who?"

Use:

```text
Endpoints
Conversations
Address filters
```

### "What?"

Use:

```text
Protocol hierarchy
Packet dissection
Display filters
```

### "When?"

Use:

```text
Packet timestamps
I/O graphs
Time-relative analysis
```

### "How?"

Use:

```text
Packet details
TCP sequence/ACK analysis
Protocol fields
Stream following
```

### "How much?"

Use:

```text
Statistics
I/O graphs
Packet/byte counts
Conversation statistics
```

### "Where did it fail?"

Use:

```text
Timeline
Conversation analysis
Protocol sequence
Expert information
TCP analysis
```

### "What exactly was transmitted?"

Use:

```text
Packet bytes
Stream reconstruction
Protocol fields
```

This decision framework reduces random tool usage.

## Build a Hypothesis Tree

For ambiguous problems, list plausible explanations.

Example:

```text
Application is slow
│
├── DNS delay
│
├── TCP connection delay
│
├── TLS handshake delay
│
├── Server response delay
│
├── Retransmissions
│
├── Window limitations
│
└── Capture/measurement limitation
```

Then test each branch using packet evidence.

Do not assume the first plausible explanation is correct.

## Example: Connectivity Failure

Problem:

```text
"The client cannot connect to the service."
```

Independent investigation:

```text
1. Identify client and server.
2. Identify the expected protocol/port.
3. Find connection attempts.
4. Inspect SYN packets.
5. Look for SYN-ACK.
6. Look for ACK.
7. Look for RST.
8. Check timing.
9. Check retransmissions.
10. Determine where the visible sequence changes.
```

Possible observations:

```text
SYN only
```

Possible interpretation:

```text
No response is visible at the capture point.
```

Another:

```text
SYN
SYN-ACK
ACK
RST
```

Possible interpretation:

```text
The connection established and was subsequently reset.
```

The next question depends on the evidence.

## Example: Slow Application

Problem:

```text
"The web application is slow."
```

Investigation:

```text
1. Identify client.
2. Identify server.
3. Identify DNS activity.
4. Measure DNS timing.
5. Identify TCP handshake.
6. Measure connection establishment.
7. Identify TLS handshake if applicable.
8. Measure TLS timing.
9. Identify application request.
10. Measure request-to-response timing.
11. Check retransmissions and duplicate ACKs.
12. Correlate the slow transaction with surrounding traffic.
```

The objective is to locate the delay.

```text
DNS
→ TCP
→ TLS
→ application request
→ server response
```

## Example: Suspicious DNS Activity

Problem:

```text
"Investigate unusual DNS behavior."
```

Start broad:

```text
DNS protocol
```

Then:

```text
clients
→ queried names
→ frequency
→ response codes
→ timing
→ repeated behavior
```

Then ask:

```text
Does the behavior correlate with another communication?
```

For example:

```text
DNS query
→ response
→ TCP connection
→ TLS handshake
```

The correlation may provide useful context.

Do not label traffic malicious solely from a domain name, query frequency, or unusual-looking string.

## Example: TCP Performance Problem

Problem:

```text
"The application intermittently stalls."
```

Investigate:

```text
TCP stream
→ retransmissions
→ duplicate ACKs
→ out-of-order packets
→ zero-window behavior
→ timing
→ resets
→ application-layer timing
```

Then determine whether the observed TCP behavior actually correlates with the application stall.

A TCP event outside the affected transaction may be irrelevant.

## Time as a Decision Tool

When the problem is intermittent, time becomes critical.

Create a timeline:

```text
T0:
DNS query

T1:
DNS response

T2:
TCP SYN

T3:
SYN-ACK

T4:
ACK

T5:
TLS handshake

T6:
Application request

T7:
Delayed response
```

Then calculate or inspect the gaps.

Ask:

```text
Where does the largest unexpected delay occur?
```

That question is often more useful than simply asking:

```text
"Is the network slow?"
```

## Correlation Rules

When correlating events, use multiple anchors.

Strong anchors include:

```text
timestamp
source IP
destination IP
source port
destination port
TCP stream
frame number
protocol
```

For example:

```text
DNS response
+
same client
+
same destination IP
+
subsequent TCP connection
```

is stronger evidence of a relationship than:

```text
DNS response
+
another connection several seconds later
```

Correlation should be demonstrated, not assumed.

## The "What Would Prove This?" Question

When you think you have an explanation, ask:

```text
What packet evidence would prove or disprove this explanation?
```

Example:

```text
Hypothesis:
The connection failed because the server never responded.
```

Evidence to examine:

```text
SYN
SYN-ACK
capture location
retransmissions
timing
```

Another:

```text
Hypothesis:
The application is slow because of packet loss.
```

Evidence:

```text
retransmissions
duplicate ACKs
out-of-order packets
timing
stream behavior
```

This prevents confirmation bias.

## The "What Else Could Explain It?" Question

After finding evidence supporting an explanation, ask:

```text
What else could produce the same observation?
```

Example:

```text
Observation:
No response is visible.
```

Possible explanations include:

```text
server did not respond
firewall dropped traffic
capture point did not observe response
packet was lost during capture
capture was filtered
routing path was outside visibility
```

This does not mean all explanations are equally likely.

It means the analyst should understand the limits of the evidence.

## The Stop Condition

Independent analysis can become inefficient if there is no clear stopping point.

Stop when:

```text
The original question has been answered
AND
the evidence is sufficient
AND
important uncertainty is documented.
```

Do not continue collecting unrelated packet details simply because more information exists.

Example:

```text
Question:
Did the client complete the TCP handshake?

Evidence:
SYN
SYN-ACK
ACK

Conclusion:
Yes, the handshake completed.

No need to inspect unrelated DNS packets unless another question requires them.
```

## When to Ask a New Question

Create a new question when:

```text
the current evidence is ambiguous
a new event changes the investigation
the original hypothesis is weakened
the packet sequence reveals a different problem
the current question has been answered
```

Example:

```text
Question:
Is DNS causing the delay?

Result:
DNS completes immediately.

New question:
Where does the delay appear after DNS?
```

This is how independent analysis progresses.

## Evidence Before Explanation

Use this sequence:

```text
Observation
→ Correlation
→ Interpretation
→ Conclusion
```

Avoid:

```text
Conclusion
→ Search for supporting packets
```

The second approach increases the risk of confirmation bias.

## The Independence Test

An analyst should be able to receive a new capture and answer:

```text
What is here?

What is important?

What question should I ask first?

Which Wireshark feature should I use?

What does the evidence show?

What should I investigate next?

What can I safely conclude?
```

without requiring a predefined command sequence.

That is the purpose of this section.

## Practical Exercise 1: Unknown Capture

Take an authorized PCAP without reading its description first.

Do not start with a specific protocol.

Perform only:

```text
1. Capture metadata
2. Protocol hierarchy
3. Endpoints
4. Conversations
```

Then write:

```text
What is this capture?

What systems are communicating?

Which protocols dominate?

What appears unusual or important?

What should be investigated next?
```

Do not attempt to explain every packet.

The objective is orientation.

## Practical Exercise 2: Choose the Next Action

For each question, decide what Wireshark capability should be used before running any command.

```text
Question A:
Which hosts communicated?

Question B:
Which TCP stream contains a reset?

Question C:
When did traffic spike?

Question D:
What DNS names were queried?

Question E:
What exact bytes were transmitted?

Question F:
Which protocols dominate the capture?
```

Your answer should map the question to an appropriate capability.

Example:

```text
Which hosts communicated?
→ Endpoints / Conversations
```

The exercise is about decision-making, not command memorization.

## Practical Exercise 3: Hypothesis Testing

Use an authorized capture containing a connectivity problem.

Start with:

```text
Hypothesis:
The client cannot connect because the server does not respond.
```

Test the hypothesis.

Record:

```text
Expected evidence:

Observed evidence:

Alternative explanation:

Next question:

Final interpretation:
```

The goal is to practice changing your investigation when the evidence disagrees with your initial assumption.

## Practical Exercise 4: Find the Break in a Sequence

Choose a protocol sequence such as:

```text
DNS
→ TCP
→ TLS
→ application
```

Determine:

```text
Which stages completed?

Which stage did not complete?

Where does the expected sequence change?

What evidence supports the finding?
```

Do not assume the failure is at the first unusual packet.

Look for the first meaningful break in the expected sequence.

## Practical Exercise 5: Independent Investigation

Take an unfamiliar authorized capture.

Do not use a predefined walkthrough.

Your process should be:

```text
1. Orient yourself.
2. Form an initial question.
3. Select the appropriate Wireshark capability.
4. Investigate.
5. Record observations.
6. Update your hypothesis.
7. Ask the next question.
8. Repeat.
9. Stop when the question is answered.
10. Document limitations.
```

Write a final investigation record containing:

```text
Capture

Initial question

Investigation path

Important evidence

Observations

Interpretation

Conclusion

Uncertainty

Next possible investigation
```

## Decision Checklist

Before choosing your next action, ask:

```text
[ ] What exact question am I answering?
[ ] What evidence would answer it?
[ ] What Wireshark capability exposes that evidence?
[ ] Can I start with a smaller analysis?
[ ] What did the previous result actually show?
[ ] What assumptions am I making?
[ ] What alternative explanations exist?
[ ] What should I investigate next?
[ ] Can another analyst reproduce the finding?
```

## Independent Analysis Checklist

You are becoming independent when you can:

```text
[ ] Start from a problem rather than a command
[ ] Convert vague problems into packet-level questions
[ ] Identify observable evidence
[ ] Choose an appropriate Wireshark feature
[ ] Start broad and narrow progressively
[ ] Use evidence to change the investigation direction
[ ] Distinguish observations from interpretations
[ ] Handle negative evidence carefully
[ ] Account for capture visibility
[ ] Build and test hypotheses
[ ] Correlate events using multiple anchors
[ ] Identify alternative explanations
[ ] Know when enough evidence has been collected
[ ] Document the investigation path
[ ] Reproduce important findings
```

## Core Principle

Independent packet analysis is not:

```text
"I know every Wireshark filter."
```

It is:

```text
"I know how to decide what evidence I need next."
```

The mature workflow is:

```text
Problem
  ↓
Question
  ↓
Evidence
  ↓
Wireshark capability
  ↓
Observation
  ↓
Correlation
  ↓
Interpretation
  ↓
Alternative explanations
  ↓
Conclusion
  ↓
Next question
```

Once this decision loop becomes natural, unfamiliar captures stop being collections of packets and become structured investigations.
