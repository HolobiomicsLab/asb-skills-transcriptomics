---
name: constraint-based-model-statistics-verification
description: Use when RNA-seq datasets from multiple conditions (e.g. GSE147507, GSE148729)
  must be converted to one context-specific COBRA model per sample, and per-model
  metabolite/reaction counts must be tallied and checked against published medians.
when_to_use_negative:
- The A549_0.02 condition showed no differentially expressed metabolic genes and was
  discarded — do not force a model from a condition lacking DEGs for metabolic pathway
  analysis
- The goal is drug repositioning or essential-gene identification rather than reconstructing
  and verifying the model set itself (those are downstream skills)
- Input is microarray data rather than RNA-seq — use FASTCORMICS (BARCODE-transformed
  microarray input) instead of the RNA-seq rFASTCORMICS path
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: rFASTCORMICS
  role: Prunes the consistent generic model per sample into a context-specific model
    using binarized RNA-seq data
  repo: https://github.com/sysbiolux/rFASTCORMICS
- name: DESeq2
  role: Computes differentially expressed genes per condition (|log2FC| >1, adjusted
    p-value <0.05) used to binarize expression
- name: COBRA Toolbox V3
  role: MATLAB framework in which the rFASTCORMICS pruning and model handling is executed
  repo: https://github.com/opencobra/cobratoolbox
- name: MATLAB
  role: Runtime environment (2013 or higher) for rFASTCORMICS and COBRA Toolbox scripts
- name: R
  role: Environment for running DESeq2 differential expression analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:c5232f89f66a4c6f4f4035db63c4b259a8aef6a45f9b84bfe69084ed64da782b
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
  - rFASTCORMICS
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# constraint-based-model-statistics-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs condition-specific metabolic models from infected and mock lung expression data with rFASTCORMICS and verifies their size statistics (metabolite and reaction counts, medians and ranges) against reported values (50 models: 28 infected, 22 mock).

## When to use

Use when RNA-seq datasets from multiple conditions (e.g. GSE147507, GSE148729) must be converted to one context-specific COBRA model per sample, and per-model metabolite/reaction counts must be tallied and checked against published medians.

## When NOT to use

- The A549_0.02 condition showed no differentially expressed metabolic genes and was discarded — do not force a model from a condition lacking DEGs for metabolic pathway analysis
- The goal is drug repositioning or essential-gene identification rather than reconstructing and verifying the model set itself (those are downstream skills)
- Input is microarray data rather than RNA-seq — use FASTCORMICS (BARCODE-transformed microarray input) instead of the RNA-seq rFASTCORMICS path

## Inputs

- Expression datasets GSE147507 (severity study) and GSE148729 (time-series study) from GEO
- Consistent generic genome-scale metabolic model (Recon3D_01-based, COBRA format) with VBOF added for infected conditions
- DEG gene identifier dictionary (dico) mapping expression rownames to model genes
- DEG thresholds: absolute log2 fold change >1, adjusted p-value <0.05 (DESeq2)

## Outputs

- 50 context-specific metabolic models (28 infected and 22 mock)
- Per-model metabolite and reaction counts (expected medians: 3646 metabolites, range 2465-5088; 2456.5 reactions, range 1790-3474)
- model_statistics_summary.csv with per-model counts and verification results

## How to apply

Retrieve the severity (GSE147507) and time-series (GSE148729) expression metadata, compute DEGs per condition with DESeq2 using |log2 fold change| >1 and adjusted p-value <0.05, and binarize expression onto generic model genes (1 expressed, -1 not expressed, 0 unknown). Load the consistent generic model (Recon3D_01-based, with the viral biomass function added for infected conditions) and run the rFASTCORMICS workflow in MATLAB with COBRA Toolbox V3, pruning one context-specific model per infected and mock sample. Count metabolites and reactions per model, confirm 28 infected and 22 mock models were produced, and compare the computed medians to the reported values (3646 metabolites, range 2465-5088; 2456.5 reactions, range 1790-3474). Write per-model counts and verification results to a summary CSV (model_statistics_summary.csv).

## Related tools

- **rFASTCORMICS** (Prunes the consistent generic model per sample into a context-specific model using binarized RNA-seq data) — https://github.com/sysbiolux/rFASTCORMICS
- **DESeq2** (Computes differentially expressed genes per condition (|log2FC| >1, adjusted p-value <0.05) used to binarize expression)
- **COBRA Toolbox V3** (MATLAB framework in which the rFASTCORMICS pruning and model handling is executed) — https://github.com/opencobra/cobratoolbox
- **MATLAB** (Runtime environment (2013 or higher) for rFASTCORMICS and COBRA Toolbox scripts)
- **R** (Environment for running DESeq2 differential expression analysis)

## Examples

```
[model, A_final] = fastcormics_RNAseq(model, data, rownames, dico, 'biomass_reaction', 0, 0.9, 1e-4, opt);
```

## Evaluation signals

- Model count invariant: exactly 50 models produced — 28 infected and 22 mock
- Median metabolite count matches 3646 with range 2465-5088
- Median reaction count matches 2456.5 with range 1790-3474
- Each output model contains the expected biomass reaction (biomass_rxn forced to carry flux; VBOF present in infected models)
- model_statistics_summary.csv exists with per-model counts and verification results consistent with the reported values

## Limitations

- A549_0.02 condition had no differentially expressed metabolic genes and was discarded, so not every condition yields a model
- Study limited by modeled lung cell lines (A549, Calu-3, H1299, NHBE)
- rFASTCORMICS expects RNA-seq data preferably FPKM/TPM transformed and a consistent genome-scale model (e.g. Recon 2.04 or Recon3D_01) plus a gene identifier dictionary

## Evidence

- [readme] rFASTCORMICS function signature used to build each context-specific model in MATLAB: [evidence span withheld — anchor status: not_found]
- [readme] Binarized expression encoding expected by rFASTCORMICS input data: [evidence span withheld — anchor status: external_doc]
