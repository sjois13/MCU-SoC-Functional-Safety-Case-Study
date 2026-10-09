# Functional Failure Analysis

**Version:** v0.1 — Work in Progress

## 1. Purpose

This work product develops the initial functional failure analysis of the conceptual MCU/SoC defined in the [Architecture Baseline](../01-architecture%20/conceptual_architecture.md).

The analysis follows:

```text
Architecture element
        ↓
Intended function
        ↓
Functional failure
        ↓
Local effect
        ↓
Propagation through SoC
        ↓
MCU boundary failure
        ↓
Safety relevance
```

The objective at this stage is qualitative failure reasoning, not quantitative FMEDA.

Only functions and failures for which a meaningful propagation argument has been developed are included. Absence of an element from the analysis does not imply that it is safety-irrelevant.

---

## 2. Analysis Scope

The first iteration focuses on:

- ADC;
- CPU;
- SRAM;
- interconnect; and
- clock.

Communication, timer, NVM, reset, power-related functions and the output peripheral remain to be analysed.

The functional failures below describe incorrect behaviour of an IP/SoC function. They are not claims about transistor-, gate- or physical-layout-level failure mechanisms.

### Relationship to the Architecture Baseline

The architecture baseline identifies broad initial functional failure candidates (`FF-xxx`).

This analysis retains those identifiers and refines a broad candidate into subordinate functional failures where additional distinction is required for propagation analysis.

For example:

```text
ARCH-INT-001
Interconnect
        ↓
FF-INT-001
Incorrect transaction transfer
        ↓
        ├── FF-INT-001.1  Data corruption
        ├── FF-INT-001.2  Non-delivery
        ├── FF-INT-001.3  Incorrect timing
        ├── FF-INT-001.4  Wrong destination
        ├── FF-INT-001.5  Unintended additional delivery
        ├── FF-INT-001.6  Duplication
        ├── FF-INT-001.7  Incorrect ordering
        └── FF-INT-001.8  Wrong source association
```

This refinement allows failures with different propagation behaviour to be analysed separately while retaining traceability to the architecture baseline.

---

## 3. Functional Failure Analysis

| ID | Element / Intended Function | Functional Failure | Local Effect | Example Propagation | MCU Boundary Effect |
|---|---|---|---|---|---|
| FF-ADC-001 | **ADC** — convert an analog feedback input into a digital representation | Incorrect digital conversion result | Digital value does not correctly represent the applied analog input | CPU receives apparently valid but incorrect feedback and may calculate an incorrect control action | FB-001 / FB-005 |
| FF-CPU-001 | **CPU** — execute software and calculate the local control action | Incorrect computation | Calculated result differs from the expected result for otherwise correct input data and execution context | Incorrect calculated command propagates toward the output path | FB-001 |
| FF-SRAM-001 | **SRAM** — retain working data required during execution | Incorrect stored working data | Subsequent use of the affected location provides incorrect working data | CPU may use corrupted state or control data during computation and calculate an incorrect command | FB-001 / FB-005 |
| FF-INT-001 | **Interconnect** — transfer transactions between SoC elements | Incorrect transaction transfer | One or more required properties of the transaction transfer are violated | Refined by FF-INT-001.1 through FF-INT-001.8 below | FB-001 / FB-002 / FB-003 / FB-004 / FB-005 depending on affected transaction |
| FF-INT-001.1 | **Interconnect** — preserve information during transfer | Transaction data is corrupted during transfer | Destination receives information different from that produced by the source | Incorrect information may be consumed by CPU, memory or peripheral depending on the affected path | FB-001 / FB-005; path dependent |
| FF-INT-001.2 | **Interconnect** — deliver required transactions | Required transaction is not delivered | Intended destination does not receive the requested information or operation | Destination may retain stale information or fail to perform the requested operation | FB-003; additional effects depend on transaction |
| FF-INT-001.3 | **Interconnect** — satisfy required transfer timing | Transaction is delivered outside required timing | Information or operation is presented at an unintended time | Receiving function may use stale information or perform an action too early or too late | FB-002 / FB-003 |
| FF-INT-001.4 | **Interconnect** — route transaction to the intended destination | Transaction is delivered to an unintended destination instead of the intended destination | Intended destination misses the transaction and another element receives it | Required action may be absent while an unintended action may occur elsewhere | FB-003 / potentially FB-004 |
| FF-INT-001.5 | **Interconnect** — restrict delivery to the intended destination(s) | Transaction is additionally delivered to an unintended destination | Intended destination may operate correctly while another element also receives an unintended transaction | Unintended receiving element may perform an unintended operation | Potentially FB-004; path dependent |
| FF-INT-001.6 | **Interconnect** — preserve required transaction occurrence | Transaction is duplicated | Destination observes the same transaction more times than intended | A repeated write, command or peripheral action may occur depending on transaction semantics | TBD; potentially FB-004 |
| FF-INT-001.7 | **Interconnect** — preserve required transaction ordering | Required transaction ordering is violated | Destination observes otherwise valid transactions in an unintended sequence | Dependent operations may execute in the wrong sequence and produce an incorrect downstream state | TBD |
| FF-INT-001.8 | **Interconnect** — preserve intended source association | Transaction/information is associated with an unintended source | Destination acts on information as though it originated from another source | Downstream behaviour depends on how source identity affects interpretation or access | TBD |
| FF-CLK-001 | **Clock** — provide the timing reference required by dependent functions | Incorrect clock behaviour | Timing behaviour of functions using the affected clock is altered | Computation, acquisition or output activity may occur at an incorrect time | FB-002; additional effects TBD |

