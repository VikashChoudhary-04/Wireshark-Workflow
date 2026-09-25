# Capture Filters

## Objective

Capture filters control which traffic Wireshark captures.

They operate before the packet becomes part of the capture being analyzed.

The key mental model is:

```text
Network traffic
    ↓
Capture filter
    ↓
Captured evidence
    ↓
Display filter
    ↓
Visible packets
```

A capture filter can reduce unnecessary traffic, but an overly restrictive filter can permanently remove evidence from the capture.

The objective of this file is to make capture filters a deliberate evidence-collection decision rather than a collection of memorized expressions.

## Capture Filters vs Display Filters

This distinction must become automatic.

### Capture Filter

A capture filter determines what traffic is collected.

```text
Traffic
    ↓
Capture filter
    ↓
PCAP
```

### Display Filter

A display filter determines what is shown from traffic that has already been captured.

```text
PCAP
    ↓
Display filter
    ↓
Visible packets
```

The most important consequence is:

```text
A display filter hides packets temporarily.

A capture filter can prevent packets from ever entering the capture.
```

## Why This Matters

Suppose an investigation requires:

```text
DNS
+
TCP
+
TLS
```

If you capture only TCP:

```text
DNS
  ✗
TCP
  ✓
TLS
  ✓
```

you cannot later recover the missing DNS packets from that capture.

By contrast, if you capture all relevant traffic and later apply a DNS display filter, you can return to the full capture whenever necessary.

## When Capture Filters Are Useful

Capture filters are especially useful when:

* Traffic volume is very high
* Storage is constrained
* The scope is precisely known
* Only a small class of traffic matters
* A long-running capture needs to be controlled
* You have a well-defined collection objective

They are less appropriate when the investigation is exploratory.

## When to Prefer Broad Capture

Prefer broad capture followed by display filtering when:

* You do not know the cause of the problem
* Multiple protocols may be involved
* You need context around an event
* You are investigating an unfamiliar system
* You expect the problem to cross protocol layers
* You may need to investigate unexpected traffic later

A useful rule is:

```text
Uncertain question → capture broadly, filter later.

Narrow question → capture selectively when justified.
```

This is not an absolute rule.

Privacy, storage, legal, operational, and performance constraints can require narrower collection.

## Capture Filter Syntax

Wireshark uses a capture-filter syntax based on the packet-capture filtering language used by libpcap and related capture mechanisms.

This syntax is different from Wireshark's display-filter language.

Do not assume that an expression that works in the display filter bar will work as a capture filter.

Keep the two languages mentally separate:

```text
Capture filter language
        ≠
Display filter language
```

## Start With the Question

Do not begin by writing syntax.

Begin with:

```text
What traffic do I need to capture?
```

Then translate that requirement into a capture-filter condition.

For example:

```text
Question:
Capture traffic involving one host.
```

Then:

```text
Requirement:
Traffic where that host is a source or destination.
```

Then:

```text
Capture filter:
An expression matching the host's network address.
```

This question-first approach reduces syntax mistakes.

## Capture Filter Building Blocks

Capture filters commonly reason about:

* Hosts
* Networks
* Ports
* Protocols
* Source
* Destination
* Logical combinations

The exact syntax should be verified against Wireshark/libpcap documentation for the environment being used.

The important skill is constructing the requirement correctly.

## Host Filtering

A host filter limits capture to traffic associated with a particular network address.

Conceptually:

```text
Host A
  ↕
Traffic
```

The question should be:

```text
Do I need traffic involving this host?
```

If yes, the filter should reflect both relevant directions when the investigation requires them.

Do not accidentally capture only one direction when the response traffic matters.

## Source vs Destination

Direction matters.

Conceptually:

```text
Source
    ↓
Destination
```

A source-specific condition means:

```text
Traffic originating from the selected host.
```

A destination-specific condition means:

```text
Traffic going toward the selected host.
```

An investigation involving a complete conversation usually requires both directions.

## Network Filtering

You can also scope capture to a network rather than one host.

This is useful when:

* Investigating traffic within a known subnet
* Monitoring a controlled lab segment
* Limiting collection to a specific network boundary

The important question is:

```text
Is the network boundary actually where the relevant traffic exists?
```

A correct-looking network filter can still produce the wrong evidence if the traffic is routed elsewhere.

