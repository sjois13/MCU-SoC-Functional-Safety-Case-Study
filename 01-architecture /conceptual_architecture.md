# Conceptual MCU/SoC Architecture

**Version:** v0.2 — Working Draft   
**Status:** Conceptual architecture baseline; selected ADC decomposition added

This document establishes the initial architecture and failure-analysis basis for a conceptual safety-related MCU/SoC. The ADC acquisition subsystem is decomposed in greater detail as the first selected IP study. Anything not defined is **TBD**; no quantitative values are assumed without a stated basis.

## 1. Boundary and Assumptions

The conceptual MCU/SoC is the primary controller of one humanoid Joint Control Unit (JCU).

<img width="1112" height="208" alt="System context and MCU/SoC boundary within the JCU" src="https://github.com/user-attachments/assets/e65143d4-6995-4cdf-9faf-dfe240bad8a8" />

*Figure 1 — System context and boundary of the conceptual MCU/SoC within the Joint Control Unit.*

Robot-level environment interpretation and hazard analysis remain outside the case-study boundary.

**ASM-001 — MCU allocation:** One conceptual MCU/SoC is the primary controller of each JCU.

**ASM-002 — Upstream safety context:** The application is assumed to have an upstream safety need concerning unintended or uncontrolled joint motion caused by incorrect JCU behaviour. Integrity classification is outside scope.

**ASM-003 — Feedback acquisition:** At least one analog feedback path is acquired through the ADC. Timer/capture functionality may acquire timing- or pulse-based digital feedback. Exact sensors and interfaces are TBD.

**ASM-004 — ADC use:** At least one ADC result may influence the local control computation. The precise feedback quantity, update rate and permissible error are TBD.

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

<img width="1173" height="820" alt="Conceptual MCU/SoC architecture and principal information paths" src="https://github.com/user-attachments/assets/b363ccdd-c0cd-488b-a8b4-d10f469660c2" />

*Figure 2 — Initial conceptual MCU/SoC architecture and principal information paths.*

`ARCH-CLK-001`, `ARCH-RST-001` and `ARCH-PWR-001` are not shown in this information-flow view. Their relationships to the other SoC elements remain TBD.

Watchdog, DMA, debug and security functions are outside this iteration.

### 3.1 ADC Acquisition Subsystem — Initial Decomposition

`ARCH-ADC-001` represents the conceptual analog-to-digital acquisition function used to obtain feedback that may be relevant to joint or motor control. The purpose of this decomposition is to identify functional boundaries, interfaces and dependencies **before** detailed failure analysis or safety-mechanism selection.

#### 3.1.1 Functional path

```text
External analog feedback signal
              |
              v
     Analog input path
              |
              v
  Channel selection (if used)
              |
              v
     Sampling / conversion <---- Voltage reference
              ^
              |
       Conversion control
       (trigger / clock)
              |
              v
      Digital result / status
              |
              v
        SoC interconnect
              |
              v
    CPU / feedback processing
              |
              v
    Local control computation
```

*Figure 3 — Conceptual ADC acquisition path. This is a functional decomposition, not an implementation-level block diagram. The arrows show principal information/control dependencies; the exact conversion architecture and connectivity are TBD.*

#### 3.1.2 ADC subfunctions

| Subfunction | Intended role | Boundary or implementation question |
|---|---|---|
| Analog input path | Convey the external analog signal to the conversion function | Input conditioning and protection location: TBD |
| Channel selection | Select the intended input where multiplexing is used | Multiplexer presence and configuration: TBD |
| Sampling and conversion | Produce a digital representation of the selected analog quantity | ADC topology, resolution and sampling behaviour: TBD |
| Reference interface | Supply the reference used by the conversion function | Reference source and sharing: TBD |
| Conversion control/status | Initiate or sequence conversions and report status | Trigger source, sequencing and completion signalling: TBD |
| Result storage/interface | Make converted data available to another SoC function | Register organization, access and buffering: TBD |

The ADC result is conceptually consumed by CPU-executed feedback processing via the SoC interconnect. Whether transfers use polling, interrupts or another mechanism is TBD. The table describes **functions**, not an asserted partition into physical IP sub-blocks.

#### 3.1.3 Interfaces and dependencies

| Dependency or interface | Architectural relevance | Current status |
|---|---|---|
| External sensor/analog signal | Source of the quantity represented by the conversion result | Signal type and electrical characteristics: TBD |
| Analog and digital supplies | Support acquisition and conversion circuitry | Supply topology and sharing: TBD |
| Voltage reference | Influences the mapping from analog input to digital result | Source, stability and independence: TBD |
| Conversion clock / trigger | Influences when conversion occurs and whether a result is current | Clock domain and trigger source: TBD |
| Reset and configuration | Establish operating mode, channel selection and conversion setup | Initialization sequence and reset effects: TBD |
| SoC interconnect | Makes result/status information accessible to its consumer | Addressing, access control and transaction path: TBD |
| CPU/software consumer | Interprets feedback and uses it in control computation | Data-validity checks and timing assumptions: TBD |

