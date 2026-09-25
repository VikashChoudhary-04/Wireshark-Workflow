# Investigation Completion Criteria

## Purpose

Independent packet analysis requires knowing not only how to investigate, but also when the investigation is complete.

A common mistake is to treat packet analysis as an endless search for more information.

A professional investigation should instead have a clear stopping condition:

```text id="k0m5ae"
Question
→ Evidence
→ Analysis
→ Conclusion
→ Limitations
→ Stop
```

This file defines the criteria for deciding when a Wireshark investigation has collected sufficient evidence to answer its question.

The goal is not to prove everything about the network.

The goal is to answer the defined question as accurately as the available evidence allows.

Use only authorized captures, laboratory traffic, supplied PCAPs, or traffic you are permitted to analyze.

## The Completion Principle

An investigation is complete when:

```text id="2e0z3j"
The original question has been answered
AND
the relevant evidence has been examined
AND
important uncertainty has been identified
AND
the conclusion does not exceed the evidence
```

This does not mean:

```text id="w7t9c5"
every packet was inspected
```

It means:

```text id="r5x0pu"
enough relevant evidence was examined to support the conclusion.
```

## The Four Completion Conditions

Every investigation should satisfy four conditions.

### Condition 1: The Question Is Defined

You should be able to state:

```text id="b4m7sh"
"What exactly was I trying to determine?"
```

Examples:

```text id="j5qf0f"
Did the TCP connection establish?

Where did the application delay occur?

Which client generated the DNS queries?

Did the reported failure occur during the capture?

Which endpoints communicated?
```

If the question cannot be stated clearly, the investigation may not yet have a meaningful stopping point.

## Condition 2: Relevant Evidence Has Been Examined

You should identify the packet evidence required to answer the question.

For example:

```text id="j5p9ko"
Question:
Did the TCP handshake complete?

Relevant evidence:
SYN
SYN-ACK
ACK
```

Or:

```text id="9h8p1m"
Question:
Where did application delay occur?

Relevant evidence:
DNS timing
TCP timing
TLS timing
application request
application response
transport events
```

Do not collect unrelated evidence simply because it is available.

## Condition 3: The Evidence Supports the Interpretation

A conclusion should be traceable to observations.

Example:

```text id="a0z9d3"
Observation:
SYN, SYN-ACK, and ACK were observed.

Interpretation:
The TCP three-way handshake completed.

Conclusion:
The connection was established at the observed capture point.
```

The conclusion is supported by the evidence.

## Condition 4: Important Limitations Are Documented

A technically correct conclusion can become misleading if its limitations are omitted.

Examples:

```text id="p7h1vz"
The capture was taken from the client.

Only one direction of traffic was visible.

Application payload was encrypted.

Packets may have been lost during capture.

The capture began after the reported event.

The capture ended before the transaction completed.
```

State limitations that materially affect the conclusion.

## Define the Original Question

At the beginning of an investigation, write:

```text id="m9m4r8"
Primary Question:
```

For example:

```text id="h0t9z8"
Primary Question:
Did the client successfully connect to the application server?
```

Additional questions can emerge later.

For example:

```text id="c6d7u2"
Primary Question:
Did the connection establish?

Secondary Question:
If it established, where did the transaction fail?
```

Do not allow secondary questions to replace the original objective without documenting the change.

## Primary vs Secondary Questions

Separate questions into:

```text id="g4x2z8"
Primary
```

and:

```text id="p3q5yd"
Secondary
```

Example:

```text id="0s5n0u"
Primary:
Why did the application fail?

Secondary:
Did DNS work?
Did TCP establish?
Did TLS establish?
Was application data exchanged?
```

The secondary questions support the primary investigation.

Once they have answered the primary question sufficiently, further exploration may not be necessary.

## Define the Scope

Before deciding that an investigation is complete, know what was actually examined.

Record:

```text id="8v3z5w"
Capture file
Capture period
Relevant hosts
Relevant protocols
Relevant streams
Relevant time window
```

Example:

```text id="f5x7g9"
Capture:
application-failure.pcapng

Time:
14:30–14:40

Client:
10.10.10.20

Server:
10.10.10.50

Protocol:
TCP/443

Relevant stream:
17
```

This prevents conclusions from being broader than the analysis scope.

## Define the Expected Sequence

Many investigations become easier when you know what should happen.

Example:

```text id="g2w5zq"
DNS query
→ DNS response
→ TCP SYN
→ SYN-ACK
→ ACK
→ TLS handshake
→ application request
→ application response
```

