# Capture Troubleshooting

## Objective

A packet capture can fail in many ways before packet analysis even begins.

Common problems include:

* No interfaces available
* Wrong interface selected
* Permission problems
* No packets captured
* Expected traffic missing
* Capture filter excluding traffic
* Virtual machine traffic appearing elsewhere
* Container traffic using another interface
* VPN traffic being observed at the wrong layer
* Wireless visibility limitations
* Truncated packets
* Packet loss
* Large captures becoming difficult to manage
* Unexpected checksum warnings
* Timestamp confusion
* Capture working but not providing the expected evidence

The objective of this file is to troubleshoot the **capture process itself**.

The central principle is:

```text id="z3tqak"
Unexpected capture
    ↓
Verify the capture environment
    ↓
Verify the traffic
    ↓
Verify the capture point
    ↓
Verify filtering
    ↓
Verify packet completeness
    ↓
Only then analyze the network behavior
```

## Capture Troubleshooting Mental Model

When expected traffic is missing, do not immediately conclude that the network behavior is missing.

Use:

```text id="3vyd9u"
Expected traffic
      ↓
Was it generated?
      ↓
Was the correct interface selected?
      ↓
Was capture access available?
      ↓
Was the traffic visible at that capture point?
      ↓
Was a capture filter excluding it?
      ↓
Was the packet captured completely?
      ↓
Is the traffic encrypted or encapsulated?
```

This prevents analysis mistakes caused by incorrect collection.

## The First Question

When a capture looks wrong, ask:

```text id="u1yqz4"
Did Wireshark fail to capture the traffic,
or did the traffic never reach this capture point?
```

These are different problems.

The investigation should determine which one is more likely.

## Problem Classification

Most capture problems can be placed into one of these categories:

```text id="p8h4fb"
1. Interface problem
2. Permission problem
3. Traffic-generation problem
4. Filtering problem
5. Capture-point problem
6. Environment problem
7. Capture-completeness problem
8. Performance problem
9. Interpretation problem
```

Classify the problem before changing multiple settings.

## Troubleshooting Workflow

Use this general workflow:

```text id="h8b8gu"
Problem observed
    ↓
Define expected behavior
    ↓
Generate known traffic
    ↓
Check interface
    ↓
Check permissions
    ↓
Check capture filter
    ↓
Check capture point
    ↓
Check packet visibility
    ↓
Check packet completeness
    ↓
Check environment-specific limitations
    ↓
Repeat short validation capture
```

## Problem: No Interfaces Are Available

If Wireshark does not show usable capture interfaces, possible causes include:

* Capture support not installed correctly
* Required capture component unavailable
* Insufficient permissions
* Interface state problems
* Operating-system restrictions
* Virtualization or driver issues

Do not begin with display filters.

There are no packets to filter if capture access itself is not working.

## Interface Troubleshooting

First identify what interfaces the operating system itself sees.

Then compare that with what Wireshark exposes.

Ask:

```text id="r4k8fg"
Does the operating system have the interface?

Is the interface enabled?

Can normal network traffic use it?

Does Wireshark expose it?

Can a short capture be started?
```

This separates network configuration problems from Wireshark capture problems.

## Problem: Wrong Interface

This is one of the most common capture mistakes.

Symptoms may include:

* Packets exist, but not the expected packets
* Only unrelated traffic appears
* DNS appears but application traffic does not
* Host traffic is absent
* Capture appears active but contains irrelevant traffic

The solution is to trace the expected traffic path.

## Interface Selection Test

Use:

```text id="4rqw1j"
Select candidate interface
    ↓
Start short capture
    ↓
Generate known traffic
    ↓
Stop capture
    ↓
Check for expected packet
```

Repeat for another candidate interface if necessary.

Do not rely solely on interface activity indicators.

## Problem: No Packets Captured

If the capture starts successfully but contains no packets:

```text id="m58o4d"
Check:
1. Correct interface?
2. Interface active?
3. Traffic generated?
4. Capture permissions?
5. Capture filter?
6. Traffic actually present on this interface?
7. Virtual/VPN/container path?
```

Work from the simplest explanation to the more complex one.

## Problem: Expected Traffic Missing

Suppose you generate an HTTP request but do not see the expected traffic.

Do not immediately conclude:

