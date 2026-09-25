# Suspicious Communications

## Objective

Wireshark can help identify and investigate communications that appear unusual, unexpected, or inconsistent with normal network behavior.

Examples include:

```text
Unexpected outbound connection
Repeated connections to an unfamiliar destination
Unusual DNS activity
Unexpected external services
Periodic network communication
Unexpected protocols
Large outbound transfers
Connections to unusual ports
Unexpected internal-to-internal communication
```

The purpose of this workflow is not to declare that a packet is malicious simply because it looks unusual.

Instead, the objective is to determine:

* Who communicated with whom
* When the communication occurred
* Which protocol was used
* How frequently communication occurred
* Whether the communication was expected
* What data or metadata is visible
* Whether the behavior differs from a known baseline
* What evidence supports suspicion
* What additional evidence is required

The central investigation model is:

```text id="f7j3k1"
Suspicious observation
        ↓
Identify source
        ↓
Identify destination
        ↓
Identify protocol
        ↓
Establish timeline
        ↓
Understand normal behavior
        ↓
Inspect communication pattern
        ↓
Inspect DNS / HTTP / TLS / TCP evidence
        ↓
Correlate related traffic
        ↓
Separate observation from interpretation
        ↓
Determine what requires further investigation
```

A professional security analyst should be able to say:

```text id="p2m8r4"
"This communication is unusual because..."
```

without automatically saying:

```text id="q6t1v9"
"This communication is malicious."
```

## Security Analysis Starts With a Baseline

Unusual does not automatically mean malicious.

Networks routinely contain:

```text id="c4n7x2"
Cloud services
CDNs
Telemetry
Software updates
Security scanners
Monitoring systems
Backup systems
Load balancers
Proxies
VPNs
Management protocols
Automated jobs
```

A destination that is unfamiliar to an analyst may be completely legitimate.

Therefore ask:

```text id="h8k3p5"
What normally communicates?
When does it normally communicate?
How frequently?
To which destinations?
Using which protocols?
```

Without a baseline, anomaly detection becomes guesswork.

## Define the Suspicious Event

Start with the exact observation.

Examples:

```text id="w1s6y9"
"Host 192.0.2.10 repeatedly connects to an unfamiliar external IP."

"One workstation generates many DNS queries."

"A server makes periodic outbound HTTPS connections."

"A host communicates with an unexpected internal system."

"An application suddenly begins using a new destination port."
```

Record:

| Item        | Question                              |
| ----------- | ------------------------------------- |
| Source      | Which host initiated the traffic?     |
| Destination | Which host received it?               |
| Time        | When did it occur?                    |
| Protocol    | What protocol was used?               |
| Port        | Which port was involved?              |
| Frequency   | How often did it happen?              |
| Volume      | How much traffic was exchanged?       |
| Context     | What system/application generated it? |
| Baseline    | Is this normal for the host?          |

## Identify the Source

Determine the originating endpoint.

For IPv4:

```text id="d5k9r2"
ip.src
```

For IPv6:

```text id="m3q7t1"
ipv6.src
```

Also inspect:

```text id="a8c2f6"
Ethernet source
TCP source port
UDP source port
```

The source IP identifies the network-layer sender visible at the capture point.

It does not necessarily identify the specific process or user.

For that, additional endpoint evidence may be required.

## Identify the Destination

Determine:

```text id="v4y8n0"
Destination IP
Destination port
Protocol
```

Then ask:

```text id="z2b6d9"
Is the destination internal?
External?
Expected?
Shared?
Dynamic?
Part of a known service?
```

Do not treat an unfamiliar IP address as malicious solely because it is unfamiliar.

## Internal vs External Communication

A useful first distinction is:

```text id="g6j1l4"
Internal host → Internal host
```

versus:

```text id="n8p3s5"
Internal host → External host
```

External communications often deserve additional context because they cross organizational boundaries.

But legitimate systems communicate externally all the time.

The investigation must determine whether the behavior is expected.

## Identify the Protocol

Determine whether the communication uses:

```text id="r7t2x5"
DNS
HTTP
TLS
SSH
SMB
LDAP
Kerberos
SMTP
FTP
SNMP
NTP
ICMP
TCP
UDP
```

or another protocol.

Protocol identification provides context.

For example:

```text id="y3u8w1"
Repeated DNS queries
```

mean something different from:

```text id="q5s0v4"
Repeated TCP connections to port 443
```

The same destination can also support multiple legitimate services.

## Ports Are Context, Not Verdicts

An unusual port can attract attention.

For example:

```text id="k2m6o9"
TCP connection to an uncommon port
```

But the port number alone does not establish malicious activity.

Applications can legitimately use non-standard ports.

Similarly:

```text id="c4e8g2"
Port 443
```

does not automatically mean the traffic is safe.

Always inspect the actual communication behavior.

## Establish the Timeline

Time is one of the strongest tools in security analysis.

Determine:

```text id="p6r0t3"
First observed communication
Last observed communication
Number of connections
Time between connections
Duration of each connection
```

Example:

```text id="a9d2f5"
10:00:00 → connection
10:05:00 → connection
10:10:00 → connection
10:15:00 → connection
```

A repeated pattern may be operationally meaningful.

But periodic communication is not automatically malicious.

Scheduled updates, monitoring, telemetry, and management tasks can behave similarly.

## Frequency Analysis

Count communication events.

Questions:

```text id="m7q1s4"
How many connections?
Over what period?
Is the rate stable?
Does it spike?
Does it stop?
Does it change after a user action?
```

A baseline helps determine whether the frequency is unusual.

## Periodic Communication

Repeated connections at regular intervals can be interesting.

Example:

```text id="v8x2z6"
Connection
↓ 60 seconds
Connection
↓ 60 seconds
Connection
↓ 60 seconds
Connection
```

This can be described as:

```text id="b4d9f1"
Periodic communication
```

Do not automatically label it as command-and-control or beaconing.

Possible legitimate explanations include:

* Monitoring
* Telemetry
* Health checks
* Scheduled synchronization
* Software updates
* Application polling

The next step is contextual investigation.

## Beacon-Like Behavior

Security analysts may investigate communications that resemble beaconing.

Potential characteristics include:

```text id="h5j0l3"
Repeated destination
Regular timing
Similar packet sizes
Similar connection sequence
Consistent protocol
```

These characteristics can justify further investigation.

They do not independently prove malicious intent.

Use evidence such as:

```text id="n7p2r6"
Destination reputation
Endpoint process information
DNS history
Application context
Authentication logs
Host telemetry
```

when available.

## Destination Diversity

A host communicating with many destinations may be normal or suspicious depending on the system.

For example:

```text id="q9s4u8"
Browser workstation
    ↓
Many external destinations
```

may be normal.

But:

```text id="w1y6a0"
Minimal server workload
    ↓
Sudden communication with many unfamiliar destinations
```

may deserve investigation.

The baseline matters.

## Port Diversity

Similarly, inspect whether a source communicates across many ports.

For example:

```text id="e3g8i2"
One source
    ↓
Many destination ports
```

This could represent:

* Legitimate application behavior
* Service discovery
* Monitoring
* Network scanning
* Misconfiguration
* Security testing

Packet evidence can reveal the pattern, but context determines interpretation.

## Connection Failures as a Security Signal

Repeated failed connections can be meaningful.

Example:

```text id="k5m9q1"
SYN
RST
SYN
RST
SYN
RST
...
```

Possible explanations include:

* Application retries
* Service discovery
* Misconfiguration
* Monitoring
* Scanning
* Security testing

Do not immediately classify the behavior.

Instead determine:

```text id="r2v6x0"
Who initiated the connections?
Which destinations?
Which ports?
How many?
How quickly?
Was the pattern sequential or distributed?
```

## Sequential Port Activity

A host contacting many ports on one destination can be a useful observation.

For example:

```text id="u4y8a2"
Destination: 192.0.2.50

Port 21
Port 22
Port 23
Port 25
Port 53
Port 80
Port 110
...
```

