# Repeatable Packet Analysis

## Purpose

This file turns TShark analysis into repeatable, consistent, and auditable packet-analysis workflows.

The objective is to move from:

```text
"I found something interesting in Wireshark."
```

to:

```text
"I can reproduce exactly how I found it, extract the relevant evidence,
and explain the result without relying on memory or manual clicking."
```

The central workflow is:

```text
Capture
  ↓
Question
  ↓
Known inputs
  ↓
Repeatable command
  ↓
Structured output
  ↓
Validation
  ↓
Evidence
  ↓
Interpretation
  ↓
Documentation
```

Use this workflow only with captures and traffic you are authorized to analyze.

## Why Repeatability Matters

Manual packet analysis is valuable, but manual actions can be difficult to reproduce.

For example:

```text
Open capture
→ click packet
→ apply filter
→ inspect field
→ scroll
→ compare packets
→ write notes
```

Another analyst may perform the same process differently.

A repeatable workflow instead records:

```text
Input:
capture.pcapng

Question:
Which hosts generated DNS queries?

Filter:
dns.qry.name

Fields:
frame.time
ip.src
ip.dst
dns.qry.name

Output:
structured evidence
```

The result can be rerun later.

This is especially useful for:

* incident investigations
* troubleshooting
* recurring packet analysis
* large captures
* evidence preservation
* peer review
* training
* reporting
* validation of GUI findings

## The Repeatability Model

A good packet-analysis workflow has five properties.

### Defined Input

Know exactly which capture is being analyzed.

```text
capture.pcapng
```

Avoid ambiguous inputs such as:

```text
latest.pcap
test.pcap
new_capture.pcap
```

when the analysis needs to be reproducible.

### Defined Question

Write the question before constructing the command.

Examples:

```text
Which clients contacted the DNS server?

Which TCP streams experienced retransmissions?

Which hosts communicated with the server?

When did the connection failure begin?

Which TLS handshakes occurred?
```

### Defined Filter

Record the exact display filter.

```text
dns.qry.name
```

or:

```text
tcp.analysis.retransmission
```

### Defined Fields

Record exactly what evidence is extracted.

```text
frame.number
frame.time_epoch
ip.src
ip.dst
tcp.stream
```

### Defined Output

Know whether the output is intended for:

```text
human inspection
structured analysis
documentation
comparison
automation
```

## Preserve the Original Capture

Never modify the only copy of an important capture.

Use a workflow such as:

```text
Evidence/
├── original/
│   └── incident.pcapng
├── working/
│   └── analysis-copy.pcapng
├── output/
│   ├── dns.txt
│   ├── tcp.txt
│   └── endpoints.txt
└── notes/
    └── investigation.md
```

The exact directory structure can vary.

The principle is:

```text
Original evidence
        ↓
Working analysis
        ↓
Derived outputs
```

Keep the original capture unchanged.

## Record Capture Metadata

Start every repeatable analysis with metadata.

```bash
capinfos capture.pcapng
```

Record information such as:

```text
capture filename
file format
packet count
capture duration
first packet time
last packet time
encapsulation
```

For investigations where integrity matters, calculate a cryptographic hash of the original file using an appropriate operating-system utility.

For example, on Linux:

```bash
sha256sum capture.pcapng
```

On PowerShell:

```powershell
Get-FileHash .\capture.pcapng -Algorithm SHA256
```

The purpose is to identify the exact input used for the analysis.

## Create an Investigation Question

Do not begin by writing a large command.

Start with:

```text
Question:
What am I trying to determine?
```

Then define:

```text
Known:
What do I already know?

Unknown:
What am I trying to establish?

Evidence:
What packet fields could answer the question?

Scope:
Which protocol, host, stream, or time period matters?
```

Example:

```text
Question:
Did the application server experience TCP retransmissions?

Known:
Client IP is 10.10.10.20.

Unknown:
Whether retransmissions occurred and which stream contained them.

Evidence:
TCP analysis fields, source, destination, stream, timestamp.

Scope:
Traffic involving 10.10.10.20.
```

This makes command construction much easier.

## Build a Minimal Reproducible Command

A command should contain only what is needed for the question.

Example:

```bash
tshark -r capture.pcapng \
  -Y "tcp.analysis.retransmission && ip.addr == 10.10.10.20" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e tcp.stream
```

The command documents:

