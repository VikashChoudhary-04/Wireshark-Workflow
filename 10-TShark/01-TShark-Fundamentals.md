# TShark Fundamentals

## Objective

TShark is Wireshark's command-line packet analyzer.

It provides many of the same protocol-dissection capabilities as Wireshark while allowing packet analysis from a terminal.

The purpose of this section is not to replace Wireshark's graphical interface.

The goal is to understand when command-line analysis is useful and how to move between:

```text id="p7x3m9"
Wireshark GUI
      ↕
TShark
      ↕
Repeatable analysis
```

TShark becomes especially useful for:

* Large PCAP files
* Remote systems
* Automation
* Scripting
* Repeatable investigations
* Extracting selected fields
* Batch processing
* Quick protocol statistics
* Headless environments

All packet captures used for analysis should come from systems, labs, or traffic you are authorized to inspect.

## TShark Mental Model

The basic workflow is:

```text id="n4q8v2"
Input
  ↓
Capture or PCAP
  ↓
Packet dissection
  ↓
Filter
  ↓
Field extraction
  ↓
Output
```

For example:

```text id="c6m2r7"
PCAP
 ↓
Display filter
 ↓
Selected packets
 ↓
Selected fields
 ↓
Structured output
```

The important distinction is:

```text id="y8p3k5"
Capture filter
```

versus:

```text id="m4q7v1"
Display filter
```

The same conceptual distinction learned in Wireshark applies to TShark.

## Verify TShark Installation

Start by checking whether TShark is installed:

```text id="q2w6n9"
tshark --version
```

A successful installation should return version and build information.

You can also inspect available command-line help:

```text id="r5m8c3"
tshark --help
```

For a concise option overview:

```text id="v7k4p2"
tshark -h
```

The exact output varies by TShark version.

## Check the TShark Version

Record the version when reproducibility matters:

```text id="x3n9q6"
tshark --version
```

This matters because:

* Protocol dissectors change
* Field names can change
* Output behavior can change
* Supported protocols can change
* Command-line options can differ between versions

For professional investigations, record the tool version alongside the PCAP and analysis notes.

## Read a PCAP

The basic operation is:

```text id="k6m2x8"
tshark -r capture.pcapng
```

This tells TShark to read an existing capture.

The output generally includes packet information such as:

```text id="w4q9n7"
Packet number
Time
Source
Destination
Protocol
Length
Info
```

This is conceptually similar to Wireshark's packet list.

## Limit Output

A large capture can generate enormous terminal output.

Use a display filter:

```text id="p8c3m6"
tshark -r capture.pcapng -Y "dns"
```

This asks TShark to read the capture and display only packets matching the display filter.

## Display Filters in TShark

TShark uses Wireshark display-filter syntax.

Examples:

```text id="y5r8v2"
ip.addr == 192.0.2.10
```

```text id="m7q3k9"
dns
```

```text id="c4n8p6"
tcp.port == 443
```

```text id="x2v7m5"
icmp
```

The same question-driven filter-building approach applies:

```text id="h9k4r1"
Question
  ↓
Filter
  ↓
Observation
```

## `-Y` Display Filter

The `-Y` option applies a display filter.

Example:

```text id="j3p8w6"
tshark -r capture.pcapng -Y "dns"
```

Another:

```text id="n6c2x9"
tshark -r capture.pcapng -Y "ip.addr == 192.0.2.10"
```

This is one of the most important TShark options.

## `-f` Capture Filter

The `-f` option is used for capture filters when capturing live traffic.

Example:

```text id="q8m4v2"
tshark -i eth0 -f "tcp port 443"
```

This is different from:

```text id="w3n7c5"
-Y "tcp.port == 443"
```

Remember:

```text id="k5r2x8"
-f
→ Capture filter

-Y
→ Display filter
```

## Why the Difference Matters

Capture filters determine what is captured.

Display filters determine what is displayed from the captured traffic.

Conceptually:

```text id="v6p3m9"
Live traffic
   ↓
Capture filter
   ↓
Captured packets
   ↓
Display filter
   ↓
Displayed packets
```

A restrictive capture filter can permanently remove traffic from the resulting capture.

A display filter does not remove packets from the original PCAP.

## List Interfaces

To see available capture interfaces:

```text id="m8q4n2"
tshark -D
```

Example output may look conceptually like:

```text id="x7c3v5"
1. Ethernet
2. Wi-Fi
3. Loopback
```

The actual interface names and numbering depend on the system.

## Select an Interface

Use:

