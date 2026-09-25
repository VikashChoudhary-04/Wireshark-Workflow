# PCAP Workflow

## Objective

A PCAP is not simply a file containing packets.

It is a record of captured network evidence that must be:

* Opened correctly
* Validated
* Understood in context
* Preserved appropriately
* Analyzed systematically
* Documented with its limitations

The goal of this file is to establish a repeatable workflow for working with existing packet captures.

The core model is:

```text
PCAP
 ↓
Context
 ↓
Validation
 ↓
Scope
 ↓
Initial overview
 ↓
Question-driven analysis
 ↓
Evidence extraction
 ↓
Correlation
 ↓
Conclusion
 ↓
Documentation
```

## What a PCAP Represents

A packet capture records traffic visible to the capture mechanism at a particular location and time.

It does not automatically represent:

* Everything that happened on the network
* Every packet transmitted by every host
* The complete application state
* The entire communication path
* Events outside the capture point

Therefore:

```text
PCAP evidence
    ≠
Complete reality
```

A PCAP is evidence from a particular observation point.

## Why Capture Context Matters

Before analyzing an unfamiliar PCAP, determine what is known about it.

Useful context includes:

* Why it was captured
* Who or what was captured
* Capture location
* Capture interface
* Approximate time
* Expected activity
* Reproduction steps
* Capture filters
* Known limitations

Without context, interpretation becomes more uncertain.

## First Rule: Preserve the Original

When a PCAP is important, preserve the original file.

Use:

```text
Original
   ↓
Preserved copy
   ↓
Working analysis
```

Avoid repeatedly overwriting the original during investigation.

For important investigations, maintain a clear distinction between:

```text
Original evidence
```

and:

```text
Working analysis artifacts
```

## PCAP File Naming

Use meaningful filenames.

For example:

```text
2026-09-25_dns-failure_client01.pcapng
```

A useful naming pattern is:

```text
<date>_<purpose>_<target>.<format>
```

Avoid ambiguous names such as:

```text
capture.pcapng
test2.pcapng
final.pcapng
newcapture.pcapng
```

Good naming improves investigation continuity.

## Record PCAP Metadata

Before analysis, record available information such as:

```text
File:
Purpose:
Capture date/time:
Capture location:
Capture interface:
Capture filter:
Expected activity:
Source:
Known limitations:
```

If some information is unavailable, record that as unknown.

Do not invent missing context.

## Opening a PCAP

Open the capture in Wireshark.

Before applying filters, inspect the overall state.

Look at:

* Packet count
* Time range
* Protocol column
* Source and destination
* Packet lengths
* General traffic patterns

Do not immediately jump to the first suspicious-looking packet.

Start by understanding the capture.

## Validate That the PCAP Is Usable

Ask:

```text
Does the file open correctly?

Are packets present?

Are timestamps available?

Are protocol layers being decoded?

Is the packet count plausible?

Does the capture appear complete?

Is the expected traffic present?
```

A PCAP that opens successfully is not necessarily a complete or useful capture.

## Initial PCAP Triage

Perform a short first-pass review.

Use:

```text
1. Open PCAP
2. Check packet count
3. Check time range
4. Identify protocols
5. Identify major endpoints
6. Look for obvious conversations
7. Identify potential events
8. Define the next question
```

The objective is orientation, not complete analysis.

## Determine the Time Window

Identify:

* First packet time
* Last packet time
* Approximate duration
* Significant events within the window

Ask:

```text
What happened during this period?
```

If the capture covers only a few seconds, it may represent one event.

If it covers hours, the investigation strategy may need to change.

## Packet Count

Packet count gives an initial sense of capture size.

It can help determine whether the capture is:

* Very small
* Moderate
* Large
* Extremely large

Do not assume a large packet count means a complex incident.

A single high-volume application can generate many packets.

## Identify Protocols

Establish which protocols appear in the capture.

This gives you a first understanding of the traffic.

For example:

```text
Ethernet
IPv4
IPv6
ARP
TCP
UDP
DNS
TLS
HTTP
ICMP
```

The exact set depends on the capture.

Use protocol presence to guide your next questions.

## Protocol Hierarchy

Wireshark provides protocol statistics that can help you understand the composition of the capture.

Use this to answer:

```text
Which protocol families dominate the capture?

Which protocols are present?

Is the traffic mostly TCP, UDP, or something else?

Are there unexpected protocols?
```

Protocol hierarchy is an orientation tool.

It is not automatically an explanation of the traffic.

