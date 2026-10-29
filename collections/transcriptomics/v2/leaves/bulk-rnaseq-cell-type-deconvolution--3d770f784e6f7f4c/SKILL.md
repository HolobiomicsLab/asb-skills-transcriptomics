---
name: bulk-rnaseq-cell-type-deconvolution
description: Use when bulk RNA-seq read counts must be deconvolved into cell type
  proportions using a multi-subject scRNA-seq reference, especially for tissues with
  closely related cell types (e.g., pancreatic islet alpha/beta/delta/gamma, kidney
  nephron segments).
when_to_use_negative:
- Input expression data are TPM — cell type proportions cannot be estimated with TPM
  as the input (RPKM is usable only with cell-type-specific total RNA abundance estimates)
- Bulk samples come from multiple clinical conditions differing from the scRNA-seq
  reference — use MuSiC2's iterative algorithm instead
- Only trends (not absolute proportions) are needed under assumptions A1–A3, or the
  scRNA-seq reference lacks cross-subject replicates needed for cross-subject weighting
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: MuSiC
  role: 'Core W-NNLS deconvolution: music_prop, music_basis, music_prop.cluster, bulk_construct,
    relative.ab, Eval_multi'
  repo: https://github.com/xuranw/MuSiC
- name: R
  role: R package environment (Bioconductor/CRAN dependencies); MuSiC installed via
    devtools::install_github('xuranw/MuSiC')
- name: Biobase
  role: Provides the ExpressionSet containers for bulk and single-cell input data
- name: ggplot2
  role: Visualization of estimated proportions, regression fits, and sample correlation
    heatmaps
provenance:
  source_task_ids:
  - task_007
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-08023-x
    title: Bulk tissue cell type deconvolution with multi-subject single-cell expression
      reference
  source_content_hash: sha256:c49f2b703b35fa47d6c5ffca3e8ceb7ed4ecfc94bb824150b9cf1c6c5eb28a2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xuranw/MuSiC
  tools:
  - Biobase
  - MuSiC
  - R
  - ggplot2
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# bulk-rnaseq-cell-type-deconvolution

## Summary

Estimate cell type proportions in bulk RNA-seq with MuSiC, a weighted non-negative least squares (W-NNLS) deconvolution method that uses cross-subject single-cell RNA-seq references and weights genes by cross-subject and cross-cell consistency, avoiding pre-selected marker genes.

## When to use

Use when bulk RNA-seq read counts must be deconvolved into cell type proportions using a multi-subject scRNA-seq reference, especially for tissues with closely related cell types (e.g., pancreatic islet alpha/beta/delta/gamma, kidney nephron segments).

## When NOT to use

- Input expression data are TPM — cell type proportions cannot be estimated with TPM as the input (RPKM is usable only with cell-type-specific total RNA abundance estimates)
- Bulk samples come from multiple clinical conditions differing from the scRNA-seq reference — use MuSiC2's iterative algorithm instead
- Only trends (not absolute proportions) are needed under assumptions A1–A3, or the scRNA-seq reference lacks cross-subject replicates needed for cross-subject weighting

## Inputs

- Bulk RNA-seq raw read counts as Biobase ExpressionSet (e.g., GSE50244bulkeset.rds, 32581 features x 89 samples)
- Single-cell RNA-seq reference with cell type and subject annotations (e.g., EMTABesethealthy.rds, 25453 genes x 1097 cells; XinT2Deset.rds; Mousesub.eset from GSE107585) as ExpressionSet or SingleCellExperiment
- Character vectors naming the cluster column (e.g., 'cellType'), sample column (e.g., 'sampleID'), and selected cell types
- Optional: phenotype table (age, bmi, hba1c, gender) to join with estimated proportions for association testing

## Outputs

- Est.prop.weighted: matrix of weighted cell type proportion estimates per bulk sample (non-negative, normalized over selected cell types)
- Optional Est.prop.allgene / cluster-level proportions from music_prop.cluster
- Benchmark metrics from Eval_multi (RMSD, mAD, Pearson R) e.g., written to dropout_benchmark_results.csv
- Regression results (slope, SE, p-value) when proportions are joined to phenotypes, e.g., beta_hba1c_regression_results.csv

## How to apply