```text id="82jz6r"
The HTTP request did not happen.
```

Check:

```text id="j6f8ob"
Was DNS involved?

Was the connection made over TLS instead?

Did the application use a proxy?

Was traffic routed through a VPN?

Was the wrong interface captured?

Was the application communicating over IPv6?

Did a capture filter exclude the traffic?
```

The application may have communicated differently than expected.

## Problem: Capture Filter Excluded Traffic

If a capture filter was used, temporarily remove it in a controlled test.

Then:

```text id="5xq16f"
Start broad capture
    ↓
Generate known traffic
    ↓
Verify traffic exists
```

If the traffic appears without the filter, investigate the filter logic.

## Capture Filter vs Display Filter Troubleshooting

Remember:

```text id="24h7g1"
Capture filter
    ↓
Controls collection
```

while:

```text id="m9qzv3"
Display filter
    ↓
Controls visibility
```

If a packet is missing from the capture entirely, a display filter cannot bring it back.

If a packet is present but hidden, clearing the display filter can reveal it.

## Problem: Display Filter Makes Packets Disappear

This is not necessarily a capture problem.

First clear the display filter.

Then verify whether the packets return.

If they do:

```text id="r4f5cm"
Capture is intact.
Filter logic needs investigation.
```

If they do not:

```text id="3okg7h"
Investigate whether the traffic was captured at all.
```

## Problem: Wrong Address

The expected host may have:

* A different IPv4 address
* An IPv6 address
* A dynamic address
* A VPN address
* A container address
* A virtual-machine address

Verify the actual addresses used during the event.

Do not rely on an old address assumption.

## Problem: IPv6 Traffic Was Missed

An application may prefer IPv6.

You may expect:

```text id="m7m8x2"
IPv4
```

but the actual communication may use:

```text id="e6o2p8"
IPv6
```

If the capture scope is limited to IPv4, the expected communication may appear to be missing.

When troubleshooting:

```text id="p85l8v"
Check both address families when appropriate.
```

## Problem: Loopback Traffic Is Missing

Applications communicating with services on the same machine may use a loopback interface.

Example:

```text id="b8v1gq"
Application
    ↓
127.0.0.1 / local IPv6 loopback
    ↓
Local service
```

If you capture only the physical network interface, you may not see this traffic.

Check the loopback capture capability and relevant interface.

## Problem: Virtual Machine Traffic Is Missing

Virtual machines introduce additional network layers.

For example:

```text id="3v2w7k"
VM
 ↓
Virtual NIC
 ↓
Virtual switch
 ↓
Host interface
 ↓
Physical network
```

A packet may be visible at one layer but not another depending on the capture location.

Determine:

```text id="xk20fo"
VM network mode
+
Relevant virtual interface
+
Host interface
```

Then validate with a short capture.

## Problem: Container Traffic Is Missing

Containers may communicate through:

```text id="9d3r2s"
Container interface
    ↓
Virtual bridge
    ↓
Host
    ↓
Physical network
```

Traffic destined for another local container may never appear on the physical interface.

When troubleshooting container networking:

```text id="08m1dx"
Identify container interface
Identify bridge
Identify host path
Capture at the relevant point
```

## Problem: VPN Traffic Is Missing

VPNs can change the visible traffic path.

Conceptually:

```text id="2yqcv8"
Application
    ↓
VPN interface
    ↓
Encrypted tunnel
    ↓
Physical interface
```

A capture on the VPN interface may expose inner traffic.

A capture on the physical interface may expose encrypted tunnel traffic.

Neither necessarily provides the same evidence.

## Problem: Wireless Traffic Is Missing

Wireless capture has additional constraints.

Normal host capture may show traffic associated with the local machine.

Specialized wireless analysis may require:

* Compatible hardware
* Driver support
* Appropriate capture mode
* Channel control
* Operating-system support
* Proper permissions

Do not assume that a Wi-Fi interface allows capture of arbitrary nearby wireless traffic.

## Problem: Permission Denied

If capture cannot start because of permissions:

1. Identify the operating system.
2. Check whether the capture component has required access.
3. Verify that the interface can be accessed.
4. Follow the platform's supported permission configuration.
5. Retry a short capture.

Avoid changing unrelated security settings just to make capture work.

Use the minimum access required.

## Problem: Capture Starts but Immediately Stops

