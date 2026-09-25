# Wireshark Mental Model

## Objective

Build the correct mental model for using Wireshark before learning individual features, filters, or protocols.

After completing this file, you should understand:

* What Wireshark is actually used for
* What a packet capture represents
* The difference between capturing traffic and analyzing traffic
* How packets become evidence
* How to begin an investigation with a question
* How to move from a question to relevant packets
* The difference between observation, interpretation, and conclusion
* When Wireshark cannot answer a question
* Why professional packet analysis is an iterative process

## What Wireshark Actually Does

Wireshark captures and analyzes network traffic.

At its simplest:

```text
Network Traffic
↓
Packets
↓
Wireshark
↓
Decoded Protocol Information
↓
Human Analysis
```

Wireshark does not automatically tell you why a network problem occurred.

It gives you packet-level evidence that you can use to investigate the problem.

Think of Wireshark as an instrument for observing network communication.

It is not the investigator itself.

## The Packet as Evidence

A packet contains information about network communication at a particular point in time.

Depending on the traffic, that information can include:

* Source and destination addresses
* Protocols
* Ports
* Packet length
* Timestamps
* Protocol fields
* Flags
* Sequence information
* Application metadata
* Application data when it is visible

Wireshark decodes this information into a structure that can be inspected.

A simplified view is:

```text
Packet
│
├── Frame
│
├── Ethernet
│
├── IP
│
├── TCP / UDP / Other Transport
│
└── Application Protocol
```

The exact layers depend on the traffic.

The important idea is:

> **A packet is evidence of an observed network event.**

## The Core Investigation Question

Do not begin packet analysis by randomly clicking through packets.

Begin with:

> **What question am I trying to answer?**

Examples:

```text
Why can this host not connect to the server?
```

```text
Why is this application slow?
```

```text
Which hosts communicated with this system?
```

```text
Which DNS server answered this query?
```

```text
Did the TCP connection succeed?
```

```text
What happened immediately before the connection failed?
```

```text
Does this traffic support the hypothesis that scanning occurred?
```

The question determines what evidence matters.

## The Core Workflow

The fundamental Wireshark workflow used throughout this repository is:

```text
Question
↓
Required Evidence
↓
Capture / PCAP
↓
Relevant Traffic
↓
Filter / Analysis Feature
↓
Packet Evidence
↓
Interpretation
↓
Decision
↓
Next Question
```

Do not reverse this process unnecessarily.

Avoid:

```text
Open Wireshark
↓
Look at random packets
↓
Click random menus
↓
Apply random filters
↓
Guess what happened
```

Instead:

```text
Define the question
↓
Determine what evidence could answer it
↓
Find that evidence
↓
Analyze it
```

## Question → Evidence

Every investigation should connect the question to evidence.

### Example

Question:

> Why did the client fail to connect to the server?

Potential evidence:

```text
DNS query
↓
DNS response
↓
TCP SYN
↓
SYN/ACK
↓
ACK
↓
Application traffic
```

If the SYN is repeatedly sent but no SYN/ACK returns, that is relevant evidence.

If the TCP handshake succeeds but the application does not respond, the investigation moves to a different layer.

The question determines what you need to inspect.

## Question → Wireshark Capability

Different questions require different Wireshark capabilities.

| Question                                     | Useful Capability            |
| -------------------------------------------- | ---------------------------- |
| What protocols exist?                        | Protocol Hierarchy           |
| Which hosts communicate?                     | Endpoints                    |
| Which conversations matter?                  | Conversations                |
| Which packets are relevant?                  | Display Filters              |
| What happened in one connection?             | Follow Stream                |
| Why is communication failing?                | Packet / TCP / ICMP analysis |
| Why is traffic slow?                         | Timing / I/O analysis        |
| What errors are visible?                     | Expert Information           |
| What exactly is inside this packet?          | Packet Details               |
| What bytes produced this field?              | Packet Bytes                 |
| Can this analysis be repeated automatically? | TShark                       |

The goal is not to memorize this table.

The goal is to learn the reasoning behind it.

## Capture vs Analysis

There are two fundamentally different situations.

### Situation 1 — You Need to Capture Traffic

You control the capture process.

You must decide:

* Which interface?
* What traffic?
* When should capture start?
* When should it stop?
* Should a capture filter be used?
* How much traffic should be collected?
* How should the capture be preserved?