Load the bulk read-count ExpressionSet (e.g., GSE50244bulkeset.rds, 89 samples) and a single-cell reference ExpressionSet (e.g., EMTABesethealthy.rds from E-MTAB-5060 or XinT2Deset.rds) from the MuSiC vignette data repository. Keep raw read counts — cell type proportions cannot be estimated with TPM as the input. Run music_prop(bulk.eset, sc.eset, clusters='cellType', samples='sampleID', select.ct=...) to obtain Est.prop.weighted, with proportions constrained to major cell types that make up the bulk of the tissue (e.g., alpha, beta, delta, gamma, acinar, ductal for islets). When closely related cell types cause collinearity (e.g., 13 kidney cell types), first inspect the design matrix with music_basis, hierarchically cluster cell types on the log design matrix with complete linkage, and use music_prop.cluster with a clusters.type grouping (e.g., Group 3: Endo, CD-PC, CD-IC, LOH, DCT, PT) so stage-1 cluster proportions constrain stage-2 W-NNLS. Validate estimates against known truth using Eval_multi (RMSD, mAD, Pearson R) against relative.ab-computed real proportions from bulk_construct artificial bulks, or via biological concordance such as dominant cell type agreeing with anatomical position along the nephron.

## Related tools

- **MuSiC** (Core W-NNLS deconvolution: music_prop, music_basis, music_prop.cluster, bulk_construct, relative.ab, Eval_multi) — https://github.com/xuranw/MuSiC
- **R** (R package environment (Bioconductor/CRAN dependencies); MuSiC installed via devtools::install_github('xuranw/MuSiC'))
- **Biobase** (Provides the ExpressionSet containers for bulk and single-cell input data)
- **ggplot2** (Visualization of estimated proportions, regression fits, and sample correlation heatmaps)

## Examples

```
Est.prop.Xin <- music_prop(bulk.eset = GSE50244.bulk.eset, sc.eset = EMTAB.eset, clusters = 'cellType', samples = 'sampleID', select.ct = c('alpha','beta','delta','gamma','acinar','ductal'))
```

## Evaluation signals

- Estimated proportions for every subject are non-negative and normalized over the included cell types
- Against artificial bulk from bulk_construct with known relative.ab truth, Eval_multi reports low error: RMSD 0.09881, mAD 0.06357, R 0.9378 at zero dropout, and MuSiC remains accurate up to ~30% dropout in the single-cell reference
- Beta cell proportion vs HbA1c linear model in the 89 Fadista/GSE50244 samples reproduces p-value 0.00126 and a 3.07% ± 2.49% beta proportion drop per 0.5% HbA1c increase
- Cross-species application (mouse GSE107585 reference on rat GSE56743 tubule segments): samples from each renal segment cluster together by pairwise proportion correlation, and dominant deconvolved cell type matches the segment's anatomical position (e.g., DCT cells from the DCT segment)
- Collinear closely related cell types are grouped before estimation (e.g., 13 kidney cell types cut into 4 groups) rather than estimated independently

## Limitations

- Correlation in gene expression can lead to collinearity, making it difficult to reliably estimate proportions of closely related or rare cell types
- Assumptions A1–A3 allow recovery of trends of cell type proportion change across subjects but not absolute cell type proportions
- Most scRNA-seq data do not reflect true cell type proportions in intact tissues due to biased cell dissociation; they provide cell type-specific gene expression only
- Cell type proportions cannot be estimated with TPM as input; RPKM requires external cell type-specific total RNA abundance estimates

## Evidence

- [discussion] W-NNLS without marker preselection: "MuSiC is a weighted non-negative least squares regression (W-NNLS), which does not require pre-selected marker genes."
- [discussion] TPM not usable: "However, cell type proportions cannot be estimated with TPM as the input."
- [discussion] Collinearity limitation: "Correlation in gene expression can lead to collinearity, making it difficult to reliably estimate cell type proportions, especially for less frequent and rare cell types."
- [methods] music_prop function: "The cell type proportions is estimated by function `music_prop`"
- [methods] Benchmark construction: "Benchmark dataset is constructed by summing up single cell data from `XinT2D.eset`."
- [methods] Clustered estimation: "Here we cut 13 cell types into 4 groups"
- [readme] Installation: "devtools::install_github('xuranw/MuSiC')"
- [readme] SingleCellExperiment support: [evidence span withheld — anchor status: external_doc]