```text
Input:
capture.pcapng

Filter:
TCP retransmissions involving the host

Output:
frame number
timestamp
source
destination
stream
```

That is much easier to reproduce than a screenshot of a Wireshark window.

## Separate Discovery From Extraction

A useful repeatable workflow has two stages.

### Discovery

Find what matters.

```bash
tshark -r capture.pcapng \
  -Y "tcp.analysis.retransmission"
```

### Extraction

Once relevant traffic is identified, extract structured evidence.

```bash
tshark -r capture.pcapng \
  -Y "tcp.analysis.retransmission" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e tcp.stream
```

This separation reduces unnecessary output.

Think:

```text
Discover
→ Identify
→ Extract
```

rather than:

```text
Print everything
→ search manually
```

## Use Stable Fields

When creating repeatable output, prefer fields that directly represent the evidence you need.

Common fields include:

```text
frame.number
frame.time
frame.time_epoch
ip.src
ip.dst
ip.proto
tcp.srcport
tcp.dstport
tcp.stream
udp.srcport
udp.dstport
```

Protocol-specific fields can then be added.

For DNS:

```text
dns.qry.name
dns.flags.rcode
```

For HTTP:

```text
http.request.method
http.host
http.request.uri
http.response.code
```

For TLS:

```text
TLS fields available in the installed Wireshark version
```

Field availability can change between Wireshark versions.

Check your environment when necessary:

```bash
tshark -G fields
```

## Use Frame Numbers as Anchors

Frame numbers provide a useful bridge between TShark and Wireshark.

Example:

```bash
tshark -r capture.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -e frame.number \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name
```

Output might conceptually look like:

```text
42    12:01:03.100    10.10.10.20    10.10.10.1    example.local
57    12:01:08.412    10.10.10.20    10.10.10.1    api.example.local
```

The frame number can then be used to locate the exact packet in Wireshark.

This creates a useful relationship:

```text
TShark output
     ↓
Frame number
     ↓
Wireshark packet
     ↓
Full packet inspection
```

## Use Stream IDs as Conversation Anchors

Frame numbers identify individual packets.

Stream IDs identify TCP conversations.

Extract them together:

```bash
tshark -r capture.pcapng \
  -Y "tcp" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e tcp.srcport \
  -e tcp.dstport \
  -e tcp.stream
```

Then isolate a stream:

```bash
tshark -r capture.pcapng \
  -Y "tcp.stream == 12"
```

This creates a repeatable transition:

```text
Capture
→ packet
→ stream
→ conversation
```

## Build Reusable Investigation Views

A useful technique is to create standard analysis views.

### Connection View

```bash
tshark -r capture.pcapng \
  -Y "tcp" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e tcp.srcport \
  -e ip.dst \
  -e tcp.dstport \
  -e tcp.stream
```

Purpose:

```text
Identify TCP conversations and their timing.
```

### DNS View

```bash
tshark -r capture.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name
```

Purpose:

```text
Identify DNS requests and their origin.
```

### TCP Problem View

```bash
tshark -r capture.pcapng \
  -Y "tcp.analysis.retransmission || tcp.analysis.duplicate_ack || tcp.analysis.out_of_order || tcp.flags.reset == 1" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e tcp.stream
```

Purpose:

```text
Locate potentially important TCP events.
```

These are analysis views, not universal indicators of problems.

## Build a Timeline

A timeline is often more useful than a long packet list.

Extract:

```bash
tshark -r capture.pcapng \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e tcp.stream \
  -e _ws.col.Protocol \
  -e _ws.col.Info
```

The `_ws.col.*` fields are Wireshark-generated display columns.

Availability can depend on the installed version and protocol dissection.

A timeline can answer:

```text
What happened first?

What happened immediately afterward?

When did the connection become active?

When did an error occur?

Did DNS resolution happen before the application connection?

Did a TCP reset occur after an application request?
```

## Correlate Different Protocols

Repeatable analysis becomes more powerful when multiple protocols are correlated.

Example:

```text
DNS query
    ↓
DNS response
    ↓
TCP connection
    ↓
TLS handshake
    ↓
Application traffic
```

Extract each stage separately.

DNS:

```bash
tshark -r capture.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name
```

TCP:

```bash
tshark -r capture.pcapng \
  -Y "tcp.flags.syn == 1" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e tcp.stream
```

TLS:

```bash
tshark -r capture.pcapng \
  -Y "tls.handshake" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e tcp.stream
```

