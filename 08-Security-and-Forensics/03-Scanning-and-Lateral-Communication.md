# Scanning and Lateral Communication

## Objective

Network captures can provide useful evidence about host discovery, service discovery, scanning behavior, and communication between internal systems.

The goal of this workflow is not to label traffic as malicious simply because it resembles scanning or lateral movement.

The goal is to determine:

```text
Who communicated?
With whom?
When?
Using which protocol?
Which ports or services were contacted?
How frequently?
In what direction?
What happened after the connection?
```

A useful investigation model is:

```text
Source
  ↓
Destination set
  ↓
Ports/services
  ↓
Timing
  ↓
Response pattern
  ↓
Follow-on communication
  ↓
Context
```

This file focuses on using Wireshark to reconstruct these behaviors from packet evidence.

## What Scanning Looks Like at the Packet Level

Scanning can produce repeated connection attempts involving:

```text
One source → Many destinations
One source → Many ports
Many sources → One destination
Repeated probes
Short-lived connections
Failed connection attempts
Successful connections followed by protocol traffic
```

These patterns are not automatically malicious.

Legitimate causes include:

* Vulnerability scanners
* Monitoring systems
* Asset discovery
* Service discovery
* Inventory systems
* Configuration management
* Load balancer health checks
* Administrative tools
* Application discovery
* Security testing

Context determines meaning.

## Establish the Scope

Before investigating scanning behavior, define:

```text
Source host:
Destination range:
Time window:
Expected activity:
Known scanner:
Known administrative system:
Relevant protocols:
```

Without a defined scope, a large capture can produce misleading conclusions.

## Step 1: Identify the Source

Start with the suspected source system.

For IPv4:

```text
ip.addr == 192.0.2.10
```

For a specific communication direction:

```text
ip.src == 192.0.2.10
```

For IPv6:

```text
ipv6.src == 2001:db8::10
```

The purpose is to isolate the initiating system before examining its destination pattern.

## Step 2: Identify the Destination Set

Once the source is isolated, determine:

```text
How many destinations?
Which addresses?
Which subnets?
Which ports?
Which protocols?
```

A useful first observation is whether communication is concentrated or distributed.

Example:

```text
Source:
192.0.2.10

Destinations:
192.0.2.20
192.0.2.21
192.0.2.22
192.0.2.23
192.0.2.24
```

This establishes a destination pattern.

It does not establish intent.

## One Source to Many Hosts

A common discovery pattern is:

```text
Source
 ↓
Host A
Host B
Host C
Host D
Host E
```

Investigate:

```text
Destination count
Destination subnet
Port consistency
Timing
Response behavior
```

For example:

```text
192.0.2.10 → 192.0.2.20:22
192.0.2.10 → 192.0.2.21:22
192.0.2.10 → 192.0.2.22:22
192.0.2.10 → 192.0.2.23:22
```

This may represent SSH service discovery.

It could also represent legitimate administrative activity.

## One Source to Many Ports

Another pattern is:

```text
192.0.2.10 → 192.0.2.20:21
192.0.2.10 → 192.0.2.20:22
192.0.2.10 → 192.0.2.20:25
192.0.2.10 → 192.0.2.20:53
192.0.2.10 → 192.0.2.20:80
```

This may indicate service discovery.

Investigate:

```text
Port sequence
Timing
Connection success
Protocol response
Follow-on communication
```

A port list alone is not enough to determine the activity's purpose.

## Many Sources to One Host

The reverse pattern can also be important:

```text
Host A ─┐
Host B ─┤
Host C ─┼→ Server
Host D ─┤
Host E ─┘
```

Possible explanations include:

* Normal client activity
* Service usage
* Monitoring
* Load testing
* Distributed scanning
* Application behavior

Determine whether the destinations and timing match the expected role of the target.

## TCP SYN Patterns

TCP connection attempts often begin with:

```text
SYN
SYN/ACK
ACK
```

A source repeatedly sending SYN packets can provide useful evidence about connection attempts.

Investigate whether the destination responds with:

```text
SYN/ACK
RST
No visible response
```

These outcomes can help distinguish:

```text
Open-looking service
Closed-looking port
No observable response
```

Do not overinterpret the result because filtering, firewalls, capture location, and packet loss can affect visibility.

## SYN → SYN/ACK

