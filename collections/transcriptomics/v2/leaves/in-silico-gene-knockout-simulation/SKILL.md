---
name: in-silico-gene-knockout-simulation
description: Use when you have mock and infected context-specific FBA models with
  a viral biomass objective function and need to rank host genes/gene pairs as antiviral
  targets by essentiality-vs-safety trade-off.
when_to_use_negative:
- You need direct antiviral compound docking on the viral proteome — here the virus
  is only modeled through its biomass function, so targets arise from network effects
  on the host metabolome
- You lack a functional FBA model (no consistent genome-scale reconstruction or no
  biomass objective) — knockout simulation requires a solvable COBRA model
- You need drug potency data (IC-50, response curves, synergy scores) — in silico
  knockouts do not produce these, and the authors note drug-response experiments were
  not performed
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: COBRA Toolbox V3
  role: Runs the in silico single- and double-gene deletion simulations (singleDeletion)
    on the context-specific models in MATLAB
  repo: https://github.com/opencobra/cobratoolbox
- name: rFASTCORMICS
  role: Reconstructs the mock and infected context-specific genome-scale metabolic
    models (22 mock, 28 infected) that are the substrate for the knockouts
  repo: https://github.com/sysbiolux/rFASTCORMICS
- name: MATLAB
  role: Execution environment for the COBRA Toolbox deletion analyses and aggregation
    of essentiality/safety scores
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:c53cad94f9d700703a4507046e845abb927ab840325c6a5b145e8fe8aabf10b3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sysbiolux/DCcov
  repo_url: https://github.com/opencobra/cobratoolbox
  tools:
  - COBRA Toolbox
  - COBRA Toolbox V3
  - MATLAB
  - rFASTCORMICS
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# In silico gene knockout simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Systematic single- and double-gene deletion simulations on context-specific genome-scale metabolic models (built with rFASTCORMICS from Recon2.04/Recon3D_01 plus a viral biomass objective function) using the COBRA Toolbox V3 in MATLAB, to identify host genes whose knockout reduces SARS-CoV-2 viral biomass without affecting host biomass maintenance, and to score them for essentiality (infected models) and safety (mock models).

## When to use

Use when you have mock and infected context-specific FBA models with a viral biomass objective function and need to rank host genes/gene pairs as antiviral targets by essentiality-vs-safety trade-off.

## When NOT to use

- You need direct antiviral compound docking on the viral proteome — here the virus is only modeled through its biomass function, so targets arise from network effects on the host metabolome
- You lack a functional FBA model (no consistent genome-scale reconstruction or no biomass objective) — knockout simulation requires a solvable COBRA model
- You need drug potency data (IC-50, response curves, synergy scores) — in silico knockouts do not produce these, and the authors note drug-response experiments were not performed

## Inputs

- 22 mock (healthy) and 28 infected context-specific genome-scale metabolic models in COBRA format, built with rFASTCORMICS from Recon3D_01/Recon2.04
- Viral biomass objective function (VBOF) added to the infected models
- Host (human) biomass reaction definitions for biomass-maintenance checks
- Predicted gene pairs for double-gene deletion screening

## Outputs

- Per-gene essentiality scores (count of infected models where the knockout reduces viral biomass without affecting host biomass)
- Per-gene safety scores (count of mock models where the knockout is safe)
- Per-gene-pair essentiality and safety scores from double-gene deletions
- Essentiality-vs-safety scatter plot highlighting candidate targets (e.g. GUK1, CRLS1/SGMS1/SLC27A4 pairs), e.g. figure3_essentiality_safety_scatter.png

## How to apply

Load the 22 mock and 28 infected context-specific genome-scale metabolic models (rFASTCORMICS-built from Recon3D_01/Recon2.04 with the viral biomass objective function) into MATLAB with the COBRA Toolbox V3. Run single-gene deletion simulations (COBRA Toolbox singleDeletion), flagging a knockout as essential when viral biomass production is reduced while host biomass maintenance is unaffected; repeat as double-gene deletions for predicted gene pairs. Aggregate per gene/pair into an essentiality score (number of infected models in which the knockout is essential) and a safety score (number of mock/healthy models in which the knockout is safe). Rank candidates by combining both scores — e.g. GUK1 scored moderately for essentiality with a high safety score, whereas CRLS1 and SGMS1 scored highest for essentiality but lowest for safety (high toxicity) — and plot an essentiality-vs-safety scatter for candidate selection.

## Related tools

- **COBRA Toolbox V3** (Runs the in silico single- and double-gene deletion simulations (singleDeletion) on the context-specific models in MATLAB) — https://github.com/opencobra/cobratoolbox
- **rFASTCORMICS** (Reconstructs the mock and infected context-specific genome-scale metabolic models (22 mock, 28 infected) that are the substrate for the knockouts) — https://github.com/sysbiolux/rFASTCORMICS
- **MATLAB** (Execution environment for the COBRA Toolbox deletion analyses and aggregation of essentiality/safety scores)

## Examples

```
[model, A_final] = fastcormics_RNAseq(model, data, rownames, dico, biomass_rxn, already_mapped_tag, consensus_proportion, epsilon, optional_settings); % then COBRA singleDeletion on mock & infected models
```

## Evaluation signals

- A knockout is only flagged essential if viral biomass production is reduced while host biomass maintenance is unaffected in that model
- Aggregation yields one essentiality score and one safety score per gene/pair (counts over 28 infected and 22 mock models, respectively)
- Reproduced scatter must place GUK1 (single gene) and the CRLS1/SGMS1/SLC27A4 gene pairs in the essential-but-safe region, matching the qualitative pattern of Figure 3
- CRLS1 and SGMS1 should rank highest for essentiality but among the lowest for safety, indicating high toxicity
- Model set must contain 50 models (28 infected, 22 mock) as reconstructed, before running deletions

## Limitations

- Essentiality must be confirmed experimentally: the authors state 'further in vitro single- and double-gene deletion studies are needed to determine the essentiality of the identified single genes and gene pairs'
- The virus is only represented through its biomass objective function, so targets reflect host-metabolome network effects rather than direct viral interactions
- The study is limited by the modeled lung cell lines (A549, Calu-3, H1299, NHBE); further tests on other tissues and with other optimization functions, plus in vitro/in vivo validation, are required before treating predictions as drug candidates

## Evidence

- [results] Essentiality/safety scoring of knockouts: [evidence span withheld — anchor status: not_found]
