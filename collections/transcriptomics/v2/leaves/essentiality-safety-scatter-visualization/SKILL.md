---
name: essentiality-safety-scatter-visualization
description: Use when you have single-gene deletion results from both infected and
  mock context-specific models and must rank knockouts on an essentiality-vs-safety
  plane to select low-toxicity antimetabolic drug targets.
when_to_use_negative:
- When only infected models exist without matching mock/healthy controls — safety
  scores cannot be computed and the scatter's essential-but-safe region is undefined
- When the goal is direct docking or viral-proteome inhibition — the targets here
  derive from network effects on the host metabolome, as the virus is only modeled
  through its biomass function
- When input is differential expression data alone — deletion simulations require
  context-specific FBA models, not expression matrices
edam_operation: http://edamontology.org/operation_3660
tools:
- name: MATLAB
  role: Environment in which the single-gene deletion simulations, score aggregation,
    and Figure 3 scatter are executed
- name: COBRA Toolbox V3
  role: Provides the singleDeletion function used to run in silico single-gene knockouts
    on the mock and infected models
  repo: https://github.com/opencobra/cobratoolbox/tree/master/src
- name: rFASTCORMICS
  role: Upstream reconstruction of the mock and infected context-specific models (Recon2.04/Recon3D_01
    + VBOF) that the deletion analysis and scoring are applied to
  repo: https://github.com/sysbiolux/rFASTCORMICS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:2a9fb4f4f186a4119155962cff03a8ada2bdfa4a1bcac4e9004b36fe48af940f
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
  edam_operation: http://edamontology.org/operation_3660
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# essentiality-safety-scatter-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates in silico single- and double-gene deletion results across mock and SARS-CoV-2-infected context-specific metabolic models into per-gene essentiality and safety scores, and plots them as the Figure 3 essentiality-vs-safety scatter to identify essential-but-safe antiviral drug targets (e.g., GUK1).

## When to use

Use when you have single-gene deletion results from both infected and mock context-specific models and must rank knockouts on an essentiality-vs-safety plane to select low-toxicity antimetabolic drug targets.

## When NOT to use

- When only infected models exist without matching mock/healthy controls — safety scores cannot be computed and the scatter's essential-but-safe region is undefined
- When the goal is direct docking or viral-proteome inhibition — the targets here derive from network effects on the host metabolome, as the virus is only modeled through its biomass function
- When input is differential expression data alone — deletion simulations require context-specific FBA models, not expression matrices

## Inputs

- 22 mock and 28 infected context-specific genome-scale metabolic models (rFASTCORMICS-built from Recon3D_01/Recon2.04 with VBOF, COBRA format)
- List of predicted essential genes and gene pairs (from in silico knockout screening)
- Mock vs. infected sample labels for the expression datasets (e.g., GSE147507, GSE148729)

## Outputs

- Per-gene essentiality score (count of infected models where knockout reduces viral biomass without affecting host biomass)
- Per-gene/per-gene-pair safety score (count of mock models where knockout is safe)
- figure3_essentiality_safety_scatter.png highlighting GUK1 and the CRLS1/SGMS1/SLC27A4 gene pairs

## How to apply

Load the 22 mock and 28 infected context-specific genome-scale metabolic models (built with rFASTCORMICS from Recon3D_01 or Recon2.04 with the viral biomass objective function, VBOF) into MATLAB with COBRA Toolbox V3. Run in silico single-gene deletions (COBRA Toolbox singleDeletion) and, for predicted pairs, double-gene deletions, flagging a knockout as essential when it reduces viral biomass production without affecting host biomass maintenance. Aggregate per gene and per gene pair: the essentiality score counts infected models where the knockout is essential, and the safety score counts mock/healthy models where the knockout is safe. Plot the essentiality-vs-safety scatter corresponding to Figure 3, explicitly highlighting GUK1 (single gene) and the CRLS1/SGMS1/SLC27A4 gene pairs, and save as figure3_essentiality_safety_scatter.png. Genes without pathways in Recon3D_01 subSystems are excluded from the downstream tripartite gene-drug-pathway network.

## Related tools

- **MATLAB** (Environment in which the single-gene deletion simulations, score aggregation, and Figure 3 scatter are executed)
- **COBRA Toolbox V3** (Provides the singleDeletion function used to run in silico single-gene knockouts on the mock and infected models) — https://github.com/opencobra/cobratoolbox/tree/master/src
- **rFASTCORMICS** (Upstream reconstruction of the mock and infected context-specific models (Recon2.04/Recon3D_01 + VBOF) that the deletion analysis and scoring are applied to) — https://github.com/sysbiolux/rFASTCORMICS

## Examples

```
[grouppedRes,~,gDel] = singleDeletion(model); % run per mock/infected model; tally essential (VBOF down, host biomass maintained) vs. safe counts
```

## Evaluation signals

- The reproduced scatter places GUK1 and the CRLS1/SGMS1/SLC27A4 knockouts in the essential-but-safe region: nonzero essentiality across infected models with unaffected human biomass maintenance in mock models
- Model count check: exactly 50 models are loaded (28 infected, 22 mock), consistent with 'resulted in 50 models (28 infected and 22 mock)'
- Essentiality scores must fall in the range 0–28 (infected models) and safety scores in 0–22 (mock models)
- CRLS1 and SGMS1 should show top essentiality but bottom-tier safety, whereas GUK1 shows moderate essentiality with a higher safety score
- Genes (and their connected drugs) lacking pathways in Recon3D_01 subSystems must be absent from the plotted candidate set

## Limitations

- Study is limited by the modeled lung cell lines (A549, Calu-3, H1299, NHBE); other tissues may yield different essentiality/safety profiles
- Further in vitro single- and double-gene deletion studies are needed to confirm the in silico essentiality calls
- The virus is modeled only through its biomass function, so targets reflect host metabolome network effects rather than direct antiviral action; toxicity of predicted drugs requires careful follow-up in vitro and in vivo assessment

## Evidence