## Port Filtering

Ports can be useful when the application or service is known.

Conceptually:

```text
TCP/UDP
    ↓
Specific service port
```

Examples of investigation questions:

```text
Which traffic uses the service port?

Is this host communicating with the expected service?

What traffic is reaching the test application?
```

Port-based filtering can be useful, but do not treat ports as proof of application identity.

Applications can use non-standard ports.

## Protocol Filtering

Capture filters can also be scoped around supported protocol-level conditions.

This can reduce noise when the investigation is explicitly about a known protocol.

For example:

```text
Question:
Collect only the protocol relevant to this controlled test.
```

Before using a protocol restriction, confirm that excluding other protocols will not remove context you later need.

## Logical Operators

Capture filters can combine conditions logically.

The common concepts are:

```text
AND
OR
NOT
```

Think in terms of requirements rather than syntax.

For example:

```text
Requirement A
AND
Requirement B
```

means both conditions must be satisfied.

Whereas:

```text
Requirement A
OR
Requirement B
```

means either condition may satisfy the filter.

## Parentheses and Grouping

When conditions become more complex, grouping matters.

Conceptually:

```text
A AND (B OR C)
```

is different from:

```text
(A AND B) OR C
```

Before applying a complex capture filter, write the requirement in plain language.

Example:

```text
Capture traffic involving the test client
AND
traffic associated with either of two test services.
```

Then translate that requirement into filter syntax.

## Directional Reasoning

A frequent mistake is filtering only one direction.

Suppose the question is:

```text
Did the client successfully communicate with the server?
```

You may need:

```text
Client → Server
Server → Client
```

If the filter captures only:

```text
Client → Server
```

you may miss:

```text
Server → Client
```

and lose the evidence needed to determine whether the server responded.

## Capture Filter Example Strategy

Instead of memorizing a list of filters, use this construction process:

```text
1. Identify the relevant host.
2. Identify the relevant network or service.
3. Determine whether direction matters.
4. Determine whether multiple protocols are required.
5. Determine whether excluding other traffic is safe.
6. Translate the requirement into capture-filter syntax.
7. Validate with a short test capture.
```

## Validate Every Important Capture Filter

Never assume a filter is correct because Wireshark accepts it.

A syntactically valid filter can still be logically wrong.

Use:

```text
Write filter
    ↓
Start short capture
    ↓
Generate known traffic
    ↓
Check expected packets
    ↓
Check expected response packets
    ↓
Confirm unrelated traffic is excluded as intended
```

## Positive Validation

First prove that traffic you expect is captured.

Example:

```text
Expected:
Test client communicates with test server.
```

Generate the traffic.

Then verify:

```text
Client traffic appears
Server response appears
```

## Negative Validation

Then check whether traffic that should be excluded is actually excluded.

For example:

```text
Expected:
Only traffic associated with the controlled test should be captured.
```

Generate unrelated traffic if safe and appropriate.

Verify that the filter behaves as intended.

This is especially useful for complex filters.

## Capture Filter Testing

Use a simple test matrix.

| Test                      | Expected result                         |
| ------------------------- | --------------------------------------- |
| Relevant traffic          | Captured                                |
| Relevant response         | Captured                                |
| Clearly unrelated traffic | Excluded                                |
| Alternate direction       | Captured if required                    |
| Unexpected protocol       | Excluded only if intentionally excluded |

This turns filter validation into an evidence-quality check.

## Common Capture Filter Mistake: Using Display Filter Syntax

One of the most common errors is mixing filter languages.

For example, a display filter might use field-oriented syntax such as:

```text
ip.addr == 192.0.2.10
```

That does not mean the same expression is valid as a capture filter.

Capture filters use a different syntax.

The lesson is:

```text
Do not copy display-filter expressions into the capture-filter field.
```

When unsure, consult the capture-filter syntax documentation or test the expression in a controlled capture.

## Common Capture Filter Mistake: Overly Narrow Scope

Example:

```text
Only capture one protocol.
```

Later you discover that another protocol was essential to the investigation.

The result:

```text
Missing evidence
```

Before narrowing scope, ask:

```text
What dependencies might exist outside this protocol?
```

## Common Capture Filter Mistake: Wrong Direction

Example:

```text
Capture traffic only from the client.
```

But the investigation requires the server's response.

