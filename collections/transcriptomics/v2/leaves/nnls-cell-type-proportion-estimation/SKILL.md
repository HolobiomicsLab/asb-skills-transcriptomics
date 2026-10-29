---
name: nnls-cell-type-proportion-estimation
description: Use when bulk RNA-seq read counts must be deconvolved into known cell
  type proportions using a single-cell reference, and an unweighted NNLS baseline
  (Est.prop.allgene) is needed to benchmark against weighted estimators like MuSiC.
when_to_use_negative:
- When the target cell types are closely related and collinear (e.g., PT and DCT in
  mouse kidney); use the tree-guided two-stage music_prop.cluster instead, since plain
  NNLS has limited power to resolve similar cell types
- When the input bulk matrix is TPM-normalized — cell type proportions cannot be estimated
  with TPM as the input
- When the goal is deconvolution of multi-condition bulk data whose clinical conditions
  differ from the scRNA-seq reference; MuSiC2 is the indicated iterative method
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MuSiC
  role: Provides music_prop, whose Est.prop.allgene output is the all-gene NNLS estimator
    and whose iterative gene weights are initialized by NNLS before W-NNLS
  repo: https://github.com/xuranw/MuSiC
- name: BSEQ-sc
  role: Marker-gene/support-vector-regression-based extension of CIBERSORT that uses
    NNLS-style deconvolution with scRNA-seq references; benchmarked alongside NNLS
    on the artificial bulk dataset
- name: R
  role: Implementation environment; MuSiC is an R package installed from GitHub via
    devtools
  repo: https://github.com/xuranw/MuSiC
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-08023-x
    title: Bulk tissue cell type deconvolution with multi-subject single-cell expression
      reference
  source_content_hash: sha256:affa0975ecf80bb67c27d21527b94e820d3aff79b4ee23f2c2b64fe88d7a3e2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xuranw/MuSiC
  tools:
  - BSEQ-sc
  - MuSiC
  - R
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# nnls-cell-type-proportion-estimation

## Summary

Estimates cell type proportions in bulk RNA-seq by ordinary (all-gene) non-negative least squares against a design matrix built from a multi-subject scRNA-seq reference; in MuSiC it is the baseline estimator returned by music_prop (Est.prop.allgene) and the initialization step for the gene-weighted W-NNLS procedure.

## When to use

Use when bulk RNA-seq read counts must be deconvolved into known cell type proportions using a single-cell reference, and an unweighted NNLS baseline (Est.prop.allgene) is needed to benchmark against weighted estimators like MuSiC.

## When NOT to use

- When the target cell types are closely related and collinear (e.g., PT and DCT in mouse kidney); use the tree-guided two-stage music_prop.cluster instead, since plain NNLS has limited power to resolve similar cell types
- When the input bulk matrix is TPM-normalized — cell type proportions cannot be estimated with TPM as the input
- When the goal is deconvolution of multi-condition bulk data whose clinical conditions differ from the scRNA-seq reference; MuSiC2 is the indicated iterative method

## Inputs

- Bulk RNA-seq read counts matrix (e.g., artificial bulk from bulk_construct on XinT2Deset.rds)
- Single-cell reference ExpressionSet (e.g., EMTABesethealthy.rds, E-MTAB-5061) with cell type cluster labels (cellType) and sample identifiers (sampleID)
- List of target cell types (select.ct, e.g., alpha, beta, delta, gamma)

## Outputs

- Est.prop.allgene matrix of per-sample cell type proportion estimates from all-gene NNLS
- Comparison metrics table (RMSD, mAD, Pearson R) from Eval_multi, e.g., benchmark_evaluation_metrics.csv
- Optionally a real-vs-estimated proportion heatmap (e.g., benchmark.jpeg via Prop_comp_multi)

## How to apply

Construct a scRNA-seq reference ExpressionSet (e.g., EMTABesethealthy.rds from E-MTAB-5061 with cellType and sampleID columns) and a bulk counts matrix (e.g., artificial bulk built from XinT2Deset.rds via bulk_construct with clusters='cellType', samples='SubjectName'). Run music_prop with clusters='cellType', samples='sampleID', select.ct=c('alpha','beta','delta','gamma') to obtain both the all-gene NNLS estimates (Est.prop.allgene) and the weighted MuSiC estimates (Est.prop.weighted); NNLS is also the initialization for the iterative gene-weight estimation in W-NNLS. Rationale: NNLS requires no pre-selected marker genes, making it a marker-gene-threshold-free baseline, but it treats all genes equally and thus ignores cross-subject heterogeneity and within-cell type stochasticity. Evaluate against true proportions (relative.ab(..., by.col=FALSE)) using Eval_multi with RMSD, mAD, and Pearson correlation; on the Segerstolpe-based artificial bulk benchmark NNLS yields RMSD 0.17161 and R 0.8159, underperforming MuSiC (RMSD 0.09881, R 0.9378).

## Related tools

- **MuSiC** (Provides music_prop, whose Est.prop.allgene output is the all-gene NNLS estimator and whose iterative gene weights are initialized by NNLS before W-NNLS) — https://github.com/xuranw/MuSiC
- **BSEQ-sc** (Marker-gene/support-vector-regression-based extension of CIBERSORT that uses NNLS-style deconvolution with scRNA-seq references; benchmarked alongside NNLS on the artificial bulk dataset)
- **R** (Implementation environment; MuSiC is an R package installed from GitHub via devtools) — https://github.com/xuranw/MuSiC

## Examples

```
Est.prop.Xin = music_prop(bulk.eset = XinT2D.construct.full$Bulk.counts, sc.eset = EMTAB.eset, clusters = 'cellType', samples = 'sampleID', select.ct = c('alpha','beta','delta','gamma'))
```

## Evaluation signals

- Estimated proportions are non-negative and, per sample, consistent with a simplex (proportions of the select.ct types)
- On the Segerstolpe artificial bulk benchmark, NNLS metrics reproduce the reported values within rounding: RMSD 0.17161, Pearson R 0.8159
- NNLS RMSD/R are worse than the weighted MuSiC estimates (RMSD 0.09881, mAD 0.06357, R 0.9378), confirming NNLS as the weaker baseline when bulk and reference come from the same study
- Est.prop.allgene rows span exactly the selected cell types (e.g., alpha, beta, delta, gamma) and samples match the bulk_construct subject names
- Eval_multi output table round-trips to benchmark_evaluation_metrics.csv without dropped methods

## Limitations

- All-gene NNLS ignores cross-subject heterogeneity in cell type-specific gene expression and within-cell type stochasticity, reducing accuracy relative to weighted MuSiC
- Correlation in gene expression among closely related cell types causes collinearity, making NNLS proportions unreliable, especially for less frequent and rare cell types
- Cell type proportions cannot be estimated with TPM as the input; RPKM is usable only if cell type specific total RNA abundance estimates are provided
- Under assumptions A1-A3 only trends of proportion change across subjects are recoverable, not absolute cell type proportions

## Evidence

- [methods] NNLS baseline within music_prop: "The cell type proportions is estimated by function `music_prop`"
- [discussion] NNLS initialization of W-NNLS: "MuSiC is a weighted non-negative least squares regression (W-NNLS), which does not require pre-selected marker genes."
- [intro] Existing-method limitation motivating NNLS baseline framing: "These methods rely on pre-selected cell type-specific marker genes, and thus are sensitive to the choice of significance threshold."
- [readme] Scope of the estimator from README: [evidence span withheld — anchor status: external_doc]
- [methods] Evaluation function used to score NNLS against true proportions: "Eval_multi(prop.real = data.matrix(XinT2D.construct.full$prop.real),"
