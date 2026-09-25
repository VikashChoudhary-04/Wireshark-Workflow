# Capture Workflow

## Objective

A packet analysis investigation is only as reliable as the evidence captured.

Before analyzing packets, you need to understand how to:

* Choose the correct capture interface
* Define what you are trying to observe
* Start a capture deliberately
* Generate or wait for the relevant traffic
* Stop the capture at the right time
* Preserve the evidence
* Verify that the capture actually contains what you expected
* Recognize capture limitations before drawing conclusions

The core workflow is:

```text
Question
    ↓
Capture plan
    ↓
Correct interface
    ↓
Capture
    ↓
Generate or observe relevant traffic
    ↓
Stop capture
    ↓
Validate evidence
    ↓
Analyze
```

## Capture vs Analysis

Do not treat capturing and analyzing as the same activity.

Capture asks:

```text
What traffic should I collect?
```

Analysis asks:

```text
What does the collected traffic tell me?
```

A poor capture can make a correct analysis impossible.

For example:

```text
Expected:
Client → Server traffic

Captured:
Unrelated interface traffic
```

The analysis may be technically correct but still irrelevant to the original question.

## Start With a Question

Avoid capturing traffic without a reason.

Begin with a concrete question.

Examples:

```text
Why can this client not reach the server?

Why is DNS resolution slow?

Why is this application taking several seconds to respond?

What happens when the client connects to the service?

Is this host communicating with the expected destination?

What traffic is generated when this application starts?
```

The question determines what you need to capture.

## Define the Capture Scope

Before starting the capture, determine:

* Which host is relevant?
* Which interface is relevant?
* Which direction matters?
* Which protocol or application matters?
* When will the relevant event occur?
* How long should the capture run?
* What traffic can safely be collected?

A simple planning model is:

```text
Host
+
Interface
+
Event
+
Time window
+
Expected traffic
```

## Identify the Correct Interface

A system can have multiple network interfaces.

Examples include:

* Ethernet
* Wi-Fi
* Loopback
* VPN interface
* Virtual machine interface
* Docker or container interface
* Other virtual interfaces

The interface that is physically or logically involved in the communication is the one that matters.

Do not assume the first interface shown by Wireshark is automatically correct.

## Interface Selection Workflow

Before capturing:

1. Open Wireshark's capture interface selection.
2. Review the available interfaces.
3. Identify interfaces that are currently active.
4. Consider the expected communication path.
5. Select the interface most likely to contain the traffic.
6. Start a short test capture.
7. Generate known traffic.
8. Verify that the expected packets appear.

This is safer than starting a long capture and discovering later that the wrong interface was selected.

## Interface Activity Indicators

Wireshark may provide visual indications of interface activity.

Use these as clues, not as absolute proof.

An interface can appear active while the traffic you care about is occurring elsewhere.

The final test is:

```text
Expected traffic generated
        ↓
Expected traffic appears
```

If that does not happen, investigate the capture setup.

## Common Interface Types

### Ethernet

Usually represents a wired network connection.

Useful when the target traffic travels through the wired network interface.

### Wi-Fi

Represents wireless network connectivity.

Normal operating-system captures generally provide traffic associated with the host's network activity, but wireless-specific analysis can require additional hardware, driver, operating-system, and capture-mode support.

Do not assume that seeing a Wi-Fi interface means you can capture arbitrary wireless traffic.

### Loopback

Used for traffic that stays on the local host.

This is especially important when troubleshooting:

```text
Application
    ↓
Local service
```

rather than communication across the physical network.

### Virtual Interfaces

Virtual machines, containers, VPNs, and other software can create interfaces that carry traffic.

Examples include:

```text
VM interface
Container bridge
VPN tunnel interface
Virtual Ethernet pair
```

The correct interface depends on where the traffic exists in the network path.

## Capture Path Thinking

When unsure which interface to capture, think about the traffic path.

For example:

```text
Application
    ↓
Operating system
    ↓
VPN
    ↓
Physical interface
    ↓
Network
    ↓
Server
```

Different interfaces may expose different portions of that path.

Ask:

```text
Where does the traffic exist at the point I need to observe it?
```

That is a better question than:

```text
Which interface looks active?
```

## Test Before Long Captures

A short validation capture can prevent major mistakes.

Use this pattern:

```text
Start capture
    ↓
Generate known traffic
    ↓
Stop capture
    ↓
Find expected packet
```

For example, if investigating a web connection:

```text
Start capture
    ↓
Perform the authorized connection
    ↓
Stop capture
    ↓
Look for the expected traffic
```

If the expected traffic is absent, fix the capture setup before performing a long investigation.