The investigation asks:

```text id="2x1p6w"
Where does the observed sequence differ?
```

If the sequence completes, investigate later stages.

If it breaks, determine whether the available evidence explains the break.

## The First Meaningful Break

A useful completion strategy is to find the first meaningful deviation from expected behavior.

Example:

```text id="k7x3z1"
DNS
✓

TCP
✓

TLS
✓

Application request
✓

Application response
✗
```

The investigation can then focus on the application response stage.

Avoid continuing to analyze unrelated lower-layer traffic after the relevant failure point has been established unless another question requires it.

## Sufficient Evidence vs Maximum Evidence

These are different concepts.

### Maximum Evidence

```text id="r0z5y6"
Inspect every packet.
Extract every field.
Read every byte.
```

This is usually unnecessary.

### Sufficient Evidence

```text id="w2s4j9"
Inspect the evidence required to answer the question.
Validate the important observation.
Document uncertainty.
Stop.
```

Professional analysis aims for sufficient evidence.

## The Evidence Sufficiency Test

Ask:

```text id="x3w7v9"
If another analyst asked "How do you know?",
could I show them the relevant packet evidence?
```

If yes, continue to the next test.

If no, more investigation may be necessary.

Next:

```text id="c8n2m5"
Could a reasonable alternative explanation
still fit the available evidence?
```

If yes, either investigate further or explicitly document the uncertainty.

Finally:

```text id="m5j1z8"
Does my conclusion say more than the capture proves?
```

If yes, narrow the conclusion.

## The Reproduction Test

Important findings should be reproducible.

Ask:

```text id="d8x4v6"
Can I reproduce the finding using:
```

```text id="q9p2s1"
the same capture
the same filter
the same relevant fields
the same frame/stream
```

For TShark:

```bash id="m4x6q8"
tshark -r capture.pcapng -Y "RELEVANT_FILTER"
```

For Wireshark:

```text id="z7c3v5"
same capture
+
same display filter
+
same packet/frame
```

If the finding cannot be reproduced, determine why before treating it as established evidence.

## The Evidence Chain

A completed investigation should have a traceable chain:

```text id="f3g8h2"
Question
  ↓
Relevant traffic
  ↓
Relevant packet(s)
  ↓
Relevant fields
  ↓
Observed behavior
  ↓
Interpretation
  ↓
Conclusion
```

Example:

```text id="w4m8c1"
Question:
Did the client receive a response?

Traffic:
10.10.10.20 → 10.10.10.50:443

Packets:
Frames 100–104

Observation:
SYN, SYN-ACK, ACK visible.

Interpretation:
TCP connection established.

Conclusion:
The client successfully established the TCP connection
during the visible capture period.
```

This is a complete evidence chain.

## When the Answer Is "Unknown"

A professional analyst must be comfortable with:

```text id="q5r8n2"
The capture does not establish the answer.
```

Examples:

```text id="w9x2m4"
Only one side of the communication was captured.

The relevant traffic was outside the capture period.

Payload was encrypted and no decryption material was available.

The capture contains insufficient packets.

The relevant protocol was not successfully dissected.
```

Unknown is not a failed investigation.

If the evidence genuinely cannot answer the question, documenting that limitation is the correct outcome.

## Distinguish "Not Observed" From "Did Not Happen"

This distinction is essential.

Incorrect:

```text id="g3m7p2"
"The server never responded."
```

when only the client side was captured.

More accurate:

```text id="r8k4z1"
"No server response is visible in the capture."
```

Similarly:

```text id="n6p3v5"
"No DNS response is visible."
```

does not automatically prove:

```text id="s2x7q9"
"The DNS server did not respond."
```

Use evidence-aware language.

## Completion in Troubleshooting

For a troubleshooting investigation, completion should identify:

```text id="c4n8w1"
Observed symptom
→ Relevant transaction
→ First meaningful deviation
→ Supporting evidence
→ Likely interpretation
→ Limitations
```

Example:

```text id="m2q6x9"
Symptom:
Application connection fails.

Transaction:
Client → Server TCP/443.

Deviation:
SYN retransmissions occur without a visible SYN-ACK.

Evidence:
Frames 210, 215, 220.

Interpretation:
No server response is visible from this capture point.

Limitation:
The capture does not establish whether the server received the SYN.
```

This is a complete troubleshooting result even though the ultimate cause remains unknown.

## Completion in Performance Analysis

For performance investigations, identify:

```text id="x7v4m3"
Affected transaction
→ timing stages
→ abnormal delay
→ supporting evidence
→ correlation
→ limitation
```

Example:

```text id="n4p8s2"
DNS:
fast

TCP:
fast

TLS:
fast

Application request:
observed

Application response:
large delay

TCP retransmissions:
not correlated with the delay

Conclusion:
The visible delay occurs after the application request,
rather than during DNS, TCP, or TLS establishment.
```

The conclusion should remain within what the capture can demonstrate.

## Completion in Security Investigation

Security-oriented packet analysis requires additional discipline.

Do not treat:

```text id="y2m7c9"
unusual
unexpected
rare
external
high-volume
```

as automatic proof of malicious activity.

A completed security investigation should instead document:

```text id="f8c2n6"
Observed communication
Relevant endpoints
Protocols
Timing
Frequency
Application metadata
Connection behavior
Correlations
Evidence limitations
```

Then identify whether the behavior:

```text id="b4k7m1"
requires additional investigation
```

rather than making unsupported attribution.

## Completion in DNS Investigation

A DNS investigation can be complete when you can answer the relevant question, such as:

```text id="v6x9p3"
Which client queried the name?

Which DNS server responded?

What response was returned?

When did it occur?

Was the query repeated?

Did another communication correlate with it?
```

You do not need to analyze every DNS packet in the capture if the investigation concerns one specific transaction.

## Completion in TCP Investigation

A TCP investigation may be complete when you can establish:

```text id="c9m4x2"
Who initiated the connection

Whether the handshake completed

Whether the connection was reset

Whether retransmissions occurred

Which stream was involved

When the relevant events occurred

What the visible evidence supports
```

Additional TCP details should be examined only if they are necessary to answer the question.

## Completion in HTTP Investigation

For an HTTP transaction, completion may require:

```text id="n8x3v7"
client
server
method
host
URI
status code
timestamp
TCP stream
```

Then ask:

```text id="j2p5m9"
Did the transaction complete?

Was an error returned?

Did the same client perform successful requests?

Did transport-level problems correlate with the failure?
```

Stop when the defined question has sufficient evidence.

## Completion in TLS Investigation

For TLS:

```text id="q7m3x5"
TCP established?
        ↓
TLS handshake visible?
        ↓
Handshake progressed?
        ↓
Alert observed?
        ↓
Application traffic visible?
```

If application traffic is encrypted, explicitly state what cannot be inspected.

Do not continue searching for plaintext application content that the capture cannot provide.

## Completion in TShark Analysis

A TShark investigation should preserve:

```text id="s4x8c2"
Input file
TShark version
Command
Display filter
Fields
Output
Important frames
Interpretation
Limitations
```

Example:

```bash id="a9c5m2"
tshark --version
```

Then:

```bash id="h3v7x1"
tshark -r capture.pcapng \
  -Y "tcp.analysis.retransmission" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e tcp.stream
```

If the output answers the question and the evidence has been validated, the TShark portion may be complete.

## The Stop-or-Continue Decision

At every stage, ask:

```text id="m8q2v4"
Do I have enough evidence?
```

If yes:

```text id="w3x7n1"
Document and stop.
```

If no:

```text id="k6p4y9"
Define the missing evidence.
Ask the next question.
Continue.
```

This creates a controlled investigation loop.

## The Three-Question Stop Test

Before stopping, answer:

### Question 1

```text id="p4n8x6"
Did I answer the original question?
```

### Question 2

```text id="v7m2c9"
Can I show the evidence supporting the answer?
```

### Question 3

```text id="x5q3m8"
Have I documented important uncertainty?
```

If all three are satisfied, the investigation is usually ready for closure.

## When Not to Stop

Continue investigating when:

```text id="f2m7k4"
The evidence is contradictory.

The key packet sequence is incomplete.

The relevant stream has not been identified.

The conclusion depends on an unsupported assumption.

An alternative explanation remains important.

The capture visibility is insufficient to answer the question.

The original question has not actually been answered.
```

Do not stop merely because you found an interesting packet.

## When to Stop Immediately

Stop the current investigation path when:

```text id="n3x8p5"
The question has already been answered.

Additional packets cannot affect the conclusion.

The remaining information is outside the capture.

The investigation has moved into an unrelated question.
```

If a new question is important, document it separately.

## Secondary Questions That Become New Investigations

Suppose the original question is:

```text id="r6m1x9"
Did the TCP connection establish?
```

