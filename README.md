# Functional Safety MCU/SoC Case Study

> An independent engineering case study exploring how Functional Safety analysis can be applied at MCU/SoC and IP level.

**Status: v0.1 — Work in Progress**

## Overview

Functional Safety analysis at system or ECU level eventually depends on the behaviour of the semiconductor devices implementing the safety-related functions.

This project explores that next level of abstraction.

The case study starts with a conceptual MCU/SoC used for a safety-related motor-control function and progressively examines what happens inside the semiconductor: how functions are allocated to IP blocks, how those blocks can fail, how failures can propagate through the SoC, and how safety requirements and safety mechanisms can be derived from that analysis.

The intention is not to design a production MCU. The project is an engineering exercise in building and documenting the reasoning chain from a system-level safety need down into semiconductor-level Functional Safety.

## Case Study

A humanoid joint controller is used as the reference application.

The application provides a concrete boundary:

**robot-level control → joint control unit → MCU/SoC → power stage → motor/joint**

The detailed robot safety concept is outside the scope of this repository. The focus begins at the boundary of the MCU/SoC and moves inward.

The conceptual MCU/SoC is developed from basic functional needs such as acquiring sensor information, executing control calculations, storing data, maintaining timing and generating control outputs.

From there, the analysis asks questions such as:

- What functions must the MCU/SoC perform correctly?
- How can an individual IP block fail?
- Can correct information become corrupted while moving between IP blocks?
- How can an internal failure propagate to an externally visible MCU failure?
- Which failures require safety mechanisms?
- Can supposedly independent safety mechanisms share a common dependency?
- How can the resulting safety requirements and mechanisms be verified?

## Engineering Method

The project is developed incrementally rather than starting with a predefined list of safety mechanisms.

```text id="eqg1if"
Application and MCU boundary
            ↓
Functional decomposition
            ↓
Conceptual IP/SoC architecture
            ↓
Functional failures and failure propagation
            ↓
Semiconductor safety requirements
            ↓
Safety mechanisms
            ↓
FTA / FMEDA
            ↓
Dependent Failure Analysis
            ↓
Fault reactions
            ↓
Verification
            ↓
Traceability
```

For example, the analysis distinguishes between a memory element storing incorrect information and correct information becoming corrupted while travelling through the SoC interconnect. These may lead to similar system-level effects but originate from different parts of the semiconductor architecture and therefore require different failure analysis.

Safety mechanisms such as redundancy, monitoring or data-integrity protection are not assumed at the start. They are introduced only when supported by the preceding failure analysis.

## Work Products

### v0.1 — Architecture and Initial Failure Reasoning

[`docs/01_conceptual_architecture.md`](docs/01_conceptual_architecture.md)

The first work product establishes:

- the system-to-semiconductor boundary;
- the initial conceptual MCU/SoC architecture;
- the purpose of the initial IP blocks;
- initial MCU and IP-level failure behaviours; and
- initial failure-propagation reasoning, including interconnect failures.

Later work products will be added as the analysis progresses. Empty FTA, FMEDA, DFA or verification documents are intentionally not created in advance.

## Project Status

The project is currently at the **architecture and qualitative failure-analysis stage**.

The next engineering activities are to formalize the MCU/IP functional decomposition and trace selected IP-level failures through the SoC to MCU-boundary effects. These results will provide the basis for deriving semiconductor safety requirements.

Quantitative FMEDA, diagnostic coverage, SPFM/LFM, detailed dependent failure analysis and verification evidence have **not yet been developed**.

## Project Basis

This is an independent research and learning project.

The MCU/SoC architecture is conceptual and is based on general semiconductor architecture concepts and public technical information. It does not represent the internal architecture of a specific commercial semiconductor product and does not represent semiconductor architecture from previous professional work.

AI tools are used as learning and review aids to explain concepts, challenge engineering reasoning and support documentation. Engineering decisions, assumptions and analyses are developed explicitly as part of the case study.

Where external technical sources are used, they will be identified. Unknown information is marked **TBD** or documented as a project assumption.

No semiconductor failure rates, diagnostic coverage values, SPFM/LFM results or hardware capabilities are invented for the case study.