## Capture Timing

Capture should begin before the event you are investigating.

If you start too late, you may miss:

* TCP handshake
* DNS query
* DHCP exchange
* TLS handshake
* Initial application request
* Authentication-related traffic

A useful rule is:

```text
Start slightly before the event.
Stop shortly after the event has completed or failed.
```

Do not capture indefinitely unless there is a specific reason.

## Generate Known Traffic

For a controlled investigation, deliberately generate the event you want to observe.

Examples include:

* Open an authorized test website
* Resolve a known test hostname
* Connect to a lab service
* Send a controlled request
* Start an application
* Reproduce a known failure

Known actions create known time windows.

This makes later analysis easier.

## Reproduce the Problem

When troubleshooting, the capture should contain the actual problem.

Use:

```text
Start capture
    ↓
Reproduce problem
    ↓
Stop capture
```

Avoid generating unrelated traffic between the start and the reproduction if possible.

For example:

```text
Start capture
    ↓
Immediately reproduce connection failure
    ↓
Stop capture
```

This reduces noise and improves correlation.

## Establish a Baseline

When troubleshooting, a baseline can be extremely useful.

Capture a known-good transaction first.

Then capture the failing transaction.

Conceptually:

```text
Known-good
    ↓
Compare
    ↓
Failing
```

Possible comparison points include:

* DNS response
* TCP handshake
* TLS handshake
* Application response
* Timing
* Retransmissions
* Connection termination

The baseline helps identify what changed.

## Capture Duration

The required duration depends on the question.

Short event:

```text
Capture for the event.
```

Intermittent problem:

```text
Capture long enough to reproduce the behavior.
```

Background issue:

```text
Use an appropriate capture strategy that avoids unnecessary data collection.
```

Do not use a huge capture simply because more packets exist.

More data can make analysis harder.

## Stop the Capture Deliberately

Stop the capture when:

* The target event has occurred
* The failure has been reproduced
* The relevant conversation is complete
* Enough evidence has been collected

Stopping at the right time reduces irrelevant traffic.

The workflow should be:

```text
Capture
    ↓
Event occurs
    ↓
Confirm enough evidence exists
    ↓
Stop
```

## Verify the Capture

Never assume a capture succeeded.

After stopping, check:

* Did packets arrive?
* Does the expected protocol appear?
* Does the expected host appear?
* Does the relevant time window exist?
* Is the packet count plausible?
* Is the target conversation present?
* Does the capture appear complete?

The first analysis question should often be:

```text
Did I actually capture the evidence I expected?
```

## Capture Validation Workflow

Use:

```text
1. Open or stop capture
2. Review packet count
3. Identify protocols
4. Identify expected endpoints
5. Search for the target event
6. Inspect representative packets
7. Confirm capture completeness
```

Only after this should detailed analysis begin.

## Expected Traffic Checklist

Before starting, write down what you expect.

Example:

```text
Expected event:
Client connects to HTTPS service

Expected traffic:
DNS
TCP
TLS
Application traffic
```

After capture, verify whether these actually appeared.

If they did not, that absence is itself a clue.

But first determine whether the capture setup could explain the absence.

## Capture Filters vs Display Filters

Capture filters and display filters operate at different stages.

Conceptually:

```text
Capture filter:
Controls what traffic is captured.

Display filter:
Controls what captured traffic is displayed.
```

The workflow is:

```text
Network traffic
    ↓
Capture filter
    ↓
Captured packets
    ↓
Display filter
    ↓
Visible packets
```

This distinction is fundamental.

## When to Use a Capture Filter

Use a capture filter when you have a clear reason to limit traffic before it is captured.

This can be useful when:

* The traffic volume is extremely high
* Storage is limited
* The investigation has a tightly defined scope
* You know exactly what traffic is relevant
* You need to reduce collection before capture

However, capture filters can permanently exclude traffic from the resulting capture.

That makes mistakes costly.

## When to Avoid an Aggressive Capture Filter

Avoid overly restrictive capture filters when:

* You are still discovering the problem
* You do not know which protocol is involved
* You may need surrounding traffic
* You are investigating an unfamiliar event
* You need broad evidence for later analysis

When uncertain, capturing broadly and narrowing later with display filters is often more flexible, subject to privacy, storage, and operational constraints.

## The Cost of a Wrong Capture Filter

Suppose you capture only one protocol:

```text
Capture:
Only selected traffic
```

Later you discover that the problem actually involved:

```text
DNS
    ↓
TCP
    ↓
TLS
```

If the capture excluded DNS, you cannot recover those packets from that capture.

This is why capture filtering should be deliberate.

