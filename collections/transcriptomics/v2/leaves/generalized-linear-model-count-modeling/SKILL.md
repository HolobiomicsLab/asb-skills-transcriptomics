---
name: generalized-linear-model-count-modeling
description: Use when raw single-cell gene count data (e.g. Smart-seq2) has cells
  assigned to clone/subclone groups and you must test which genes differ between groups
  while accounting for plate and detection-rate confounders.
when_to_use_negative:
- Input consists of already-normalized values (TPM/CPM/log-expression) instead of
  raw counts, since edgeR GLMs model raw count dispersion
- The goal is calling clonally informative mtDNA variants or assigning cells to clones
  — use the cellSNP-MQuad-vireoSNP pipeline instead
- Cells lack clone/subclone labels or groups are so small that the edgeR design matrix
  (detection rate + plate + clone) cannot be estimated
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: edgeR
  role: Negative-binomial GLM fit with quasi-likelihood F-test for the clone1.MT0
    vs clone1.MT1 differential expression contrast
- name: IHW
  role: Independent hypothesis weighting for multiple-testing correction of gene and
    gene-set p-values using average expression as the covariate
- name: limma
  role: Two-sided camera test on 50 MSigDB Hallmark gene sets using edgeR log2-fold-changes
    as input
- name: Molecular Signatures Database (MsigDB)
  role: Source of the 50 Hallmark gene sets tested with camera
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:424a9030c935324f0aae2482a511e2594e54dd3c4286e24bde7c358f6ac65bb4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IHW
  - Molecular Signatures Database (MsigDB)
  - edgeR
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# generalized-linear-model-count-modeling

## Summary

Fits negative-binomial generalized linear models (edgeR quasi-likelihood) to single-cell gene count matrices to test differential expression between mtDNA-defined subclones, with IHW multiple-testing correction and limma camera pathway enrichment on MSigDB Hallmark sets.

## When to use

Use when raw single-cell gene count data (e.g. Smart-seq2) has cells assigned to clone/subclone groups and you must test which genes differ between groups while accounting for plate and detection-rate confounders.

## When NOT to use

- Input consists of already-normalized values (TPM/CPM/log-expression) instead of raw counts, since edgeR GLMs model raw count dispersion
- The goal is calling clonally informative mtDNA variants or assigning cells to clones — use the cellSNP-MQuad-vireoSNP pipeline instead
- Cells lack clone/subclone labels or groups are so small that the edgeR design matrix (detection rate + plate + clone) cannot be estimated

## Inputs

- Single-cell gene expression count matrix (raw counts, e.g. Smart-seq2 scRNA-seq of fibroblasts)
- Per-cell mtDNA/nuclear clone assignments from upstream clonal reconstruction (e.g. vireoSNP output from task_003)
- Per-cell metadata: cellular detection rate, plate/batch identifiers
- MSigDB Hallmark gene sets (50 sets)

## Outputs

- de_gene_table.csv — per-gene edgeR quasi-likelihood statistics with FDR values, DE genes at FDR < 0.1
- gene_set_enrichment_table.csv — camera results for 50 Hallmark sets with IHW-adjusted FDR < 0.05 flags

## How to apply

Load the fibroblast Smart-seq2 single-cell gene expression counts together with mtDNA clone assignments, restricting to cells within the nuclear clone of interest (e.g. clone1.MT0 vs clone1.MT1). Fit an edgeR generalized linear model with cellular detection rate, plate, and assigned mitochondrial clone as predictor variables, testing the subclone contrast with the two-sided quasi-likelihood F-test. Apply independent hypothesis weighting (IHW) using average gene expression as the independent covariate and retain DE genes at FDR < 0.1 (the paper reports 847 such genes between clone1.MT0 and clone1.MT1). Then run the two-sided camera test from limma on the 50 MSigDB Hallmark gene sets using the edgeR log2-fold-changes, adjusting gene set p-values with IHW the same way and calling enrichments at FDR < 0.05 (expecting MYC targets, E2F targets, G2M checkpoint, and Reactive Oxygen Species pathway).

## Related tools

- **edgeR** (Negative-binomial GLM fit with quasi-likelihood F-test for the clone1.MT0 vs clone1.MT1 differential expression contrast)
- **IHW** (Independent hypothesis weighting for multiple-testing correction of gene and gene-set p-values using average expression as the covariate)
- **limma** (Two-sided camera test on 50 MSigDB Hallmark gene sets using edgeR log2-fold-changes as input)
- **Molecular Signatures Database (MsigDB)** (Source of the 50 Hallmark gene sets tested with camera)

## Examples

```
R: fit <- glmQLFit(y, design); qlf <- glmQLFTest(fit, contrast="clone1.MT1-clone1.MT0"); ihw_fdr <- ihw(qlf$table$PValue, qlf$table$logCPM, alpha=0.1)
```

## Evaluation signals

- de_gene_table.csv contains a gene-level statistics column with FDR values (not only raw p-values)
- Number of significant DE genes at FDR < 0.1 is consistent with the paper's reported 847 genes
- The four expected Hallmark sets — MYC targets, E2F targets, G2M checkpoint, and ROS Pathway — are flagged significant at FDR < 0.05 in gene_set_enrichment_table.csv
- Design matrix explicitly includes cellular detection rate, plate, and mtDNA clone terms before testing the clone1.MT0 vs clone1.MT1 contrast
- Multiple-testing correction used IHW with average gene expression as the covariate (not plain BH) for both gene and gene-set p-values

## Limitations

- Result quality depends on correct upstream clone assignments from mtDNA-based clonal reconstruction; noisy or uninformative mtDNA SNVs confound the group labels
- Single-cell assays such as Smart-seq2 have high dropout, so subclone DE calls rely on the count GLM handling sparsity and on detection-rate modeling
- IHW power depends on the chosen independent covariate (here average gene expression); a poorly informative covariate weakens FDR control efficiency

## Evidence

- [other] edgeR GLM with cellular detection rate, plate, and mtDNA clone covariates: [evidence span withheld — anchor status: not_found]
- [other] IHW correction at FDR < 0.1 using average expression covariate: [evidence span withheld — anchor status: not_found]
- [discussion] camera test on 50 Hallmark sets: "Using 50 Hallmark gene sets from Molecular Signatures Database (MsigDB)"