Then correlate the results using:

```text
time
addresses
ports
stream
protocol
```

Do not assume that packets belong together simply because they occur close in time.

## Repeated DNS Analysis

A repeatable DNS workflow can extract names:

```bash
tshark -r capture.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -e frame.time_epoch \
  -e ip.src \
  -e dns.qry.name
```

Then summarize frequency:

```bash
tshark -r capture.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -e dns.qry.name |
sort |
uniq -c |
sort -nr
```

This can reveal:

```text
high-frequency names
repeated failed lookups
periodic lookups
clients generating unusual query volumes
```

Frequency is an observation.

It is not automatically an indicator of malicious activity.

## Extract Unique Endpoints

For IPv4 traffic:

```bash
tshark -r capture.pcapng \
  -T fields \
  -e ip.src \
  -e ip.dst |
tr '\t' '\n' |
sort -u
```

The exact pipeline depends on the shell.

A statistics-based alternative is:

```bash
tshark -r capture.pcapng -q -z endpoints,ip
```

Prefer the statistics approach when you want Wireshark's endpoint accounting.

Use shell extraction when you specifically need a simple list for another analysis step.

## Extract Unique Destination Ports

For TCP:

```bash
tshark -r capture.pcapng \
  -Y "tcp" \
  -T fields \
  -e tcp.dstport |
sort -u
```

For UDP:

```bash
tshark -r capture.pcapng \
  -Y "udp" \
  -T fields \
  -e udp.dstport |
sort -u
```

Port lists can help establish the communication landscape.

They should not be interpreted as proof of service identity without additional evidence.

## Use Statistics Before Large Extractions

For a large capture, begin with:

```bash
tshark -r large.pcapng -q -z io,phs
```

Then:

```bash
tshark -r large.pcapng -q -z endpoints,ip
```

Then:

```bash
tshark -r large.pcapng -q -z conv,tcp
```

Only after identifying the relevant traffic should you extract packet-level details.

This reduces:

```text
processing effort
output volume
manual review
```

and improves investigative focus.

## Use Time Windows

When you know approximately when an event occurred, narrow the analysis.

For example, use a display filter based on packet timestamps when the relevant fields and syntax are appropriate for your capture.

The important concept is:

```text
Entire capture
        ↓
Relevant time period
        ↓
Relevant protocol
        ↓
Relevant endpoint
        ↓
Relevant stream
```

Do not narrow the capture prematurely if doing so could remove context required to understand the event.

## Use Capture Filters Carefully

Capture filters and display filters serve different purposes.

Capture filter:

```bash
tshark -i 1 -f "tcp port 443"
```

Display filter:

```bash
tshark -r capture.pcapng -Y "tcp.port == 443"
```

The first controls what is captured.

The second controls what is displayed or analyzed from an existing capture.

For repeatable investigations, preserve which type of filter was used.

A missing capture-filter detail can make an investigation difficult to reproduce because traffic may never have been recorded.

## Create Output Files

For important analysis, save outputs rather than relying on terminal history.

Example:

```bash
tshark -r capture.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name \
  > dns_queries.txt
```

Then inspect the file:

```bash
cat dns_queries.txt
```

On PowerShell:

```powershell
Get-Content .\dns_queries.txt
```

The output file becomes part of the derived evidence.

## Use Structured Formats When Appropriate

For structured processing, TShark can produce JSON output.

```bash
tshark -r capture.pcapng \
  -Y "dns" \
  -T json \
  > dns.json
```

It can also produce Elastic-compatible JSON:

```bash
tshark -r capture.pcapng \
  -Y "dns" \
  -T ek \
  > dns-ek.json
```

Use structured output when another tool or script needs to process packet information.

Do not choose JSON simply because it looks more advanced.

For manual review, field output is often easier to understand.

## Repeatability Through Exact Commands

A strong investigation record should preserve:

```text
Tool:
TShark

Version:
installed TShark version

Input:
capture.pcapng

Question:
What was observed?

Command:
exact command used

Filter:
exact display filter

Fields:
exact fields extracted

Output:
saved output filename

Interpretation:
what the evidence indicates

Limitations:
what the capture cannot establish
```

Record the TShark version:

```bash
tshark --version
```

This matters because protocol dissectors, fields, and behavior can change between versions.

## Compare Two Analysis Runs

Repeatability becomes particularly useful when comparing captures.