## Capture Evidence Lifecycle

Treat the capture as evidence.

A useful lifecycle is:

```text
Plan
  ↓
Capture
  ↓
Validate
  ↓
Preserve
  ↓
Analyze
  ↓
Document
```

Do not modify the original evidence unnecessarily.

If you need a working copy, preserve the original and analyze the copy.

## Saving a Capture

Save the capture in an appropriate PCAP/PCAPNG format according to your workflow.

Use a meaningful filename.

A useful naming structure is:

```text
<date>_<case-or-purpose>_<host-or-event>.<format>
```

For example:

```text
2026-09-25_dns-troubleshooting_client01.pcapng
```

Avoid names such as:

```text
capture1.pcapng
test.pcapng
new.pcapng
final-final.pcapng
```

Good naming improves evidence management.

## Preserve the Original Capture

When the capture is important:

```text
Original capture
    ↓
Preserved
    ↓
Working copy
    ↓
Analysis
```

Avoid repeatedly overwriting the original.

This becomes especially important for incident response, troubleshooting records, and forensic workflows.

## Record Capture Context

A capture without context can become difficult to interpret later.

Record information such as:

```text
Purpose:
Target host:
Capture interface:
Start time:
Stop time:
Reproduction steps:
Expected traffic:
Observed traffic:
Capture limitations:
File name:
```

This does not need to be complicated.

The purpose is reproducibility.

## Capture Notes Example

```text
Purpose:
Investigate intermittent HTTPS connection failure.

Target:
Authorized lab client and test server.

Interface:
Interface carrying the client-to-server traffic.

Capture window:
Started immediately before reproducing the issue.

Reproduction:
Initiated the HTTPS connection three times.

Expected:
DNS resolution, TCP connection, TLS handshake.

Observed:
Capture contains DNS and TCP traffic but the expected TLS exchange is incomplete.

Limitation:
Capture was performed on a host interface rather than at the server.
```

The final limitation is important.

The capture location affects what conclusions are possible.

## Capture Location Matters

The same network event can look different depending on where you capture it.

For example:

```text
Client
  ↓
Switch
  ↓
Firewall
  ↓
Server
```

A capture at the client may show:

```text
Client-side perspective
```

A capture at the server may show:

```text
Server-side perspective
```

These perspectives can differ because of:

* Routing
* NAT
* Firewall behavior
* VPNs
* Load balancers
* Packet loss
* Network segmentation
* Capture limitations

Always document where the evidence was collected.

## Host Capture vs Network Capture

A capture performed directly on an endpoint observes traffic available to that endpoint's capture mechanism.

A network-level capture may provide a different perspective.

Do not assume:

```text
I did not see the packet
```

means:

```text
The packet never existed anywhere on the network.
```

It may mean:

```text
The packet was not visible at my capture point.
```

This distinction is essential.

## Promiscuous Mode

Promiscuous mode can affect which frames a network interface makes available for capture, depending on the network environment and interface behavior.

However, enabling promiscuous mode does not magically provide visibility into all network traffic.

For example, switched networks generally do not send every host's unicast traffic to every other port.

Therefore:

```text
Promiscuous mode
≠
Capture everything on the network
```

Capture visibility depends on the network path and capture location.

## Virtualized Environments

Virtual machines can introduce multiple network paths.

For example:

```text
VM
 ↓
Virtual NIC
 ↓
Hypervisor
 ↓
Host NIC
 ↓
Network
```

The relevant traffic may appear at different layers depending on where the capture is performed.

When troubleshooting a VM:

1. Identify the VM interface.
2. Determine its network mode.
3. Verify where the traffic exits.
4. Perform a short test capture.
5. Confirm the expected packets appear.

## Containers

Containers can also introduce multiple interfaces and bridges.

Conceptually:

```text
Container
    ↓
Virtual interface
    ↓
Bridge
    ↓
Host network
```

If the traffic does not appear where expected, investigate the container networking path before assuming the application is not communicating.

## VPN Traffic

VPNs can create another capture consideration.

You may see:

```text
Application traffic
    ↓
VPN interface
    ↓
Encrypted tunnel
    ↓
Physical interface
```

Capturing at different interfaces can expose different representations of the same communication.

One location may show the original inner traffic.

Another may show encrypted tunnel traffic.

Do not assume that the absence of application-layer visibility means the application generated no traffic.

## Wireless Capture Limitations

Wireless packet capture requires more than simply having Wireshark installed.

Depending on the goal, useful wireless capture may depend on:

* Wireless adapter capabilities
* Driver support
* Operating-system support
* Capture mode
* Channel configuration
* Encryption
* Network environment