```text id="r4m9k2"
tshark -i 1
```

where `1` represents the interface number shown by `tshark -D`.

You can also specify an interface by name when supported:

```text id="c8n3v7"
tshark -i eth0
```

Always verify the interface before capturing.

## Start a Basic Live Capture

A basic capture is:

```text id="y6p2m8"
tshark -i 1
```

Stop it with:

```text id="v9k4q1"
Ctrl+C
```

Use live capture only on systems and networks where you are authorized to capture traffic.

## Live Capture With a Capture Filter

Example:

```text id="m3x7c9"
tshark -i 1 -f "port 53"
```

This limits captured traffic to packets matching the capture filter.

For TCP port 443:

```text id="q5n8r2"
tshark -i 1 -f "tcp port 443"
```

## Live Capture With a Display Filter

You can also capture broadly and display selectively.

Example:

```text id="a7c4m9"
tshark -i 1 -Y "dns"
```

Conceptually:

```text id="j2v6p8"
Capture
↓
All visible packets
↓
Display only DNS
```

This preserves more capture context than a restrictive capture filter.

## Reading a PCAP and Counting Packets

A simple way to inspect a capture is:

```text id="w8r3k5"
tshark -r capture.pcapng | wc -l
```

The exact result depends on shell behavior and output formatting.

For more reliable analysis, use TShark's statistics features rather than treating terminal line counts as authoritative packet statistics.

## Packet Numbers

TShark normally displays packet numbers.

Example:

```text id="f4m7x2"
1  ...
2  ...
3  ...
```

These packet numbers can be useful when documenting findings.

Record:

```text id="n9q3v6"
Packet number
Timestamp
Source
Destination
Protocol
Observation
```

## Verbose Packet Details

Use:

```text id="c5x8r1"
tshark -r capture.pcapng -V
```

`-V` displays detailed packet information.

This is similar to expanding protocol trees in Wireshark's packet-details pane.

It can produce a large amount of output.

Use it selectively.

## Packet Summary vs Detailed Dissection

Use normal output when you need:

```text id="p6m2w9"
Overview
Packet sequence
Endpoints
Protocol
Timing
```

Use `-V` when you need:

```text id="r8k4c3"
Detailed fields
Protocol structure
Specific packet values
```

A professional workflow usually moves between both levels.

## Hex and ASCII Output

TShark can display packet bytes.

Use:

```text id="x3q7m5"
tshark -r capture.pcapng -x
```

This can help when:

* A field needs raw verification
* Dissection appears incomplete
* You need to inspect packet bytes

Do not inspect raw bytes without a specific analytical question.

## Packet Bytes and Protocol Dissection

Use:

```text id="v5n8c2"
Normal output
```

to identify the relevant packet first.

Then:

```text id="j7m3q9"
-x
```

to inspect raw bytes if necessary.

This is more efficient than examining raw bytes for every packet.

## Read Specific Packets

If you know a relevant packet number, you can narrow your analysis using filters or shell tools.

For example:

```text id="b4r9x6"
tshark -r capture.pcapng -Y "frame.number == 143"
```

This is useful when documenting a finding.

## Filter by IP Address

Example:

```text id="k2m7v4"
tshark -r capture.pcapng -Y "ip.addr == 192.0.2.25"
```

Source only:

```text id="q6x3n8"
tshark -r capture.pcapng -Y "ip.src == 192.0.2.25"
```

Destination only:

```text id="r8v5c2"
tshark -r capture.pcapng -Y "ip.dst == 192.0.2.25"
```

## Filter by Protocol

Examples:

```text id="m3q7x9"
tshark -r capture.pcapng -Y "dns"
```

```text id="p8c4v1"
tshark -r capture.pcapng -Y "http"
```

```text id="y6n2r5"
tshark -r capture.pcapng -Y "tls"
```

```text id="w9k3m7"
tshark -r capture.pcapng -Y "tcp"
```

## Filter by Port

Examples:

```text id="c7x2m8"
tshark -r capture.pcapng -Y "tcp.port == 443"
```

```text id="v4n9q6"
tshark -r capture.pcapng -Y "udp.port == 53"
```

The protocol should be included when it makes the question clearer.

## Combine Filters

Example:

```text id="j5m8r3"
tshark -r capture.pcapng -Y "ip.addr == 192.0.2.25 && tcp.port == 443"
```

This asks:

```text id="q2v7n9"
Which TCP/443 traffic involves this host?
```

## OR Conditions

Example:

```text id="x8c4m6"
tshark -r capture.pcapng -Y "dns || tls"
```