This pattern may be consistent with service discovery or scanning.

But legitimate network-management tools can produce similar traffic.

Document the observable pattern first.

## Distributed Destination Activity

Another pattern is:

```text id="m6q0s4"
One source
   ↓
Many destinations
   ↓
Same or multiple ports
```

Possible explanations include:

* Application distribution
* CDN communication
* Software update infrastructure
* Monitoring
* Scanning
* Discovery
* Compromised-host behavior

Again, packet behavior creates a hypothesis, not a verdict.

## DNS as a Security Signal

DNS can reveal useful security context because many applications communicate through hostnames.

Investigate:

```text id="c8e2g6"
Query frequency
Query diversity
Failed lookups
Long or unusual names
Repeated queries
Unusual record types
Unexpected DNS servers
```

A sudden change in DNS behavior may be relevant.

But unusual DNS activity has many legitimate explanations.

## Repeated Failed DNS Queries

Example:

```text id="p0t4v8"
Query A
NXDOMAIN
Query B
NXDOMAIN
Query C
NXDOMAIN
...
```

Possible explanations include:

* Application searching for multiple service names
* Misconfiguration
* Missing internal records
* Software discovery
* Browser/application behavior
* Security-related behavior

Investigate the names, timing, and application context.

## DNS Query Diversity

Count the number of distinct names queried by a host.

A workstation may generate many names during normal browsing.

A server with a narrow role may normally generate fewer.

Therefore:

```text id="x2z6b0"
High DNS diversity
```

is meaningful only relative to the system's expected behavior.

## DNS and Unusual Naming Patterns

Some DNS names may deserve closer inspection because they contain:

* Long labels
* Repeated encoded-looking data
* High query frequency
* Large amounts of changing subdomain content

These characteristics can occur in legitimate systems as well.

Examples include:

```text id="d4f8h2"
Tracking systems
CDNs
Telemetry
Security products
Application identifiers
```

Therefore investigate the full pattern instead of treating naming characteristics as proof of DNS tunneling or data exfiltration.

## HTTP Security Investigation

HTTP traffic can expose useful information when it is unencrypted.

Investigate:

```text id="j6n0r4"
Host
URI
Method
Status code
User-Agent
Headers
Redirects
Request timing
Response timing
```

Questions include:

```text id="q8s2w6"
What resource was requested?
Who requested it?
Where did the request go?
What response was returned?
Was the behavior expected?
```

## Unexpected HTTP Requests

A workstation may normally communicate with known services.

A sudden request to an unfamiliar host may justify investigation.

Document:

```text id="m4p8t2"
Source:
Destination:
Host:
URI:
Method:
Status:
Time:
```

Then determine whether the request corresponds to:

* Software update
* CDN content
* Advertisement
* Application dependency
* API call
* User activity
* Security tooling

## HTTP POST Requests

POST requests can contain application data.

In authorized environments where the traffic is visible, inspect:

```text id="v0x4z8"
Destination
URI
Content type
Request size
Response
Timing
```

Do not assume a POST is suspicious.

Many legitimate applications rely heavily on POST requests.

## Large Outbound Transfers

Large outbound transfers can deserve investigation depending on the host and context.

First establish:

```text id="b2f6j0"
Source:
Destination:
Duration:
Bytes:
Protocol:
Application:
Expected behavior:
```

Possible legitimate explanations include:

* Backup
* Cloud synchronization
* Software update
* File transfer
* Business application
* Media upload

Possible security hypotheses may include:

* Unauthorized transfer
* Data exfiltration
* Compromised application

The capture alone may not reveal which explanation is correct.

## Asymmetric Data Volume

Consider:

```text id="h8l2n4"
Small inbound request
Large outbound response
```

or:

```text id="p6r0t4"
Small outbound request
Large inbound response
```

The direction and volume can help characterize communication.

But data volume is not intent.

A legitimate file download can be very large.

A legitimate upload can also be very large.

## Encrypted Communications

Modern suspicious communications are often encrypted.

