# Safety Architecture and Mechanism Allocation

**Version:** v0.1 — Work in Progress

## 1. Purpose

This work product proposes an initial safety architecture for the conceptual MCU/SoC and allocates candidate safety mechanisms to the preliminary [Semiconductor Safety Requirements](../safety-requirements/semiconductor_safety_requirements.md).

The reasoning follows:

```text
Functional failure
        ↓
MCU boundary effect
        ↓
Semiconductor safety requirement
        ↓
Candidate safety mechanism
        ↓
Fault detection
        ↓
Fault reaction
```

The mechanisms below are conceptual candidates. They are not claims about the implementation of any commercial MCU/SoC.

No diagnostic coverage is assumed.

## 2. Safety Architecture Principle

The initial safety architecture uses three conceptual layers:

```text
Normal function
      ↓
Failure detection / control
      ↓
Fault handling and reaction
```

A safety mechanism is useful only if its relationship to the relevant functional failure is understood.

The presence of a mechanism alone does not establish sufficient diagnostic coverage or independence.

## 3. Initial Mechanism Allocation

| Safety Requirement | Failure Concern | Candidate Safety Mechanism | Intended Purpose |
|---|---|---|---|
| SSR-001 | Incorrect ADC result | Independent plausibility/range monitoring and, where justified, redundant or diverse acquisition | Detect feedback information inconsistent with expected behaviour or an independent reference |
| SSR-002 | Incorrect CPU computation | Redundant computation, lockstep-style execution or independent result monitoring | Detect disagreement or incorrect execution before an unsafe command propagates |
| SSR-003 | Corrupted SRAM working data | ECC/parity with appropriate error handling | Detect and, where supported, correct memory data corruption |
| SSR-004 | Incorrect interconnect transaction | End-to-end information protection and/or interconnect error-detection mechanisms | Detect corruption or other protected transaction failures between source and destination |
| SSR-005 | Incorrect clock behaviour | Independent clock monitoring | Detect clock behaviour outside defined operating expectations |
| SSR-006 | Detected safety-relevant fault | Fault collection/handling and defined fault signalling or transition | Initiate the required fault reaction within the required reaction time |

The final mechanism choice depends on the detailed architecture and safety requirement.

## 4. Mechanism Reasoning

### 4.1 ADC

Duplicating an ADC does not automatically provide independent protection.

Two acquisition paths may still share:

- analog supply;
- reference voltage;
- clock;
- sensor;
- interconnect; or
- downstream processing.

A comparison mechanism can therefore detect disagreement without necessarily identifying which channel is correct.

The required independence and system response remain TBD.

### 4.2 CPU

A lockstep-style architecture can compare equivalent processing results or execution behaviour.

However:

```text
CPU A ──┐
        ├── comparison
CPU B ──┘
```

does not by itself guarantee independence.

Common clock, power, reset, interconnect, design or comparison logic may introduce shared dependencies.

The exact CPU safety architecture remains conceptual.

### 4.3 SRAM

Memory protection may use parity or error-correcting codes depending on the required capability.

Conceptually:

```text
Stored data + protection information
              ↓
            read
              ↓
        integrity check
          ↙       ↘
       valid      error
```

The capability of the code, protected memory regions, handling of detected errors, latent-fault considerations and diagnostic coverage remain TBD.

### 4.4 Interconnect

Protecting only stored data does not necessarily protect the information while it is transferred.

An end-to-end protection concept may associate protection information with safety-relevant data so that corruption between producer and consumer can be detected.

However, different interconnect failures require different reasoning.

For example:

```text
Data corruption
      ≠
Wrong destination
      ≠
Transaction loss
      ≠
Incorrect timing
      ≠
Incorrect ordering
```

A single protection mechanism should therefore not be assumed to cover every failure identified under `FF-INT-001`.

### 4.5 Clock

Clock supervision requires a reference sufficiently suitable for detecting the clock failure of concern.

If the monitored clock and its monitor depend on the same failing resource, the assumed detection capability may be compromised.

Clock-monitor independence and clock-domain architecture remain TBD.

## 5. Fault Handling

Detection alone is insufficient.

The conceptual chain is:

```text
Fault occurs
     ↓
Safety mechanism detects fault
     ↓
Fault indication
     ↓
Fault-handling logic
     ↓
Defined MCU/JCU reaction
```

Possible reactions could include suppressing an affected output, preventing further command updates, signalling the external system, reset, or requesting/triggering an external safe reaction.

No specific reaction is allocated yet because the JCU safe behaviour and external power-stage responsibilities remain TBD.

## 6. Shared Dependencies

The mechanisms introduce an important dependent-failure question.

For example:

```text
CPU A ──┐
        ├── comparator
CPU B ──┘
  ↑
shared clock
```

A common clock fault could affect both processing channels.

Similar reasoning applies to:

- shared power;
- shared reset;
- shared interconnect;
- common reference sources;
- common fault-handling logic; and
- shared safety-mechanism resources.

These dependencies will be analysed during the Dependent Failure Analysis (DFA)

## 7. Initial Traceability

| Functional Failure | Safety Requirement | Candidate Mechanism |
|---|---|---|
| FF-ADC-001 | SSR-001 | Plausibility monitoring / redundant or diverse acquisition |
| FF-CPU-001 | SSR-002 | Redundant computation / lockstep-style monitoring |
| FF-SRAM-001 | SSR-003 | ECC / parity |
| FF-INT-001 | SSR-004 | End-to-end and/or interconnect protection |
| FF-CLK-001 | SSR-005 | Clock supervision |
| Detected safety-relevant fault | SSR-006 | Fault collection and reaction logic |

This table records candidate allocation only. It does not establish diagnostic coverage or sufficient effectiveness.

## 8. Open Items

The following remain TBD:

- detailed ADC monitoring architecture;
- CPU redundancy architecture;
- memory protection capability;
- detailed interconnect protection;
- clock-monitor architecture;
- fault collection and signalling;
- safe-state responsibilities;
- fault-reaction timing;
- independence requirements;
- latent-fault detection;
- diagnostic coverage; and
- verification of each mechanism.

## 9. Next Step

The next work products will challenge the proposed safety architecture from different directions:

```text
FTA
    Can combinations of internal failures lead to an MCU boundary failure?

FMEDA
    How would failure modes, safety mechanisms and diagnostic coverage
    eventually be quantified?

DFA
    Can shared dependencies defeat apparently independent protection?

Verification
    How would the safety requirements and mechanisms be demonstrated?
```

The safety architecture will be revised if those analyses identify gaps.
