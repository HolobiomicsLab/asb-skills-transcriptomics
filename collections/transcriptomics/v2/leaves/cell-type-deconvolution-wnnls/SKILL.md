---
name: cell-type-deconvolution-wnnls
description: Use when estimating cell type proportions from bulk RNA-seq read counts
  with a multi-subject single-cell reference, especially for closely related cell
  types and without pre-selected marker genes.
when_to_use_negative:
- Bulk input is TPM-normalized — cell type proportions cannot be estimated with TPM
  as the input; read counts are required.
- Bulk data are generated from samples with multiple clinical conditions differing
  from the single-cell reference — use MuSiC2 instead of the plain W-NNLS fit.
- No multi-subject scRNA-seq reference with per-subject replication is available —
  the gene weighting requires cross-subject statistics (M.theta, Sigma).
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MuSiC
  role: R package providing music_basis (Disgn.mtx, M.theta, Sigma) and music_prop,
    which fits the W-NNLS estimator with weights initialized by NNLS and refined iteratively
  repo: https://github.com/xuranw/MuSiC
- name: R
  role: Runtime environment; MuSiC is implemented as an R package installed from GitHub
    via devtools::install_github('xuranw/MuSiC')
- name: Biobase
  role: Supplies the ExpressionSet containers holding bulk and single-cell read counts
    passed to music_basis and music_prop
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-08023-x
    title: Bulk tissue cell type deconvolution with multi-subject single-cell expression
      reference
  source_content_hash: sha256:4efbe41d6c30c73fe35ad7cc2fabf022b4e8080ce70fc82ba4d63260d4e99f67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xuranw/MuSiC
  tools:
  - Biobase
  - MuSiC
  - R
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cell-type-deconvolution-wnnls

## Summary

MuSiC-style cell type deconvolution: a weighted non-negative least squares (W-NNLS) estimator that uses cross-subject scRNA-seq statistics to weight genes and estimate cell type proportions in bulk RNA-seq read counts, without pre-selected marker genes.

## When to use

Use when estimating cell type proportions from bulk RNA-seq read counts with a multi-subject single-cell reference, especially for closely related cell types and without pre-selected marker genes.

## When NOT to use

- Bulk input is TPM-normalized — cell type proportions cannot be estimated with TPM as the input; read counts are required.
- Bulk data are generated from samples with multiple clinical conditions differing from the single-cell reference — use MuSiC2 instead of the plain W-NNLS fit.
- No multi-subject scRNA-seq reference with per-subject replication is available — the gene weighting requires cross-subject statistics (M.theta, Sigma).

## Inputs

- Bulk RNA-seq read-count ExpressionSet (e.g., GSE50244bulkeset.rds from GEO GSE50244)
- Multi-subject scRNA-seq reference ExpressionSet with cellType and sampleID phenotype columns (e.g., EMTABesethealthy.rds from E-MTAB-5060/5061)
- Character vector of cell types to deconvolve (select.ct, e.g., alpha, beta, delta, gamma, acinar, ductal)

## Outputs

- Per-sample cell type proportion matrix returned as 'Est.prop.weighted' by music_prop (89 bulk samples x 6 cell types in the vignette workflow)
- Design matrix Disgn.mtx, cross-subject mean of relative abundance M.theta, and cross-subject variance Sigma from music_basis

## How to apply

Load the bulk read-count ExpressionSet (e.g., GSE50244bulkeset.rds from GEO GSE50244) and a multi-subject scRNA-seq reference ExpressionSet (e.g., EMTABesethealthy.rds from E-MTAB-5060/5061) with readRDS via a gzcon URL connection. Summarize the reference per cell type and subject using music_basis(clusters='cellType', samples='sampleID', select.ct=c('alpha','beta','delta','gamma','acinar','ductal')) to obtain the design matrix Disgn.mtx, the cross-subject mean of relative abundance M.theta, and the cross-subject variance Sigma. Run music_prop on the bulk eset against the single-cell eset with the same clusters/samples/select.ct: MuSiC weights genes by cross-subject variation — up-weighting low-variance (informative) genes and down-weighting high-variance (non-informative) genes — initializes the weights by NNLS, then iteratively refines the W-NNLS fit. Extract the 'Est.prop.weighted' result component, normalize the estimated proportions over the included cell types, and write the per-sample proportion matrix (e.g., music_prop_GSE50244.csv). Input must be raw read counts; TPM cannot be used, and RPKM is only usable if cell type-specific total RNA abundance estimates are supplied from another dataset.

## Related tools

- **MuSiC** (R package providing music_basis (Disgn.mtx, M.theta, Sigma) and music_prop, which fits the W-NNLS estimator with weights initialized by NNLS and refined iteratively) — https://github.com/xuranw/MuSiC
- **R** (Runtime environment; MuSiC is implemented as an R package installed from GitHub via devtools::install_github('xuranw/MuSiC'))
- **Biobase** (Supplies the ExpressionSet containers holding bulk and single-cell read counts passed to music_basis and music_prop)

## Examples

```
prop <- music_prop(bulk.eset = GSE50244bulkeset, sc.eset = EMTABesethealthy, clusters = 'cellType', samples = 'sampleID', select.ct = c('alpha','beta','delta','gamma','acinar','ductal'))
```

## Evaluation signals

- Output file music_prop_GSE50244.csv exists with 89 rows (one per bulk sample) and 6 cell-type columns
- All estimated proportions are non-negative
- Each row of estimated proportions sums to 1 within numerical tolerance
- The music_prop result object contains an 'Est.prop.weighted' component
- In benchmark evaluation (Eval_multi against prop.real from bulk_construct via relative.ab), the weighted estimates track true proportions better than the all-gene NNLS baseline

## Limitations

- Cell type proportions cannot be estimated with TPM as the input; MuSiC can utilize RPKM only if estimates of cell type-specific total RNA abundance are provided (e.g., estimated from another dataset).
- Correlation in gene expression can lead to collinearity, making it difficult to reliably estimate cell type proportions, especially for less frequent and rare cell types.
- Under assumptions A1-A3 only the trend of cell type proportion change across subjects can be recovered, not absolute cell type proportions; when the efficiency assumption is not satisfied, only the fraction of RNA molecules from each cell type can be estimated.

## Evidence

- [abstract] By appropriate weighting of genes showing cross-subject and cross-cell consistency, MuSiC enables the transfer of cell type-specific gene expression information from one dataset to another.: "By appropriate weighting of genes showing cross-subject and cross-cell consistency, MuSiC enables the transfer of cell type-specific gene expression information from one dataset to another."
- [methods] `music_basis` is the function to produce the design matrix (`Disgn.mtx`), cross-subject mean of relative abundance (`M.theta`): "`music_basis` is the function to produce the design matrix (`Disgn.mtx`), cross-subject mean of relative abundance (`M.theta`)"
- [methods] The cell type proportions is estimated by function `music_prop`: "The cell type proportions is estimated by function `music_prop`"
- [discussion] MuSiC is a weighted non-negative least squares regression (W-NNLS), which does not require pre-selected marker genes.: "MuSiC is a weighted non-negative least squares regression (W-NNLS), which does not require pre-selected marker genes."
- [discussion] However, cell type proportions cannot be estimated with TPM as the input.: "However, cell type proportions cannot be estimated with TPM as the input."
- [readme] The original release of `MuSiC` is a deconvolution method that utilizes cross-subject scRNA-seq to estimate cell type proportions in bulk RNA-seq data.: [evidence span withheld — anchor status: external_doc]