## Identify Endpoints

Determine which systems communicate in the capture.

Look for:

* IP addresses
* IPv6 addresses
* MAC addresses
* Relevant hostnames when available

Ask:

```text
Which hosts generate the most traffic?

Which hosts communicate with each other?

Which hosts are relevant to the investigation?
```

Do not assume that the host generating the most packets is the most important host.

## Identify Conversations

Endpoints tell you who exists.

Conversations tell you who communicated with whom.

Use conversation information to identify:

* Source
* Destination
* Protocol
* Ports
* Packet counts
* Byte counts
* Duration when available

This can quickly reduce a large capture to a smaller set of meaningful interactions.

## Initial Questions

After opening a PCAP, ask:

```text
What is this capture about?

Which hosts matter?

Which protocols matter?

Which conversations matter?

What event am I looking for?

What evidence would confirm it?

What evidence would contradict it?
```

This prevents random packet inspection.

## Do Not Begin With a Giant Filter

A common mistake is immediately constructing a complicated display filter.

Instead:

```text
PCAP
 ↓
Understand scope
 ↓
Identify likely target
 ↓
Ask focused question
 ↓
Build filter
```

Filtering should support analysis, not replace thinking.

## Use Broad-to-Narrow Analysis

Start broad.

Then narrow.

```text
Entire PCAP
    ↓
Relevant protocol
    ↓
Relevant host
    ↓
Relevant conversation
    ↓
Relevant packet
    ↓
Relevant field
```

This approach preserves context while reducing complexity.

## Display Filters as Analysis Tools

Display filters are reversible views over the capture.

They allow you to ask focused questions without modifying the underlying PCAP.

Examples of question types include:

```text
Which packets involve this host?

Which traffic uses this protocol?

Which packets belong to this conversation?

Where are TCP retransmissions?

Which DNS queries were made?

Which HTTP responses returned errors?
```

The syntax will be developed in the next section of the repository.

Here the focus is workflow.

## Save Your Analysis State Carefully

When an investigation becomes complex, record important filters or observations.

Do not rely entirely on memory.

Record:

```text
Filter:
Purpose:
Result:
Important packets:
```

This makes the investigation reproducible.

## Packet Marking

When a packet becomes important, consider marking it or otherwise recording its packet number.

Examples:

```text
Packet 142:
TCP connection attempt

Packet 184:
DNS response

Packet 217:
Unexpected reset
```

The exact annotation method can depend on the investigation.

The principle is:

```text
Important evidence
    ↓
Stable reference
```

## Packet Comments

When appropriate, packet comments can help preserve investigation context.

For example:

```text
"First observed failed connection attempt."
```

Keep comments factual.

Prefer:

```text
TCP SYN sent to test server; no response observed in subsequent packets.
```

over:

```text
Server is definitely down.
```

The first records evidence.

The second is a conclusion that may require more evidence.

## Follow the Communication Story

A useful PCAP workflow is to reconstruct events.

For example:

```text
DNS query
    ↓
DNS response
    ↓
TCP connection
    ↓
TLS handshake
    ↓
Application request
    ↓
Application response
```

Or:

```text
ARP request
    ↓
ARP response
    ↓
TCP SYN
    ↓
TCP SYN/ACK
    ↓
TCP ACK
    ↓
Application traffic
```

This turns packets into a sequence of events.

## PCAP Timeline

A timeline can be more useful than a packet list.

Think in terms of:

```text
Time
 ↓
Event
 ↓
Response
 ↓
Next event
```

For example:

```text
10:00:01 DNS query
10:00:01 DNS response
10:00:01 TCP SYN
10:00:01 TCP SYN/ACK
10:00:02 TLS handshake
10:00:05 Application response
```

The exact interpretation depends on the capture.

The important point is to correlate timestamps.

## Establishing a Baseline

If a PCAP contains normal and abnormal behavior, compare them.

For example:

```text
Normal transaction
        ↓
Compare
        ↓
Abnormal transaction
```

Compare:

* Protocol sequence
* Packet timing
* Response codes
* TCP behavior
* Retransmissions
* Connection duration

A baseline can reveal meaningful differences.

## PCAP and Reproduction

When a PCAP represents a reproduced problem, document the reproduction.

For example:

```text
Action:
Attempted connection to authorized test service.

Expected:
Successful application response.

Observed:
Connection established but application response delayed.

Capture:
Contains complete client-side transaction.
```

This connects the packet evidence to the real-world event.