Example:

```text
Client → Server: SYN
Server → Client: SYN/ACK
Client → Server: ACK
```

This indicates that a TCP connection was established at the packet level.

Then inspect what happened next.

For example:

```text
TCP handshake
↓
SSH traffic
```

or:

```text
TCP handshake
↓
HTTP request
```

A successful connection provides more evidence than a single probe.

## SYN → RST

Example:

```text
Client → Server: SYN
Server → Client: RST
```

This can indicate that the connection was actively rejected.

Possible explanations include:

* No service listening
* Host firewall behavior
* Application behavior
* Network device behavior

Treat it as an observed response pattern rather than a definitive statement about the target's configuration.

## SYN With No Visible Response

Example:

```text
Client → Server: SYN

No visible response
```

Possible explanations include:

* Packet filtering
* Host unavailable
* Routing issue
* Capture point limitation
* Packet loss
* Firewall silently dropping traffic

Do not automatically conclude that the destination is offline.

## UDP Scanning and Discovery

UDP does not use a TCP handshake.

Therefore, investigation requires different evidence.

Look for:

```text
UDP request
↓
UDP response
```

or:

```text
UDP request
↓
ICMP response
```

or:

```text
UDP request
↓
No visible response
```

The absence of a response can be ambiguous.

## ICMP Responses to UDP

An ICMP response may provide evidence about an unreachable UDP destination.

For example:

```text
UDP probe
↓
ICMP Destination Unreachable
```

Inspect the ICMP message details and the embedded packet information.

This can help associate the response with the original UDP communication.

## ARP-Based Discovery

On local networks, host discovery may involve ARP.

A common sequence is:

```text
ARP request:
Who has 192.0.2.20?

ARP response:
192.0.2.20 is at <MAC>
```

Investigate:

```text
Requester
Target IP
Response
Timing
Frequency
```

Repeated ARP requests across a local subnet can indicate discovery or normal address-resolution activity.

Again, context matters.

## IPv6 Neighbor Discovery

IPv6 networks use Neighbor Discovery mechanisms rather than ARP.

Useful traffic includes:

```text
Neighbor Solicitation
Neighbor Advertisement
Router Solicitation
Router Advertisement
```

When investigating IPv6 discovery, identify:

```text
Source
Target
Message type
Timing
Follow-on communication
```

Do not assume that an absence of ARP means the network has no local discovery traffic.

## Port Distribution

For a suspected scanner, summarize contacted ports.

Example:

```text
22
80
443
445
3389
8080
```

Then ask:

```text
Are the same ports contacted across many hosts?
Are many ports contacted on one host?
Which ports succeeded?
Which failed?
Which produced application traffic?
```

This creates a more useful picture than simply counting packets.

## Destination Distribution

Similarly, summarize destinations.

Example:

```text
192.0.2.20
192.0.2.21
192.0.2.22
192.0.2.23
192.0.2.24
```

Then compare:

```text
Destination count
Port count
Protocol count
Time range
Response rate
```

## Timing Patterns

Timing can be highly informative.

Compare:

```text
Sequential
Burst
Periodic
Random-looking
Slow and distributed
```

For example:

```text
10:00:00.000 → Host A
10:00:00.050 → Host B
10:00:00.100 → Host C
10:00:00.150 → Host D
```

This indicates a regular sequence.

It does not by itself prove automated malicious scanning.

## Bursty Discovery

Another pattern:

```text
10:00:00
→ dozens of destinations

10:00:01
→ dozens of destinations
```

Investigate:

```text
Packet count
Destination count
Port count
Protocol
Source process context if available
```

Security tools and legitimate inventory systems can both create bursty traffic.

## Periodic Probing

Example:

```text
10:00:00
10:00:30
10:01:00
10:01:30
```

A regular interval can indicate:

* Monitoring
* Health checks
* Service discovery
* Scheduled automation
* Beacon-like behavior

The pattern should be correlated with application and host context.

## Scan Direction

Determine whether the behavior is:

```text
Internal → Internal
Internal → External
External → Internal
External → External
```

For lateral communication, the most relevant pattern is generally:

```text
Internal host → Other internal host(s)
```

But internal-to-external activity can also matter depending on the investigation.

## Lateral Communication

Lateral communication refers to movement or communication between systems within an environment.