For example:

```text
Before change
    ↓
capture-before.pcapng

After change
    ↓
capture-after.pcapng
```

Run the same analysis against both.

Example:

```bash
tshark -r capture-before.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name
```

Then:

```bash
tshark -r capture-after.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name
```

Compare:

```text
query names
query frequency
response behavior
timing
endpoints
```

The same workflow is especially useful for troubleshooting before/after changes.

## Build a Small Analysis Notebook

For recurring work, maintain a simple analysis record.

Example:

```text
Investigation: Application Connectivity

Capture:
application-failure.pcapng

TShark version:
5.x.x

Question 1:
Did DNS resolution occur?

Command:
...

Result:
...

Question 2:
Did TCP establish?

Command:
...

Result:
...

Question 3:
Did TLS establish?

Command:
...

Result:
...

Question 4:
Where did the failure occur?

Command:
...

Result:
...

Conclusion:
...

Limitations:
...
```

The exact format can be Markdown, text, or another suitable documentation format.

The important property is traceability.

## Avoid Hidden Manual Steps

A workflow is less reproducible when it depends on undocumented actions.

Weak:

```text
"I clicked around until I found the packet."
```

Stronger:

```text
Applied:
dns.qry.name == "example.local"

Found:
Frame 842

Validated with:
tshark -r capture.pcapng -Y 'dns.qry.name == "example.local"'
```

The second approach allows another analyst to reproduce the observation.

## Reproducibility Does Not Mean Blind Automation

Automation should not replace reasoning.

A command can reliably show:

```text
50 retransmissions
```

but that does not automatically establish:

```text
"The network was congested."
```

A repeatable workflow must preserve the reasoning chain:

```text
Observed:
50 retransmission events.

Correlated:
events occurred within a narrow time period.

Additional evidence:
duplicate ACKs and timing behavior were also observed.

Interpretation:
the TCP exchange experienced packet-delivery problems.

Remaining uncertainty:
capture location does not provide visibility into every network segment.
```

Repeatability makes reasoning auditable.

It does not make an interpretation automatically correct.

## Handling Large Captures

Large captures require deliberate scope.

Start with:

```bash
capinfos large.pcapng
```

Then:

```bash
tshark -r large.pcapng -q -z io,phs
```

Then identify endpoints:

```bash
tshark -r large.pcapng -q -z endpoints,ip
```

Then identify conversations:

```bash
tshark -r large.pcapng -q -z conv,tcp
```

Then narrow:

```text
protocol
→ endpoint
→ port
→ stream
→ event
```

Avoid repeatedly running expensive full-detail output against a huge capture when a narrower query can answer the question.

## When to Use Wireshark GUI

TShark is not a replacement for the GUI.

Use Wireshark when you need:

* visual packet navigation
* packet-tree inspection
* byte-level inspection
* coloring
* interactive filtering
* stream following
* graphical I/O analysis
* rapid exploration
* protocol-field discovery

Use TShark when you need:

* repeatable commands
* structured extraction
* command-line workflows
* large-scale filtering
* saved outputs
* integration with shell tools
* reproducible evidence generation

A professional workflow often uses both.

```text
Wireshark
→ discover and understand

TShark
→ reproduce and extract

Wireshark
→ validate important packet details
```

## Common Mistakes

### Mistake: Over-Automating Too Early

Do not build scripts before you understand the packet-analysis question.

First establish:

```text
question
→ filter
→ fields
→ expected output
```

Then automate repetitive work.

### Mistake: Saving Only the Output

A result without the command that produced it is difficult to reproduce.

Preserve:

```text
input
command
filter
fields
output
```

### Mistake: Using Ambiguous Filenames

Prefer:

```text
dns-queries-2026-09-25.txt
```

over:

```text
output.txt
```

Use naming conventions appropriate to your investigation.

### Mistake: Forgetting the Tool Version

Record:

```bash
tshark --version
```

when reproducibility matters.

### Mistake: Treating Shell Pipelines as Universal

Commands such as:

```text
sort
uniq
grep
awk
tr
```

are environment-specific.

Know what each pipeline stage does and adapt it to the operating system.

### Mistake: Losing Context

Do not reduce an important finding to:

```text
10.10.10.20 → 10.10.10.1
```

Preserve enough context to understand:

```text
when
what protocol
which port
which stream
which frame
what happened
```