A shared dependency may affect multiple acquisition channels or undermine a proposed comparison mechanism. Identifying such dependencies here does **not** establish their failure behaviour or diagnostic coverage; those questions belong to later analysis.

#### 3.1.4 MCU boundary relevance

An ADC result may influence the state estimate or feedback used by the local control computation and thus the command issued toward the external power stage. Consequently, the acquisition path is relevant to the MCU boundary behaviours in Section 4.

The actual propagation from an ADC malfunction to a boundary failure depends on the software's use of the data, timing, any other feedback sources and the external system response. These conditions will be investigated in the Functional Failure Analysis rather than assumed here.

#### 3.1.5 ADC open architectural decisions

- Feedback quantity, sensor interface and analog input conditioning
- ADC topology, resolution, channels and channel selection
- Sampling/trigger schedule and data freshness expectations
- Reference generation and distribution
- Result registers, status indications and interconnect access
- Clock, reset, power and configuration dependencies
- Consumer software interface and control-cycle relationship
- Potential diagnostic provisions, independence and fault response (not yet allocated)

No redundant ADC, internal diagnostic, fault reaction or quantitative performance is assumed in this architecture baseline.

## 4. MCU Boundary Failure Behaviours

`FB` is a project-specific identifier for an externally observable MCU failure behaviour.

| ID | Failure behaviour |
|---|---|
| FB-001 | Incorrect control output value |
| FB-002 | Control output at incorrect time |
| FB-003 | Expected control output missing or stale |
| FB-004 | Unintended/spurious control output |
| FB-005 | Incorrect response to otherwise correct input/feedback |

The list is preliminary and not claimed to be exhaustive. These are **MCU boundary behaviours**, not individual IP failure modes or a completed system-level hazard analysis.

## 5. Initial IP/SoC Functional Failure Candidates

This section is a preliminary index of failure-analysis topics. It is not a completed or exhaustive IP failure analysis; the detailed causes, local effects, propagation conditions and resulting boundary effects belong in the [Functional Failure Analysis](../02-failure-analysis/functional_failure_analysis.md).

| ID | Element | Candidate failure | Potential boundary effect |
|---|---|---|---|
| FF-CLK-001 | Clock | Incorrect clock behaviour | FB-002; others TBD |
| FF-CPU-001 | CPU | Incorrect computation | FB-001 |
| FF-SRAM-001 | SRAM | Incorrect stored working data | FB-001 / FB-005 |
| FF-INT-001 | Interconnect | Incorrect transaction transfer | FB-001 / FB-002 / FB-003 / FB-005 |
| FF-RST-001 | Reset | Failure to establish intended defined state | TBD |

For `FF-INT-001`, candidate transfer failures currently include **data corruption, loss, incorrect timing, wrong destination, unintended additional delivery, duplication, incorrect ordering and unintended source**.

A storage failure is distinguished from a transfer-path failure: correct information in SRAM that becomes incorrect during transfer is not automatically classified as an SRAM failure.

**ADC next:** The decomposition in Section 3.1 provides the basis for examining incorrect conversion results, wrong-channel acquisition, stale results and conversion timing issues in WP02. An ADC failure identifier and propagation mapping will be established there, rather than implying that these candidates have already been analysed.

Failure candidates for communication, timer, NVM, power and output functions remain TBD.

## 6. Safety Mechanisms

No safety mechanisms or diagnostic coverage are **established by this architecture baseline**. Candidate allocations, assumptions and limitations are addressed separately in the [Safety Architecture and Mechanism Allocation](../04-safety-architecture/safety_mechanism_allocation.md).

Mechanism choices must be justified against analysed functional failures and propagation paths. Any use of redundancy will require consideration of shared dependencies such as clock, power, reference, reset and interconnect.

## 7. Next Iteration and Traceability

The work products are developed iteratively:

```text
Architecture and interfaces (WP01)
              |
              v
Functional failures and propagation (WP02)
              |
              v
Semiconductor safety requirements (WP03)
              |
              v
Candidate mechanisms and fault reactions (WP04)
              |
              v
FTA / FMEDA / DFA and verification planning
              |
              +---- findings feed back into architecture,
                    failure analysis and requirements
```

The **next focused activity** is to analyse the ADC acquisition path in WP02, tracing selected functional failures through their local effects and conditional propagation to the MCU boundary. Findings may require changes to this decomposition, existing requirements or candidate safety mechanisms.

Failure rates, diagnostic coverage, SPFM/LFM values, reaction-time budgets and verification evidence remain TBD pending a justified technical basis.
