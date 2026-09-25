# Wireshark Workflow

A practical, question-driven workflow for learning and mastering Wireshark from absolute beginner to independent professional-level packet analysis.

## Purpose

This repository is designed to develop the ability to independently use Wireshark to:

* Capture network traffic
* Open and preserve packet captures
* Navigate the Wireshark interface
* Inspect packets and protocol fields
* Build useful display filters
* Use capture filters appropriately
* Analyze conversations and streams
* Interpret common network protocols
* Troubleshoot network communication
* Investigate suspicious traffic
* Extract packet-level evidence
* Document findings
* Determine what additional evidence is required
* Decide what to investigate next

The goal is not to memorize Wireshark features.

The goal is to develop the ability to answer unfamiliar network questions using packet evidence.

## Core Mental Model

The central workflow used throughout this repository is:

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

The most important question is always:

> **What am I trying to determine from the traffic?**

Only after defining that question should the analysis method be selected.

## Learning Objective

The repository is designed to take a learner from:

```text
"I have never used Wireshark."
```

to:

```text
"Give me an unfamiliar packet capture or network-traffic problem
and I can independently determine what to inspect, how to analyze it,
what the traffic proves, what remains uncertain, and what I should
investigate next."
```

## Learning Philosophy

This repository follows several principles.

### Practical Work First

Every major concept should support a practical task.

The preferred learning pattern is:

```text
Goal
↓
Situation
↓
Question
↓
Wireshark Action
↓
Observed Traffic
↓
Interpretation
↓
Decision
↓
Next Action
```

### Questions Before Filters

Filters are tools for answering questions.

The repository therefore teaches:

```text
Question
↓
Evidence required
↓
Relevant protocol / field
↓
Filter
↓
Validation
```

rather than requiring memorization of large filter lists.

### Investigation Before Memorization

The learner should understand why a particular Wireshark feature is useful before memorizing where it is located.

### Evidence Before Conclusions

The repository distinguishes between:

```text
Observation
↓
Interpretation
↓
Conclusion
```

For example:

```text
Observation:
Multiple SYN packets were sent to sequential destination ports.

Interpretation:
The traffic is consistent with port-scanning behavior.

Conclusion:
The activity warrants further investigation.
```

The interpretation or conclusion must not be stronger than the available evidence.

### Decision-Making Over Button Memorization

The objective is not to memorize:

> "Click this menu."

The objective is to understand:

> "I need this evidence, therefore this Wireshark capability is appropriate."

## Learning Progression

The repository progresses through the following stages:

```text
Orientation & Mental Model
↓
UI & Packet Navigation
↓
Capture & Evidence Handling
↓
Question-Driven Filtering
↓
Packet & Protocol Analysis
↓
Conversations, Streams & Correlation
↓
Statistics, Timing & Anomaly Analysis
↓
Troubleshooting Workflows
↓
Security & Forensic Workflows
↓
Professional Analysis & Documentation
↓
TShark & Repeatable Analysis
↓
Decision-Making & Independent Investigation
↓
Capstone / Independence Assessment
```

The learner gradually moves from exact instructions toward independent investigation.

## Guided → Independent Progression

Practical work follows three major stages.

### Guided

The repository provides:

* Exact objective
* Exact action
* Exact filter where appropriate
* Expected result
* Interpretation
* Troubleshooting

### Partially Guided

The repository provides:

* Objective
* Context
* Evidence
* Constraints
* Success criteria

The learner determines the analysis path.

### Independent

The repository provides only:

* Scenario
* Available evidence
* Objective
* Constraints
* Expected deliverable

The learner determines:

* Where to start
* What to inspect
* Which filters to construct
* Which analysis features to use
* Which packets matter
* What the evidence means
* What remains uncertain
* What should happen next

## Repository Structure

