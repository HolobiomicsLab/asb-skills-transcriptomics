---
name: in-silico-single-gene-deletion-analysis
description: Use when you have context-specific genome-scale metabolic models (e.g.
  built with rFASTCORMICS on Recon2.04/Recon3D_01) with a VBOF and need to find genes
  essential to viral biomass but not host growth.
when_to_use_negative:
- You need synergistic gene-pair effects (essential + non-essential combinations)
  — use double gene deletion (doubleGeneDeletion) instead, as single deletion alone
  cannot capture pair synergy
- The model is a generic reconstruction without the VBOF or without a host biomass
  reaction to constrain — knockout effects would not be attributable to viral replication
- The question is drug repositioning or pathway enrichment rather than essentiality
  calling — those downstream steps require the essential gene list as input but use
  different tools (DrugBank mapping, KEGG/Enrichr)
edam_operation: http://edamontology.org/operation_3663
edam_topics:
- http://edamontology.org/topic_2259
tools:
- name: COBRA Toolbox V3
  role: Provides the singleGeneDeletion routine and FBA solvers used to score each
    knockout by viral biomass flux
  repo: https://github.com/opencobra/cobratoolbox
- name: rFASTCORMICS
  role: Reconstructs the mock and infected context-specific lung models that serve
    as the deletion-analysis input
  repo: https://github.com/sysbiolux/rFASTCORMICS
- name: MATLAB
  role: Runtime environment in which the models are loaded and singleGeneDeletion
    is executed
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:ccfead72a5459d9ce033ee0eaacbb46011bc975bfa0155d9a75a27f64c305480
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
  edam_operation: http://edamontology.org/operation_3663
  edam_topics:
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# in-silico-single-gene-deletion-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Systematic single-gene deletion via flux balance analysis on context-specific infected-lung metabolic models carrying a viral biomass objective function (VBOF), to identify host genes whose knockout abolishes viral biomass production while preserving host biomass maintenance.

## When to use

Use when you have context-specific genome-scale metabolic models (e.g. built with rFASTCORMICS on Recon2.04/Recon3D_01) with a VBOF and need to find genes essential to viral biomass but not host growth.

## When NOT to use

- You need synergistic gene-pair effects (essential + non-essential combinations) — use double gene deletion (doubleGeneDeletion) instead, as single deletion alone cannot capture pair synergy
- The model is a generic reconstruction without the VBOF or without a host biomass reaction to constrain — knockout effects would not be attributable to viral replication
- The question is drug repositioning or pathway enrichment rather than essentiality calling — those downstream steps require the essential gene list as input but use different tools (DrugBank mapping, KEGG/Enrichr)

## Inputs

- Infected context-specific metabolic models in COBRA format carrying the VBOF (built from GSE147507 severity-study and GSE148729 time-series expression data with rFASTCORMICS)
- Genome-scale reconstructions (Recon2.04, Recon3D_01) with the viral biomass objective function
- Host biomass reaction definition and VBOF reaction identifiers in the MATLAB model structure

## Outputs

- essential_genes.csv — deduplicated list of 23 unique essential genes (e.g. CRLS1, SGMS1, GUK1, DTYMK, CMPK1, TYMS) with shared essentiality across conditions
- Per-gene grRatio scores (post-knockout viral biomass flux relative to pre-knockout maximum) for every gene in each infected model

## How to apply

Load the infected context-specific models (reconstructed with rFASTCORMICS, VBOF added) in MATLAB with the COBRA Toolbox V3. Run FBA to obtain the maximal host biomass flux, then cap the host biomass reaction upper bound at 10% of that maximum so host growth cannot mask changes in viral biomass. Execute the COBRA Toolbox singleGeneDeletion routine over all genes, optimizing the viral biomass objective for each knockout. Classify a gene as essential when grRatio (post-knockout viral biomass flux / pre-knockout maximum) falls below 0.2; aggregate calls across models, deduplicate, and export the unique essential gene list (23 genes in the DCcov study) to essential_genes.csv.

## Related tools

- **COBRA Toolbox V3** (Provides the singleGeneDeletion routine and FBA solvers used to score each knockout by viral biomass flux) — https://github.com/opencobra/cobratoolbox
- **rFASTCORMICS** (Reconstructs the mock and infected context-specific lung models that serve as the deletion-analysis input) — https://github.com/sysbiolux/rFASTCORMICS
- **MATLAB** (Runtime environment in which the models are loaded and singleGeneDeletion is executed)

## Examples

```
[grRatio, grRateKO, geneKO] = singleGeneDeletion(infectedModel); essentialGenes = geneKO(grRatio < 0.2); writetable(array2table(essentialGenes), 'essential_genes.csv');
```

## Evaluation signals

- The final essential_genes.csv contains exactly 23 unique genes and reproduces the reported set including CRLS1, SGMS1, GUK1, DTYMK, CMPK1, and TYMS
- Each called essential gene has grRatio < 0.2 (post-knockout viral biomass below 20% of the pre-knockout maximum)
- Essentiality calls are consistent across the severity (GSE147507) and time-series (GSE148729) studies for shared genes such as CRLS1, GUK1, and SGMS1
- Host biomass flux remains above the 10% cap for each knockout, confirming the deletion affects viral rather than host biomass maintenance

## Limitations

- Essentiality predictions are in silico only; the paper states 'further in vitro single- and double-gene deletion studies are needed to determine the essentiality of the identified single genes and gene pairs'
- Results are limited by the modeled lung cell lines (A549, Calu-3, H1299, NHBE) and may not transfer to other tissues or other optimization functions
- The virus is represented only through its biomass function, so essentiality reflects host-metabolome network effects rather than direct viral protein targets

## Evidence

- [methods] classify a gene as essential when grRatio < 0.2: [evidence span withheld — anchor status: not_found]
- [methods] host biomass capped at 10% of maximum: [evidence span withheld — anchor status: not_found]
