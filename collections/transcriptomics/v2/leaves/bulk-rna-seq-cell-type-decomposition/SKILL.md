---
name: bulk-rna-seq-cell-type-decomposition
description: Use when bulk RNA-seq/snRNA-seq data from the same tissue must be decomposed
  into cell-type proportions, especially when single-cell vs bulk technical biases
  are expected or when only marker genes (not a reference profile) are available.
when_to_use_negative:
- When few individuals have single-cell data, since the reference profile and gene-specific
  transformations may become unreliable
- When single-cell-captured cell-type proportions differ significantly from true physiological
  distributions (e.g., severe underrepresentation of a cell type)
- When cross-cell-type comparisons of absolute proportions are needed from marker-based
  mode, since BisqueMarker yields relative abundances that cannot be compared across
  cell types
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: BisqueRNA
  role: Core R package implementing ReferenceBasedDecomposition and MarkerBasedDecomposition
  repo: https://github.com/cozygene/bisque
- name: Biobase
  role: Provides the ExpressionSet format required for bulk and single-cell inputs
- name: CIBERSORT
  role: Comparator SVM-regression-based decomposition method
- name: BSEQ-sc
  role: Comparator that generates a single-cell reference for the CIBERSORT model
- name: MuSiC
  role: Comparator using weighted non-negative least-squares regression for decomposition
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:ea2728b58d150690474553d7f98bef08fd404dc9b8d795ad225c7c9bc5e4f311
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cozygene/bisque
  tools:
  - BSEQ-sc
  - Biobase
  - Bisque
  - BisqueRNA
  - CIBERSORT
  - MuSiC
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 20
---

# bulk-rna-seq-cell-type-decomposition

## Summary

Estimate cell-type proportions in bulk RNA-seq data using Bisque, which learns gene-specific transformations between a snRNA-seq/scRNA-seq reference and bulk expression, or BisqueMarker when only marker genes are available. Outperforms CIBERSORT/BSEQ-sc and MuSiC under significant technical variation.

## When to use

Use when bulk RNA-seq/snRNA-seq data from the same tissue must be decomposed into cell-type proportions, especially when single-cell vs bulk technical biases are expected or when only marker genes (not a reference profile) are available.

## When NOT to use

- When few individuals have single-cell data, since the reference profile and gene-specific transformations may become unreliable
- When single-cell-captured cell-type proportions differ significantly from true physiological distributions (e.g., severe underrepresentation of a cell type)
- When cross-cell-type comparisons of absolute proportions are needed from marker-based mode, since BisqueMarker yields relative abundances that cannot be compared across cell types

## Inputs

- Bulk expression matrix converted to a Biobase ExpressionSet (bulk.eset)
- scRNA-seq or snRNA-seq count matrix with cell-type and individual labels as a Biobase ExpressionSet (sc.eset), or a Seurat object via BisqueRNA::SeuratToExpressionSet
- Marker gene data frame with gene, cluster, and log-fold-change weight columns (for marker-based mode)

## Outputs

- Estimated cell-type proportions per bulk sample (res$bulk.props)
- Correlation (R) and RMSD metrics against true proportions when validation labels are available

## How to apply

Convert both the bulk expression matrix and the scRNA-seq/snRNA-seq counts (with cell-type and individual labels) to Biobase ExpressionSet objects, then run BisqueRNA::ReferenceBasedDecomposition with use.overlap=TRUE so gene-specific linear transformations are learned from overlapping samples. Bisque addresses gene-specific biases that arise from differences in technologies (e.g., nuclear pre-mRNA in snRNA-seq) that break direct proportionality assumptions used by CIBERSORT/BSEQ-sc (SVM regression) and MuSiC (weighted NNLS). When no single-cell reference exists, use BisqueRNA::MarkerBasedDecomposition: restrict markers via GetUniqueMarkers, weight columns by avg_logFC, run a weighted PCA per cell type after correcting for the first global expression PC, and treat the first PC as relative abundance. Simulate bias robustness checks by drawing gene-specific linear coefficient/intercept perturbations from a half-normal distribution over sigma levels (e.g., 0, 5, 10, 20). Validate by correlating res$bulk.props with true proportions (e.g., R ≈ 0.85 and RMSD ≈ 0.07 at sigma = 20 in adipose simulations) and confirm positive correlation of marker-based estimates with scaled true proportions.

## Related tools

- **BisqueRNA** (Core R package implementing ReferenceBasedDecomposition and MarkerBasedDecomposition) — https://github.com/cozygene/bisque
- **Biobase** (Provides the ExpressionSet format required for bulk and single-cell inputs)
- **CIBERSORT** (Comparator SVM-regression-based decomposition method)
- **BSEQ-sc** (Comparator that generates a single-cell reference for the CIBERSORT model)
- **MuSiC** (Comparator using weighted non-negative least-squares regression for decomposition)

## Examples

```
res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)
```

## Evaluation signals

- res$bulk.props is populated for all samples and cell types
- Correlation of estimates with true proportions is high (e.g., R ≈ 0.85 at sigma = 20 in adipose bias simulations)
- RMSD against true.props is low (e.g., ≈ 0.07 at the highest bias level)
- Bisque outperforms or matches CIBERSORT/BSEQ-sc and MuSiC on the same biased bulk inputs
- Marker-based estimates show positive correlation with scaled true proportions, t(scale(t(true.props)))

## Limitations

- Reference profile and gene-specific transformations may become unreliable when few individuals have single-cell data
- Key assumption that single-cell-based estimates of cell proportions accurately reflect the true proportions to be estimated
- Accuracy may decrease if single-cell-captured cell type proportions differ significantly from true physiological distributions
- BisqueMarker estimates relative differences in abundances that cannot be compared across cell types

## Evidence

- [abstract] Bisque learns gene-specific bulk expression transformations: "These transformations significantly improve decomposition performance compared to existing methods when there is significant technical variation"
- [methods] ExpressionSet input requirement: "Bisque requires expression data in the ExpressionSet format from the Biobase package."
- [methods] Reference-based decomposition call: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [readme] Marker-based mode description: "This method utilizes marker genes alone to decompose bulk expression when a reference profile is not available."
- [intro] Bias in single-cell vs bulk technologies: "these differences may introduce gene-specific biases that break down the correlation between cell-type-specific and bulk tissue measurements"
