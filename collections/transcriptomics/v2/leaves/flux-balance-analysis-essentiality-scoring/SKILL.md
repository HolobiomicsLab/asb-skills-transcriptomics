---
name: flux-balance-analysis-essentiality-scoring
description: Use when context-specific metabolic models of infected and healthy cells
  (e.g., rFASTCORMICS models with a VBOF) are available and you need to rank which
  gene knockouts reduce viral biomass without impairing host biomass.
when_to_use_negative:
- When the virus cannot be represented by a biomass objective function (e.g., direct
  viral-proteome docking is required — the article notes targets come from network
  effects on the host metabolome, not the virus itself)
- When models are generic genome-scale reconstructions without context-specific pruning
  or a VBOF — essentiality calls will not reflect infected-lung conditions
- When the question is drug-toxicity assessment rather than target essentiality/safety
  ranking — toxicity here is only inferred from knockout safety scores, not measured
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3172
tools:
- name: COBRA Toolbox V3
  role: Provides FBA and the singleGeneDeletion/doubleGeneDeletion routines used to
    compute grRatio and essentiality calls on each model
  repo: github.com/opencobra/cobratoolbox
- name: rFASTCORMICS
  role: Builds the infected and mock context-specific models from RNA-seq expression
    data that are the input to the deletion simulations
  repo: https://github.com/sysbiolux/rFASTCORMICS
- name: MATLAB
  role: Runtime environment in which COBRA Toolbox V3 deletion analyses and scoring
    aggregation are executed
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:92d211c4f2495d36e711acd41439f7ee506f2fceee072f00a6cc3593a261dd1e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sysbiolux/DCcov
  tools:
  - COBRA Toolbox
  - COBRA Toolbox V3
  - MATLAB
  - rFASTCORMICS
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# Flux Balance Analysis Essentiality Scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores gene knockouts by running FBA single- and double-gene deletions on context-specific infected (viral biomass objective) and mock (host biomass) metabolic models, aggregating results into an essentiality score (infected models) and a safety score (mock models) to rank toxicity-favorable antiviral targets.

## When to use

Use when context-specific metabolic models of infected and healthy cells (e.g., rFASTCORMICS models with a VBOF) are available and you need to rank which gene knockouts reduce viral biomass without impairing host biomass.

## When NOT to use

- When the virus cannot be represented by a biomass objective function (e.g., direct viral-proteome docking is required — the article notes targets come from network effects on the host metabolome, not the virus itself)
- When models are generic genome-scale reconstructions without context-specific pruning or a VBOF — essentiality calls will not reflect infected-lung conditions
- When the question is drug-toxicity assessment rather than target essentiality/safety ranking — toxicity here is only inferred from knockout safety scores, not measured

## Inputs

- Infected context-specific genome-scale metabolic models with viral biomass objective function (VBOF), built from Recon2.04/Recon3D_01 via rFASTCORMICS
- Mock (healthy) control context-specific metabolic models from the same workflow
- Expression datasets (e.g., GEO: GSE147507 severity study, GSE148729 time-series study) used to build the models

## Outputs

- Per-gene and per-gene-pair essentiality scores (count of infected models where knockout is essential, grRatio < 0.2)
- Per-gene safety scores (count of mock models where human biomass maintenance is unaffected)
- List of unique essential genes (expected: 23 genes written to essential_genes.csv)
- Essentiality-vs-safety scatter plot (figure3_essentiality_safety_scatter.png)

## How to apply

Load the 28 infected and 22 mock context-specific models (rFASTCORMICS-built from Recon2.04/Recon3D_01) into MATLAB with COBRA Toolbox V3. For each infected model, run FBA on the viral biomass objective function, then fix the host biomass reaction upper bound to 10% of its maximum so host growth cannot mask viral-biomass changes. Execute the COBRA Toolbox singleGeneDeletion routine (and doubleGeneDeletion for gene pairs) optimizing the viral biomass objective; classify a knockout as essential when grRatio (post-knockout viral biomass divided by pre-knockout maximum) is < 0.2. Aggregate per gene: essentiality score = count of infected models where the knockout reduces viral biomass without affecting host biomass maintenance; safety score = count of mock/healthy models where the knockout leaves human biomass unaffected. This yielded 23 unique essential genes (CRLS1, GUK1, SGMS1, DTYMK, CMPK1, TYMS, ...), with GUK1 emerging as moderately essential but safer, while CRLS1/SGMS1 were highly essential yet toxic.

## Related tools

- **COBRA Toolbox V3** (Provides FBA and the singleGeneDeletion/doubleGeneDeletion routines used to compute grRatio and essentiality calls on each model) — github.com/opencobra/cobratoolbox
- **rFASTCORMICS** (Builds the infected and mock context-specific models from RNA-seq expression data that are the input to the deletion simulations) — https://github.com/sysbiolux/rFASTCORMICS
- **MATLAB** (Runtime environment in which COBRA Toolbox V3 deletion analyses and scoring aggregation are executed)

## Examples

```
grRatio = singleGeneDeletion(infectedModel, 'viral_biomass'); essential = grRatio.grRatio < 0.2;
```

## Evaluation signals

- single-gene deletion output contains exactly 23 unique essential genes and reproduces the reported set including CRLS1, SGMS1, GUK1, DTYMK, CMPK1, and TYMS
- Every knockout classified as essential has grRatio < 0.2 relative to the pre-knockout maximum viral biomass flux
- Host biomass reaction upper bound was fixed to 10% of its maximal FBA flux before running deletions on infected models
- Essentiality-vs-safety scatter places GUK1 (single gene) and the CRLS1/SGMS1/SLC27A4 pairs in the essential-but-safe region; CRLS1 and SGMS1 score highest for essentiality but lowest for safety
- Model set matches 50 reconstructions: 28 infected and 22 mock context-specific models

## Limitations

- Essentiality predictions are in silico only; the article states further in vitro single- and double-gene deletion studies are needed to confirm the identified genes and gene pairs
- Limited by the modeled lung cell lines (A549, Calu-3, H1299, NHBE); results may not transfer to other tissues or optimization functions without in vitro/in vivo validation
- The virus is modeled only through its biomass function, so drug targets reflect host-metabolome network effects rather than direct antiviral activity, and predicted-drug toxicity requires careful follow-up assessment

## Evidence

- [readme] rFASTCORMICS README defining the model-construction method that generates the deletion-analysis inputs: [evidence span withheld — anchor status: external_doc]