The first decision is therefore:

> **What traffic do I need to observe?**

### Situation 2 — You Already Have a PCAP

The traffic has already been captured.

You generally do not control what was collected.

Your task becomes:

```text
Understand the capture
↓
Scope the traffic
↓
Ask questions
↓
Filter
↓
Analyze
↓
Extract evidence
```

This distinction becomes important throughout the repository.

## Capture Filters vs Display Filters

These two concepts must remain separate in your mind.

### Capture Filter

A capture filter controls what traffic is collected during capture.

Conceptually:

```text
Network Traffic
↓
Capture Filter
↓
Captured Packets
```

Traffic excluded by the capture filter is not available in that capture.

Therefore, capture filters should be used deliberately.

### Display Filter

A display filter controls which packets are currently shown during analysis.

Conceptually:

```text
Captured Packets
↓
Display Filter
↓
Displayed Packets
```

The underlying capture remains available.

Therefore, display filters are generally the primary tool for narrowing an existing capture during investigation.

The practical decision is:

> **Do I want to prevent traffic from being captured, or do I already have the traffic and simply want to focus on part of it?**

That question should become automatic.

## Observation vs Interpretation vs Conclusion

Professional analysis requires discipline.

### Observation

Something directly supported by the traffic.

Example:

```text
The client sent three TCP SYN packets to the server.
No SYN/ACK packet is visible in the capture.
```

### Interpretation

What that observation may indicate.

Example:

```text
The client did not receive a TCP response during the observed exchange.
```

### Conclusion

A conclusion supported by the available evidence.

Example:

```text
The supplied capture does not show successful TCP connection establishment.
```

Notice the difference.

The capture does **not automatically prove** why the SYN/ACK was absent.

Possible explanations could include:

* The server did not respond.
* The response traveled through a path not included in the capture.
* The capture point could not observe the response.
* A filter excluded relevant traffic.
* The capture ended before the response occurred.
* Another network condition prevented the response.

Therefore:

> **Do not claim more than the evidence establishes.**

## Evidence Has Limits

A packet capture represents what was visible from a particular capture point during a particular period.

That creates limitations.

For example:

```text
You see a client SYN.
You do not see a SYN/ACK.
```

You can say:

> The capture does not contain a visible SYN/ACK corresponding to the observed SYN.

You should not automatically say:

> The server never sent a SYN/ACK.

The capture may not contain all traffic.

This distinction is fundamental to professional packet analysis.

## What Wireshark Can and Cannot Tell You

Wireshark can provide strong evidence about observed network behavior.

For example, it can help establish:

* Which hosts communicated
* Which protocols were observed
* Which ports were involved
* When packets were observed
* What TCP flags were present
* Whether retransmissions occurred
* Whether a DNS response was returned
* What HTTP information is visible
* What TLS metadata is visible
* How traffic was sequenced
* How much traffic was exchanged
* What packet-level anomalies are visible

But Wireshark cannot automatically establish everything.

For example, packet evidence alone may not prove:

* The identity of a human user
* The intent behind traffic
* Why an application made a particular decision
* What happened on a host outside the captured traffic
* The contents of properly encrypted application data without appropriate decryption context
* Whether an unusual pattern is malicious without additional context

When Wireshark cannot answer the question, identify the additional evidence required.

Possible sources include:

* Host logs
* Application logs
* Server logs
* Authentication logs
* Firewall logs
* DNS logs
* Endpoint telemetry
* Configuration data
* Additional packet captures

A professional analyst knows when to stop asking the packet capture to answer a question it cannot answer.

## The Iterative Investigation Model

Packet analysis rarely follows one straight path.

Instead:

```text
Question
↓
Evidence
↓
Observation
↓
New Question
↓
More Evidence
↓
New Observation
↓
Refined Hypothesis
↓
Additional Analysis
```

For example:

```text
Question:
Why is the application slow?

↓
Observation:
DNS response arrives quickly.

↓
New Question:
Is TCP establishment delayed?

↓
Observation:
TCP handshake is fast.

↓
New Question:
Is TLS establishment delayed?

↓
Observation:
TLS handshake takes significantly longer.

↓
New Question:
Is the delay visible inside the network traffic,
or do we need additional application/server evidence?
```

The investigation changes direction as evidence is discovered.

## The "What Next?" Principle

After every meaningful observation, ask:

> **What does this tell me, and what should I investigate next?**