This displays DNS or TLS packets.

Use parentheses for complex expressions:

```text id="r3n7k5"
tshark -r capture.pcapng -Y "(dns || tls) && ip.addr == 192.0.2.25"
```

## Negation

Example:

```text id="m6q2v8"
tshark -r capture.pcapng -Y "!(arp)"
```

This excludes ARP packets from the displayed results.

Do not use negative filters without understanding what context they remove.

## Field Extraction

One of TShark's most useful capabilities is extracting specific protocol fields.

Use:

```text id="y4c8n2"
tshark -r capture.pcapng -T fields -e ip.src -e ip.dst
```

This can produce a compact representation of selected fields.

## `-T fields`

The `-T fields` option tells TShark to output selected fields instead of the normal packet summary.

Example:

```text id="p7m3x9"
tshark -r capture.pcapng -T fields \
  -e frame.number \
  -e frame.time \
  -e ip.src \
  -e ip.dst
```

This is useful for:

* Tables
* Scripts
* Data processing
* Investigation notes

## Extract DNS Fields

Example:

```text id="v8q4c1"
tshark -r capture.pcapng -Y "dns" -T fields \
  -e frame.number \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name
```

Depending on the packet, some fields may be empty.

That is expected.

## Extract HTTP Fields

For captures where HTTP is visible:

```text id="k3n7r5"
tshark -r capture.pcapng -Y "http" -T fields \
  -e frame.number \
  -e ip.src \
  -e ip.dst \
  -e http.request.method \
  -e http.host \
  -e http.request.uri \
  -e http.response.code
```

Encrypted HTTPS will not necessarily expose equivalent HTTP fields.

## Extract TLS Fields

Example:

```text id="m9x2q6"
tshark -r capture.pcapng -Y "tls" -T fields \
  -e frame.number \
  -e ip.src \
  -e ip.dst \
  -e tls.handshake.type
```

Available fields depend on the TShark/Wireshark version and the packet.

## Inspect Available Fields

TShark can list protocol fields:

```text id="c5r8v3"
tshark -G fields
```

The output is large.

You can combine it with shell filtering where appropriate.

For example:

```text id="x7m4q2"
tshark -G fields | grep dns
```

The exact command depends on the operating system shell.

## Search for a Field Name

If you need to determine whether a field exists:

```text id="q9n3k6"
tshark -G fields | grep http.request
```

This is useful when a field name is uncertain.

## Output Separators

For field extraction, define a separator when necessary.

Example:

```text id="v6c2m8"
tshark -r capture.pcapng -T fields \
  -E separator=, \
  -e frame.number \
  -e ip.src \
  -e ip.dst
```

This makes the output easier to process as delimited data.

## Header Output

For structured field output:

```text id="m4x7p1"
tshark -r capture.pcapng -T fields \
  -E header=y \
  -e frame.number \
  -e ip.src \
  -e ip.dst
```

This can make results easier to read and import.

## JSON Output

TShark can produce JSON output.

Example:

```text id="r8k3v5"
tshark -r capture.pcapng -T json
```

JSON is useful when:

* Processing results programmatically
* Preserving structured packet information
* Integrating TShark into analysis workflows

The output can be large.

Use display filters when appropriate.

## JSON for Filtered Traffic

Example:

```text id="c6n9q2"
tshark -r capture.pcapng \
  -Y "dns" \
  -T json
```

This produces structured information for DNS packets matching the filter.

## EK JSON

TShark also supports an Elasticsearch-compatible JSON format in versions that provide it.

Example:

```text id="y3m8r6"
tshark -r capture.pcapng -T ek
```

Check the installed version's help output before relying on a specific output mode.

## Read From Standard Input

TShark can be used in pipelines where supported.

Conceptually:

```text id="k7q2v4"
Input
 ↓
TShark
 ↓
Filter
 ↓
Output
```

This becomes particularly useful when integrating packet analysis into scripts.

## Reading Compressed Captures

Support for compressed capture files depends on the TShark/Wireshark build and file format.

When possible, prefer:

```text id="p5r8x2"
Native PCAP/PCAPNG input
```

and verify support using:

```text id="m6c3n9"
tshark --help
```

or the installed documentation.

Do not assume every compressed format is directly readable.

## Capture File Formats

Common capture formats include:

```text id="w4k9q7"
PCAP
PCAPNG
```

PCAPNG is common in modern Wireshark workflows because it can preserve richer capture metadata.

Use the format appropriate to the investigation and tooling environment.

