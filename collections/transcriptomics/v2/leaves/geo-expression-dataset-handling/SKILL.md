---
name: geo-expression-dataset-handling
description: Use when raw/processed RNA-seq lung expression data must be fetched from
  GEO accessions (e.g. GSE147507, GSE148729) and converted into per-sample binarized
  gene expression vectors to drive context-specific metabolic model reconstruction.
when_to_use_negative:
- The data are microarray .CEL files rather than RNA-seq counts — use the FASTCORMICS
  (microarray) branch with BARCODE transformation instead of the DESeq2/rFASTCORMICS
  RNA-seq path
- The goal is differential expression analysis alone without downstream metabolic
  model reconstruction — a plain DESeq2 workflow suffices
- A condition shows no differentially expressed metabolic genes (e.g. the A549_0.02
  condition), which must be discarded rather than forced into model building
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DESeq2
  role: Compute per-condition differentially expressed genes (absolute log2 fold change
    >1, adjusted p-value <0.05) from the GEO datasets
- name: rFASTCORMICS
  role: Consumes the binarized expression data and consistent generic model to prune
    one context-specific model per sample
  repo: https://github.com/sysbiolux/rFASTCORMICS
- name: COBRA Toolbox V3
  role: MATLAB toolbox providing the constraint-based framework in which rFASTCORMICS
    builds the per-sample models
  repo: https://github.com/opencobra/cobratoolbox
- name: edgeR
  role: filterByExpr function (Version 3.30.3) removes low-expression genes before
    DEG calling
- name: R
  role: Environment for running DESeq2/edgeR on the GEO expression data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:10bdfd51dccb5b600cc7bab5716485e61accf3b95f40675130cf376b260abb3f
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
  - edgeR
  - rFASTCORMICS
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# geo-expression-dataset-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieve SARS-CoV-2 lung expression datasets from GEO (severity study GSE147507 and time-series study GSE148729), compute per-condition differentially expressed genes with DESeq2, and binarize them onto generic metabolic model genes as rFASTCORMICS input for context-specific model reconstruction.

## When to use

Use when raw/processed RNA-seq lung expression data must be fetched from GEO accessions (e.g. GSE147507, GSE148729) and converted into per-sample binarized gene expression vectors to drive context-specific metabolic model reconstruction.

## When NOT to use

- The data are microarray .CEL files rather than RNA-seq counts — use the FASTCORMICS (microarray) branch with BARCODE transformation instead of the DESeq2/rFASTCORMICS RNA-seq path
- The goal is differential expression analysis alone without downstream metabolic model reconstruction — a plain DESeq2 workflow suffices
- A condition shows no differentially expressed metabolic genes (e.g. the A549_0.02 condition), which must be discarded rather than forced into model building

## Inputs

- GEO accession GSE147507 (severity study expression metadata and counts)
- GEO accession GSE148729 (time-series study expression metadata and counts)
- Raw or processed RNA-seq count matrix per sample (FPKM/TPM usable for rFASTCORMICS)
- Gene identifier dictionary (dico) mapping experiment rownames to model genes (e.g. from Ensembl Biomart)
- Consistent generic metabolic model in COBRA format (Recon3D_01-based; VBOF added for infected conditions)

## Outputs

- Per-condition DEG tables from DESeq2 (absolute log2FC >1, adjusted p-value <0.05)
- Discretized expression matrix (1/-1/0 per gene per sample)
- One context-specific metabolic model per infected and mock sample (50 models total: 28 infected, 22 mock)
- Per-model metabolite and reaction counts written to model_statistics_summary.csv

## How to apply

Retrieve the severity study metadata (GEO: GSE147507) and time-series study metadata (GEO: GSE148729), covering the modeled lung cell lines A549, Calu-3, H1299 and NHBE. Compute differentially expressed genes per condition with DESeq2 using an absolute log2 fold change >1 and adjusted p-value < 0.05, optionally pre-filtering low-expression genes with edgeR's filterByExpr. Binarize the expression onto the generic model genes as rFASTCORMICS expects: 1 for expressed, -1 for not expressed, and 0 for unknown. Feed the discretized data together with a consistent Recon3D_01-based generic model (with the viral biomass function added for infected conditions) into the fastcormics_RNAseq MATLAB function, which uses the GPR rules to map genes to reactions. Verify the run by counting the resulting models and their sizes: the reference execution produced 50 models (28 infected and 22 mock) with a median of 3646 metabolites (2465-5088) and 2456.5 reactions (1790-3474).

## Related tools

- **DESeq2** (Compute per-condition differentially expressed genes (absolute log2 fold change >1, adjusted p-value <0.05) from the GEO datasets)
- **rFASTCORMICS** (Consumes the binarized expression data and consistent generic model to prune one context-specific model per sample) — https://github.com/sysbiolux/rFASTCORMICS
- **COBRA Toolbox V3** (MATLAB toolbox providing the constraint-based framework in which rFASTCORMICS builds the per-sample models) — https://github.com/opencobra/cobratoolbox
- **edgeR** (filterByExpr function (Version 3.30.3) removes low-expression genes before DEG calling)
- **R** (Environment for running DESeq2/edgeR on the GEO expression data)

## Examples

```
[model, A_final] = fastcormics_RNAseq(model, data, rownames, dico, biomass_rxn, already_mapped_tag, consensus_proportion, epsilon, optional_settings)
```

## Evaluation signals

- Model count check: exactly 28 infected and 22 mock context-specific models are produced from the two GEO studies
- Size sanity check: per-model metabolite counts fall within the reported range 2465-5088 (median 3646) and reaction counts within 1790-3474 (median 2456.5)
- DEG threshold audit: every reported DEG satisfies |log2FC| > 1 and adjusted p-value < 0.05
- Binarization schema check: the input expression matrix contains only values 1 (expressed), -1 (not expressed), and 0 (unknown)
- Conditions with no differentially expressed metabolic genes (e.g. A549_0.02) are absent from downstream model building

## Limitations

- A549_0.02 condition showed no differentially expressed metabolic genes and had to be discarded from the DEGs metabolic pathway analysis
- Only modeled lung cell lines (A549, Calu-3, H1299, NHBE) are covered, limiting generalizability
- RNA-seq data should preferably be FPKM or TPM transformed for rFASTCORMICS; a gene identifier dictionary (dico) must be assembled to map rownames to model genes

## Evidence

- [readme] rFASTCORMICS purpose: [evidence span withheld — anchor status: external_doc]
- [readme] discretized data encoding: [evidence span withheld — anchor status: external_doc]