Wireshark can help reveal communication such as:

```text
Workstation → Server
Server → Server
Workstation → Workstation
Admin system → Many systems
```

Potentially relevant protocols include:

```text
SMB
RPC
LDAP
Kerberos
RDP
SSH
WinRM
FTP
HTTP/HTTPS
DNS
SNMP
```

The presence of one of these protocols does not establish lateral movement.

The investigation should establish what communication actually occurred.

## SMB Investigation

SMB traffic may reveal communication with Windows file and service infrastructure.

Investigate:

```text
Source
Destination
TCP port
Session establishment
SMB dialect/version when visible
Operations
Timing
```

Questions:

```text
Which host initiated the communication?
Which server responded?
Was the session successful?
Was there subsequent SMB activity?
```

Do not infer file access or execution unless the packet evidence supports the conclusion.

## RDP Investigation

For RDP-related traffic, identify:

```text
Source
Destination
TCP connection
Connection timing
Session duration
```

Depending on encryption and protocol visibility, detailed application content may not be available.

Record the limitation.

## SSH Investigation

For SSH:

```text
Source
Destination
TCP port
Connection establishment
Session duration
Repeated attempts
```

If authentication or command contents are encrypted, Wireshark cannot directly reveal them without appropriate decryption or endpoint evidence.

Do not claim a command was executed merely because an SSH connection existed.

## LDAP and Kerberos

Directory-related communication can be important in enterprise investigations.

Investigate:

```text
Client
Server
Protocol
Port
Timing
Request/response pattern
```

For encrypted or otherwise protected sessions, document what metadata is visible and avoid claiming content that cannot be observed.

## Windows Administrative Traffic

A system communicating with many internal hosts over administrative protocols may deserve closer investigation.

Possible explanations include:

* Domain administration
* Configuration management
* Backup
* Monitoring
* Software deployment
* Vulnerability scanning
* Security tooling

The important question is:

```text
Does the communication match the expected role of the source?
```

## Service Discovery vs Scanning

These can look similar.

For example:

```text
Host → many servers:443
```

could represent:

```text
Service discovery
Health checking
Application behavior
Security scanning
```

To differentiate them, examine:

```text
Timing
Frequency
Protocol depth
Source role
Destination role
Response behavior
Follow-on sessions
Baseline
```

## Scan Depth

A useful distinction is between:

```text
Probe only
```

and:

```text
Probe + application interaction
```

Example:

```text
SYN
SYN/ACK
RST
```

shows a limited TCP interaction.

Whereas:

```text
SYN
SYN/ACK
ACK
HTTP request
HTTP response
```

shows deeper application interaction.

This distinction can help characterize activity.

## Successful Service Discovery

Consider:

```text
Source → Host A:22
SYN
SYN/ACK
ACK
SSH protocol traffic

Source → Host B:22
SYN
RST

Source → Host C:22
SYN
SYN/ACK
ACK
SSH protocol traffic
```

The capture supports:

```text
Host A: TCP connection established
Host B: TCP connection rejected
Host C: TCP connection established
```

It does not by itself identify the tool or intent responsible.

## Repeated Failed Connections

A host repeatedly attempting connections to unavailable services may indicate:

* Misconfiguration
* Broken application
* Monitoring
* Scanner activity
* Automated discovery
* Authentication/application problems

Investigate the pattern rather than the individual packet.

## Source Port Behavior

Client applications often use ephemeral source ports.

Example:

```text
192.0.2.10:49152 → 192.0.2.20:22
192.0.2.10:49153 → 192.0.2.21:22
```

Do not interpret changing source ports as different source hosts.

Focus on:

```text
Source IP
Destination IP
Destination port
Connection timing
```

## NAT Considerations

Network Address Translation can hide the original host.

You may observe:

```text
Internal:
192.0.2.10

Translated:
198.51.100.10
```

Depending on the capture point, only one side may be visible.

Document:

```text
Capture location
Observed source
Observed destination
Expected translated address
```

Do not infer the original endpoint if the capture does not support it.

## Proxies and Load Balancers

A proxy can make many clients appear to communicate with one intermediary.

Example:

```text
Client A ─┐
Client B ─┼→ Proxy → Server
Client C ─┘
```

Likewise, a load balancer may make many backend systems appear behind one address.