You establish:

```text id="j4p8w2"
Yes.
```

You then notice:

```text id="c7n3m5"
Multiple retransmissions later.
```

Do not silently turn the original investigation into a full TCP-performance investigation.

Instead:

```text id="v9x2k6"
Primary question:
Did the connection establish?
→ Answered.

Follow-up question:
Did packet loss affect the subsequent transaction?
→ New investigation.
```

This keeps investigations bounded.

## Investigation Closure Record

Use this template when closing an investigation:

```text id="b8q4m1"
# Investigation Closure

## Original Question

## Scope

Capture:

Time period:

Hosts:

Protocols:

Streams:

## Evidence Examined

Frames:

Filters:

Statistics:

Fields:

## Key Observations

## Interpretation

## Conclusion

## Limitations

## Remaining Uncertainty

## Follow-Up Questions
```

Not every field is required for every investigation.

Use the fields that materially support the conclusion.

## Example Closure

```text id="h6m2v8"
# Investigation Closure

## Original Question

Did the client establish a TCP connection to the application server?

## Scope

Capture:
application-failure.pcapng

Client:
10.10.10.20

Server:
10.10.10.50

Protocol:
TCP/443

## Evidence Examined

Frames:
120–126

Stream:
8

## Key Observations

A SYN, SYN-ACK, and ACK were observed in sequence.

## Interpretation

The TCP three-way handshake completed.

## Conclusion

The client successfully established the TCP connection during
the visible capture period.

## Limitations

The capture does not establish what happened before the first
visible packet or after the capture ended.

## Remaining Uncertainty

Application-level success was not established by the TCP handshake alone.

## Follow-Up Questions

Did TLS negotiation complete?
Did the application return a response?
```

Notice that the investigation stops at the scope of its question.

## The "Enough Evidence" Principle

More packets do not automatically mean better analysis.

A useful rule is:

```text id="x9m3v7"
Collect evidence until additional evidence
is unlikely to change the answer.
```

This is especially important for large captures.

Once the relevant transaction is understood, unrelated packets provide diminishing value.

## Avoid Premature Closure

Premature closure occurs when an analyst stops after finding the first plausible explanation.

Example:

```text id="j8p4x2"
Observation:
TCP retransmissions exist.

Premature conclusion:
"Packet loss caused the application problem."
```

A stronger investigation asks:

```text id="c5n7m1"
Did retransmissions occur in the affected transaction?

Did they occur during the reported delay?

Were they significant enough to explain the timing?

Were there other explanations?

Does the capture provide sufficient visibility?
```

Only then should the conclusion be written.

## Avoid Endless Analysis

The opposite problem is endless analysis.

Example:

```text id="v2m8q6"
Question:
Did the TCP handshake complete?

Evidence:
SYN
SYN-ACK
ACK

Conclusion:
Yes.

Unnecessary continuation:
Inspect every HTTP packet, DNS query, ARP packet,
and unrelated TCP stream.
```

Once the defined question is answered, stop.

## The Professional Balance

Good packet analysis balances:

```text id="f7c3m9"
thoroughness
```

with:

```text id="q8n2v5"
scope control
```

The goal is:

```text id="x4m7p1"
Enough evidence
+
appropriate validation
+
clear uncertainty
=
defensible conclusion
```

## Final Independence Criteria

You are ready for the capstone section when you can consistently:

```text id="n7q3m5"
[ ] Define the original investigation question
[ ] Establish the analysis scope
[ ] Identify relevant evidence
[ ] Select appropriate Wireshark capabilities
[ ] Build a logical investigation path
[ ] Recognize the first meaningful break in expected behavior
[ ] Distinguish observation from interpretation
[ ] Test alternative explanations
[ ] Account for capture visibility
[ ] Recognize when evidence is insufficient
[ ] Recognize when enough evidence has been collected
[ ] Avoid premature conclusions
[ ] Avoid unnecessary analysis
[ ] Reproduce important findings
[ ] Document limitations
[ ] Produce a defensible conclusion
[ ] Identify useful follow-up questions
```

## The Final Decision Rule

When deciding whether an investigation is complete, use:

```text id="m5x8c2"
Can I answer the question?

Can I show why?

Can I explain what I cannot know?

If yes:
→ Close the investigation.

If no:
→ Define the missing evidence and continue.
```

This is the final skill of independent packet analysis.

The objective is not to know everything contained in a capture.

The objective is to know **what must be known, what can be known, and when you have enough evidence to stop.**
