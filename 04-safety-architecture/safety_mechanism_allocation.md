# Safety Architecture and Mechanism Allocation

**Version:** v0.2 — Working draft  
**Scope:** ADC acquisition path; preliminary allocation for other MCU/SoC functions

## 1. Design position

This document evaluates candidate mechanisms against the ADC failures identified in [WP02 — Functional Failure Analysis](../02-failure-analysis/functional_failure_analysis.md) and the proposed requirements in [WP03 — Semiconductor Safety Requirements](../03-safety-requirements/semiconductor_safety_requirements.md). The architectural starting point is [WP01](../01-architecture%20/conceptual_architecture.md).

The ADC is the first IP examined in detail. No mechanism is credited with diagnostic coverage or declared implemented. Mechanism selection remains conditional on the sensor interface, ADC implementation, timing budget, hardware/software partition and JCU fault response. The architecture below is a *candidate safety concept*, not a description of an NXP device.

A design distinction is maintained throughout:

- **Functional provision:** supplying a result, channel identifier, completion status or consistent data/status interface.
- **Diagnostic:** detecting a specified fault or its effect with a defined capability and latency.
- **Reaction:** preventing, limiting or signalling the use of affected information after detection.

The first does not establish the second; the second does not establish the third.

## 2. Proposed ADC monitoring boundary

```text
External analog feedback
         |
         v
Analog input / selection --> sampling and conversion
                                    |   ^
                                    |   | reference, clock, trigger
                                    v
                           Result + status interface
                                    |
                                    v
                              SoC interconnect
                                    |
                                    v
                       Consumer data qualification
                         /          |           \
                channel/source  freshness/time  validity/status
                         \          |           /
                                    v
                          Control computation
                                    |
                                    v
                          MCU output peripheral
                                    |
                              External power stage

Candidate monitoring inputs (not assumed present):
  - conversion-completion / sequence information
  - timing reference or deadline observation
  - independent or diverse measurement/reference
  - fault indications from ADC, clock, supply or interconnect
```

The consumer qualification block is a **logical responsibility**, not necessarily a dedicated hardware IP. Its location (ADC logic, another SoC peripheral or software) is undecided. The external power stage remains outside the MCU boundary.

## 3. Mechanism selection by failure concern

| Failure concern | Candidate measure | Detection or control opportunity | Limitation / open design question |
|---|---|---|---|
| `FF-ADC-001A` Incorrect but plausible conversion | Independent reference measurement, diverse sensing, or justified analytical plausibility check | Compare result against information not affected by the same fault | Simple range checks can miss plausible errors; independence, error bounds and comparison latency TBD |
| `FF-ADC-001B` Wrong channel | Channel identity and configuration readback; sequence checks | Detect mismatch between expected and reported channel or configuration | Identity derived from the same erroneous selection logic may agree with the wrong result |
| `FF-ADC-001C` Stale result | Conversion-instance indication (counter, toggle or completion semantics) plus consumer age check | Reject results not demonstrably refreshed within a defined interval | Counter/status faults, wraparound and trigger failures require analysis; unchanged sample value alone is not evidence of staleness |
| `FF-ADC-001D` Missing conversion | Conversion deadline supervision / timeout | Detect failure to complete a required acquisition | Monitor must have a suitable time base; timeout value and owner TBD |
| `FF-ADC-001E` Incorrect sampling time | Trigger and completion timestamp or independent timing observation, where supported | Detect deviation from allocated sampling/latency window | Completion time does not necessarily prove actual sample instant; timestamp clock dependencies TBD |
| `FF-ADC-001F` Data/status mismatch | Atomic snapshot, coherent buffering, sequence-tagged data or defined read protocol | Avoid or identify mixing data and status from different conversions | Read consistency is not proof of numerical correctness; implementation-specific register semantics TBD |

**Selection rule:** no candidate is accepted solely because it is commonly used in safety MCUs. Each requires a specified fault set, architectural interface, assumed independence, detection conditions, latency and evidence strategy.

## 4. ADC mechanism trade-offs

### 4.1 Numerical integrity is the unresolved hard case

`SSR-001A` defines an intended acquisition performance obligation. It does **not** require or prove detection of all out-of-tolerance results. A separate diagnostic requirement is needed if the safety strategy depends on identifying such results.

Three options merit comparison:

| Option | Potential value | Principal weakness |
|---|---|---|
| Fixed range/limit check | Low complexity; catches values outside a justified envelope | Cannot detect many plausible in-range errors |
| Two acquisition channels with comparison | Can detect disagreement for some channel-specific failures | Shared sensor, reference, supply, trigger, configuration or comparator may defeat independence |
| Diverse/independent reference or analytical cross-check | Can reveal errors common to identical conversion channels if the reference is genuinely independent | Requires a defensible physical relationship, tolerance model and response to disagreement |

A disagreement indicates inconsistency; it does not identify which result is correct. Any fallback to one channel therefore requires additional justification.

### 4.2 Source and freshness protection

For `SSR-001B`–`SSR-001F`, interface design can make faults *observable* to the consumer. The monitor must then compare that evidence with expectations derived from the intended acquisition schedule and configuration.

For example, a completion counter helps with freshness only if the consumer knows which conversion it refers to, observes updates at the required rate and handles rollover or discontinuity. A counter increment generated by faulty control logic does not independently prove that an analog sample occurred.

Similarly, reading a channel identifier is useful only after examining whether it can be corrupted by the same fault that selects the wrong input.

### 4.3 Time supervision

The required observation interval is not yet known. A future timing argument must distinguish:

1. Intended sampling instant and permissible jitter.
2. Maximum conversion/result-availability latency.
3. Detection latency of a missing, late or invalid conversion.
4. Fault signalling and reaction latency.

