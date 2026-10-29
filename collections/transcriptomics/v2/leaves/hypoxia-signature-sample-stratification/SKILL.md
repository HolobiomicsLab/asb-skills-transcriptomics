---
name: hypoxia-signature-sample-stratification
description: Use when a counts matrix of tumor RNA-seq samples (e.g., TCGA-LUAD from
  GDC) must be split into hypoxia-high vs hypoxia-low groups before differential expression
  or survival analysis, using a gene-set score rather than a single marker gene.
when_to_use_negative:
- Input is already a normalized TPM/FPKM matrix or lacks Ensembl IDs — ARMT expects
  a raw counts matrix with Ensembl ID row names for its normalization step
- The goal is single-cell or non-bulk RNA-seq stratification — the ARMT/GSVA workflow
  is designed for bulk transcriptome matrices (TPM)
- The samples are already partitioned by an external categorical label (e.g., tumor
  vs normal) — the skill is for splitting a continuous score into high/low groups,
  not for predefined comparisons
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GSVA
  role: Computes the per-sample hypoxia signature score from the log2(TPM+1) expression
    matrix using the 14-gene .gmt geneset
- name: ARMT
  role: Provides the GUI (Normalization&GSVA page) that normalizes counts to TPM,
    runs GSVA with the custom .gmt file, and integrates the scores with clinical data;
    the 't' cutoff parameter defines the top/bottom group split
  repo: https://github.com/Dulab2020/ARMT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:67ac56f79a39e3ff2a1f507ba02c24337914f36dcaeb1ef2adb45ae80dd60383
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Dulab2020/ARMT
  repo_url: https://github.com/Dulab2020/ARMT
  tools:
  - ARMT
  - GSVA
  - R
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# hypoxia-signature-sample-stratification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies bulk RNA-seq tumor samples (e.g., TCGA-LUAD) into hypoxia-high and hypoxia-low groups by computing per-sample GSVA scores from a 14-gene hypoxia signature (ALDOA, MIF, TUBB6, P4HA1, SLC2A1, PGAM1, ENO1, LDHA, CDKN3, TPI1, NDRG1, VEGFA, ACOT7, ADM) on log2(TPM+1) expression, enabling downstream differential and survival analysis.

## When to use

Use when a counts matrix of tumor RNA-seq samples (e.g., TCGA-LUAD from GDC) must be split into hypoxia-high vs hypoxia-low groups before differential expression or survival analysis, using a gene-set score rather than a single marker gene.

## When NOT to use

- Input is already a normalized TPM/FPKM matrix or lacks Ensembl IDs — ARMT expects a raw counts matrix with Ensembl ID row names for its normalization step
- The goal is single-cell or non-bulk RNA-seq stratification — the ARMT/GSVA workflow is designed for bulk transcriptome matrices (TPM)
- The samples are already partitioned by an external categorical label (e.g., tumor vs normal) — the skill is for splitting a continuous score into high/low groups, not for predefined comparisons

## Inputs

- Raw gene counts matrix as .csv (rows = Ensembl gene IDs, columns = sample IDs, e.g., TCGA-LUAD tumor samples from GDC)
- Hypoxia gene signature as two-column .csv (geneset name + gene symbols) used to create a .gmt file
- TCGA-LUAD clinical data (optional, for integrating groups with survival metadata)
- Grouping fraction parameter t (0.3 for top/bottom 30% hypoxia-high/low split)

## Outputs

- Per-sample hypoxia GSVA score matrix (.csv, produced by ARMT's Normalization&GSVA page)
- Hypoxia-high and hypoxia-low sample group assignments (top 30% vs bottom 30% of GSVA score distribution)
- Group labels ready for downstream edgeR/limma differential expression and survival analysis

## How to apply

First normalize the raw counts matrix to TPM and transform to log2(TPM + 1), following the ARMT/GSVA convention; the input matrix must use Ensembl gene IDs as row names and sample IDs as columns (ARMT maps Ensembl ID to gene Symbol during normalization). Provide the 14-gene hypoxia signature (ALDOA, MIF, TUBB6, P4HA1, SLC2A1, PGAM1, ENO1, LDHA, CDKN3, TPI1, NDRG1, VEGFA, ACOT7, ADM) as a two-column .csv to build a .gmt file on ARMT's Data page, then run GSVA on ARMT's 'Normalization&GSVA' page — for TCGA input, restrict to tumor samples (the default). Rank all tumor samples by their hypoxia GSVA score and define hypoxia-high and hypoxia-low groups as the top and bottom 30% of the score distribution, respectively; ARMT's adjustable cutoff 't' implements the top/bottom t*100% grouping, with the low group treated as the control in subsequent differential analysis. Feed the resulting group assignments into edgeR (TMM normalization) DE testing, and verify the pipeline by reproducing the reported LUAD hypoxia-group DEG counts (6684 DEGs at p < 0.05, -log10(FDR) > 2, logFC > 1; 1174 up-regulated at -log10(FDR) > 20).

## Related tools

- **GSVA** (Computes the per-sample hypoxia signature score from the log2(TPM+1) expression matrix using the 14-gene .gmt geneset)
- **ARMT** (Provides the GUI (Normalization&GSVA page) that normalizes counts to TPM, runs GSVA with the custom .gmt file, and integrates the scores with clinical data; the 't' cutoff parameter defines the top/bottom group split) — https://github.com/Dulab2020/ARMT

## Examples

```
devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()  # then use Data page to build the 14-gene hypoxia .gmt and Normalization&GSVA page to score tumor samples
```

## Evaluation signals

- Every one of the 14 signature genes (ALDOA through ADM) is present in the TPM matrix after Ensembl-to-Symbol mapping; missing genes should be flagged before scoring
- GSVA is computed on log2(TPM + 1) values, not raw counts, and only tumor samples are scored (ARMT default for TCGA input)
- Hypoxia-high and hypoxia-low groups each contain ~30% of LUAD tumor samples with no overlap, consistent with the top/bottom 30% split
- Downstream edgeR DE between the groups at p < 0.05, -log10(FDR) > 2, logFC > 1 should recover ~6684 DEGs, and the stringent filter (-log10(FDR) > 20) should yield ~1174 up-regulated genes
- Hypoxia-high vs hypoxia-low survival separation should be checkable: the article reports higher hypoxia score associated with poorer LUAD survival

## Limitations

- ARMT's built-in grouping convention for continuous factors is top/bottom 50% (used for survival analysis); the 30% hypoxia-high/low split relies on the adjustable 't' cutoff parameter, so the threshold must be set explicitly
- Differential tools based on different statistical principles may lead to different results, so downstream DEG counts can vary between edgeR and limma runs on the same stratified groups
- No software version numbers for GSVA/edgeR/limma or ARMT itself are stated in the article or README, complicating exact reproducibility

## Evidence