For ordinary host troubleshooting, you may primarily see traffic associated with the local system.

For specialized wireless analysis, additional capture capabilities may be required.

Do not confuse:

```text
Wi-Fi connectivity
```

with:

```text
Full wireless protocol visibility
```

## Permissions and Capture Access

A capture may fail because the operating system does not permit the capture mechanism to access the selected interface.

Symptoms may include:

* Interface unavailable
* Permission errors
* Capture failing to start
* No packets despite known traffic

The correct response is to troubleshoot the capture environment rather than changing analysis filters blindly.

## No Packets Captured

If a capture contains no packets, investigate systematically.

Use:

```text
1. Is the correct interface selected?
2. Is the interface active?
3. Is traffic actually being generated?
4. Does the operating system permit capture?
5. Is a capture filter excluding everything?
6. Is the traffic occurring on another interface?
7. Is virtualization or VPN routing involved?
```

Do not immediately conclude:

```text
There is no network traffic.
```

The capture setup may simply be wrong.

## Wrong Interface

A common failure is selecting an interface that is not carrying the relevant traffic.

Example:

```text
Expected:
Wi-Fi traffic

Selected:
VPN interface
```

or:

```text
Expected:
Container traffic

Selected:
Physical host interface
```

The solution is to trace the network path and validate with a short capture.

## Capture Filter Excluded the Traffic

If you used a capture filter, verify that it was not too restrictive.

A useful troubleshooting method is:

```text
Remove unnecessary capture restrictions
    ↓
Run a short test capture
    ↓
Generate known traffic
    ↓
Verify visibility
```

Do not keep debugging a filter when the underlying problem is interface selection.

## Large Captures

Large captures can become difficult to handle.

Potential problems include:

* High memory usage
* Slow filtering
* Difficult manual navigation
* Large storage requirements
* Increased analysis time

The solution is not always to capture less.

Better approaches may include:

* Shorter capture windows
* Appropriate capture filters
* Display filters
* Targeted reproduction
* Capture rotation where appropriate
* Command-line preprocessing when needed

The correct choice depends on the investigation.

## Capture Rotation

For long-running collection, capture rotation can help prevent a single capture file from becoming excessively large.

The exact configuration depends on the capture workflow and environment.

The important concept is:

```text
Long-running collection
    ↓
Manageable capture segments
```

This can improve storage management and analysis.

## Snapshot Length

Capture mechanisms can be configured to capture only a portion of each packet in some workflows.

This can reduce storage requirements but can also remove application data or other evidence.

The tradeoff is:

```text
Smaller capture
    ↔
Less available packet content
```

Do not use a reduced snapshot length when you need full packet content.

## Validate Packet Completeness

When packet contents appear incomplete, determine whether:

* The original packet was short
* The capture truncated it
* The capture configuration limited the captured length
* The protocol itself carried no additional data

This prevents false conclusions.

## Checksum Offloading

Endpoint captures can sometimes show checksum-related information that does not represent what actually traversed the physical network because of hardware or software offloading behavior.

When investigating checksum warnings, consider:

```text
Application/OS
    ↓
Network stack
    ↓
Offloading
    ↓
Capture point
    ↓
Network
```

Do not automatically conclude that a checksum warning proves corrupted network traffic.

First understand where and how the packet was captured.

## Name Resolution

Wireshark can perform name resolution depending on configuration and available information.

Name resolution can make captures easier to read, but it can also change how endpoints are displayed.

For evidence-oriented analysis, be aware of the difference between:

```text
Observed address
```

and:

```text
Resolved name
```

When documenting evidence, retain the underlying address information when relevant.

## Timestamp Considerations

Capture timestamps can be affected by:

* Host clock configuration
* Timestamp source
* Capture environment
* Synchronization
* Time display settings

For relative timing analysis, compare packets within the same capture carefully.

For multi-capture correlation, verify that the clocks and timestamp assumptions are compatible.

## Capture Validation Questions

After every important capture, ask:

```text
Did I capture the correct interface?

Did I capture the relevant time window?

Did I reproduce the event?

Did the expected traffic appear?

Could a capture filter have removed useful evidence?

Was the packet content captured completely?

Where was the capture taken?

What traffic could I not observe from this location?
```

These questions should become routine.

## Capture Troubleshooting Decision Tree

Use this sequence when the capture does not contain what you expected:

```text
Expected traffic missing
        ↓
Was traffic actually generated?
        ↓
No ──→ Reproduce the event
        │
        Yes
        ↓
Correct interface?
        ↓
No ──→ Select the interface carrying the traffic
        │
        Yes
        ↓
Capture permissions working?
        ↓
No ──→ Fix capture access
        │
        Yes
        ↓
Capture filter too restrictive?
        ↓
Yes ──→ Remove or broaden it
        │
        No
        ↓
Virtualization/VPN/container path?
        ↓
Yes ──→ Identify the actual capture point
        │
        No
        ↓
Traffic may not be visible at this capture location
```

## Capture Before Analysis

A strong analyst does not begin by applying random filters to an empty or irrelevant capture.

The workflow should always be:

```text
Question
    ↓
Capture plan
    ↓
Correct capture point
    ↓
Short validation
    ↓
Targeted reproduction
    ↓
Evidence validation
    ↓
Detailed analysis
```

## Practical Capture Exercise 1: Known DNS Traffic

Use an authorized system and a controlled hostname.

Perform:

```text
Start capture
    ↓
Generate DNS activity
    ↓
Stop capture
```

Then verify:

* The DNS traffic exists.
* The expected client is visible.
* The expected DNS server is visible.
* The query is present.
* A response is present when expected.

The objective is to validate the entire capture workflow.

## Practical Capture Exercise 2: TCP Connection

Capture an authorized connection to a known test service.

Verify:

```text
Connection attempt
    ↓
TCP handshake
    ↓
Subsequent traffic
```

Do not analyze the protocol deeply yet.

The objective is simply to prove that you captured the expected connection.

## Practical Capture Exercise 3: Failed Connection

Use a controlled environment where a connection failure can be safely reproduced.

Start the capture before the attempt.

Then:

```text
Reproduce failure
    ↓
Stop capture
    ↓
Verify evidence
```

Ask:

```text
Did the client send traffic?

Did the server respond?

Was the failure visible?

Did the capture begin early enough?

```

Detailed failure analysis comes later.

## Practical Capture Exercise 4: Wrong Interface

If your environment has multiple active interfaces:

1. Select one interface.
2. Generate known traffic.
3. Capture briefly.
4. Determine whether the expected traffic appears.
5. Repeat with another relevant interface if necessary.

The objective is to develop interface-selection intuition.

## Practical Capture Exercise 5: Baseline vs Failure

Create two captures:

```text
Known-good transaction
Failing transaction
```

Record:

```text
Capture point
Interface
Time
Expected behavior
Observed behavior
```

Do not try to explain the failure yet.

Simply confirm that both captures contain enough evidence for later comparison.

## Practical Capture Exercise 6: Capture Completeness

Choose a capture where packet length information can be inspected.

Determine whether:

```text
Captured length
```

and:

```text
Original packet length
```

differ.

Then answer:

```text
Could missing bytes affect the investigation?
```

## Practical Capture Exercise 7: Capture Context

Perform a short controlled capture.

Create a capture note containing:

```text
Purpose:
Target:
Interface:
Start:
Stop:
Reproduction:
Expected traffic:
Observed traffic:
Limitations:
File:
```

This builds the habit of treating captures as evidence rather than disposable files.

## Capture Quality Checklist

Before analysis:

```text
[ ] Investigation question defined
[ ] Target identified
[ ] Correct interface selected
[ ] Capture permissions verified
[ ] Capture started before event
[ ] Relevant event reproduced
[ ] Capture stopped after event
[ ] Expected traffic confirmed
[ ] Capture completeness considered
[ ] Capture location recorded
[ ] Important limitations recorded
[ ] Original evidence preserved
```

## What You Should Be Able to Do

After completing this file, you should be able to:

* Start a capture deliberately
* Choose a likely correct interface
* Validate interface selection with known traffic
* Reproduce a controlled event
* Capture the relevant time window
* Stop a capture intentionally
* Verify that expected traffic was captured
* Recognize when capture filters can remove evidence
* Understand capture location limitations
* Recognize virtualization and VPN capture considerations
* Understand basic wireless capture limitations
* Recognize permission-related capture failures
* Recognize empty or irrelevant captures
* Consider packet completeness
* Preserve useful capture evidence
* Record enough context for later analysis

## Completion Criteria

You are ready to continue when you can independently perform this workflow:

```text
Define question
    ↓
Identify target
    ↓
Identify capture interface
    ↓
Run a short validation capture
    ↓
Generate known traffic
    ↓
Confirm visibility
    ↓
Capture the real event
    ↓
Stop at the appropriate time
    ↓
Validate the evidence
    ↓
Preserve the capture
    ↓
Begin analysis
```

The key skill is not simply:

```text
"I know how to start Wireshark."
```

It is:

```text
"I can deliberately collect the evidence required
to answer a network question."
```

That is the foundation of reliable packet analysis.