You may not see application contents.

You may still observe:

```text id="y2c6e0"
Source
Destination
Port
TLS metadata
Timing
Packet sizes
Connection frequency
Connection duration
Handshake behavior
```

This can support behavioral analysis.

It cannot reliably reveal encrypted payload content without appropriate decryption or endpoint evidence.

## TLS Metadata

Depending on the protocol and configuration, inspect visible information such as:

* TLS version
* Cipher information
* Certificate information
* Server Name Indication when visible
* Handshake timing
* Alerts
* Connection lifecycle

These provide metadata rather than complete application visibility.

## TLS Does Not Automatically Mean Safe

Encryption protects traffic contents.

It does not establish that the communication itself is legitimate.

A useful security workflow is:

```text id="f4j8l2"
Encrypted traffic
    ↓
Who?
    ↓
Where?
    ↓
When?
    ↓
How often?
    ↓
How much?
    ↓
What surrounding DNS behavior?
    ↓
What endpoint process?
```

The last question generally requires endpoint telemetry outside Wireshark.

## ICMP as a Security Signal

ICMP can reveal:

* Reachability testing
* Network diagnostics
* Path behavior
* Errors
* Discovery behavior

Repeated ICMP activity can be legitimate.

Interpret it according to:

```text id="n0p4r8"
Source
Destination
Type
Code
Frequency
Timing
Context
```

## Internal Lateral Communication

Unexpected internal communication can be important during incident response.

For example:

```text id="v6x2z4"
Workstation A
   ↓
Workstation B
   ↓
SMB
```

or:

```text id="q8s0u2"
One internal host
   ↓
Many internal destinations
```

Possible explanations include:

* Normal administration
* File sharing
* Service discovery
* Monitoring
* Backup
* Software deployment
* Security scanning
* Lateral movement

The communication pattern should be documented before assigning intent.

## Service Discovery vs Scanning

Both can involve multiple connection attempts.

A security analyst should ask:

```text id="w4y6a8"
Which hosts?
Which ports?
How quickly?
What order?
What response?
Is the source a known management system?
Does the behavior match the host's role?
```

A scanner may generate systematic patterns.

A management platform may generate similarly systematic traffic.

Context distinguishes them.

## SMB Communications

SMB can be legitimate in many enterprise environments.

When unexpected SMB activity appears, investigate:

```text id="b0d2f4"
Source
Destination
Ports
Session setup
Authentication-related traffic
File/share operations when visible
Frequency
Timing
```

Do not classify SMB itself as suspicious.

The question is whether the communication is expected for the hosts involved.

## SSH Communications

Unexpected SSH connections can be important depending on environment.

Inspect:

```text id="h6j8l0"
Source
Destination
Port
Connection frequency
Connection duration
Handshake
Authentication-related behavior when visible
```

Encrypted SSH payloads generally prevent direct inspection of command content.

Endpoint logs may be required.

## RDP and Remote Administration

Remote-administration protocols can generate substantial legitimate traffic.

When analyzing unexpected remote administration, identify:

```text id="n2p4r6"
Source host
Destination host
Time
Protocol
Frequency
Whether the source is an expected administrator or management system
```

Wireshark can establish the network communication.

Identity and authorization usually require other evidence.

## Correlation With Endpoint Evidence

Network traffic alone often cannot answer:

```text id="t8v0x2"
Which process created this connection?
Which user initiated it?
Why did the process make the connection?
Was the executable legitimate?
```

Additional evidence may include:

* Endpoint process telemetry
* DNS logs
* Proxy logs
* Firewall logs
* Authentication logs
* EDR telemetry
* Application logs
* System logs

A professional investigation combines network and host evidence.

## Observation vs Interpretation vs Conclusion

Use three separate layers.

### Observation

What the packet capture directly shows.

```text id="z4b6d8"
Host A initiated 25 TCP connections to Host B over 10 minutes.
```

### Interpretation

What the behavior may indicate.

```text id="c2e6g0"
The repeated communication may represent application polling or another periodic process.
```

