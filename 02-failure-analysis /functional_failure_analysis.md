# Functional Failure Analysis — Conceptual MCU/SoC

**Version:** v0.2 — Working Draft 

**Status:** Focused ADC analysis; other IPs retained as preliminary candidates

## 1. Purpose and Scope

This work product analyses how failures of selected MCU/SoC functions could propagate to externally observable MCU boundary behaviours. It builds on the [Conceptual MCU/SoC Architecture](../01-architecture%20/conceptual_architecture.md), particularly its ADC decomposition in Section 3.1.

The first detailed study covers the **ADC acquisition path** (`ARCH-ADC-001`). CPU, SRAM, interconnect, clock and reset remain at preliminary-analysis depth. This staged approach is deliberate: a sufficiently detailed analysis of one representative IP provides a reusable method and exposes architectural questions before extending the same method to other IPs.

This is a **functional failure analysis**, not a transistor-level failure-mode analysis, silicon FMEDA, completed FTA, or assertion that any particular commercial MCU implements these functions. No failure rates, diagnostic coverage, safety integrity classification or reaction times are assumed.

## 2. Analysis Method and Boundary

For each selected failure, the analysis follows:

```text
Intended IP function
       ↓
Functional deviation
       ↓
Local effect at the IP/interface
       ↓
Propagation conditions and dependencies
       ↓
Effect on MCU processing or command generation
       ↓
Potential MCU boundary failure behaviour
```

**Terminology:**

- **Functional failure:** Deviation from the intended behaviour of an architectural function, independent of its physical root cause.
- **Local effect:** What becomes incorrect or unavailable at the affected function's interface.
- **Propagation condition:** An additional condition needed for the local effect to influence another function or the MCU boundary; it must not be silently assumed.
- **MCU boundary effect:** A potentially observable failure behaviour at the MCU interface toward the JCU or external power stage.
- **Candidate cause:** A possible contributing hardware/configuration/dependency failure, not a proven root cause or quantified failure mode.

The external sensor, analog signal quality, power stage and robot-level response are outside the MCU/SoC boundary. Their behaviour can influence propagation but is not attributed to internal ADC hardware without evidence.

### 2.1 MCU boundary failure behaviour reference

| ID | Behaviour |
|---|---|
| FB-001 | Incorrect control output value |
| FB-002 | Control output at incorrect time |
| FB-003 | Expected control output missing or stale |
| FB-004 | Unintended/spurious control output |
| FB-005 | Incorrect response to otherwise correct input/feedback |

These project-specific behaviours are not necessarily mutually exclusive. In particular, `FB-005` describes an input-to-response relationship and may overlap with `FB-001` or `FB-002`.

## 3. ADC Intended Function and Analysis Assumptions

The ADC acquisition path is expected to make a digital representation of the **intended analog input** available to its consumer, with sufficient validity and timeliness for the application. Exact accuracy, update timing and freshness limits are TBD.

```text
External analog feedback
        ↓
Analog input / channel selection
        ↓
Sampling and conversion  ←  reference / clock / trigger
        ↓
Result and status interface
        ↓
SoC interconnect
        ↓
CPU / feedback processing
        ↓
Local control computation
        ↓
MCU output peripheral → external power stage
```

**Working assumptions for conditional propagation only:**

- **FA-ASM-001:** At least one ADC result is used in local control or state estimation (consistent with `ASM-004` in WP01).
- **FA-ASM-002:** A consumer may use an ADC value unless it rejects or qualifies that value; actual consumer behaviour is TBD.
- **FA-ASM-003:** No ADC-specific diagnostic or fault reaction is credited in this analysis. This does **not** assert that a real implementation lacks diagnostics.
- **FA-ASM-004:** A plausible-looking incorrect value may not be distinguishable from a valid value without additional information; the presence and effectiveness of such information are TBD.

A functional failure in the acquisition path does not automatically produce an incorrect MCU output. Propagation depends on data usage, control sensitivity, software behaviour, timing and any protection or fallback behaviour.