Possible causes include:

* Interface failure
* Driver issue
* Permission problem
* Resource issue
* Capture backend problem
* Interface state change

Test with another interface if available.

Then perform a short capture and observe whether the problem is reproducible.

## Problem: Capture Is Extremely Large

Large captures can be difficult to analyze.

Possible solutions include:

* Narrowing the capture scope
* Shortening the capture window
* Using a validated capture filter
* Capturing only during reproduction
* Using capture rotation for long-running collection
* Analyzing in stages
* Using TShark for targeted extraction later

Do not automatically delete traffic simply to make analysis easier.

First determine what evidence is necessary.

## Problem: Capture Is Too Small

A capture can also be suspiciously small.

Possible causes include:

* Wrong interface
* Capture filter too restrictive
* Event never occurred
* Short capture duration
* Capture started too late
* Packet loss
* Traffic occurring elsewhere

A tiny capture is not automatically better evidence.

## Problem: Packet Loss During Capture

Packet loss can occur under high traffic rates or resource pressure.

Potential contributing factors include:

* High packet rate
* Limited processing capacity
* Storage performance
* Capture-buffer limitations
* Interface or driver behavior
* Virtualization overhead

When packet loss is suspected, investigate capture statistics and environment constraints.

Do not interpret missing packets as network behavior until capture loss has been considered.

## Capture Loss vs Network Loss

This distinction is critical.

### Network loss

Packets may be dropped somewhere in the network path.

### Capture loss

Packets may have existed but not been recorded by the capture mechanism.

The PCAP alone may not always distinguish these.

Therefore:

```text id="tvnq6w"
Missing packet
    ≠
Proven network packet loss
```

You need appropriate evidence to distinguish them.

## Problem: Truncated Packets

If a packet is only partially captured, inspect:

* Captured length
* Original length
* Capture configuration
* Packet bytes

A truncated packet can make application analysis incomplete.

Record this limitation.

## Problem: Unexpected Checksum Warnings

Endpoint captures can show checksum warnings because of checksum offloading.

A packet may be captured before the network hardware calculates the final checksum.

Therefore:

```text id="pp3d4g"
Checksum warning
    ≠
Automatically corrupted network packet
```

Consider:

* Where the capture was taken
* Whether checksum offloading is enabled
* Whether the warning occurs consistently
* Whether the packet was observed on the wire or before transmission processing

Use additional evidence before concluding there is a network integrity problem.

## Problem: Malformed Packet Warnings

If Wireshark reports malformed or invalid protocol data:

1. Inspect the packet structure.
2. Check captured length.
3. Inspect raw bytes.
4. Compare with surrounding packets.
5. Determine whether the protocol is known.
6. Consider dissector limitations.
7. Check whether the capture is complete.

Do not immediately classify the traffic as malicious.

## Problem: Protocol Not Decoded

Possible causes include:

* Unsupported protocol
* Encrypted payload
* Non-standard port
* Encapsulation
* Incomplete packet
* Malformed traffic
* Dissector limitation

Use the workflow:

```text id="s9yqwm"
Protocol not decoded
    ↓
Check transport ports
    ↓
Check packet structure
    ↓
Inspect raw bytes
    ↓
Inspect surrounding packets
    ↓
Determine encryption/encapsulation
    ↓
Investigate further
```

## Problem: Wrong Protocol Dissection

Sometimes traffic may be interpreted differently depending on protocol context or port information.

When the decoded protocol appears incorrect:

* Inspect packet structure
* Check ports
* Examine payload
* Review surrounding packets
* Determine whether protocol dissection can be adjusted appropriately

Do not force a protocol interpretation simply because it looks convenient.

## Problem: Encrypted Traffic Appears Unreadable

Encryption may prevent direct application-content inspection.

That does not mean the capture is useless.

You may still analyze:

* Endpoints
* Ports
* Timing
* Handshake behavior
* Packet sizes
* Connection attempts
* Certificate information when visible
* Other protocol metadata

State clearly what the capture does and does not reveal.

## Problem: Application Uses a Proxy

An application may not communicate directly with the destination you expected.

The traffic path may be:

```text id="hjf1w2"
Application
    ↓
Proxy
    ↓
Destination
```

The capture may therefore show the proxy as the network peer.

Do not assume the destination based solely on application intent.

