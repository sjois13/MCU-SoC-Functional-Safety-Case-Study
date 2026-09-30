# Functional Safety MCU/SoC Case Study

An independent case study on how Functional Safety analysis changes when you move from the ECU down into the MCU/SoC itself.

**Status: v0.1, work in progress**

## Why this project

Safety analysis at system or ECU level eventually depends on how the semiconductor behaves. I work with that boundary from the integration side, and I wanted to understand the other side of it: how functions are allocated to IP blocks, how those blocks can fail, how failures spread through the SoC, and how safety requirements and mechanisms follow from that.

This is not a production MCU design. It is an exercise in documenting the reasoning chain from a system-level safety need down to semiconductor-level Functional Safety.

## Reference application

A humanoid joint controller:

**robot-level control → joint control unit → MCU/SoC → power stage → motor/joint**

The robot-level safety concept is out of scope. The analysis starts at the MCU/SoC boundary and moves inward, from basic needs such as sensing, control calculation, data storage, timing and output generation.

## Method

I develop the analysis step by step. Safety mechanisms are not assumed at the start; they are added only when the failure analysis justifies them.

```text
Application and MCU boundary
  → Functional decomposition
  → Conceptual IP/SoC architecture
  → Functional failures and failure propagation
  → Semiconductor safety requirements
  → Safety mechanisms
  → FTA / FMEDA
  → Dependent failure analysis
  → Fault reactions
  → Verification and traceability
```

One example of the reasoning: a memory element storing wrong data and correct data being corrupted in the interconnect can look identical at system level, but they start in different parts of the architecture and need different analysis.

## Work products

**v0.1: architecture and initial failure reasoning**
[`docs/01_conceptual_architecture.md`](docs/01_conceptual_architecture.md)

- system-to-semiconductor boundary
- initial conceptual MCU/SoC architecture and IP blocks
- initial MCU and IP-level failure behaviours
- first failure-propagation reasoning, including interconnect failures

Later documents will be added as the analysis progresses. I do not create empty FTA, FMEDA, DFA or verification files in advance.

## Status

The project is at the architecture and qualitative failure-analysis stage. Next: formalise the functional decomposition and trace selected IP-level failures to MCU-boundary effects, which will be the basis for deriving safety requirements.

Quantitative FMEDA, diagnostic coverage, SPFM/LFM, detailed dependent failure analysis and verification evidence are **not yet developed**.

## Project basis

This is an independent learning project. The architecture is conceptual, built from general semiconductor concepts and public information. It does not represent a specific commercial product or any architecture from my professional work.

I use AI tools as learning and review aids to explain concepts and challenge my reasoning. Decisions, assumptions and analyses are my own. Unknown information is marked **TBD** or listed as an assumption, and no failure rates, diagnostic coverage values or metrics are invented.
