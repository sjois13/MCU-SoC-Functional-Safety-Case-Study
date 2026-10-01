# Functional Failure Analysis

**Version:** v0.1 — Work in Progress

## 1. Purpose

This work product develops the initial functional failure analysis of the conceptual MCU/SoC defined in the [architecture baseline](../architecture/conceptual_architecture.md).

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

## 2. Analysis Scope

The first iteration focuses on:

- ADC;
- CPU;
- SRAM;
- interconnect; and
- clock.

Communication, timer, NVM, reset, power-related functions and the output peripheral remain to be analysed.

The functional failures below describe incorrect behaviour of an IP/SoC function. They are not claims about transistor-, gate- or physical-layout-level failure mechanisms.

## 3. Functional Failure Analysis

| ID | Element / Intended Function | Functional Failure | Local Effect | Example Propagation | MCU Boundary Effect |
|---|---|---|---|---|---|
| FF-ADC-001 | **ADC** — convert an analog feedback input into a digital representation | Incorrect digital conversion result | Digital value does not correctly represent the applied analog input | CPU receives apparently valid but incorrect feedback and may calculate an incorrect control action | FB-001 / FB-005 |
| FF-CPU-001 | **CPU** — execute software and calculate the local control action | Incorrect computation result | Calculated result differs from the result expected for otherwise correct input data and software execution context | Incorrect calculated command propagates through the output path | FB-001 |
| FF-SRAM-001 | **SRAM** — retain working data required during execution | Stored working data is corrupted | Subsequent read returns incorrect working data | CPU uses corrupted state or control data during computation and may calculate an incorrect command | FB-001 / FB-005 |
| FF-INT-001 | **Interconnect** — transfer transactions between intended SoC elements | Transaction data is corrupted during transfer | Destination receives information different from that produced by the source | Incorrect information may be consumed by CPU, memory or peripheral depending on the affected path | FB-001 / FB-005; path dependent |
| FF-INT-002 | **Interconnect** — transfer required transactions | Required transaction is not delivered | Intended destination does not receive the requested information or operation | Destination may retain stale information or fail to perform the requested operation | FB-003; additional effects depend on transaction |
| FF-INT-003 | **Interconnect** — transfer transactions within required timing | Transaction is delivered outside its required timing | Correct information is available at the wrong time | Receiving function may operate on stale information or perform an action too early/late | FB-002 / FB-003 |
| FF-INT-004 | **Interconnect** — deliver transactions to the intended destination | Transaction is delivered to an unintended destination | Intended destination may miss the transaction while another element receives an unintended transaction | Required action may be absent and/or an unintended action may occur elsewhere | FB-003 / potentially FB-004 |
| FF-INT-005 | **Interconnect** — preserve required transaction occurrence and ordering | Transaction is duplicated or required ordering is violated | Destination observes an additional operation or operations in an unintended sequence | Result depends on the affected peripheral and transaction semantics | TBD |
| FF-CLK-001 | **Clock** — provide the timing reference required by dependent functions | Incorrect clock timing/frequency | Timing behaviour of functions using the affected clock is altered | Computation, acquisition or output activity may occur at an incorrect time | FB-002; additional effects TBD |

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

In this case, the observed incorrect value at the CPU does not by itself establish an SRAM storage failure. At the current abstraction level, the failure is attributed to the transfer path if the SRAM provided the correct information and the information changed during transfer.

Conversely:

```text
Data stored incorrectly in SRAM
        ↓
SRAM returns the corrupted stored value
        ↓
Interconnect transfers that value correctly
        ↓
CPU receives incorrect data
```

Here the interconnect has performed its transfer function correctly; the originating failure is associated with storage.

This distinction is important because different originating failures may require different safety mechanisms.

## 5. Interconnect Functional Properties

The interconnect analysis identified several properties that may need to be preserved for a transaction to be considered correct.

| Required property | Candidate functional failure |
|---|---|
| Information integrity | Data corruption |
| Required delivery | Transaction loss / non-delivery |
| Intended destination | Wrong-destination delivery |
| Intended source association | Information associated with an unintended source |
| Required occurrence | Missing, duplicated or unintended transaction |
| Required timing | Early or late delivery |
| Required ordering | Incorrect transaction sequence |

These are conceptual functional properties. The exact guarantees, timing constraints and ordering rules depend on the eventual interconnect architecture and are currently TBD.

## 6. Initial Observations

The analysis shows that an externally similar MCU failure can originate from different internal functions.

For example, an incorrect control output could result from:

```text
Incorrect ADC conversion
        ↓
incorrect feedback used by CPU
        ↓
incorrect calculated control action
        ↓
FB-001
```

or:

```text
Correct feedback
        ↓
incorrect CPU computation
        ↓
incorrect calculated control action
        ↓
FB-001
```

or:

```text
Correct information at source
        ↓
interconnect corruption
        ↓
incorrect information at destination
        ↓
incorrect downstream behaviour
        ↓
FB-001 / FB-005 depending on the affected path
```

Therefore, the MCU boundary failure behaviour alone is not sufficient to identify the originating internal failure or the appropriate safety mechanism.

## 7. Shared Dependencies

Clock, reset, power and shared interconnect resources may influence multiple IP elements.

At this stage these are treated only as **candidates for later dependent failure analysis**. Independence between duplicated or redundant functions is not assumed merely because more than one implementation exists.

## 8. Open Analysis

The following remain open:

- communication peripheral functional failures;
- timer/capture functional failures;
- program NVM functional failures;
- reset functional failures;
- power-related functional failures;
- output peripheral functional failures;
- more precise transaction timing and ordering requirements;
- propagation through specific SoC paths;
- safe-state and fault-reaction behaviour; and
- physical failure mechanisms and quantitative failure data.

No diagnostic coverage, failure rates, SPFM or LFM values are assigned in this work product.

## 9. Next Step

The next analysis step is to determine which functional failures require prevention, detection or control in order to satisfy the assumed upstream safety context.

That analysis will provide the basis for deriving preliminary semiconductor safety requirements before selecting specific safety mechanisms.