## 4. ADC Functional Failure Catalogue

`FF-ADC-001` is the parent topic **incorrect or invalid ADC acquisition information**. The suffixes below identify distinct deviations for analysis; they are not separate physical fault classifications.

| ID | Functional deviation | Local effect at ADC interface | Illustrative candidate contributors (not exhaustive) |
|---|---|---|---|
| FF-ADC-001A | Incorrect conversion value | Result differs from the intended analog quantity beyond a yet-to-be-defined tolerance | Conversion circuitry, reference deviation, sampling disturbance |
| FF-ADC-001B | Wrong input channel acquired | Result corresponds to a different analog input | Channel-selection logic, configuration corruption, sequencing |
| FF-ADC-001C | Result stale or repeated | Previously acquired value presented as if current | Trigger/sequence failure, result-update failure, status handling |
| FF-ADC-001D | Conversion absent or incomplete | No usable new result when expected | Conversion control, clock, trigger, supply/reset dependency |
| FF-ADC-001E | Conversion/result available at incorrect time | Sample age or delivery time violates the intended schedule | Trigger timing, clock deviation, conversion sequencing |
| FF-ADC-001F | Result/status inconsistency | Data and validity/completion indication do not describe the same conversion | Result/status interface, synchronization, buffering |

**Attribution boundary:** Corruption after the ADC result has correctly left its defined interface may belong to `FF-INT-001` rather than `FF-ADC-001`. The exact interface boundary requires refinement when the bus/register architecture is defined. Likewise, an erroneous external sensor voltage is an input-path/system-interface concern, not automatically an internal ADC failure.

## 5. Selected ADC Failure Propagation Analyses

### 5.1 FF-ADC-001A — Incorrect but plausible conversion result

**Intended behaviour:** The result represents the sampled analog input within defined accuracy limits (limits TBD).

**Functional deviation:** The ADC reports a numerically plausible but inaccurate value.

**Local effect:** The result register/interface exposes incorrect feedback data without necessarily indicating an error.

**Conditional propagation:**

```text
Incorrect but plausible ADC result
               ↓
Consumer accepts result as valid
               ↓
Incorrect feedback / estimated joint state
               ↓
Control computation uses affected feedback
               ↓
Potential incorrect output command (FB-001 / FB-005)
```

**Conditions that must hold:** The affected signal is used in the relevant computation; its error is not rejected or compensated; and the computation is sensitive enough to change the resulting command. Whether these conditions hold is TBD.

**Important limitation:** A simple range check may accept an incorrect value that lies within the permitted range. This observation motivates later diagnostic analysis but does not select a mechanism here.

**Open questions:** What is the feedback quantity? What error magnitude matters? Is an independent reference or other feedback available? How does software qualify the value?

### 5.2 FF-ADC-001B — Wrong channel acquired

**Intended behaviour:** The requested conversion corresponds to the configured channel and intended physical signal.

**Functional deviation:** A result from another channel is delivered or labelled as the intended channel.

**Local effect:** The numerical result may be internally consistent but semantically associated with the wrong signal.

**Conditional propagation:** If the consumer cannot distinguish the channel mismatch and uses the value for the wrong feedback variable, the MCU may calculate an incorrect command (`FB-001` / `FB-005`). If the consumer detects the mismatch or ignores the result, boundary propagation may be prevented.

**Important distinction:** This is not equivalent to a bit error in the converted value. A data-integrity check over a correctly transmitted wrong-channel result would not, by itself, establish correct channel identity.

**Open questions:** How is channel identity represented? Are conversions multiplexed? Is the sequence controlled by software or hardware? What configuration dependencies are shared?

### 5.3 FF-ADC-001C — Stale or repeated result

**Intended behaviour:** A newly required sample is distinguishable from an earlier sample and available within the application's freshness requirement (TBD).

**Functional deviation:** The result remains unchanged because a new acquisition or result update did not occur, or an earlier value is reused.

