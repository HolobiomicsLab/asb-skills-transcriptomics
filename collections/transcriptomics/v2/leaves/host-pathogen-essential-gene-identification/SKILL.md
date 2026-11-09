---
name: host-pathogen-essential-gene-identification
description: Use when you have context-specific host metabolic models with a viral
  biomass reaction and need to find gene knockouts that reduce viral growth (grRatio
  < 0.2) without compromising host biomass.
when_to_use_negative:
- You only have the generic reconstruction (Recon2.04/Recon3D_01) without expression
  data — context-specific models built with rFASTCORMICS are required for cell-line-relevant
  essentiality calls
- The models lack a viral biomass objective function — knockout of host genes against
  a host-only biomass objective identifies host-essential rather than viral-specific
  targets
- The goal is direct antiviral docking against the viral proteome — this approach
  models the virus only through its biomass function, so targets arise from host metabolome
  network effects
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_2259
tools:
- name: COBRA Toolbox V3
  role: Provides the singleGeneDeletion routine and FBA solver used to compute per-knockout
    viral biomass fluxes in MATLAB
  repo: https://github.com/opencobra/cobratoolbox
- name: rFASTCORMICS
  role: Builds the infected and mock context-specific lung-cell models (from RNA-seq
    data) on which single-gene deletion is performed
  repo: https://github.com/sysbiolux/rFASTCORMICS
- name: MATLAB
  role: Runtime environment hosting the COBRA Toolbox deletion and FBA workflow
- name: Recon3D_01
  role: Generic genome-scale reconstruction used as the base model to which the VBOF
    is added before context-specific reconstruction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:0167a32d476ad83025fc79b4d07641919c6787e71ae39cc7f3e654e6b144968f
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
  - Recon3D_01
  - rFASTCORMICS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# host-pathogen-essential-gene-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies host genes essential for viral biomass production in SARS-CoV-2-infected lung cells by running in silico single-gene deletion (FBA) on context-specific metabolic models reconstructed with rFASTCORMICS and carrying a viral biomass objective function (VBOF).

## When to use

Use when you have context-specific host metabolic models with a viral biomass reaction and need to find gene knockouts that reduce viral growth (grRatio < 0.2) without compromising host biomass.

## When NOT to use

- You only have the generic reconstruction (Recon2.04/Recon3D_01) without expression data — context-specific models built with rFASTCORMICS are required for cell-line-relevant essentiality calls
- The models lack a viral biomass objective function — knockout of host genes against a host-only biomass objective identifies host-essential rather than viral-specific targets
- The goal is direct antiviral docking against the viral proteome — this approach models the virus only through its biomass function, so targets arise from host metabolome network effects

## Inputs

- Infected context-specific metabolic models in COBRA format (MATLAB structures), built with rFASTCORMICS and carrying the viral biomass objective function (VBOF)
- Matching mock/control models for host biomass reference fluxes
- Viral biomass objective function (VBOF) definition added to the generic reconstruction (Recon2.04 or Recon3D_01)
- Expression datasets (e.g. GEO: GSE147507 severity study, GSE148729 time-series) used to build the context-specific models

## Outputs

- essential_genes.csv — deduplicated list of unique essential genes (23 in the DCcov study, including CRLS1, GUK1, SGMS1, DTYMK, CMPK1, TYMS)
- Per-gene deletion grRatio values (post-knockout / pre-knockout viral biomass flux) for each infected model
- Essential gene pairs from double-knockout analysis for drug-combination mapping to DrugBank V5

## How to apply

In MATLAB with the COBRA Toolbox V3, load the infected context-specific models (built with rFASTCORMICS on generic reconstructions such as Recon2.04/Recon3D_01 with the VBOF from Renz et al. 2020 added). Run FBA on each model to get the maximal host biomass flux, then fix the host biomass reaction upper bound to 10% of that maximum so host growth cannot mask viral-biomass changes. Execute the COBRA Toolbox singleGeneDeletion routine over all genes, optimizing the viral biomass objective function for each knockout, and compute the grRatio (post-knockout viral biomass flux divided by the pre-knockout maximum). Classify a gene as essential when grRatio < 0.2, then aggregate calls across all infected models (e.g. the severity-study GSE147507 and time-series GSE148729 conditions), deduplicate gene names, and write the list (23 unique genes in the DCcov study, including CRLS1, GUK1, SGMS1, DTYMK, CMPK1, TYMS) to essential_genes.csv. Essentiality under the viral objective, not the host objective, is what distinguishes candidate anti-viral drug targets for downstream DrugBank repositioning.

## Related tools

- **COBRA Toolbox V3** (Provides the singleGeneDeletion routine and FBA solver used to compute per-knockout viral biomass fluxes in MATLAB) — https://github.com/opencobra/cobratoolbox
- **rFASTCORMICS** (Builds the infected and mock context-specific lung-cell models (from RNA-seq data) on which single-gene deletion is performed) — https://github.com/sysbiolux/rFASTCORMICS
- **MATLAB** (Runtime environment hosting the COBRA Toolbox deletion and FBA workflow)
- **Recon3D_01** (Generic genome-scale reconstruction used as the base model to which the VBOF is added before context-specific reconstruction)

## Examples

```
[grRatio, ~, grRateWT, ~, ~] = singleGeneDeletion(model); essential = model.genes(strcmp(grRatio, '')); idx = cellfun(@isempty, grRatio); essential_idx = find(cell2mat(grRatio( ~idx )) < 0.2 | strcmp(grRatio, ''))
```

## Evaluation signals

- Output essential_genes.csv contains exactly 23 unique genes after deduplication across all infected models
- The reproduced set includes the reported genes CRLS1, SGMS1, GUK1, DTYMK, CMPK1, and TYMS
- Every gene classified essential has grRatio < 0.2 under the viral biomass objective
- Host biomass reaction upper bound was capped at 10% of the pre-knockout FBA maximum before deletions, so host growth cannot mask viral-biomass changes
- Several essential genes (CRLS1, GUK1, SGMS1) are shared across conditions of the severity study, indicating cross-model consistency

## Limitations

- Essentiality predictions require in vitro confirmation: 'further in vitro single- and double-gene deletion studies are needed to determine the essentiality of the identified single genes and gene pairs.'
- Results are limited to the modeled lung cell lines (A549, Calu-3, H1299, NHBE); other tissues and optimization functions would need retesting
- Drug targets identified this way rely on network effects on the host metabolome (the virus is only modeled through its biomass function), not direct interaction with the viral proteome
- One condition (A549_0.02) showed no differentially expressed metabolic genes and was excluded, so coverage of conditions is not uniform

## Evidence

- [methods] grRatio threshold and singleGeneDeletion: [evidence span withheld — anchor status: not_found]
- [methods] host biomass 10% cap: [evidence span withheld — anchor status: not_found]
- [methods] essentiality classification rule: [evidence span withheld — anchor status: not_found]
- [results] 23 unique essential genes finding: [evidence span withheld — anchor status: not_found]
- [readme] rFASTCORMICS model construction: [evidence span withheld — anchor status: external_doc]