```text
Wireshark-Workflow/
│
├── README.md
│
├── 00-Orientation/
│   ├── 01-Wireshark-Mental-Model.md
│   └── 02-Lab-Environment-and-Safety.md
│
├── 01-UI-and-Packet-Navigation/
│   ├── 01-Interface-Orientation.md
│   ├── 02-Packet-Inspection.md
│   └── 03-UI-Fluency-Exercises.md
│
├── 02-Capture-and-Evidence/
│   ├── 01-Capture-Workflow.md
│   ├── 02-Capture-Filters.md
│   ├── 03-PCAP-Workflow.md
│   └── 04-Capture-Troubleshooting.md
│
├── 03-Question-Driven-Filtering/
│   ├── 01-Display-Filter-Mental-Model.md
│   ├── 02-Filter-Building.md
│   └── 03-Filter-Workflows.md
│
├── 04-Packet-and-Protocol-Analysis/
│   ├── 01-Packet-Layers-and-Fields.md
│   ├── 02-Addressing-and-Basic-Protocols.md
│   ├── 03-TCP-and-UDP-Analysis.md
│   ├── 04-DNS-DHCP-and-ICMP-Analysis.md
│   └── 05-HTTP-TLS-and-Application-Traffic.md
│
├── 05-Conversations-Streams-and-Correlation/
│   ├── 01-Endpoints-and-Conversations.md
│   ├── 02-Following-and-Reconstructing-Traffic.md
│   └── 03-Timeline-and-Correlation.md
│
├── 06-Statistics-Timing-and-Anomalies/
│   ├── 01-Statistics-Workflow.md
│   ├── 02-Timing-Performance-and-I-O.md
│   └── 03-Expert-Information-and-Anomalies.md
│
├── 07-Troubleshooting-Workflows/
│   ├── 01-Connectivity-Failures.md
│   ├── 02-DNS-and-Name-Resolution.md
│   ├── 03-Slow-Application.md
│   ├── 04-Packet-Loss-and-TCP-Problems.md
│   └── 05-Wireshark-Analysis-Problems.md
│
├── 08-Security-and-Forensics/
│   ├── 01-Suspicious-Communications.md
│   ├── 02-DNS-and-HTTP-Investigation.md
│   ├── 03-Scanning-and-Lateral-Communication.md
│   ├── 04-Encrypted-Traffic-Analysis.md
│   └── 05-Evidence-and-Forensic-Reasoning.md
│
├── 09-Professional-Workflow/
│   ├── 01-Professional-Investigation-Workflow.md
│   ├── 02-Advanced-Wireshark-Workflow.md
│   └── 03-Analysis-Documentation.md
│
├── 10-TShark/
│   ├── 01-TShark-Fundamentals.md
│   ├── 02-TShark-Analysis-Workflows.md
│   └── 03-Repeatable-Packet-Analysis.md
│
├── 11-Independent-Analysis/
│   ├── 01-Decision-Framework.md
│   ├── 02-Progressive-Analysis-Challenges.md
│   └── 03-Investigation-Completion-Criteria.md
│
└── 12-Capstone/
    ├── 01-Capstone-Rules.md
    ├── 02-Guided-Capstone.md
    ├── 03-Independent-Capstone.md
    └── 04-Final-Independence-Test.md
```

## Capability Areas

The repository develops the following practical capabilities.

| Area                 | Capability                                     |
| -------------------- | ---------------------------------------------- |
| Environment          | Prepare a safe packet-analysis environment     |
| Capture              | Select interfaces and capture traffic          |
| Evidence             | Preserve and work with PCAP/PCAPNG files       |
| UI                   | Navigate Wireshark independently               |
| Filtering            | Construct filters from investigative questions |
| Packet Analysis      | Inspect packet layers, fields, and bytes       |
| Protocol Analysis    | Interpret relevant network protocols           |
| TCP Analysis         | Investigate connection behavior and failures   |
| DNS Analysis         | Investigate name-resolution behavior           |
| Application Analysis | Analyze HTTP, TLS, and other relevant traffic  |
| Streams              | Reconstruct useful communication               |
| Statistics           | Scope and analyze large captures               |
| Timing               | Investigate latency and performance            |
| Troubleshooting      | Diagnose capture and network-analysis problems |
| Security             | Investigate suspicious communication           |
| Forensics            | Build evidence-based conclusions               |
| TShark               | Perform repeatable packet analysis             |
| Documentation        | Produce concise professional findings          |
| Decision-Making      | Determine what to investigate next             |
| Independence         | Solve unfamiliar packet-analysis problems      |