## File Information

Use:

```text id="x2n7m5"
capinfos capture.pcapng
```

`capinfos` is part of the Wireshark command-line toolset and provides capture metadata.

Useful information can include:

```text id="j8q4c1"
File type
File size
Packet count
Capture duration
First packet
Last packet
Data rate
Encapsulation
```

This is often a better first step than immediately dumping every packet.

## Basic Professional Workflow With `capinfos`

Use:

```text id="v3m6r9"
capinfos capture.pcapng
```

Then:

```text id="q7x2k5"
tshark -r capture.pcapng -Y "relevant filter"
```

Then:

```text id="c9n4m8"
tshark -r capture.pcapng -T fields ...
```

This creates:

```text id="p6r3w1"
Capture Metadata
↓
Focused Packets
↓
Structured Evidence
```

## TShark and Wireshark Together

A strong workflow is:

```text id="m8c5q2"
Wireshark
↓
Understand traffic visually
↓
Identify useful fields/filter
↓
TShark
↓
Repeat extraction
↓
Automate or process results
```

Do not force every task into TShark.

Use the GUI when visual investigation is more efficient.

## When to Prefer Wireshark

Use Wireshark when you need:

```text id="y2v7n4"
Interactive packet exploration
Visual protocol trees
Stream reconstruction
Graphs
Coloring
Manual correlation
Rapid hypothesis testing
```

## When to Prefer TShark

Use TShark when you need:

```text id="r5m8c3"
Batch analysis
Large capture processing
Field extraction
Repeatable commands
Remote analysis
Scripting
Structured output
Headless environments
```

## Reproducible TShark Commands

A good command should make its purpose clear.

Example:

```text id="n7q4x2"
tshark -r incident.pcapng \
  -Y "ip.addr == 192.0.2.25 && tcp.port == 443" \
  -T fields \
  -E header=y \
  -E separator=, \
  -e frame.number \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e tcp.srcport \
  -e tcp.dstport
```

This documents:

```text id="k3m8p6"
Input
Filter
Output format
Fields
```

## Shell Quoting

Display filters contain characters interpreted by shells.

Therefore, quote the filter:

```text id="q8v4n1"
-Y "ip.addr == 192.0.2.25 && tcp.port == 443"
```

Do not assume the shell will interpret an unquoted filter correctly.

## Windows vs Linux Shells

Command syntax around TShark is generally similar, but shell behavior differs.

Examples:

```text id="c5x7m2"
Linux:
grep

PowerShell:
Select-String

Windows command shell:
findstr
```

The TShark options remain the same, but surrounding shell commands may differ.

## TShark Errors

If TShark reports an invalid field or filter:

Check:

```text id="v9n3k6"
Field name
Filter syntax
TShark version
Protocol dissection
```

For example, a field available in one Wireshark release may differ in another.

## Verify Filters Before Automation

Before putting a TShark command into a script:

```text id="m2r8c4"
1. Run the filter manually.
2. Inspect several matching packets.
3. Verify the fields.
4. Confirm the output.
5. Only then automate it.
```

This prevents scripts from repeatedly extracting the wrong information.

## Practical Exercise 1 — Version and Interface Discovery

Run:

```text id="x6q3n8"
tshark --version
```

Then:

```text id="p4m7r2"
tshark -D
```

Record:

```text id="v8c5k1"
TShark version:
Available interfaces:
```

## Practical Exercise 2 — Read a PCAP

Choose an authorized PCAP and run:

```text id="r3n9q6"
tshark -r capture.pcapng
```

Observe:

```text id="j7m4x2"
Packet numbering
Timestamp
Source
Destination
Protocol
Length
Info
```

Compare the output with Wireshark's packet list.

## Practical Exercise 3 — Protocol Filtering

Run:

```text id="c8v2m5"
tshark -r capture.pcapng -Y "dns"
```

Then:

```text id="q5n7r3"
tshark -r capture.pcapng -Y "tcp"
```

Compare the results.

## Practical Exercise 4 — Host Filtering

Choose a host and run:

```text id="m9x4k7"
tshark -r capture.pcapng \
  -Y "ip.addr == 192.0.2.25"
```

Identify:

```text id="w6p2c8"
Destinations
Protocols
Ports
```

## Practical Exercise 5 — Field Extraction

Extract:

```text id="a4r8m2"
frame.number
frame.time
ip.src
ip.dst
```

Example:

```text id="f7q3n9"
tshark -r capture.pcapng -T fields \
  -E header=y \
  -E separator=, \
  -e frame.number \
  -e frame.time \
  -e ip.src \
  -e ip.dst
```

