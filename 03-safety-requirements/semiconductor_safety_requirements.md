# Semiconductor Safety Requirements

**Version:** v0.1 — Work in Progress

## 1. Purpose

This work product derives an initial set of MCU/SoC-level safety requirements from the functional failures and propagation paths identified in the [Functional Failure Analysis](../02-failure-analysis/functional_failure_analysis.md)

The derivation follows:

```text
Functional failure
        ↓
Failure propagation
        ↓
MCU boundary failure
        ↓
Potential contribution to upstream safety concern
        ↓
Required MCU/SoC safety behaviour
```

The requirements are intentionally mechanism-independent where possible. Safety mechanisms are allocated in a subsequent work product.

No integrity level or quantitative fault-reaction time is assigned because these have not been derived from an application-level hazard analysis.

## 2. Safety Context

The architecture baseline assumes an upstream safety need concerning unintended or uncontrolled joint motion caused by incorrect JCU behaviour.

The semiconductor contribution considered here is therefore the propagation of MCU/SoC faults into incorrect, unintended, missing or incorrectly timed control behaviour at the MCU boundary.

A system-level safe state and numerical fault-tolerant time interval (FTTI) have not yet been established. Where fault reaction or timing is required, these remain explicit integration dependencies.

## 3. Initial Safety Requirements

| ID | Derived from | Semiconductor safety requirement |
|---|---|---|
| SSR-001 | FF-ADC-001 → FB-001 / FB-005 | The MCU/SoC shall provide means to detect or otherwise control safety-relevant incorrect feedback information resulting from an ADC functional failure before it can lead to an unacceptable control output. |
| SSR-002 | FF-CPU-001 → FB-001 | The MCU/SoC shall provide means to detect or otherwise control safety-relevant incorrect control computation before the resulting command can lead to an unacceptable control output. |
| SSR-003 | FF-SRAM-001 → FB-001 / FB-005 | The MCU/SoC shall provide means to detect or otherwise control corruption of safety-relevant working data before the corrupted data can lead to an unacceptable control output or response. |
| SSR-004 | FF-INT-001 → multiple FBs | The MCU/SoC shall provide means to detect or otherwise control safety-relevant transaction-transfer failures that can propagate to an unacceptable MCU boundary behaviour. |
| SSR-005 | FF-CLK-001 → FB-002 | The MCU/SoC shall provide means to detect or otherwise control safety-relevant incorrect clock behaviour that can cause incorrect temporal behaviour of MCU/SoC functions. |
| SSR-006 | SSR-001 to SSR-005 | On detection of a safety-relevant fault requiring reaction, the MCU/SoC shall initiate the defined fault reaction within the required fault-reaction time. |

## 4. Fault Reaction and Timing

`SSR-006` cannot yet be completed quantitatively.

The required reaction depends on information not yet established in this case study:

- the JCU/system safe behaviour;
- whether the MCU can independently achieve that behaviour;
- responsibilities of the external power stage;
- the applicable FTTI or other reaction-time constraint; and
- assumptions on the robot/JCU integrator.

These items are therefore treated as integration assumptions/TBDs rather than assigned arbitrary values.

## 5. Traceability

```text
ARCH
  ↓
FF
  ↓
FB
  ↓
SSR
  ↓
Safety mechanism
  ↓
Verification
```

The next work product allocates candidate safety mechanisms to these requirements and evaluates whether shared dependencies could undermine the intended protection.

## 6. Limitations

These requirements are preliminary and conceptual.

They do not claim compliance with a specific ASIL or SIL, and no diagnostic coverage or hardware safety metric is assigned.

The wording will be refined as the architecture, safe behaviour, timing assumptions and safety mechanisms mature.