Example:

```text
You identified the client.
↓
What next?
```

Possible answers:

```text
Need the destination?
→ Inspect conversations/endpoints.

Need to know whether the connection succeeded?
→ Inspect TCP handshake.

Need to understand application behavior?
→ Follow the stream.

Need to understand timing?
→ Inspect timestamps and timing statistics.

Need DNS context?
→ Investigate DNS traffic.

Need broader context?
→ Inspect protocol hierarchy or statistics.
```

The exact path depends on the problem.

## Do Not Chase Packets Without a Question

A large PCAP may contain:

```text
100,000+
500,000+
1,000,000+
```

packets.

You cannot effectively analyze a large capture by reading every packet sequentially.

Instead:

```text
Question
↓
Scope
↓
Filter
↓
Relevant packets
```

The purpose of filtering is therefore not:

> "Make the packet list look smaller."

The purpose is:

> **Reduce the available evidence to the evidence relevant to the current question.**

## A Practical Example

Suppose a user reports:

> "The website is slow."

Do not immediately inspect every HTTP packet.

Start by breaking the problem into observable stages:

```text
Client
↓
DNS
↓
TCP connection
↓
TLS handshake
↓
HTTP/application request
↓
Server response
↓
Data transfer
```

Then investigate each stage.

For example:

```text
DNS fast?
    ↓
Yes

TCP connection fast?
    ↓
Yes

TLS handshake slow?
    ↓
Investigate TLS timing

If not:
    ↓
Inspect application request/response timing
```

The workflow is driven by the question.

## Another Example

Suppose:

> "The host cannot connect to the server."

You might investigate:

```text
Name resolution
↓
Address resolution where relevant
↓
Routing-related evidence
↓
TCP SYN
↓
SYN/ACK
↓
ACK
↓
Application exchange
```

If the TCP handshake never completes, there may be little value in immediately investigating application-layer behavior.

The packets tell you where to investigate next.

## The Professional Mental Model

When you open Wireshark, think:

```text
I have a question.
        ↓
What evidence would answer it?
        ↓
Where is that evidence?
        ↓
How can I isolate it?
        ↓
What do the packets actually show?
        ↓
What does that evidence mean?
        ↓
What can I prove?
        ↓
What remains uncertain?
        ↓
What should I investigate next?
```

This should eventually become automatic.

## Practical Exercise

Use any small, authorized PCAP or a capture generated in your own lab.

Do not worry about advanced filtering yet.

Open the capture and answer only these questions:

### Question 1

What is the capture?

Determine:

* Approximate packet count
* Capture duration
* Whether multiple protocols are present
* Whether multiple hosts are present

### Question 2

What can you observe?

Identify at least:

* One source
* One destination
* One protocol
* One packet field that provides useful evidence

### Question 3

What can you prove?

Write one statement based directly on the packets.

Use this format:

```text
Observation:
...

Evidence:
...

Interpretation:
...

Uncertainty:
...
```

Do not attempt to explain everything in the capture.

The objective is to practice evidence-based thinking.

## Decision Point

Before moving forward, you should be able to answer:

### If I receive a PCAP and no instructions are provided, what do I do first?

You should think:

```text
Understand the problem
↓
Understand the capture
↓
Establish an initial overview
↓
Identify relevant hosts/protocols
↓
Define the next question
```

### If I see something unusual, should I immediately call it malicious?

No.

First determine:

```text
What exactly did I observe?
↓
What does it indicate?
↓
What alternative explanations exist?
↓
What additional evidence is required?
```

### If I do not know which Wireshark feature to use?

Return to the question.

Ask:

> **What evidence am I trying to obtain?**

Then choose the feature that provides that evidence.

## Completion Criteria

Do not move forward until you can explain, in your own words:

* What Wireshark is used for
* What a packet capture represents
* The difference between capturing and analyzing traffic
* The difference between capture filters and display filters
* Why analysis should begin with a question
* How a question determines the evidence you need
* The difference between observation and interpretation
* Why a missing packet does not necessarily prove that an event never occurred
* When Wireshark may be unable to answer a question
* Why investigations are iterative
* Why every observation should lead to a next question

Most importantly, you should be able to use this mental model:

```text
Question
↓
Evidence
↓
Analysis
↓
Interpretation
↓
Decision
↓
Next Question
```

That mental model will be used throughout the rest of the repository.