Inspect the resulting structure.

## Practical Exercise 6 — DNS Extraction

Run:

```text id="k2m8v5"
tshark -r capture.pcapng -Y "dns" -T fields \
  -E header=y \
  -E separator=, \
  -e frame.number \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e dns.qry.name
```

Determine which fields are populated for different DNS packet types.

## Practical Exercise 7 — TCP/443 Extraction

Run:

```text id="n5c7x2"
tshark -r capture.pcapng \
  -Y "tcp.port == 443" \
  -T fields \
  -E header=y \
  -E separator=, \
  -e frame.number \
  -e frame.time \
  -e ip.src \
  -e ip.dst \
  -e tcp.srcport \
  -e tcp.dstport
```

Use the results to identify conversations.

## Practical Exercise 8 — Packet-Level Verification

Find an important packet in Wireshark.

Record its packet number.

Then use:

```text id="p8r4m6"
tshark -r capture.pcapng \
  -Y "frame.number == <packet-number>" \
  -V
```

Compare the TShark details with the Wireshark packet-details pane.

## Practical Exercise 9 — Capture Metadata

Run:

```text id="x3m7q9"
capinfos capture.pcapng
```

Record:

```text id="c6n2v5"
File type
Packet count
Capture duration
First packet
Last packet
Encapsulation
```

Explain why this information matters before analysis.

## Practical Exercise 10 — Build a Reproducible Command

Choose one investigation question.

Example:

```text id="y8q4m1"
Which HTTPS connections involve the affected host?
```

Build a TShark command that:

```text id="r2v7c9"
Reads the capture
↓
Applies the relevant display filter
↓
Extracts useful fields
↓
Produces readable output
```

Document:

```text id="m5x8k3"
Question:
Command:
Filter:
Fields:
Expected output:
Actual observation:
```

## Common Mistakes

### Mistake 1: Confusing `-f` and `-Y`

Remember:

```text id="q7m3v9"
-f
→ Capture filter

-Y
→ Display filter
```

### Mistake 2: Running Huge Unfiltered Captures

A large PCAP can produce overwhelming output.

Start with:

```text id="x4n8c2"
capinfos
```

then narrow the analysis.

### Mistake 3: Guessing Field Names

Verify fields using:

```text id="k6r2m5"
tshark -G fields
```

### Mistake 4: Automating an Incorrect Filter

Always validate the filter manually first.

### Mistake 5: Forgetting Shell Quoting

Use:

```text id="v9c3q7"
-Y "filter expression"
```

### Mistake 6: Treating Field Extraction as Analysis

TShark can extract data.

The analyst still has to interpret it.

### Mistake 7: Ignoring Version Differences

Record the TShark/Wireshark version when reproducibility matters.

### Mistake 8: Assuming Every Field Exists in Every Packet

Protocol fields depend on packet type and dissection.

### Mistake 9: Replacing Wireshark With TShark Everywhere

Use the right interface for the problem.

### Mistake 10: Forgetting Capture Context

Command-line output does not eliminate capture visibility limitations.

## Professional TShark Workflow

Use:

```text id="b7m4x8"
1. Check TShark version.
2. Inspect capture metadata.
3. Define the question.
4. Identify the relevant protocol.
5. Build a display filter.
6. Test the filter.
7. Extract relevant fields.
8. Verify important packets.
9. Save reproducible commands.
10. Document observations.
```

The workflow is:

```text id="n3q8v5"
Question
  ↓
PCAP Metadata
  ↓
Filter
  ↓
Field Extraction
  ↓
Verification
  ↓
Interpretation
  ↓
Documentation
```

## Completion Criteria

You should be able to:

```text id="c4m9r7"
Explain what TShark is.

Read a PCAP with TShark.

List capture interfaces.

Perform a live authorized capture.

Distinguish -f from -Y.

Apply display filters.

Extract protocol fields.

Inspect detailed packet information.

Inspect packet bytes.

Use capinfos to understand a capture.

Produce structured output.

Verify TShark findings against Wireshark.

Record reproducible commands.

Explain the limitations of command-line analysis.
```

The key mental model is:

```text id="x8p2k6"
TShark
  ↓
Filter
  ↓
Extract
  ↓
Verify
  ↓
Interpret
  ↓
Automate
```

TShark is most valuable when packet analysis needs to become **repeatable, scriptable, and efficient without losing the evidence-based reasoning developed in Wireshark**.
