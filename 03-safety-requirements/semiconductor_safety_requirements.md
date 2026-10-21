# Semiconductor Safety Requirements

**Version:** v0.2 — Working draft  
**Scope:** ADC acquisition path; preliminary requirements for other SoC functions

## 1. Basis and status

This work product translates the selected functional failures in [WP02 — Functional Failure Analysis](../02-failure-analysis/functional_failure_analysis.md) into candidate MCU/SoC safety requirements. The architecture reference is [WP01 — Conceptual MCU/SoC Architecture](../01-architecture%20/conceptual_architecture.md).

The ADC is the first IP examined in detail. Its requirements below are **proposed allocations**, not approved hardware safety requirements for a real device. In particular, the required accuracy, sample age, diagnostic time, fault reaction and allocation between hardware and software cannot be finalized without application-level inputs.

A requirement marked **Provisional** describes intended behaviour but still needs one or more measurable parameters or an allocation decision. **Open** indicates that the obligation itself depends on a design choice not yet made.

## 2. ADC requirements

The existing parent identifier `SSR-001` is retained as a grouping reference. The child identifiers provide the detail needed for design and verification. They do not imply that each requirement must be implemented by a separate hardware mechanism.

| ID | Proposed requirement | Source | Status |
|---|---|---|---|
| SSR-001A | The ADC acquisition function shall make the result of a safety-relevant conversion available with an error within the allocated acquisition tolerance under the specified operating conditions. | FF-ADC-001A | Provisional |
| SSR-001B | The acquisition interface shall provide a means for the consumer to associate each safety-relevant conversion result with its intended input channel. | FF-ADC-001B | Provisional |
| SSR-001C | The acquisition interface shall provide a means for the consumer to distinguish a newly completed safety-relevant conversion from a previously available result. | FF-ADC-001C | Provisional |
| SSR-001D | The acquisition function shall provide conversion completion or failure status sufficient for the consumer to identify when a required conversion has not completed within the allocated acquisition deadline. | FF-ADC-001D | Provisional |
| SSR-001E | The acquisition function shall support the allocated sampling and result-availability timing constraints for safety-relevant feedback. | FF-ADC-001E | Provisional |
| SSR-001F | The acquisition interface shall preserve the defined association between conversion data and its completion/validity status when the consumer accesses that information. | FF-ADC-001F | Provisional |

**Requirement boundary:** `SSR-001A` specifies expected conversion performance, not diagnostic detection of every conversion error. Likewise, `SSR-001C` provides evidence of a new conversion; it does not prove that the external physical signal has changed. Fault detection obligations will need separate allocation once the safety strategy is defined.

### 2.1 Required refinements before baselining

| Requirement | Parameter or interface decision needed | Reason |
|---|---|---|
| SSR-001A | Signal range, acquisition tolerance, environmental/operating conditions and calibration assumptions | Without these, “incorrect result” is not objectively testable |
| SSR-001B | Channel identity encoding, configuration ownership and interface boundary | A numerically correct value can belong to the wrong channel |
| SSR-001C | Definition of a new conversion; completion indicator, counter or equivalent interface behaviour | Equal consecutive codes can be valid and do not establish freshness |
| SSR-001D | Required conversion deadline and expected consumer timeout behaviour | Missing data becomes safety-relevant only in relation to its use |
| SSR-001E | Trigger source, sampling instant, latency and permissible jitter | Timing failures may change the meaning of feedback |
| SSR-001F | Register/buffer update and read consistency rules | Data and status must refer to the same acquisition instance |

The word **shall** is used to record candidate obligations. It does not mean the requirements are complete, measurable or approved.

## 3. Consumer and fault-response allocation

The ADC cannot, by itself, guarantee a safe JCU output. A correct ADC interface may still be used incorrectly by software, and a detected acquisition fault requires an allocated response. The following requirements are therefore held at the MCU integration boundary pending hardware/software partitioning.

| ID | Candidate MCU integration requirement | Allocation / open issue |
|---|---|---|
| SSR-007 | The MCU integration shall prevent safety-relevant control processing from treating acquisition data identified as invalid or expired as valid current feedback. | Consumer qualification and permissible fallback: TBD |
| SSR-008 | The MCU integration shall provide a defined response when required safety-relevant ADC feedback is unavailable or invalid beyond its allocated limit. | Reaction owner, external power-stage responsibility and timing: TBD |
| SSR-009 | Safety-relevant acquisition faults requiring system action shall be communicated to the allocated fault-handling function through a defined indication path. | Hardware/software detection and signalling path: TBD |