This affects interpretation of:

```text
Destination counts
Connection counts
Source/destination relationships
```

Always consider infrastructure intermediaries.

## VLANs and Capture Visibility

If traffic between hosts appears absent, determine whether the capture point can see that traffic.

Possible limitations:

* Switched network
* VLAN segmentation
* SPAN configuration
* Host-based capture
* Wireless capture
* Remote capture
* Tunnel boundaries

Absence of traffic in a capture is not always proof that the communication did not occur.

## Scan Pattern Investigation Workflow

Use this workflow:

```text
1. Identify the source.
2. Define the time window.
3. Enumerate destinations.
4. Enumerate destination ports.
5. Identify protocols.
6. Measure timing.
7. Examine TCP handshake outcomes.
8. Examine UDP/ICMP responses where applicable.
9. Identify successful connections.
10. Follow successful sessions.
11. Identify application protocols.
12. Determine communication direction.
13. Consider NAT/proxy/load-balancer effects.
14. Compare against the source's expected role.
15. Separate observation from interpretation.
16. Identify additional evidence required.
```

## Practical Exercise 1 — One Source to Many Hosts

Find a source contacting multiple destinations.

Record:

```text
Source:
Destination count:
Destination addresses:
Ports:
Protocols:
Time window:
Response pattern:
```

Determine whether the activity appears sequential, bursty, or periodic.

## Practical Exercise 2 — One Host, Many Ports

Find a source contacting multiple ports on one destination.

Record:

```text
Destination:
Ports:
Protocol:
Connection result:
Application traffic:
```

Explain what the packet evidence supports.

## Practical Exercise 3 — TCP Probe Classification

Find three TCP connection attempts with different outcomes:

```text
SYN → SYN/ACK
SYN → RST
SYN → no visible response
```

For each, document the exact packet sequence.

Then list possible explanations without selecting one unless additional evidence supports it.

## Practical Exercise 4 — UDP Investigation

Find UDP traffic involving multiple destinations or ports.

Record:

```text
Source:
Destination:
Port:
Response:
ICMP response:
Timing:
```

Explain why the absence of a UDP response is not automatically conclusive.

## Practical Exercise 5 — Internal Communication

Find communication between two internal hosts.

Determine:

```text
Source role:
Destination role:
Protocol:
Port:
Session duration:
Application behavior:
```

Ask whether the communication is consistent with the expected roles.

## Practical Exercise 6 — SMB/RDP/SSH Investigation

Choose one protocol:

```text
SMB
RDP
SSH
```

Trace the connection from establishment to termination where visible.

Document:

```text
Source:
Destination:
Port:
Handshake:
Protocol:
Session duration:
Visible application evidence:
Encryption limitations:
```

## Practical Exercise 7 — Repeated Internal Connections

Find a source repeatedly connecting to internal destinations.

Measure:

```text
Connection count
Destination count
Port count
Time intervals
Success rate
```

Compare the pattern with an expected administrative or monitoring workflow.

## Practical Exercise 8 — Scan vs Application Discovery

Find activity that could plausibly be interpreted as scanning.

Then investigate whether it could instead represent:

```text
Health checking
Monitoring
Inventory
Service discovery
Configuration management
```

Write two competing hypotheses.

For each hypothesis, list:

```text
Evidence supporting it
Evidence against it
Evidence still missing
```

Do not force a conclusion when the capture cannot distinguish the explanations.

## Practical Exercise 9 — Lateral Communication Timeline

Build a timeline:

```text
Source host
↓
First internal destination
↓
Second destination
↓
Successful service
↓
Follow-on protocol traffic
↓
Additional destination
```

Record exact timestamps.

Then explain how the sequence changes your understanding of the activity.

## Practical Exercise 10 — Independent Investigation

Select an unfamiliar internal communication pattern.

Without immediately searching for a known signature:

```text
1. Identify the source.
2. Identify destinations.
3. Identify ports.
4. Identify protocols.
5. Measure timing.
6. Examine connection outcomes.
7. Follow successful conversations.
8. Identify application behavior.
9. Compare against expected roles.
10. Document observations.
11. State competing interpretations.
12. Identify missing evidence.
```

The objective is to investigate the traffic rather than simply classify it.

## Observation vs Interpretation

Use precise language.

### Observation