## Practical Exercise 1: Create a Repeatable DNS Workflow

Choose an authorized PCAP containing DNS traffic.

First inspect it:

```bash
capinfos capture.pcapng
```

Then identify DNS traffic:

```bash
tshark -r capture.pcapng -Y "dns"
```

Create a structured extraction:

```bash
tshark -r capture.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -E header=y \
  -E separator=, \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name
```

Save it:

```bash
tshark -r capture.pcapng \
  -Y "dns.qry.name" \
  -T fields \
  -E header=y \
  -E separator=, \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name \
  > dns-analysis.csv
```

Then document:

```text
Question
Command
Filter
Fields
Result
Interpretation
Limitations
```

## Practical Exercise 2: Create a Repeatable TCP Workflow

Identify TCP conversations:

```bash
tshark -r capture.pcapng -q -z conv,tcp
```

Find retransmissions:

```bash
tshark -r capture.pcapng \
  -Y "tcp.analysis.retransmission"
```

Extract streams:

```bash
tshark -r capture.pcapng \
  -Y "tcp.analysis.retransmission" \
  -T fields \
  -e frame.number \
  -e frame.time_epoch \
  -e ip.src \
  -e ip.dst \
  -e tcp.stream
```

Choose one relevant stream.

Then:

```bash
tshark -r capture.pcapng \
  -Y "tcp.stream == STREAM_ID"
```

Record:

```text
stream ID
endpoints
ports
timestamps
TCP events
relevant frames
interpretation
limitations
```

## Practical Exercise 3: GUI-to-CLI-to-GUI

Perform the following:

```text
1. Open the capture in Wireshark.
2. Find one meaningful packet.
3. Record its frame number.
4. Identify the relevant display filter.
5. Reproduce the finding in TShark.
6. Extract the key fields.
7. Return to Wireshark.
8. Inspect the complete packet.
9. Compare the CLI result with the GUI observation.
```

The goal is to become comfortable moving between:

```text
GUI exploration
↔
CLI reproduction
```

## Practical Exercise 4: Before-and-After Comparison

Use two authorized captures representing different states.

Examples:

```text
working vs failing
before configuration change vs after
normal traffic vs unusual traffic
```

Run the same TShark workflow against both.

Compare:

```text
endpoints
protocols
conversations
DNS behavior
TCP behavior
timing
errors
```

Do not change the analysis method halfway through unless the investigation question itself changes.

## Practical Exercise 5: Build Your Own Investigation Template

Create a reusable template:

```text
# Packet Analysis Record

## Capture

File:
Hash:
TShark version:
Capture time:

## Investigation Question

Question:

## Known Context

Known:

Unknown:

## Analysis

Command:

Filter:

Fields:

## Evidence

Frame(s):

Timestamp(s):

Source:

Destination:

Protocol:

Port(s):

Stream:

## Interpretation

Observation:

Interpretation:

## Limitations

## Next Question
```

Use this template for future packet-analysis work.

## Professional Evidence Checklist

Before considering an analysis reproducible, verify:

```text
[ ] Original capture identified
[ ] Capture metadata recorded
[ ] Tool version recorded when relevant
[ ] Investigation question written
[ ] Exact filter preserved
[ ] Exact fields preserved
[ ] Important frame numbers recorded
[ ] Stream IDs recorded when relevant
[ ] Relevant output saved
[ ] GUI findings validated where appropriate
[ ] Observation separated from interpretation
[ ] Limitations documented
[ ] Commands can be rerun
```

## Completion Criteria

You are ready to move beyond the core TShark workflow when you can:

* create a repeatable analysis from a capture
* begin with a question rather than a command
* preserve the exact input capture
* record relevant capture metadata
* construct minimal TShark commands
* separate discovery from extraction
* extract stable evidence fields
* use frame numbers as evidence anchors
* use TCP stream IDs for conversation analysis
* build repeatable DNS and TCP workflows
* create structured output
* save derived analysis results
* compare multiple captures using the same workflow
* correlate different protocols
* preserve commands and filters
* distinguish reproducibility from interpretation
* identify limitations in the capture
* move confidently between Wireshark GUI analysis and TShark CLI analysis

The final objective is:

```text
A packet-analysis result should not depend on
"I remember what I clicked."

It should depend on:

the capture
+
the question
+
the filter
+
the fields
+
the evidence
+
the reasoning
```

That is the foundation of repeatable packet analysis.