These are **interface and integration obligations**, not claims that the ADC IP itself implements control fallback or external power-stage shutdown. `SSR-008` must be refined against the JCU's actual safe behaviour; “reset the MCU” is not assumed to be safe.

## 4. Requirements retained for other IPs

The following identifiers are carried forward from v0.1 for continuity. They remain **topics requiring decomposition**, not sufficiently specified implementation requirements.

| ID | Element | Requirement topic | Why not yet baselined |
|---|---|---|---|
| SSR-002 | CPU | Detect or control safety-relevant incorrect execution/computation | Execution architecture, error model and reaction allocation undefined |
| SSR-003 | SRAM | Protect safety-relevant working data against specified corruption | Memory regions, error classes and handling undefined |
| SSR-004 | Interconnect | Protect safety-relevant transactions against specified transfer failures | Data, address, control, loss and timing failure coverage not partitioned |
| SSR-005 | Clock | Detect or control safety-relevant clock deviations | Clock domains, reference and detection limits undefined |
| SSR-006 | Fault handling | Convey and react to allocated safety-relevant fault indications | Fault classes, reaction path and time budget undefined |

The requirements for program NVM, timer, communication, reset, power and output functions have not yet been derived. This is a known coverage limitation, not evidence that those elements are non-safety-related.

## 5. Traceability and rationale

| Failure analysed in WP02 | Candidate requirement(s) | Principal concern addressed |
|---|---|---|
| FF-ADC-001A — Incorrect result | SSR-001A; possible future diagnostic requirement | Result integrity and allocated error tolerance |
| FF-ADC-001B — Wrong channel | SSR-001B | Source identity |
| FF-ADC-001C — Stale result | SSR-001C, SSR-007 | Acquisition instance and consumer freshness decision |
| FF-ADC-001D — Missing conversion | SSR-001D, SSR-008 | Deadline monitoring and handling of unavailable feedback |
| FF-ADC-001E — Wrong timing | SSR-001E, SSR-007 | Sample and delivery timing |
| FF-ADC-001F — Data/status mismatch | SSR-001F, SSR-007 | Consistent result interpretation |
| Multiple ADC failure scenarios | SSR-009 | Fault indication, if detection is allocated |

These links show **why a requirement is proposed**. They do not establish that satisfying it alone prevents `FB-001` or `FB-005`. That claim would require a defined consumer architecture, fault-response concept and evidence.

## 6. Safety mechanism and verification implications

No mechanism is selected or credited here. The following points constrain the next design iteration:

- A range check cannot be assumed to detect a plausible but inaccurate conversion.
- A channel identifier can support association checks, but it may be wrong if its source or configuration is affected by the same fault.
- A completion flag or counter can help identify new conversions, but only if its update behaviour is defined and its failure modes are considered.
- Comparing two channels is not necessarily independent when they share a reference, supply, trigger or downstream path.
- Detection latency and reaction latency are separate contributions to the available fault-tolerant time budget; that budget is not yet allocated.

Potential verification approaches, once requirements are completed, include interface-level tests, register/protocol tests, timing measurements, controlled fault injection and analysis of shared dependencies. No verification results are claimed.

## 7. Decisions needed for v0.3

1. Identify the actual feedback quantity and the ADC-to-consumer interface contract.
2. Allocate permissible acquisition error, sample age, update deadline and timing constraints from the JCU control and safety context.
3. Decide which failures are handled inside ADC hardware, by another SoC mechanism, or by consumer software.
4. Define fault indication, invalid-data handling and the responsibility of the external power stage.
5. Derive **separate diagnostic requirements** for failures that must be detected, including detection conditions and time limits.
6. Review the resulting allocation in WP04, then challenge it through FTA, DFA and verification planning.

**Revision note:** v0.2 develops the ADC requirement set while deliberately leaving the other IP requirements preliminary. The analysis is expected to iterate: an unachievable or unverifiable requirement may require changes to WP01 architecture assumptions or WP02 failure propagation.
