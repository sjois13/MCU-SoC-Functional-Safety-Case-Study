# Dependent Failure Analysis

**Version:** v0.1 — Work in Progress

## 1. Objective

Identify shared dependencies and couplings that could affect both a safety-relevant function and the mechanism intended to protect it, or affect multiple redundant functions simultaneously.

## 2. Dependency Analysis

| ID | Shared Dependency | Functions / Mechanisms Exposed | Dependent Failure Concern | Status |
|---|---|---|---|---|
| DFA-001 | Clock generation / distribution | CPU redundancy, interconnect, peripherals, monitors | Common timing disturbance may affect multiple protected functions or channels | Clock domains and monitor independence TBD |
| DFA-002 | Power supply / distribution | CPU, SRAM, interconnect, peripherals, safety mechanisms | Common supply disturbance may affect functional and diagnostic paths simultaneously | Power domains and supervision TBD |
| DFA-003 | Reset generation / distribution | CPU, peripherals, monitoring logic | Common reset fault may simultaneously disturb functional and diagnostic elements | Reset architecture TBD |
| DFA-004 | Shared interconnect | Functional traffic and diagnostic traffic | Same interconnect fault may affect both safety-relevant data and information required for its diagnosis | Diagnostic communication paths TBD |
| DFA-005 | ADC reference / analog supply | Redundant or compared ADC channels | Common reference disturbance may produce correlated incorrect conversions that comparison does not reveal | Analog/reference architecture TBD |
| DFA-006 | Comparator / monitoring logic | Redundant CPU channels | Failure of common comparison logic may prevent detection of channel disagreement | Monitor architecture TBD |
| DFA-007 | Fault collection / reaction path | Multiple safety mechanisms | Individual faults may be detected correctly but a shared handling-path failure may prevent the required reaction | Fault-handling architecture TBD |

## 3. Dependency Map

```text
Functional channel ─────┐
                        ├── Safety decision / reaction
Monitoring channel ─────┘
        ↑                       ↑
        │                       │
   shared resources        shared handling
        │                       │
   clock / power /         fault collection
   reset / interconnect
```

The presence of redundancy does not establish independence where these shared dependencies remain unresolved.

## 4. Findings

The current safety architecture does not yet support an independence claim for redundant or monitoring functions.

The main architectural questions to resolve are whether:

- functional and monitoring paths share resources capable of defeating both;
- redundant channels contain common dependencies;
- diagnostic information uses the same resources as the function being diagnosed; and
- fault detection and fault reaction contain single shared points of dependency.

These items will be refined when the conceptual architecture is decomposed into clock, power, reset and diagnostic domains.