```text
192.0.2.10 sent TCP SYN packets to 25 hosts on TCP port 22 within 2 seconds.
```

### Interpretation

```text
The pattern is consistent with automated SSH service discovery.
```

### Conclusion

```text
Additional endpoint and authorization context is required to determine whether the activity was expected.
```

This separation prevents overclaiming.

## Evidence Record

Use this structure:

```text
Investigation:
Capture:
Time window:

Source host:

Source role:

Destination set:

Destination ports:

Protocols:

First observed event:

Last observed event:

Connection outcomes:

Successful sessions:

Follow-on application traffic:

Repeated behavior:

Timing pattern:

Capture point:

Known infrastructure intermediaries:

Observed facts:

Interpretations:

Alternative explanations:

Unknowns:

Additional evidence required:
```

## Common Mistakes

### Mistake 1: Calling Every Multi-Host Connection a Scan

Legitimate software frequently communicates with multiple systems.

### Mistake 2: Treating SYN Packets as Proof of Scanning

Connection attempts must be evaluated in context.

### Mistake 3: Ignoring Successful Application Traffic

A successful TCP connection followed by application traffic provides much more context than the SYN alone.

### Mistake 4: Ignoring UDP

UDP discovery does not behave like TCP scanning.

### Mistake 5: Assuming No Response Means Host Down

Filtering, packet loss, and capture limitations can produce the same observation.

### Mistake 6: Ignoring Network Infrastructure

NAT, proxies, load balancers, VLANs, and monitoring architecture can change what the capture shows.

### Mistake 7: Treating Internal Traffic as Automatically Trusted

Internal communication still requires contextual analysis.

### Mistake 8: Assuming Protocol Means Intent

SSH, SMB, RDP, LDAP, and Kerberos are legitimate protocols with many normal uses.

### Mistake 9: Confusing Source Ports With Source Identity

Ephemeral ports change between connections.

### Mistake 10: Overstating Lateral Movement

A connection between two internal hosts does not by itself prove lateral movement.

## Professional Scanning and Lateral Communication Workflow

Use this workflow during authorized investigations:

```text
Question
  ↓
Source identification
  ↓
Destination mapping
  ↓
Port mapping
  ↓
Protocol identification
  ↓
Timing analysis
  ↓
Connection outcome
  ↓
Application follow-up
  ↓
Infrastructure context
  ↓
Host-role context
  ↓
Baseline comparison
  ↓
Evidence assessment
  ↓
Competing interpretations
  ↓
Additional evidence
```

The central question is not:

```text
"Does this look like a scan?"
```

Instead ask:

```text
"What communication pattern is actually supported by the capture,
and what additional context is required to explain it?"
```

## Investigation Checklist

Before closing a scanning or lateral communication investigation:

* [ ] Source identified
* [ ] Time window defined
* [ ] Destination set identified
* [ ] Destination ports identified
* [ ] Protocols identified
* [ ] TCP handshake behavior examined
* [ ] UDP behavior examined where relevant
* [ ] ICMP responses examined where relevant
* [ ] Successful sessions followed
* [ ] Application traffic examined
* [ ] Timing pattern documented
* [ ] Internal/external direction established
* [ ] NAT considered
* [ ] Proxy/load-balancer behavior considered
* [ ] VLAN/capture visibility considered
* [ ] Source role considered
* [ ] Destination roles considered
* [ ] Baseline considered
* [ ] Observations separated from interpretations
* [ ] Alternative explanations documented
* [ ] Unknowns documented
* [ ] Additional evidence identified

## Completion Criteria

You should be able to independently analyze a packet capture and answer:

```text
Which host initiated the communication?

How many destinations were contacted?

Which ports were targeted?

Which protocols were used?

What was the timing pattern?

Which connections succeeded?

Which connections failed?

What happened after successful connections?

Was application traffic visible?

Could NAT, proxies, or load balancers affect interpretation?

Could the behavior represent legitimate discovery or administration?

What evidence supports each interpretation?

What does the capture not establish?

What additional evidence is required?
```

The core mental model is:

```text
Source
 ↓
Destination Pattern
 ↓
Port Pattern
 ↓
Protocol
 ↓
Timing
 ↓
Response
 ↓
Application
 ↓
Context
 ↓
Evidence-Based Interpretation
```
