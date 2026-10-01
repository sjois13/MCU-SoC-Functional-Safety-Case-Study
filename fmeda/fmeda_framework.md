# FMEDA Framework

**Version:** v0.1 — Work in Progress

## 1. Objective

This work product establishes a preliminary FMEDA structure for the conceptual MCU/SoC.

The purpose is to connect hardware functions, functional failure modes and allocated safety mechanisms to the quantitative information that would be required for a semiconductor FMEDA.

No failure rates, diagnostic coverage, SPFM or LFM values are calculated in v0.1 because justified semiconductor failure data are not available for this conceptual architecture.

## 2. Preliminary FMEDA Structure

| Element | Functional Failure | Potential Effect | Safety Mechanism | Failure Rate | Diagnostic Coverage | Residual / Latent Contribution |
|---|---|---|---|---|---|---|
| ADC | FF-ADC-001 — Incorrect conversion | Incorrect feedback may propagate into control computation | Plausibility / redundant or diverse acquisition | TBD | TBD | TBD |
| CPU | FF-CPU-001 — Incorrect computation | Incorrect control action | Redundant computation / lockstep-style monitoring | TBD | TBD | TBD |
| SRAM | FF-SRAM-001 — Incorrect stored working data | Corrupted data may influence computation | ECC / parity | TBD | TBD | TBD |
| Interconnect | FF-INT-001.1 — Data corruption | Incorrect information received by destination | End-to-end / interconnect integrity protection | TBD | TBD | TBD |
| Clock | FF-CLK-001 — Incorrect clock behaviour | Incorrect temporal behaviour of dependent functions | Clock supervision | TBD | TBD | TBD |

The table is intentionally incomplete. Additional hardware elements and failure modes will be added as the architecture and failure analysis mature.

## 3. Quantitative Data Gaps

The current analysis does not contain the semiconductor data needed for quantitative FMEDA.

For each hardware element, the following remain TBD:

- base failure rate and its technical source;
- allocation of that failure rate to relevant hardware failure modes;
- safety classification of those failure modes;
- coverage provided by the allocated safety mechanism; and
- remaining residual or latent failure contribution.

Mission-profile and technology assumptions are also not yet defined.

Until these inputs have a justified basis, failure rates, diagnostic coverage, SPFM, LFM and PMHF will not be calculated.

## 4. Failure Classification Concept

For a safety-related hardware element, the analysis would determine how its failure contribution is classified after considering the implemented safety mechanisms.

Conceptually:

```text
Hardware failure
       ↓
Does it violate a safety requirement?
       │
   ┌───┴───┐
   │       │
  No      Yes
           ↓
     Is it detected /
     controlled by the
     safety mechanism?
           │
      ┌────┴────┐
      ↓         ↓
   covered    not covered
      ↓         ↓
 diagnostic   residual /
 contribution other safety-
              relevant contribution
```

## 5. Diagnostic Coverage

Diagnostic coverage cannot be inferred simply from the presence of a safety mechanism.

For example:

```text
SRAM
  ↓
ECC
```

does not justify assigning a diagnostic coverage value without knowing:

- the implemented code and its fault-detection/correction capability;
- the relevant SRAM failure modes;
- which failures are inside the mechanism's coverage;
- fault handling behaviour; and
- supporting implementation evidence.

The same principle applies to CPU monitoring, ADC monitoring, interconnect protection and clock supervision.

## 6. Open Items

- semiconductor failure-rate source;
- failure-mode distributions;
- complete hardware-element decomposition;
- diagnostic coverage justification;
- residual failure classification;
- latent multiple-point failure classification;
- mission/profile assumptions;
- SPFM calculation;
- LFM calculation; and
- PMHF evaluation where applicable.

Once these inputs are available, this work product can be a quantitave FMEDA framework rather than the existing qualitative FMEDA.
