---
name: single-cell-cluster-label-validation
description: Use when a scRNA-seq clustering method (e.g., SSRE) has produced cell
  type labels and you need to confirm the labels are biologically meaningful by recovering
  known marker genes via differential expression testing.
when_to_use_negative:
- The cluster labels have not yet been produced or cannot be aligned cell-by-cell
  to the expression matrix rows
- The goal is evaluating clustering accuracy against ground-truth labels (use NMI/Purity/ARI
  comparisons instead)
- The gene expression matrix is not raw/log-normalized count data suited to rank-based
  testing across cluster groups
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: R (kruskal.test)
  role: Per-gene Kruskal-Wallis rank-sum test using SSRE-inferred labels as the grouping
    factor to obtain DEG p-values
- name: SSRE
  role: Produces the inferred cluster labels and the gene expression pipeline whose
    downstream DEG extension is being validated
  repo: https://github.com/CSUBioGroup/SSRE
- name: MATLAB
  role: Runs the SSRE implementation that generates the cluster labels used as the
    grouping factor in the DEG test
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:37f5460bbff64b4b32cdd6092daf16577ed5260af0ef9b1c0714446b4c5a88a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - Python
  - R (kruskal.test)
  - SSRE
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-cluster-label-validation

## Summary

Validates computationally inferred single-cell cluster labels (e.g., SSRE-inferred labels on the Yan scRNA-seq dataset) by testing whether they support downstream differential expression analysis, using per-gene Kruskal-Wallis rank-sum tests with a P < 0.01 significance threshold.

## When to use

Use when a scRNA-seq clustering method (e.g., SSRE) has produced cell type labels and you need to confirm the labels are biologically meaningful by recovering known marker genes via differential expression testing.

## When NOT to use

- The cluster labels have not yet been produced or cannot be aligned cell-by-cell to the expression matrix rows
- The goal is evaluating clustering accuracy against ground-truth labels (use NMI/Purity/ARI comparisons instead)
- The gene expression matrix is not raw/log-normalized count data suited to rank-based testing across cluster groups

## Inputs

- scRNA-seq gene expression matrix (cells x genes) for the Yan dataset
- SSRE-inferred cell type cluster labels per cell (aligned to expression matrix rows)
- Optional: list of previously reported marker genes for validation (NLRP11, NLRP4, CLEC10A, H1FOO, GDF9, OTX2, ACCSL, TUBB8, TUBB4Q)

## Outputs

- Per-gene Kruskal-Wallis p-values and test statistics
- Top-50 differentially expressed genes table with p-values and group assignments (top50_de_genes_yan.csv)
- Marker recovery report listing which known markers were/were not recovered at P < 0.01

## How to apply

Load the scRNA-seq gene expression matrix and the inferred cluster labels (SSRE-inferred labels for the Yan dataset), aligning cells between the two tables. For each gene, run a Kruskal-Wallis rank-sum test in R (kruskal.test) with the SSRE-inferred labels as the grouping factor, obtaining one p-value per gene. Apply the significance threshold P < 0.01 and rank significant genes by test statistic/p-value to extract the top-50 differentially expressed genes, writing them (with p-values and group assignments) to a file such as top50_de_genes_yan.csv. Validate the result by checking that the recovered marker set includes the previously reported markers NLRP11, NLRP4, CLEC10A, H1FOO, GDF9, OTX2, ACCSL, TUBB8, and TUBB4Q reported in Figure S2 of the SSRE paper, and report any markers not recovered at P < 0.01. Rationale: SSRE is explicitly extensible to differential gene expression analysis, so a clustering that supports recovery of known markers is validated.

## Related tools

- **R (kruskal.test)** (Per-gene Kruskal-Wallis rank-sum test using SSRE-inferred labels as the grouping factor to obtain DEG p-values)
- **SSRE** (Produces the inferred cluster labels and the gene expression pipeline whose downstream DEG extension is being validated) — https://github.com/CSUBioGroup/SSRE
- **MATLAB** (Runs the SSRE implementation that generates the cluster labels used as the grouping factor in the DEG test) — https://github.com/CSUBioGroup/SSRE

## Examples

```
pvals <- apply(expr_matrix, 1, function(g) kruskal.test(g ~ factor(ssre_labels))$p.value); top50 <- head(sort(pvals[pvals < 0.01]), 50)
```

## Evaluation signals

- The recovered top-50 DEG list includes all nine reported markers: NLRP11, NLRP4, CLEC10A, H1FOO, GDF9, OTX2, ACCSL, TUBB8, and TUBB4Q
- All reported genes pass the P < 0.01 significance threshold; any unrecovered marker is explicitly reported
- Output table top50_de_genes_yan.csv contains gene names, p-values, and group assignments and has exactly 50 rows
- The number of Kruskal-Wallis tests performed equals the number of genes in the expression matrix (one p-value per gene)
- Cell identifiers in the labels vector match one-to-one with rows of the expression matrix after alignment

## Limitations

- SSRE is relatively time-consuming on large-scale datasets, so generating labels for the DEG step may be slow (parallel computing proposed as future acceleration)
- Validation depends on prior knowledge of marker genes (Figure S2 of the paper); without such a reference set, recovery cannot be checked
- Markers not recovered at P < 0.01 must be reported rather than assumed present — the threshold can miss weakly expressed markers

## Evidence

- [abstract] SSRE extension to DEG analysis: "In addition, SSRE can be extended to visualization of scRNA-seq data and identification of differentially expressed genes."
- [intro] Learned similarities reveal block structure: "The experimental results show that when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably."
- [intro] Two-step procedure rationale: "In order to reduce the effect of irrelevant features and improve the overall accuracy, SSRE designs a two-step procedure, i.e., 1) adaptive gene selection and 2) similarity enhancement."
- [discussion] Runtime limitation on large-scale data: "One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework"