### Conclusion

What the broader evidence supports.

```text id="f8j0l2"
Additional endpoint evidence is required to determine which process generated the connections.
```

This structure prevents overclaiming.

## Security Investigation Example

Suppose:

```text id="m4o6q8"
Internal workstation
        ↓
DNS query
        ↓
Unfamiliar hostname
        ↓
External IP
        ↓
Repeated TLS connections
        ↓
Approximately every 60 seconds
```

A professional investigation would record:

```text id="p0r2t4"
Observation:
Repeated DNS and TLS communication to the same external destination.

Pattern:
Approximately periodic communication.

Interpretation:
The behavior is unusual enough to warrant additional investigation.

Unknown:
The capture does not establish which local process initiated it or whether the destination is malicious.

Next evidence:
Endpoint process telemetry, DNS history, destination intelligence, and application context.
```

This is stronger than immediately declaring the host compromised.

## Practical Exercise 1 — Identify an Unusual Connection

Find a connection that appears unusual within your lab or supplied PCAP.

Record:

```text id="v6x8z0"
Source:
Destination:
Protocol:
Port:
Time:
Frequency:
Why it differs from baseline:
```

## Practical Exercise 2 — Periodic Communication

Find repeated connections to the same destination.

Determine:

```text id="b2d4f6"
Number of connections:
Approximate interval:
Destination:
Protocol:
Packet-size similarity:
```

Then identify at least two legitimate explanations that could produce similar behavior.

## Practical Exercise 3 — DNS Investigation

Find a host generating repeated DNS queries.

Record:

```text id="h8j0l2"
Query count:
Distinct names:
Failed queries:
Successful queries:
DNS server:
Time window:
```

Determine whether the behavior is consistent with the host's expected role.

## Practical Exercise 4 — Unusual HTTP

Find an HTTP request that deserves additional investigation.

Record:

```text id="n4p6r8"
Source:
Destination:
Host:
URI:
Method:
Status:
Time:
```

Determine what additional evidence would be needed to interpret the request.

## Practical Exercise 5 — Large Transfer

Find a large data transfer.

Record:

```text id="t0v2x4"
Source:
Destination:
Protocol:
Duration:
Approximate volume:
Direction:
```

Identify legitimate explanations before considering security hypotheses.

## Practical Exercise 6 — Internal Communication

Find communication between two internal hosts.

Determine:

```text id="z6b8d0"
Source:
Destination:
Protocol:
Port:
Frequency:
Expected purpose:
```

Explain what additional evidence would establish whether the communication is authorized.

## Practical Exercise 7 — Port Activity

Find one source communicating with multiple destination ports.

Record:

```text id="f2h4j6"
Source:
Destination:
Ports:
Timing:
Connection results:
```

Determine whether the pattern is more consistent with normal application behavior, administration, discovery, or something requiring additional investigation.

Do not force a definitive classification without context.

## Practical Exercise 8 — Encrypted Traffic

Find repeated encrypted communications.

Record:

```text id="m8o0q2"
Source:
Destination:
Port:
Frequency:
TLS metadata:
Visible information:
Unavailable information:
```

Explain what the network capture can and cannot establish.

## Practical Exercise 9 — Build an Evidence Timeline

Choose a suspicious communication and create:

```text id="q4s6u8"
Time
↓
DNS
↓
Connection
↓
TLS
↓
Application-visible event
↓
Connection termination
```

Identify which events can be directly correlated.

## Practical Exercise 10 — Observation vs Interpretation

Choose one suspicious-looking pattern.

Write three separate statements:

```text id="a0c2e4"
Observation:
Interpretation:
Conclusion / next evidence required:
```

Ensure that the conclusion does not claim more than the packet evidence supports.

## Professional Security Evidence Record

Use this structure:

```text id="g6i8k0"
Incident:
Date/Time:
Host:
Source:
Destination:
Protocol:
Port:
Capture point:

Observed behavior:

Frequency:

Volume:

Timeline:

DNS context:

HTTP context:

TLS context:

TCP context:

Baseline:

Why behavior is unusual:

What the packet capture proves:

What the packet capture does not prove:

Possible explanations:

Additional evidence required:

Final assessment:
```