Result:

```text
Incomplete conversation
```

Always consider both directions.

## Common Capture Filter Mistake: Wrong Host

A filter can be technically correct but target the wrong address.

This can happen when:

* DHCP changes an address
* Multiple interfaces exist
* NAT changes addresses
* Virtual machines use different addresses
* Containers have different addresses
* IPv4 and IPv6 are both involved

Confirm the actual addresses before capturing.

## Common Capture Filter Mistake: Ignoring IPv6

An investigation may involve both:

```text
IPv4
IPv6
```

If your capture filter targets only one protocol family, you may miss valid communication.

Before restricting address families, determine whether IPv6 is relevant.

## Common Capture Filter Mistake: Assuming Ports Identify Applications

A port number is not a guaranteed application identity.

For example, traffic associated with a familiar service port may actually contain another protocol or application.

Use ports as collection criteria when appropriate, but verify the actual protocol during analysis.

## Common Capture Filter Mistake: Forgetting Context

Suppose you capture only:

```text
TCP traffic to a service
```

but the real problem begins with DNS.

The capture may show the connection attempt but not the name-resolution event that caused it.

Before narrowing the capture, ask:

```text
What happened immediately before the event I care about?
```

## Capture Filter and Troubleshooting

For troubleshooting, capture filters should support the diagnostic question.

Example:

```text
Problem:
Application cannot reach service.

Potential causes:
DNS
TCP
TLS
Application
```

A filter that captures only application traffic may hide the evidence needed to distinguish these causes.

A broader capture may be more appropriate.

## Capture Filter and Security Investigation

Security investigations often benefit from broad contextual evidence.

For example, if you suspect suspicious communication, you may initially need:

```text
DNS
TCP
TLS
HTTP
ICMP
Other relevant protocols
```

A narrowly filtered capture can remove the context needed to determine what happened.

If collection constraints require filtering, document those limitations.

## Capture Filter and Performance

A capture filter can reduce the volume of traffic that needs to be stored or processed.

This can be valuable in high-volume environments.

However:

```text
Lower volume
    ≠
Better evidence
```

The objective is:

```text
Enough evidence
+
Manageable collection
```

## Capture Filter and Privacy

Filtering can also reduce unnecessary collection of sensitive traffic.

For example, if an investigation is specifically scoped to an authorized test service, collecting unrelated user traffic may be unnecessary.

This creates a legitimate reason to narrow capture scope.

Always balance:

```text
Investigation requirements
+
Evidence completeness
+
Privacy
+
Operational constraints
```

## Capture Filters in Long-Running Captures

For long-running collection, filtering may be necessary to keep capture size manageable.

Before deploying such a filter, validate it carefully.

A mistake that persists for hours can create a large evidence gap.

Use:

```text
Test
    ↓
Validate
    ↓
Deploy
    ↓
Periodically verify
```

## Capture Filter Documentation

For important captures, document the filter used.

Record:

```text
Capture filter:
Purpose:
Expected traffic:
Excluded traffic:
Validation performed:
Known limitations:
```

This makes later interpretation much easier.

## Example Capture Record

```text
Purpose:
Capture traffic for an authorized lab application test.

Capture filter:
Restricted to traffic involving the test client and test server.

Expected:
Client-to-server and server-to-client traffic.

Validation:
Generated a known test transaction and verified both directions.

Limitation:
Traffic outside the defined endpoints was intentionally excluded.
```

## Capture Filter Decision Framework

Before applying a filter, ask:

```text
1. What question am I answering?
2. What traffic must be captured?
3. What traffic may be safely excluded?
4. Do I need both directions?
5. Could another protocol provide important context?
6. Could IPv4 and IPv6 both matter?
7. Could NAT or virtualization change the addresses?
8. Will this filter permanently remove evidence?
9. Have I tested the filter?
10. Have I documented the scope and limitations?
```

If you cannot answer these questions, the filter may be premature.

## Broad vs Narrow Capture

Use this mental model:

```text
Exploratory investigation
        ↓
Broad capture
        ↓
Display filtering
```

versus:

```text
Precisely defined collection requirement
        ↓
Validated capture filter
        ↓
Focused evidence
```

Neither approach is universally correct.

The correct choice depends on the investigation.

## Practical Exercise 1: Host-Scoped Capture

