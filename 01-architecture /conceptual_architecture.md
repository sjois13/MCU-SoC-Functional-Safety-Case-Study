# Conceptual MCU/SoC Architecture

**Version:** v0.1 — Work in Progress

This document establishes the initial architecture and failure-analysis basis for a conceptual safety-related MCU/SoC. Anything not defined is **TBD**; no quantitative values are assumed without a stated basis.

## 1. Boundary and Assumptions

The conceptual MCU/SoC is the primary controller of one humanoid Joint Control Unit (JCU).

![System Boundary](01-architecture / diagrams /system_boundary.png) 

*Figure 1 — System context and boundary of the conceptual MCU/SoC within the Joint Control Unit.*

Robot-level environment interpretation and hazard analysis remain outside the case-study boundary.

**ASM-001 — MCU allocation:** One conceptual MCU/SoC is the primary controller of each JCU.

**ASM-002 — Upstream safety context:** The application is assumed to have an upstream safety need concerning unintended or uncontrolled joint motion caused by incorrect JCU behaviour. Integrity classification is outside scope.

**ASM-003 — Feedback acquisition:** At least one analog feedback path is acquired through the ADC. Timer/capture functionality may acquire timing- or pulse-based digital feedback. Exact sensors and interfaces are TBD.

## 2. MCU Functions

At the current level of abstraction, the MCU/SoC performs the following functions:

- receive motion requests and constraints;
- acquire joint/motor feedback;
- determine information representing the current joint state;
- calculate the local control action; and
- provide a control command toward the external power stage.

Control-law design is outside scope.

## 3. Conceptual Architecture

| ID | Element | Role |
|---|---|---|
| ARCH-CPU-001 | CPU | Software execution and control computation |
| ARCH-ADC-001 | ADC | Analog feedback acquisition |
| ARCH-SRAM-001 | SRAM | Working data storage |
| ARCH-NVM-001 | Program NVM | Non-volatile program storage |
| ARCH-INT-001 | Interconnect | Transfers transactions between SoC elements |
| ARCH-COM-001 | Communication peripheral | Interface to robot-level controller |
| ARCH-TMR-001 | Timer | Timing, measurement and applicable capture functions |
| ARCH-CLK-001 | Clock | Timing reference |
| ARCH-RST-001 | Reset | Establishes defined state |
| ARCH-PWR-001 | Power-related function | SoC supply/conditioning/distribution; boundary TBD |
| ARCH-OUT-001 | Output peripheral | Interface toward external power stage |

![Conceptual MCU/SoC Architecture](diagrams/conceptual_soc_architecture.png)

*Figure 2 — Initial conceptual MCU/SoC architecture and principal information paths.*

`ARCH-CLK-001`, `ARCH-RST-001` and `ARCH-PWR-001` are not shown in this information-flow view. Their relationships to the other SoC elements remain TBD.

Watchdog, DMA, debug and security functions are outside v0.1.

## 4. MCU Boundary Failure Behaviours

`FB` is a project-specific identifier for an externally observable MCU failure behaviour.

| ID | Failure behaviour |
|---|---|
| FB-001 | Incorrect control output value |
| FB-002 | Control output at incorrect time |
| FB-003 | Expected control output missing or stale |
| FB-004 | Unintended/spurious control output |
| FB-005 | Incorrect response to otherwise correct input/feedback |

The list is preliminary and not claimed to be exhaustive.

## 5. Initial IP/SoC Functional Failure Candidates

| ID | Element | Candidate failure | Potential boundary effect |
|---|---|---|---|
| FF-CLK-001 | Clock | Incorrect clock behaviour | FB-002; others TBD |
| FF-CPU-001 | CPU | Incorrect computation | FB-001 |
| FF-SRAM-001 | SRAM | Incorrect stored working data | FB-001 / FB-005 |
| FF-INT-001 | Interconnect | Incorrect transaction transfer | FB-001 / FB-002 / FB-003 / FB-005 |
| FF-RST-001 | Reset | Failure to establish intended defined state | TBD |

For `FF-INT-001`, candidate transfer failures currently include **data corruption, loss, incorrect timing, wrong destination, unintended additional delivery, duplication, incorrect ordering and unintended source**.

A storage failure is distinguished from a transfer-path failure: correct information in SRAM that becomes incorrect during transfer is not automatically classified as an SRAM failure.

Failure candidates for ADC, communication, timer, NVM, power and output functions remain TBD.

## 6. Safety Mechanisms

No safety mechanisms or diagnostic coverage are allocated in v0.1.

Mechanisms will be derived after functional failures and failure propagation are analysed. Any future use of redundancy will also require analysis of shared dependencies such as clock, power and interconnect.

## 7. Next

The next iteration will develop:

**functional decomposition → IP functional failures → failure propagation → semiconductor safety requirements → fault reactions → safety mechanisms → FTA/FMEDA/DFA → verification and traceability**

Failure rates, diagnostic coverage, SPFM/LFM values and verification evidence remain TBD.