**Local effect:** The consumer can observe a plausible value that no longer represents the current physical state.

**Conditional propagation:**

```text
New acquisition/result update does not occur
                 ↓
Previous result remains accessible
                 ↓
Consumer treats previous result as fresh
                 ↓
Control uses outdated feedback
                 ↓
Potential FB-001 / FB-005
```

If the stale data causes the MCU to omit or delay an expected command, `FB-002` or `FB-003` may also be relevant; that path is not established by the ADC fault alone.

**Important distinction:** An unchanged physical signal can legitimately yield repeated ADC codes. Equal successive values do not prove stale acquisition.

**Open questions:** What marks a new conversion? Is there a timestamp, sequence counter, completion flag or other freshness evidence? What happens if a new sample is late?

### 5.4 FF-ADC-001D/E — Missing or mistimed conversion

**Intended behaviour:** Acquisition occurs in the intended control-cycle relationship, and results become available when required.

**Functional deviation:** The conversion does not complete, or the sampling/delivery timing deviates from the intended schedule.

**Local effect:** A new result is missing, late, or associated with an unintended sampling instant.

**Conditional propagation:** Consumer behaviour determines whether the MCU reuses old feedback, waits for data, proceeds with a substitute value, or changes output timing. Potential effects include `FB-001`, `FB-002`, `FB-003` and `FB-005`; none is automatic.

**Open questions:** Which trigger and clock domains are involved? What is the control-cycle deadline? Does a missing completion indication cause a timeout, blocking wait or fallback?

### 5.5 FF-ADC-001F — Result/status inconsistency

**Intended behaviour:** Completion/validity status accurately describes the result that the consumer reads.

**Functional deviation:** Status indicates a new or valid conversion while the associated data belongs to a different conversion or is not yet valid.

**Local effect:** The consumer may accept mismatched data and status.

**Conditional propagation:** A consumer relying on the incorrect status could use invalid or stale feedback and influence the command (`FB-001` / `FB-005`). A software protocol that establishes consistent data/status association could interrupt this path, but its existence and effectiveness are TBD.

**Open questions:** Are data/status reads atomic or otherwise synchronized? Can buffering or interrupt sequencing create ambiguity? What is the hardware/software responsibility split?

## 6. ADC Dependency and Common-Effect Considerations

These are **questions for architectural refinement and DFA**, not claims of implemented redundancy or independent channels.

| Shared or external dependency | Potential influence on acquisition | Analysis implication |
|---|---|---|
| Voltage reference | Systematic conversion bias across one or multiple channels | Two agreeing results need not both be correct |
| Analog supply / ground | Conversion accuracy or availability | Check whether acquisition and proposed monitoring share a disturbance |
| Conversion clock / trigger | Missing, shifted or mistimed samples | Data value checks alone may miss temporal invalidity |
| Channel configuration / sequencing | Wrong-channel or wrong-order results | Correct numerical transmission does not prove correct source identity |
| Reset / initialization | Invalid setup or unexpected acquisition state | Determine what consumer assumes after reset |
| SoC interconnect | Corrupted, delayed, misrouted or lost result/status transfer | Separate ADC-local failures from transfer-path failures |
| CPU / consumer software | Incorrect interpretation, validation or use of results | Distinguish an ADC-originating failure from downstream failure |

A common reference failure is particularly important: multiple conversion paths can produce mutually consistent but incorrect results if they share the same affected reference. Whether this scenario is applicable depends on the future reference and channel architecture.

## 7. Preliminary Traceability to MCU Boundary