Inspect actual network traffic.

## Problem: NAT Changes Addresses

NAT can change the visible source or destination address between network locations.

For example:

```text id="zjv9m5"
Internal client
    ↓
NAT
    ↓
External address
```

A client-side capture and an external capture may therefore show different addresses.

Always consider capture location.

## Problem: Timestamps Look Wrong

If timestamps appear unexpected:

* Check the selected time display
* Consider the capture timestamp source
* Consider host clock configuration
* Compare relative timing between packets
* Avoid relying on absolute time without context

For multi-system correlation, verify clock assumptions.

## Problem: Name Resolution Looks Wrong

If Wireshark displays names that seem unexpected:

* Check whether name resolution is enabled
* Compare with the underlying address
* Determine whether the name is derived from local configuration, DNS, or another mechanism

For evidence, retain the actual observed address when relevant.

## Problem: Capture Contains Too Much Background Traffic

Background traffic can include:

* Broadcasts
* Multicast
* Discovery traffic
* Routine application traffic
* Operating-system services

This is not necessarily a problem.

Use display filtering and analysis views to reduce noise.

If future collection can be more targeted, consider whether a validated capture filter is appropriate.

## Problem: The Capture Is Correct but the Question Is Wrong

Sometimes the capture contains exactly what it should.

The problem is the investigation approach.

For example:

```text id="iyb7xg"
Question:
Why is the application slow?

Current analysis:
Only inspecting TCP flags.
```

The capture may require:

* Timing analysis
* Application response inspection
* DNS analysis
* TCP behavior
* Server-side comparison

Before changing the capture, reconsider the question.

## Capture Troubleshooting Decision Tree

Use this general decision tree:

```text id="z9j8gr"
Expected traffic missing
        ↓
Was the event actually generated?
        ↓
No → Reproduce it
        ↓
Yes
        ↓
Correct interface?
        ↓
No → Select the correct interface
        ↓
Yes
        ↓
Capture access working?
        ↓
No → Fix permissions/capture support
        ↓
Yes
        ↓
Capture filter applied?
        ↓
Yes → Remove/review filter and retest
        ↓
No
        ↓
Traffic visible at capture point?
        ↓
No → Check routing/VPN/NAT/VM/container path
        ↓
Yes
        ↓
Packet complete?
        ↓
No → Check truncation/snapshot length
        ↓
Yes
        ↓
Capture loss possible?
        ↓
Yes → Check capture performance/statistics
        ↓
No
        ↓
Investigate protocol/application behavior
```

## Minimal Reproduction Method

When troubleshooting capture problems, simplify.

Use:

```text id="u4j6we"
One host
One interface
One event
One short capture
```

Then generate known traffic.

If that works, gradually reintroduce complexity.

This is usually faster than changing multiple variables at once.

## Controlled Capture Test

A useful test is:

```text id="l2w4h9"
Start capture
    ↓
Generate known traffic
    ↓
Stop capture
    ↓
Find known packet
```

For example:

```text id="z4t3mc"
Known test connection
```

If the expected packet is visible, the capture mechanism is probably functioning for that scenario.

## Compare Interfaces

If several interfaces may be relevant:

```text id="9cyxgd"
Interface A
    ↓
Short test

Interface B
    ↓
Short test

Interface C
    ↓
Short test
```

Compare the results.

Do not make the capture unnecessarily long.

## Compare Capture Locations

For difficult problems, evidence from different points can be useful.

Conceptually:

```text id="q7d4dy"
Client capture
      +
Server capture
      ↓
Compare
```

This can help distinguish:

* Client-side behavior
* Network-path behavior
* Server-side behavior

Be aware that simultaneous captures require appropriate time correlation.

## Capture Troubleshooting Notes

Use:

```text id="8o1i9n"
Problem:
Expected behavior:
Observed behavior:
Interface:
Capture filter:
Test performed:
Result:
Likely capture limitation:
Next action:
```

This prevents repeated troubleshooting of the same issue.

## Practical Exercise 1: No Packets

In an authorized lab:

1. Select a valid interface.
2. Start a short capture.
3. Generate known traffic.
4. Stop the capture.
5. Confirm packets appear.

Then deliberately test a non-relevant interface if your environment allows it.

Compare the results.

The goal is to recognize wrong-interface symptoms.

