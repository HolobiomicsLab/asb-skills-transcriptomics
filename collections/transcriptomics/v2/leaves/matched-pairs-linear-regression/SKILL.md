---
name: matched-pairs-linear-regression
description: Use when a subset of individuals has both bulk RNA-seq and single-cell/scRNA-seq
  measurements (use.overlap=TRUE), so gene-specific bulk transformations can be learned
  from matched pairs before decomposition.
when_to_use_negative:
- No individuals have matched bulk and single-cell measurements — use the semisupervised
  shrinkage-variance transform (SemisupervisedTransformBulk, use.overlap=FALSE) instead
  of the supervised fit.
- Only a set of known marker genes is available and no single-cell reference exists
  — use marker-based decomposition (BisqueRNA::MarkerBasedDecomposition) instead.
- Very few individuals have single-cell data — the article warns the reference profile
  and gene-specific transformations may become unreliable.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: BisqueRNA
  role: Reference implementation of SupervisedTransformBulk and the ReferenceBasedDecomposition
    pipeline being reconstructed
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Provides ExpressionSet containers for bulk and single-cell inputs
- name: limSolve
  role: Solves the downstream NNLS decomposition with non-negativity and sum-to-one
    constraints on the transformed reference
- name: Bisque
  role: The method whose supervised gene-specific bulk transformation is implemented
  repo: github.com/cozygene/bisque
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:b7e36219dbea0612a8031cd22de448328b4714778ffd2eab9d65b33f484c74ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biobase
  - Bisque
  - BisqueRNA
  - limSolve
  - stats
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# matched-pairs-linear-regression

## Summary

Per-gene supervised linear regression on matched bulk/single-cell samples: regress bulk gene expression on single-cell-derived cell-type proportions across the overlapping individuals to learn gene-specific coefficients that map the single-cell reference profile to the bulk expression scale, correcting technical biases between the two technologies.

## When to use

Use when a subset of individuals has both bulk RNA-seq and single-cell/scRNA-seq measurements (use.overlap=TRUE), so gene-specific bulk transformations can be learned from matched pairs before decomposition.

## When NOT to use

- No individuals have matched bulk and single-cell measurements — use the semisupervised shrinkage-variance transform (SemisupervisedTransformBulk, use.overlap=FALSE) instead of the supervised fit.
- Only a set of known marker genes is available and no single-cell reference exists — use marker-based decomposition (BisqueRNA::MarkerBasedDecomposition) instead.
- Very few individuals have single-cell data — the article warns the reference profile and gene-specific transformations may become unreliable.

## Inputs

- Bulk RNA-seq expression matrix as a Biobase ExpressionSet (bulk.eset)
- Single-cell/scRNA-seq count matrix with cell-type and individual labels as an ExpressionSet (sc.eset)
- Set of individuals with both bulk and single-cell measurements (overlapping samples via GetOverlappingSamples)
- Vector of cell-type labels (e.g., Neurons, Astrocytes, Oligodendrocytes, Microglia, Endothelial Cells)

## Outputs

- Per-gene regression coefficients mapping the sc reference to the bulk expression scale
- Transformed (bulk-scale) reference expression profile used for NNLS decomposition
- Downstream cell-type proportion estimates per bulk sample (res$bulk.props)

## How to apply

First compute single-cell-derived cell-type proportions per individual (CalculateSCCellProportions logic) and the single-cell reference profile (GenerateSCReference, averaging per-individual read-count abundances within each cell type). For each gene, collect the bulk expression vector (Y) and the sc-derived proportion matrix (X) across the overlapping individuals, then fit an ordinary linear regression of bulk expression on the cell-type proportions to estimate gene-specific coefficients (SupervisedTransformBulk(gene, Y.train, X.train, X.pred)). Apply the fitted transform to the sc reference to put it on the bulk scale. This corrects gene-specific biases introduced by differences between single-cell and bulk chemistries (mRNA capture, library chemistry, nuclear pre-mRNA in snRNA-seq), which otherwise break the proportionality assumptions of regression-based decomposition. The transformed reference is then used in the NNLS decomposition of each bulk sample. If matched samples are unavailable, fall back to the semisupervised shrinkage-variance variant (SemisupervisedTransformBulk, the use.overlap=FALSE path).

## Related tools

- **BisqueRNA** (Reference implementation of SupervisedTransformBulk and the ReferenceBasedDecomposition pipeline being reconstructed) — github.com/cozygene/bisque
- **Biobase** (Provides ExpressionSet containers for bulk and single-cell inputs)
- **limSolve** (Solves the downstream NNLS decomposition with non-negativity and sum-to-one constraints on the transformed reference)
- **Bisque** (The method whose supervised gene-specific bulk transformation is implemented) — github.com/cozygene/bisque

## Examples

```
res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)
```

## Evaluation signals

- Cross-check the from-scratch implementation against BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE) as a sanity reference
- Correlate estimated proportions with true simulated proportions (SimulateData with n.ind=10, n.genes=100, n.cells=500, avg.props=c(.5,.2,.2,.07,.03))
- Verify fitted coefficients reproduce bulk-scale predictions when applied to the sc reference on the training individuals
- Confirm NNLS solutions are non-negative and sum to one per bulk sample
- Write estimates to decomposition_estimates.csv and check per-sample proportions match the expected cell-type schema

## Limitations

- If the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable.
- A key assumption of the transformation framework is that single-cell-based estimates of cell proportions accurately reflect the true proportions to estimate; accuracy may decrease if single-cell-captured proportions differ significantly from true physiological distributions.
- Bisque provides estimates of cell proportions reported by the single-cell technology used to generate the reference data — caution advised when there is known significant bias, such as severe underrepresentation of a cell type of interest.

## Evidence

- [methods] Supervised transform learns gene-specific bulk corrections from matched samples: "SupervisedTransformBulk(gene, Y.train, X.train, X.pred)"
- [methods] Semisupervised fallback when matched samples are not used: "SemisupervisedTransformBulk(gene, Y.train, X.pred)"
- [abstract] Rationale for gene-specific transformations correcting technical variation: "These transformations significantly improve decomposition performance compared to existing methods when there is significant technical variation in the generation of the reference profile and"
- [intro] Biological rationale for the transform from the introduction: "these differences may introduce gene-specific biases that break down the correlation between cell-type-specific and bulk tissue measurements"
- [readme] README statement that matched samples improve accuracy for reference-based mode: "While we don't explicitly require matched samples, we expect having samples with both single-cell and bulk expression measured will provide more accurate results."
