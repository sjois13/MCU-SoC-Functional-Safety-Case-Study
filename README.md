# Functional Safety MCU/SoC Case Study

**Status:** v0.1 — Work in Progress

## Overview

This repository is an independent engineering case study exploring Functional Safety at MCU/SoC and IP level.

A conceptual MCU/SoC for a humanoid Joint Control Unit (JCU) is used as the reference application. The focus is not the robot safety concept itself, but the engineering inside the semiconductor boundary: MCU/SoC architecture, IP functions, failure modes and propagation, safety requirements, safety mechanisms, dependent failures, and verification.

The project is developed incrementally so that safety requirements and safety mechanisms are derived from the architecture and failure analysis rather than selected in advance.

## Engineering Approach

The case study follows the engineering chain:

```text
Application / MCU boundary
        ↓
MCU and IP functions
        ↓
Conceptual MCU/SoC architecture
        ↓
Functional failures and failure propagation
        ↓
Semiconductor safety requirements
        ↓
Safety mechanisms and fault reactions
        ↓
FTA / FMEDA
        ↓
Dependent Failure Analysis
        ↓
Verification and traceability
```

## Work Products

The case study is developed through the following engineering work products:

1. [Conceptual MCU/SoC Architecture](01-architecture/conceptual_architecture.md)
2. [Functional Failure Analysis](02-failure-analysis/functional_failure_analysis.md)
3. [Semiconductor Safety Requirements](03-safety-requirements/semiconductor_safety_requirements.md)
4. [Safety Architecture and Mechanism Allocation](04-safety-architecture/safety_mechanism_allocation.md)
5. [Qualitative Fault Tree Analysis](05-fta/fault_tree_analysis.md)
6. [FMEDA Framework](06-fmeda/fmeda_framework.md)
7. [Dependent Failure Analysis](07-dfa/dependent_failure_analysis.md)
   
## Current Development

The case study is developed iteratively. The current v0.1 establishes an initial end-to-end safety analysis using selected MCU/SoC functions; it is not yet a complete analysis of every IP.

This reflects the engineering process: later analysis can expose gaps in earlier work, and findings from safety mechanisms, DFA or verification may require the architecture, failure analysis or requirements to be refined.

After the initial baseline is complete, the analysis will be expanded across the remaining IPs and deepened at semiconductor level.

## Project Basis and Limitations

This is an independent research and learning project. It does not represent work performed for an employer, customer, semiconductor manufacturer, or commercial product.

The MCU/SoC architecture is conceptual and is based on general semiconductor engineering concepts and public technical information. It is not intended to represent the internal architecture of any specific commercial MCU/SoC or semiconductor product.

AI tools are used as learning and review aids for concept explanation, challenging engineering reasoning, and documentation support.

Unknown information is identified as **TBD** or explicitly documented as a project assumption.

No semiconductor failure rates, diagnostic coverage values, SPFM/LFM results, hardware capabilities, compliance, or certification claims are made without an explicit engineering basis.