## Common Mistakes

### Mistake 1: Unfamiliar Equals Malicious

An unfamiliar destination may simply be a legitimate service.

Investigate context.

### Mistake 2: Encryption Equals Malicious

Encrypted traffic is normal.

Analyze metadata and behavior.

### Mistake 3: Periodic Traffic Equals Beaconing

Periodic traffic can come from many legitimate systems.

Establish a baseline and investigate additional indicators.

### Mistake 4: Port Number Equals Application

Ports are clues, not proof of application identity.

### Mistake 5: Large Transfer Equals Exfiltration

Backups, synchronization, updates, and normal business operations can generate large transfers.

### Mistake 6: Repeated Connections Equal Scanning

Applications and monitoring systems can generate repeated connections.

Analyze destination diversity, ports, timing, and context.

### Mistake 7: Network Capture Identifies the Process

A packet capture usually identifies network endpoints, not the local process responsible.

Endpoint evidence may be required.

### Mistake 8: Ignoring Baseline

Without knowing normal behavior, anomaly interpretation becomes unreliable.

### Mistake 9: Overclaiming From One Packet

Security conclusions should generally come from patterns and correlated evidence.

### Mistake 10: Confusing Suspicion With Proof

A behavior can justify investigation without proving malicious activity.

## Professional Suspicious-Communication Workflow

Use this workflow consistently:

```text id="k2m4o6"
1. Define the suspicious observation.
2. Identify source and destination.
3. Identify protocol and port.
4. Establish the timeline.
5. Determine frequency and volume.
6. Compare with expected host behavior.
7. Inspect DNS activity.
8. Inspect TCP/UDP behavior.
9. Inspect HTTP when visible.
10. Inspect TLS metadata when applicable.
11. Identify repeated or unusual communication patterns.
12. Check internal vs external communication.
13. Compare with known-good behavior.
14. Separate observation from interpretation.
15. Identify alternative legitimate explanations.
16. Identify additional evidence required.
17. Document what the capture proves.
18. Document what remains unknown.
```

## Security Investigation Checklist

Before closing the investigation:

* [ ] Source identified
* [ ] Destination identified
* [ ] Protocol identified
* [ ] Port identified
* [ ] Time window established
* [ ] Capture point documented
* [ ] Frequency measured
* [ ] Volume considered
* [ ] Baseline considered
* [ ] DNS behavior checked
* [ ] TCP/UDP behavior checked
* [ ] HTTP behavior checked when visible
* [ ] TLS metadata checked when applicable
* [ ] Internal/external context checked
* [ ] Repeated communication examined
* [ ] Destination diversity considered
* [ ] Port diversity considered
* [ ] Successful/normal behavior compared
* [ ] Legitimate explanations considered
* [ ] Endpoint evidence identified as needed
* [ ] Observation separated from interpretation
* [ ] Uncertainty documented
* [ ] Additional evidence identified

## Completion Criteria

You should be able to take an unusual network communication and independently answer:

```text id="o8q0s2"
Who communicated?

With whom?

When?

Using which protocol?

Using which port?

How frequently?

How much traffic was exchanged?

Is the behavior expected?

How does it compare with the host baseline?

What DNS behavior surrounds it?

What TCP behavior surrounds it?

What application metadata is visible?

What does encryption prevent you from seeing?

What does the packet capture actually prove?

What remains uncertain?

What additional evidence would strengthen the investigation?
```

The core mental model is:

```text id="u4w6y8"
Unusual Communication
        ↓
Observe
        ↓
Contextualize
        ↓
Compare With Baseline
        ↓
Correlate
        ↓
Form Hypotheses
        ↓
Seek Additional Evidence
        ↓
Document Confidence and Uncertainty
```

The objective of network security analysis is not to label every unusual packet as malicious.

It is to turn network evidence into a defensible investigation that can be correlated with other evidence when necessary.