---

## 4. Failure Attribution

Failure attribution matters because the same incorrect information observed by an IP can originate from different parts of the SoC.

For example:

```text
Correct data stored in SRAM
        ↓
SRAM provides the correct data
        ↓
Data is corrupted during transfer
        ↓
CPU receives incorrect data
```

In this case, the incorrect value observed at the CPU does not by itself establish an SRAM storage failure.

At the current abstraction level, if the SRAM provided the correct information and that information changed during transfer, the originating functional failure is associated with the transfer path.

Conversely:

```text
Data stored incorrectly in SRAM
        ↓
SRAM provides the corrupted stored value
        ↓
Interconnect transfers that value correctly
        ↓
CPU receives incorrect data
```

Here the interconnect has performed its transfer function correctly. The originating functional failure is associated with storage.

This distinction is important because different originating failures may require different safety mechanisms.

It also prevents a downstream symptom from being incorrectly treated as evidence that the receiving or transmitting IP itself caused the failure.

---

## 5. Interconnect Functional Properties

The interconnect analysis identified several properties that may need to be preserved for a transaction to be considered correct.

| Required property | Candidate functional failure |
|---|---|
| Information integrity | Data corruption |
| Required delivery | Transaction loss / non-delivery |
| Intended destination | Wrong-destination delivery |
| Delivery only to intended destination(s) | Unintended additional delivery |
| Required occurrence | Missing, duplicated or unintended transaction |
| Required timing | Early or late delivery |
| Required ordering | Incorrect transaction sequence |
| Intended source association | Information associated with an unintended source |

These are conceptual functional properties.

The exact guarantees, timing constraints, ordering rules and source/destination semantics depend on the eventual interconnect architecture and are currently TBD.

Not every transaction necessarily requires the same ordering or timing behaviour. These requirements must therefore be defined from the specific transaction and architecture rather than assumed globally.

---

## 6. Initial Failure Propagation Observations

The analysis shows that an externally similar MCU failure can originate from different internal functions.

### Example A — ADC-originated failure

```text
Incorrect ADC conversion
        ↓
Incorrect digital representation of feedback
        ↓
CPU uses incorrect feedback
        ↓
Incorrect calculated control action
        ↓
FB-001
Incorrect control output value
```

### Example B — CPU-originated failure

```text
Correct input / feedback information
        ↓
Incorrect CPU computation
        ↓
Incorrect calculated control action
        ↓
FB-001
Incorrect control output value
```

### Example C — Interconnect-originated failure

```text
Correct information produced by source
        ↓
Information corrupted during transfer
        ↓
Incorrect information received by destination
        ↓
Incorrect downstream behaviour
        ↓
FB-001 / FB-005
depending on the affected path
```

These examples demonstrate that the MCU boundary failure behaviour alone is not sufficient to identify the originating internal functional failure.

For example, observing `FB-001` does not by itself establish whether the originating failure occurred in the ADC, CPU, SRAM, interconnect, output path or another element.

The internal failure source and propagation path therefore matter when later deriving safety requirements and allocating safety mechanisms.

---

## 7. Shared Dependencies

Some architecture elements may influence multiple functions or IP blocks.

Initial candidates include:

- clock generation and distribution;
- reset generation and distribution;
- power supply, management or distribution; and
- interconnect resources shared by multiple IP elements.

At this stage these are treated only as **candidates for later dependent failure analysis**.

No claim is made that a dependent failure exists merely because a resource is shared.

Similarly, independence between duplicated or redundant functions cannot be assumed solely because more than one implementation exists. Shared resources and common dependencies must be considered when independence is later claimed.

---

## 8. Open Analysis

The following remain open:

- communication peripheral functional failures;
- timer/capture functional failures;
- program NVM functional failures;
- reset functional failures;
- power-related functional failures;
- output peripheral functional failures;
- additional ADC failure behaviours;
- additional CPU and SRAM failure behaviours;
- more precise transaction timing requirements;
- transaction-specific ordering requirements;
- propagation through specific SoC paths;
- safe-state and fault-reaction behaviour;
- fault detection and handling requirements;
- physical semiconductor failure mechanisms; and
- quantitative failure data.

The current analysis is therefore not a complete FMEA or FMEDA.

No diagnostic coverage, failure rates, SPFM or LFM values are assigned in this work product.

---

## 9. Next Step

The next analysis step is to determine which functional failures can contribute to the assumed upstream safety concern and what behaviour is required from the MCU/SoC to prevent, detect or control their propagation.

The resulting requirements will form the initial set of semiconductor-level safety requirements.

Safety mechanisms will be selected only after those requirements are established.

The intended progression is:

```text
Functional failure
        ↓
Failure propagation
        ↓
MCU boundary failure
        ↓
Safety relevance
        ↓
Semiconductor safety requirement
        ↓
Safety mechanism allocation
        ↓
Verification
```
