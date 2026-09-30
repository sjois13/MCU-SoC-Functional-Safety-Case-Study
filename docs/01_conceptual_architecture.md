# Conceptual MCU/SoC Architecture

v0.1, work in progress. Anything not defined here is TBD, and no numbers are used without a stated basis.

## Purpose

This document sets up the functional architecture of a conceptual safety-related MCU/SoC before any safety mechanism is chosen. Choosing mechanisms first would mean assuming the failures they are meant to cover, so the architecture and the failure reasoning come first, and requirements, mechanisms, FTA, FMEDA and dependent failure analysis follow from them.

## Boundary and assumptions

The MCU/SoC is the main controller of one humanoid Joint Control Unit (JCU).

```text
Robot-level controller -> JCU -> MCU/SoC -> external power stage -> motor -> joint
                                    ^                                          |
                                    +---------------- feedback ----------------+
```

Robot-level interpretation of the environment stays outside this boundary. The robot-level controller only provides the requested joint motion and its constraints.

**ASM-001, MCU allocation.** One MCU/SoC per JCU, as the primary controller.

**ASM-002, system-level safety need.** Avoid unintended or uncontrolled joint motion caused by an MCU/SoC failure. This gives the failure behaviours below something to trace to. No integrity level is assigned, because the robot-level hazard analysis is out of scope.

**ASM-003, feedback acquisition.** Feedback reaches the MCU through the ADC (analog signals) and the timer (digital position signals). The sensor types are TBD.

## What the MCU has to do

- receive motion requests and constraints;
- receive joint and motor feedback and derive the current joint state;
- calculate the local control action; and
- send a control command to the power stage.

Control-law design is out of scope.

## Architecture elements

| ID | Element | Role |
|---|---|---|
| ARCH-CPU-001 | CPU | Runs software and control computations |
| ARCH-ADC-001 | ADC | Digitises analog sensor signals |
| ARCH-SRAM-001 | SRAM | Working storage during execution |
| ARCH-NVM-001 | Program NVM | Non-volatile program storage (technology not needed at this level) |
| ARCH-INT-001 | Interconnect | Moves transactions between SoC elements |
| ARCH-COM-001 | Communication peripheral | Receives motion requests from the robot-level controller |
| ARCH-TMR-001 | Timer | Time-related control and measurement, digital feedback capture |
| ARCH-CLK-001 | Clock | Timing reference for the digital functions |
| ARCH-RST-001 | Reset | Brings elements into a defined state |
| ARCH-PWR-001 | Power supply | Supply for all elements |
| ARCH-OUT-001 | Output peripheral | Interface to the external power stage |

The communication peripheral is there because the motion requests have to enter the MCU somewhere. The power function is there because supply is a likely shared dependency, which matters later for the dependent failure analysis.

Watchdog, DMA, debug and security functions are left out of v0.1.

## Information flow

```text
Robot-level controller -> COM ----------------------+
                                                    v
External sensor -> ADC / TMR --------------------> INT <----> CPU <----> SRAM
                                                    |           ^
                                                    |           |
                                                    v          NVM
                                                   OUT -> power stage -> motor / joint
```

CLK, RST and PWR are not drawn. They affect execution, state and timing of everything above, and how they connect is still to be worked out.

## Failure behaviours at the MCU boundary

These are the effects seen from outside the MCU. The list is a starting point, not a complete or exclusive set.

| ID | Behaviour |
|---|---|
| FB-001 | Incorrect control output value |
| FB-002 | Control output at the wrong time |
| FB-003 | Expected control output missing or stale |
| FB-004 | Unintended or spurious control output |
| FB-005 | Incorrect MCU response to correct input or feedback |

## Failure candidates inside the SoC

No detailed failure-mode analysis has been done yet. What follows is the reasoning so far.

**Clock (FM-CLK-001).** Incorrect clock behaviour changes the timing of every function in the affected domain, so computation or output may happen at the wrong time (FB-002, possibly more). Failure modes and clock domains are TBD.

**CPU (FM-CPU-001).** Assuming the inputs arrive correctly, the CPU can still compute a wrong result. That becomes a wrong control command, and the output path passes it on. The physical causes are TBD.

**SRAM (FM-SRAM-001).** Corrupted stored data is an SRAM failure. Data that is correct in SRAM but arrives wrong at the receiver is not, because the fault is in the path between them. Treating both as "SRAM failure" would hide the second case and lead to the wrong mechanism, so they are kept apart.

**Interconnect (FM-INT-001).** The interconnect has the widest set of candidate failures, because everything passes through it: corrupted data, lost transactions, transactions that are too early or too late, delivery to the wrong or an extra destination, duplication, wrong order, and data taken from an unintended source. The list is preliminary and will not fit every interconnect implementation.

**Reset (FM-RST-001).** Reset may fail to bring the elements into the intended defined state. Failure modes and consequences are TBD.

## Traceability

| Failure candidate | Element | Boundary behaviour | Reasoning |
|---|---|---|---|
| FM-CLK-001 | ARCH-CLK-001 | FB-002 | Wrong clock mainly changes when things happen |
| FM-CPU-001 | ARCH-CPU-001 | FB-001, FB-004 | Wrong computation gives a wrong value, or a command nobody intended |
| FM-SRAM-001 | ARCH-SRAM-001 | FB-001, FB-005 | Corrupted working data gives a wrong value, or a wrong reaction to correct input |
| FM-INT-001 | ARCH-INT-001 | FB-001, FB-002, FB-003, FB-005 | Corruption gives wrong values, late delivery gives timing errors, lost transactions give missing output, wrong source or destination gives wrong reactions |
| FM-RST-001 | ARCH-RST-001 | TBD | Not analysed yet |

This is a working mapping and will change. COM, TMR, ADC, NVM, PWR and OUT have no failure candidates yet.

## Safety mechanisms

None are allocated in v0.1. What the analysis suggests so far is that anything generated, stored, processed or transferred incorrectly inside the SoC may need to be detected before it reaches a safety-relevant output.

Redundancy is one possible mechanism, but duplicating an element does not make it independent. If the copies share clocking, power or the interconnect, a single fault can affect both, and that has to be handled in the dependent failure analysis. No diagnostic coverage is assigned.

## Next

- formal functional decomposition at MCU and IP level;
- IP-level failure modes and their propagation to the boundary behaviours;
- semiconductor safety requirements derived from that;
- safe state and fault reaction, including timing;
- assumptions on the JCU / system integrator;
- mechanism allocation, then FTA, FMEDA and dependent failure analysis; and
- verification cases.

Failure rates, diagnostic coverage, SPFM/LFM and verification evidence do not exist yet.