| Functional failure | Local effect | Conditional propagation | Candidate boundary behaviour |
|---|---|---|---|
| FF-ADC-001A | Inaccurate digital result | Consumer accepts and uses incorrect feedback | FB-001 / FB-005 |
| FF-ADC-001B | Wrong signal represented | Channel mismatch not identified before use | FB-001 / FB-005 |
| FF-ADC-001C | Previous result reused | Staleness not identified; feedback affects control | FB-001 / FB-005; FB-002 / FB-003 only conditionally |
| FF-ADC-001D | New result unavailable | Consumer waits, reuses data or otherwise changes behaviour | FB-001 / FB-002 / FB-003 / FB-005, depending on consumer |
| FF-ADC-001E | Incorrect sample/delivery time | Timing deviation affects feedback interpretation or command timing | FB-001 / FB-002 / FB-005 |
| FF-ADC-001F | Inconsistent result and status | Consumer accepts mismatched data/status | FB-001 / FB-005 |

This table maps **possible propagation**, not demonstrated end-to-end causality. It does not imply that all listed effects occur in a given design.

## 8. Other IP/SoC Functional Failure Candidates — Current Coverage

The following topics are retained from the architecture baseline but are **not** analysed at the same depth as ADC in this revision.

| ID | Element | Candidate functional failure | Current next question |
|---|---|---|---|
| FF-CPU-001 | CPU | Incorrect computation | What execution/result path can corrupt the command, and what faults are distinguishable? |
| FF-SRAM-001 | SRAM | Incorrect stored working data | Which data is affected, and where is corruption introduced versus transferred? |
| FF-INT-001 | Interconnect | Incorrect transaction transfer | Which data, address, control and timing deviations affect which consumers? |
| FF-CLK-001 | Clock | Incorrect clock behaviour | Which domains and dependent functions are affected? |
| FF-RST-001 | Reset | Intended defined state not established | Which blocks or registers may remain incorrectly initialized? |

Communication, timer, program NVM, power-related and output-peripheral failure candidates remain to be developed. This is a **selected-IP study**, not an exhaustive SoC failure catalogue.

## 9. Inputs to Safety Requirements and Mechanism Analysis

The ADC analysis suggests several **requirement topics**, not finalized requirements:

- Accuracy/validity of safety-relevant acquisition data at the defined consumer interface
- Correct association between result, input channel and acquisition instance
- Detection or handling of missing, stale or late acquisition information
- Defined treatment of invalid data and status at the consumer boundary
- Defined fault indication and reaction responsibilities between ADC, CPU/software, MCU output and external JCU
- Shared-reference, clock, power and interconnect dependencies that may limit proposed diagnostics

The [Semiconductor Safety Requirements](../03-safety-requirements/semiconductor_safety_requirements.md) must be refined only after the application-relevant validity limits, timing assumptions and fault-response responsibilities are specified. Candidate mechanisms belong in the [Safety Architecture and Mechanism Allocation](../04-safety-architecture/safety_mechanism_allocation.md); none is credited here.

## 10. Open Items and Next Iteration

| Open item | Why it matters | Intended follow-up |
|---|---|---|
| ADC signal identity and permissible error | Determines what constitutes an incorrect result | WP01 / WP03 |
| Sampling schedule and freshness criteria | Distinguishes valid repeated values from stale data | WP01 / WP03 |
| Channel sequencing and result/status protocol | Defines wrong-channel and mismatch propagation | WP01 / WP02 |
| Reference, clock, power and reset architecture | Establishes shared dependencies and common effects | WP01 / DFA |
| Consumer acceptance, timeout and fallback behaviour | Determines whether a local failure reaches the MCU boundary | WP02 / WP03 / WP04 |
| MCU-to-power-stage interface and safe reaction | Defines responsibility for preventing or limiting an unsafe command | WP01 / WP03 / WP04 |
| Fault injection points and observable outcomes | Enables a meaningful verification strategy | Verification planning |
| ADC physical failure-mode and failure-rate data | Required before credible quantitative FMEDA | FMEDA, when justified data exists |

**Next focused step:** Review `FF-ADC-001A` and `FF-ADC-001C` against the intended JCU feedback usage, then derive a small set of measurable, interface-specific ADC safety requirements. Iterate back to WP01 if the analysis exposes missing architecture. Later, apply the same decomposition-to-propagation method to CPU, SRAM and interconnect.
