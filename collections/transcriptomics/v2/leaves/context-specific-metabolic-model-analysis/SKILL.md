---
name: context-specific-metabolic-model-analysis
description: Use when RNA-seq data from multiple disease/control conditions (e.g.,
  SARS-CoV-2 infected vs mock lung samples) must be converted into per-sample context-specific
  metabolic models and queried by FBA for essential genes or drug targets.
when_to_use_negative:
- The goal is direct docking on the viral proteome — the models represent the virus
  only through its biomass function, so targets are host-metabolome network effects
- No consistent generic model, dico gene-mapping table, or MATLAB/COBRA Toolbox environment
  is available
- The aim is condition-agnostic analysis of a single generic genome-scale model rather
  than per-sample context-specific pruning
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: rFASTCORMICS
  role: Reconstructs one context-specific metabolic model per RNA-seq sample by pruning
    the generic model via GPR-mapped binarized expression
  repo: https://github.com/sysbiolux/rFASTCORMICS
- name: COBRA Toolbox V3
  role: Loads models in MATLAB and runs FBA and singleDeletion simulations to test
    gene essentiality
  repo: https://github.com/opencobra/cobratoolbox
- name: DESeq2
  role: Computes differentially expressed genes (absolute log2 fold change >1, adjusted
    p-value <0.05) used to binarize expression
- name: Recon3D_01 / Recon2.04
  role: Generic genome-scale metabolic reconstructions used as rFASTCORMICS input
    after VBOF addition
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:b4056ac54054726183e8428e5a7c62fce76b4001c2ab2b3fe62b071ced5c65cd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sysbiolux/DCcov
  repo_url: https://github.com/sysbiolux/rFASTCORMICS
  tools:
  - COBRA Toolbox
  - COBRA Toolbox V3
  - DESeq2
  - MATLAB
  - R
  - Recon3D_01 / Recon2.04
  - rFASTCORMICS
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# context-specific-metabolic-model-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct and analyze context-specific genome-scale metabolic models from RNA-seq expression data using rFASTCORMICS with the COBRA Toolbox V3, then run in silico gene knockouts against a viral biomass objective function (VBOF) to identify host genes whose knockout reduces viral biomass without affecting host biomass.

## When to use

Use when RNA-seq data from multiple disease/control conditions (e.g., SARS-CoV-2 infected vs mock lung samples) must be converted into per-sample context-specific metabolic models and queried by FBA for essential genes or drug targets.

## When NOT to use

- The goal is direct docking on the viral proteome — the models represent the virus only through its biomass function, so targets are host-metabolome network effects
- No consistent generic model, dico gene-mapping table, or MATLAB/COBRA Toolbox environment is available
- The aim is condition-agnostic analysis of a single generic genome-scale model rather than per-sample context-specific pruning

## Inputs

- RNA-seq data (FPKM or TPM transformed) with sample colnames and gene rownames
- Consistent genome-scale metabolic reconstruction in COBRA format (Recon2.04 or Recon3D_01)
- Gene identifier mapping table (dico, from Ensembl Biomart)
- Viral biomass objective function (VBOF) added to generic models
- Optional medium composition constraints

## Outputs

- One context-specific metabolic model per sample (28 infected, 22 mock in the source study)
- Model size statistics per model (metabolites and reactions, e.g., median 3646 metabolites, 2456.5 reactions)
- In silico single/double gene deletion results with essentiality classifications
- Per-gene/pair essentiality and safety scores and an essentiality-vs-safety scatter (e.g., figure3_essentiality_safety_scatter.png)

## How to apply

Retrieve expression metadata (e.g., GEO GSE147507 severity study and GSE148729 time-series), compute DEGs with DESeq2 (absolute log2 fold change >1, adjusted p-value <0.05), and binarize expression (1/-1/0) onto a consistent generic model (Recon2.04 or Recon3D_01) with a viral biomass objective function added for infected conditions. Run fastcormics_RNAseq in MATLAB (COBRA Toolbox V3, IBM CPLEX) per sample, forcing the biomass reaction to carry flux, to obtain one context-specific model per condition. Query the models with in silico single- and double-gene deletion (COBRA Toolbox singleDeletion), flagging a knockout as essential when viral biomass production is reduced without affecting host biomass maintenance. Aggregate per gene/pair into an essentiality score (count of infected models where essential) and a safety score (count of mock models where knockout is safe) to rank candidates like GUK1 for a favorable toxicity profile.

## Related tools

- **rFASTCORMICS** (Reconstructs one context-specific metabolic model per RNA-seq sample by pruning the generic model via GPR-mapped binarized expression) — https://github.com/sysbiolux/rFASTCORMICS
- **COBRA Toolbox V3** (Loads models in MATLAB and runs FBA and singleDeletion simulations to test gene essentiality) — https://github.com/opencobra/cobratoolbox
- **DESeq2** (Computes differentially expressed genes (absolute log2 fold change >1, adjusted p-value <0.05) used to binarize expression)
- **Recon3D_01 / Recon2.04** (Generic genome-scale metabolic reconstructions used as rFASTCORMICS input after VBOF addition)

## Examples

```
[model, A_final] = fastcormics_RNAseq(model, data, rownames, dico, biomass_rxn, 0, 0.9, 1e-4, optional_settings);
```

## Evaluation signals

- Condition-specific reconstruction yields 50 models (28 infected, 22 mock) with median 3646 metabolites (range 2465-5088) and 2456.5 reactions (range 1790-3474)
- The biomass reaction carries flux in every pruned model (biomass_rxn argument forces flux)
- Essentiality criterion holds: viral biomass production is reduced while host biomass maintenance is unaffected
- Prioritized knockouts (e.g., GUK1) fall in the essential-but-safe region: nonzero essentiality across infected models and unaffected human biomass in mock models
- Knockouts with essentiality but lowest safety (e.g., CRLS1, SGMS1) are flagged as high toxicity rather than candidates

## Limitations

- The work is limited by the modeled lung cell lines (A549, Calu-3, H1299, NHBE); other tissues may behave differently
- Further in vitro single- and double-gene deletion studies are needed to confirm essentiality of predicted genes and gene pairs
- The virus is modeled only through its biomass function, so drug targets reflect host metabolome network effects rather than direct antiviral binding
- Tests on other tissues, other optimization functions, and in vitro/in vivo validation are required before treating predictions as drug candidates

## Evidence

- [readme] rFASTCORMICS reconstructs context-specific metabolic models from RNA-seq data: [evidence span withheld — anchor status: external_doc]
- [readme] Binarized experimental data and required inputs for fastcormics_RNAseq: [evidence span withheld — anchor status: external_doc]