In an authorized lab:

1. Identify a test host.
2. Determine its network address.
3. Build a capture filter targeting traffic involving that host.
4. Start a short capture.
5. Generate known traffic.
6. Stop the capture.
7. Verify the expected traffic appears.

Then ask:

```text
Did I capture both directions?
```

## Practical Exercise 2: Service-Scoped Capture

Use a controlled test service.

Build a filter based on the service's transport port.

Then:

1. Start capture.
2. Connect to the service.
3. Generate a second unrelated connection.
4. Stop capture.
5. Verify that the intended traffic was captured.
6. Determine whether the unrelated traffic was excluded.

Remember that the port alone does not prove application identity.

## Practical Exercise 3: Direction Test

Create a capture where both client and server traffic are expected.

Use a directional filter.

Then determine whether:

```text
Client → Server
```

and:

```text
Server → Client
```

are both captured.

Repeat with a broader filter if necessary.

The purpose is to understand the consequences of direction.

## Practical Exercise 4: Protocol Dependency

Choose a controlled application workflow involving more than one protocol.

For example:

```text
Name resolution
    ↓
TCP
    ↓
TLS
    ↓
Application
```

First capture broadly.

Then identify which protocols were present.

Next, imagine restricting the capture to only one protocol.

Ask:

```text
What evidence would I lose?
```

This exercise teaches the cost of premature filtering.

## Practical Exercise 5: Filter Validation

Create a capture filter for a known lab scenario.

Validate it using:

```text
Expected traffic
Expected response
Unrelated traffic
```

Record:

```text
What was captured?
What was excluded?
What was unexpectedly missing?
```

If anything is unexpected, do not trust the filter until the behavior is understood.

## Practical Exercise 6: Capture Filter Failure

Intentionally create an overly restrictive filter in a controlled lab.

Run the test.

Observe the missing evidence.

Then remove the filter and repeat the capture.

Compare:

```text
Restricted capture
vs
Broad capture
```

The objective is to experience why capture-filter mistakes can be difficult to recover from.

## Practical Exercise 7: Display Filter Recovery

Capture traffic broadly.

Then use a display filter to isolate the same traffic you previously attempted to isolate during capture.

Now clear the display filter.

Observe that the broader evidence is still available.

This demonstrates:

```text
Capture filter:
Permanent collection decision

Display filter:
Reversible analysis decision
```

## Practical Exercise 8: Capture Planning

Before starting a capture, write:

```text
Question:
Target:
Expected protocols:
Expected endpoints:
Required directions:
Capture scope:
Potential dependencies:
Potential exclusions:
Capture limitations:
```

Then decide whether a capture filter is actually necessary.

The goal is to stop filtering by habit.

## Capture Filter Checklist

Before using a capture filter:

```text
[ ] Investigation question is defined
[ ] Target is known
[ ] Required traffic is known
[ ] Required protocols are understood
[ ] Both directions considered
[ ] IPv4/IPv6 considered
[ ] NAT/virtualization considered
[ ] Dependencies considered
[ ] Excluded traffic identified
[ ] Filter syntax validated
[ ] Short test performed
[ ] Expected traffic confirmed
[ ] Limitations documented
```

## What You Should Be Able to Do

After completing this file, you should be able to:

* Explain the difference between capture and display filters
* Decide when a capture filter is appropriate
* Decide when broad capture is safer
* Build capture requirements from investigation questions
* Reason about source and destination
* Reason about host and network scope
* Reason about port and protocol scope
* Combine conditions logically
* Validate capture filters
* Recognize overly restrictive filters
* Recognize directional mistakes
* Recognize address-family limitations
* Understand capture-filter consequences
* Document filtering decisions
* Preserve awareness of evidence limitations

## Completion Criteria

You are ready to continue when you can look at an investigation and say:

```text
I need to capture this traffic
because it answers this question.
```

Then you can determine:

```text
What must be included
        ↓
What can be excluded
        ↓
Whether filtering is necessary
        ↓
How to validate the filter
        ↓
What evidence may be lost
```

The final mental model should be:

```text
Capture filter
    =
Evidence collection decision
```

not:

```text
Capture filter
    =
Another list of Wireshark commands
```

A professional analyst uses capture filters deliberately because every excluded packet is evidence that may no longer be available for later analysis.