## Capture and Evidence Principle

Only capture and analyze traffic that you are authorized to capture and analyze.

Safe practice environments include:

* Your own systems
* Your own virtual machines
* Your own containers
* Intentionally created lab traffic
* Intentionally vulnerable applications
* Supplied PCAP/PCAPNG files
* Explicitly authorized environments

Do not use this repository as justification for intercepting traffic belonging to other people or systems without authorization.

## What This Repository Is Not

This repository is not intended to be:

* A networking textbook
* A protocol specification
* A massive Wireshark filter cheat sheet
* A button-by-button UI manual
* A malware-analysis course
* A generic cybersecurity course
* A collection of random packet-capture tricks

Networking and protocol concepts are introduced only when they are necessary for practical packet analysis.

## Professional Analysis Model

A professional Wireshark investigation should generally follow this pattern:

```text
Understand the problem
↓
Establish scope
↓
Identify available evidence
↓
Preserve the source
↓
Establish an initial overview
↓
Identify relevant hosts
↓
Identify relevant protocols
↓
Narrow the traffic
↓
Investigate conversations
↓
Inspect packets
↓
Correlate timing and events
↓
Test hypotheses
↓
Look for contradictory evidence
↓
Determine uncertainty
↓
Identify missing evidence
↓
Document findings
↓
Determine the next action
```

This workflow should eventually become intuitive.

## Evidence Model

Every investigation should distinguish:

### Observation

What the traffic directly shows.

### Interpretation

What the observed traffic reasonably indicates.

### Conclusion

What can be concluded after considering the available evidence.

### Uncertainty

What cannot be established from the available evidence.

### Additional Evidence

What other information would help answer the remaining question.

## Version Awareness

Wireshark changes over time.

Interface layout, protocol dissectors, filter behavior, capture mechanisms, operating-system support, hardware support, and available features may differ between versions and environments.

When behavior is version-specific, verify it against current official Wireshark documentation rather than assuming that every environment behaves identically.

## Learning Standard

Completion is not measured by how many features the learner can name.

The real test is:

> **Can the learner independently use Wireshark to answer an unfamiliar network question and support the result with packet-level evidence?**

## Final Independence Test

The final assessment intentionally minimizes instructions.

The learner will receive an unfamiliar network-analysis scenario and available traffic evidence.

They will not be told:

* Which protocol to inspect
* Which filter to use
* Which host to investigate
* Which statistics feature to open
* Which stream to follow
* Which packets matter
* What the expected conclusion is

The learner must determine the investigation methodology.

A successful investigation should demonstrate the ability to:

1. Understand the problem.
2. Identify the required evidence.
3. Select the appropriate capture or PCAP.
4. Navigate Wireshark.
5. Establish an initial overview.
6. Identify relevant hosts.
7. Identify relevant protocols.
8. Construct useful filters.
9. Narrow the investigation.
10. Follow conversations or streams when appropriate.
11. Inspect packet details.
12. Inspect raw bytes when necessary.
13. Use statistics appropriately.
14. Interpret protocol behavior.
15. Recognize anomalies.
16. Distinguish observation from interpretation.
17. Identify uncertainty.
18. Troubleshoot unexpected results.
19. Determine whether the evidence is sufficient.
20. Identify additional evidence when necessary.
21. Document findings.
22. Support conclusions with packet-level evidence.
23. Determine the appropriate next action.

## Repository Goal

The final outcome is not:

> **"The learner knows Wireshark."**

The final outcome is:

> **"The learner can independently use Wireshark to answer unfamiliar network questions."**