## Practical Exercise 2: Capture Filter Failure

Create a deliberately restrictive filter in a controlled environment.

Generate traffic that does not match it.

Observe that the traffic is absent from the capture.

Then remove the filter.

Repeat the capture.

The objective is to understand why capture filtering can create irreversible evidence gaps.

## Practical Exercise 3: Loopback

Use an authorized local service.

Generate traffic between the local client and local service.

Determine which interface carries the traffic.

Capture it.

Then compare with a capture on the physical interface.

The goal is to understand that local traffic may never leave the host.

## Practical Exercise 4: Virtual Machine

In an authorized virtual machine environment:

1. Identify the VM interface.
2. Determine its network mode.
3. Generate known traffic.
4. Capture from the VM.
5. If possible, compare with a host-side capture.
6. Determine which traffic is visible at each point.

Record the difference.

## Practical Exercise 5: VPN

If you have an authorized VPN test environment:

1. Identify the VPN interface.
2. Identify the physical interface.
3. Generate known traffic through the VPN.
4. Capture from the relevant interfaces.
5. Compare the visible traffic.

Focus on:

```text id="0cz4ry"
Inner traffic
vs
Tunnel traffic
```

## Practical Exercise 6: Truncation

Use a controlled capture where packet-capture length can be varied.

Compare:

```text id="5xg2lo"
Full capture
vs
Reduced capture length
```

Inspect the effect on packet details and raw bytes.

The goal is to understand why incomplete packet evidence limits analysis.

## Practical Exercise 7: Capture Performance

Use a controlled high-volume environment only where appropriate.

Observe whether the capture reports packet loss or other resource limitations.

Record:

```text id="q3v6ck"
Traffic rate:
Capture behavior:
Observed loss:
Capture environment:
```

Do not mistake capture loss for network loss.

## Practical Exercise 8: Complete Capture Troubleshooting

Start with this problem:

```text id="9kgmsp"
"I generated traffic, but Wireshark does not show what I expected."
```

Do not immediately ask for a new capture.

Work through:

```text id="g91gjh"
Event
 ↓
Interface
 ↓
Permissions
 ↓
Filter
 ↓
Capture point
 ↓
Environment
 ↓
Completeness
 ↓
Performance
```

Document the root cause if you can determine it.

## Capture Troubleshooting Checklist

When a capture is unexpected:

```text id="6b5l4y"
[ ] Event actually generated
[ ] Correct interface selected
[ ] Interface active
[ ] Capture access working
[ ] No accidental capture filter
[ ] Both address families considered
[ ] Loopback considered
[ ] VM/container path considered
[ ] VPN path considered
[ ] NAT considered
[ ] Wireless limitations considered
[ ] Capture point documented
[ ] Packet loss considered
[ ] Truncation considered
[ ] Checksum offloading considered
[ ] Timestamp assumptions considered
[ ] Protocol dissection checked
[ ] Encryption considered
```

## What You Should Be Able to Do

After completing this file, you should be able to:

* Diagnose an empty capture
* Diagnose a wrong-interface capture
* Distinguish capture problems from network problems
* Validate capture permissions
* Identify capture-filter mistakes
* Reason about IPv4 and IPv6 visibility
* Handle loopback captures
* Reason about VM and container interfaces
* Reason about VPN capture locations
* Understand basic wireless capture limitations
* Recognize capture loss
* Recognize packet truncation
* Interpret checksum warnings carefully
* Investigate malformed or undecoded packets
* Recognize encryption-related visibility limitations
* Investigate timestamp and name-resolution issues
* Reduce capture troubleshooting to a controlled reproduction

## Completion Criteria

You are ready to move forward when an unexpected capture no longer leads immediately to:

```text id="c0u5g5"
"Wireshark is broken."
```

Instead, your first response should be:

```text id="5v7a0k"
What exactly did I expect?
        ↓
Did I generate it?
        ↓
Where should it have appeared?
        ↓
Did I capture that interface?
        ↓
Could filtering exclude it?
        ↓
Could the traffic be elsewhere?
        ↓
Could capture limitations explain it?
```

The final mental model is:

```text id="1q0g4q"
Capture troubleshooting
    =
Proving that the evidence collection process
is capable of answering the question.
```

Only after that proof should detailed packet analysis begin.