These intervals must be reconciled with the JCU-level time available to avoid the relevant unsafe behaviour. No numerical fault-tolerant time interval is claimed.

## 5. Fault signalling and response allocation

The proposed fault path is:

```text
ADC-related anomaly
       |
       v
Detection / qualification (hardware and/or software)
       |
       v
Fault indication with defined identity and validity
       |
       v
MCU fault-handling decision
       |
       +--> Prevent use of invalid/expired feedback
       |
       +--> Apply allocated control/output response
       |
       +--> Signal fault or request external action, if required
```

This is a *responsibility chain*, not an implementation diagram. It supports `SSR-007`, `SSR-008`, `SSR-009` and the broader `SSR-006` topic. The following questions must be resolved before claiming a complete safety path:

| Decision | Why it matters |
|---|---|
| Where is detection performed? | Determines whether CPU/software failures can disable the monitor |
| How is a fault indication transported? | A bus or interrupt fault can block delivery even when detection works |
| What does the consumer do with invalid feedback? | Detection is ineffective if invalid data continues to influence the output |
| What response is permitted at the MCU output? | Holding the last command, suppressing updates and reset can have different joint-level consequences |
| What must the external power stage do? | The MCU may only be able to request, not enforce, a safe physical response |
| What is the complete detection-to-reaction deadline? | A correct reaction delivered too late may not control the boundary failure |

**No default safe state is assumed.** In particular, MCU reset and retaining the previous actuator command are not treated as inherently safe.

## 6. Dependency review before mechanism credit

The following are candidate common dependencies to carry into WP07 (DFA):

| Shared resource / condition | Possible impact on the proposed protection |
|---|---|
| ADC voltage reference | Two conversion channels may drift or bias together |
| Analog supply / ground | Simultaneous acquisition error and monitor impairment |
| Trigger source or clock | Both acquisition and freshness evidence may stop or become misleading |
| Channel configuration logic | Selected channel and reported identity may fail consistently |
| SoC interconnect | Both result delivery and fault notification may be disrupted |
| CPU and software execution | Consumer qualification and reaction may fail together |
| Reset and fault-handling logic | Fault information may be lost or reaction prevented |
| Shared comparator/reference in redundancy | Comparison may not be an independent observation |

A dependency is not automatically a demonstrated dependent failure. The DFA must establish plausible coupling paths, affected elements and any architectural provisions that interrupt them.

## 7. Allocation record and verification intent

The following table records *candidate responsibilities*, not completed requirements verification.

| WP03 reference | Candidate architecture allocation | Evidence needed before acceptance |
|---|---|---|
| `SSR-001A` | ADC performance specification; additional independent monitoring if justified | Defined accuracy envelope; diagnostic fault set and effectiveness argument if monitoring is required |
| `SSR-001B` | Channel selection/identity interface + consumer comparison | Configuration and identity fault tests; shared-source analysis |
| `SSR-001C` | Conversion-instance interface + consumer age supervision | Update semantics, stuck-status/counter fault injection, age-limit test |
| `SSR-001D` | Completion indication + deadline monitor | Missing-conversion fault injection; timeout accuracy and time-base dependency |
| `SSR-001E` | Trigger/ADC timing contract + observation mechanism as needed | Sampling/latency measurements and timing-fault scenarios |
| `SSR-001F` | Coherent data/status transfer protocol | Boundary-condition tests for concurrent updates and reads |
| `SSR-007` | Consumer data qualification | Invalid/expired-data injection; verify affected feedback is not accepted as current |
| `SSR-008` | MCU fault-response control and external interface, allocation TBD | Reaction-path tests under specified fault and timing conditions |
| `SSR-009` | Fault indication and transport | Fault reporting and delivery-failure tests |

These tests are **proposals**. Their feasibility depends on the eventual hardware architecture and available fault-injection access. No test has been executed.

## 8. Other SoC mechanisms — not yet assessed

The previous v0.1 mechanism ideas are retained as candidates only:

| Requirement topic | Candidate direction | Key unresolved issue |
|---|---|---|
| `SSR-002` CPU | Lockstep-style comparison, diverse/redundant execution or independent result monitoring | Fault model, comparator dependencies and reaction |
| `SSR-003` SRAM | Parity/ECC and error handling | Protected regions, error classes, scrubbing/latent faults |
| `SSR-004` Interconnect | Data/address/control protection and transaction supervision | Coverage of corruption versus loss, misrouting, duplication and timing |
| `SSR-005` Clock | Clock supervision with suitable reference | Reference independence, clock-domain coverage and detection time |
| `SSR-006` Fault handling | Fault collection, escalation and defined outputs | Delivery path, reaction ownership and common dependencies |

No conclusion is drawn for NVM, timer, communication, reset, power or output peripherals. Their omission from detailed analysis is an explicit scope limitation.

## 9. Outstanding engineering decisions

Before WP04 can become a defensible safety architecture baseline, the project needs:

- ADC sensor and acquisition timing/interface contract;
- allocated permissible error and data-age limits;
- an explicit diagnostic objective for plausible numerical errors;
- a decision on hardware versus software monitoring responsibilities;
- fault reporting paths and their own failure behaviour;
- JCU output response and external power-stage responsibilities;
- an end-to-end reaction-time budget; and
- a justified dependency and verification strategy for selected mechanisms.

**Iteration trigger:** if a candidate monitor cannot distinguish a relevant failure, or its indication cannot reliably reach the reaction function, revisit WP01 architecture and WP03 requirements rather than crediting the mechanism. FTA and DFA will challenge these paths; a later FMEDA will need device-specific failure data and defensible diagnostic coverage evidence.