## PCAP Limitations

Every PCAP has limitations.

Possible limitations include:

* Wrong capture interface
* Short capture duration
* Packet loss
* Capture filtering
* Truncated packets
* Encryption
* Missing remote-side traffic
* NAT
* VPN encapsulation
* Timestamp uncertainty
* Unsupported or incomplete dissection
* Capture performed at only one network location

Document limitations explicitly.

## Missing Packets

Missing packets can occur for multiple reasons.

Possible explanations include:

* Capture loss
* Capture point limitations
* Capture filtering
* Network behavior
* Interface limitations
* Hardware or operating-system limitations

Do not automatically conclude:

```text
The packet was never transmitted.
```

A safer statement may be:

```text
The packet was not observed in this capture.
```

## Capture Point Reasoning

Consider where the PCAP was collected.

For example:

```text
Client
 ↓
Router
 ↓
Firewall
 ↓
Server
```

A client-side capture and server-side capture can provide different evidence.

Therefore:

```text
Not observed here
```

does not automatically mean:

```text
Did not happen anywhere.
```

## Encryption

Encrypted traffic limits what can be directly inspected.

However, useful metadata may still exist.

Depending on the protocol and capture, you may be able to analyze:

* Endpoints
* Ports
* Timing
* Handshake behavior
* TLS metadata
* Certificate information
* Packet sizes
* Connection patterns

Do not claim application content from encrypted packets unless it is actually available through authorized decryption or another valid source.

## Truncated Packets

If the captured length is less than the original packet length, the packet may be truncated.

This can affect:

* Payload analysis
* Application protocol decoding
* Stream reconstruction
* Raw-byte inspection

Always distinguish:

```text
Unavailable from capture
```

from:

```text
Absent from original traffic
```

## Malformed Packets

Malformed indications should be investigated rather than immediately treated as malicious activity.

Possible explanations include:

* Genuine malformed traffic
* Truncation
* Capture corruption
* Protocol variation
* Dissector limitations

Inspect:

```text
Packet structure
Raw bytes
Related packets
Capture completeness
```

before making conclusions.

## Name Resolution

Name resolution can make an analysis easier to read.

However, when documenting evidence, preserve underlying addresses where relevant.

For example:

```text
Observed IP:
192.0.2.10

Resolved name:
example.test
```

Treat the name and address as different pieces of information.

## PCAP Analysis Notes

Use a structured note format.

```text
PCAP:
Purpose:
Capture point:
Time range:
Relevant hosts:
Relevant protocols:
Relevant conversations:

Observation:
Evidence:
Interpretation:
Limitation:
Next question:
```

This prevents analysis from becoming an unstructured collection of impressions.

## Evidence vs Interpretation

Keep these separate.

### Evidence

What the PCAP directly shows.

Example:

```text
TCP SYN sent from client to server.
No corresponding SYN/ACK was observed in the relevant capture window.
```

### Interpretation

What that evidence may mean.

Example:

```text
The connection attempt did not appear to receive a TCP response at this capture point.
```

### Conclusion

What you can reasonably determine after considering additional evidence.

Example:

```text
The client-side capture does not show completion of the TCP handshake.
The capture alone does not establish why the response was absent.
```

This structure avoids overclaiming.

## PCAP Investigation Workflow

Use this repeatable sequence:

```text
1. Preserve the original
2. Record available context
3. Open the PCAP
4. Validate the file
5. Establish the time window
6. Review protocol composition
7. Review endpoints
8. Review conversations
9. Define the investigation question
10. Apply focused display filters
11. Inspect candidate packets
12. Correlate related traffic
13. Analyze timing
14. Record evidence
15. Record limitations
16. Form a conclusion
17. Document the result
```

## Practical Exercise 1: PCAP Triage

Open an unfamiliar authorized PCAP.

Without reading individual packets in depth, determine:

```text
Packet count:
Time range:
Major protocols:
Major endpoints:
Major conversations:
Potentially interesting traffic:
```

Then write one investigation question.

Example:

```text
Which host initiated the largest number of external connections?
```

The goal is to practice orientation.

## Practical Exercise 2: Protocol Inventory

Use protocol statistics to identify the protocols present.

Create a simple inventory:

```text
Protocol:
Approximate significance:
Why it matters:
```

Do not attempt to explain every protocol.

Focus on those relevant to the capture.

## Practical Exercise 3: Endpoint Inventory

Identify the major endpoints.

For each relevant endpoint, record:

```text
Address:
Observed role:
Major communication partners:
Relevant protocols:
```

Avoid assigning a role based only on the address.

Use actual traffic evidence.

## Practical Exercise 4: Conversation Inventory

Identify several significant conversations.

Record:

```text
Source:
Destination:
Protocol:
Ports:
Approximate volume:
Duration when available:
Reason it may matter:
```

Then choose one conversation for deeper analysis.

## Practical Exercise 5: Reconstruct One Transaction

Choose a simple transaction.

For example:

```text
DNS query
→ DNS response
```

or:

```text
TCP SYN
→ SYN/ACK
→ ACK
```

or:

```text
HTTP request
→ HTTP response
```

Find the packets and reconstruct the sequence.

Record the packet numbers.

## Practical Exercise 6: Timeline Reconstruction

Choose one event.

Create a timeline:

```text
Time:
Event:
Packet:
Evidence:
```

Then determine:

```text
What happened first?
What happened next?
What was the response?
Where was the delay?
```

## Practical Exercise 7: Baseline Comparison

If the PCAP contains normal and abnormal behavior, identify both.

Compare:

```text
Normal:
Protocol sequence
Timing
Response

Abnormal:
Protocol sequence
Timing
Response
```

Then identify one concrete difference.

Do not immediately explain why the difference occurred.

## Practical Exercise 8: Capture Limitation Analysis

Choose a PCAP with a known limitation or determine one from inspection.

Record:

```text
Limitation:
Evidence of limitation:
Effect on investigation:
What cannot be concluded:
```

This develops evidence discipline.

## Practical Exercise 9: Evidence Trail

Choose an important event.

Create:

```text
Event:
    ↓
Packet number
    ↓
Relevant field
    ↓
Related packet
    ↓
Supporting evidence
    ↓
Interpretation
```

The goal is to make the conclusion traceable.

## Practical Exercise 10: Complete PCAP Investigation

Start with only:

```text
Analyze this PCAP.
```

Do not use a predefined answer.

Perform:

```text
Initial triage
    ↓
Endpoint review
    ↓
Protocol review
    ↓
Conversation review
    ↓
Question selection
    ↓
Filtering
    ↓
Packet inspection
    ↓
Correlation
    ↓
Timeline
    ↓
Evidence
    ↓
Conclusion
```

Document:

```text
Primary finding:
Supporting packets:
Supporting fields:
Important timeline:
Limitations:
Next investigation step:
```

## PCAP Quality Checklist

Before trusting your analysis:

```text
[ ] Original PCAP preserved
[ ] Capture context recorded
[ ] File opens correctly
[ ] Packet count reviewed
[ ] Time range reviewed
[ ] Protocols reviewed
[ ] Endpoints reviewed
[ ] Conversations reviewed
[ ] Investigation question defined
[ ] Relevant packets identified
[ ] Related packets correlated
[ ] Timing considered
[ ] Capture limitations considered
[ ] Evidence separated from interpretation
[ ] Conclusion supported by evidence
```

## What You Should Be Able to Do

After completing this file, you should be able to:

* Open and validate an unfamiliar PCAP
* Establish capture context
* Understand the time window
* Identify major protocols
* Identify important endpoints
* Identify meaningful conversations
* Narrow a large capture progressively
* Reconstruct simple transactions
* Build timelines
* Compare normal and abnormal traffic
* Recognize capture limitations
* Distinguish missing evidence from evidence of absence
* Preserve important evidence
* Record packet-level references
* Separate observation from interpretation
* Produce a traceable conclusion

## Completion Criteria

You are ready to continue when you can take an unfamiliar PCAP and independently perform:

```text
PCAP
 ↓
Triage
 ↓
Scope
 ↓
Question
 ↓
Filter
 ↓
Inspect
 ↓
Correlate
 ↓
Timeline
 ↓
Evidence
 ↓
Conclusion
```

You should no longer open a PCAP and immediately ask:

```text
"Which filter should I use?"
```

Instead, you should first ask:

```text
"What am I trying to determine?"
```

Then use Wireshark to obtain the evidence needed to answer that question.

## Professional Standard

A professional PCAP workflow is not about finding the most interesting packet.

It is about constructing a defensible chain:

```text
Question
    ↓
Relevant evidence
    ↓
Packet-level observations
    ↓
Correlation
    ↓
Interpretation
    ↓
Supported conclusion
```

The strongest analysis is not the one with the most filters.

It is the one where another analyst can follow the evidence and understand why the conclusion was reached.
