---
name: kruskal-wallis-hypothesis-testing-scrna-seq
description: Use when you have a scRNA-seq gene expression matrix plus non-parametric
  cluster labels (e.g., from SSRE clustering) and need to detect differentially expressed
  genes across those clusters without assuming normality.
when_to_use_negative:
- No group labels are available for cells — the Kruskal-Wallis test requires a grouping
  factor (e.g., SSRE-inferred labels)
- Expression data violate the independence assumptions or contain unaligned cells
  between the expression matrix and the label table without alignment
- Goal is clustering itself rather than downstream differential expression — use SSRE/spectral
  clustering instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: R (kruskal.test)
  role: Runs the per-gene Kruskal-Wallis rank-sum test with SSRE-inferred labels as
    the grouping factor
- name: MATLAB
  role: Used to produce the SSRE-inferred cluster labels that feed the DEG test
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: SSRE implementation used to generate clustering labels for downstream DEG
    analysis
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:dc0fa71bcddc6d60080c7a8ce1f6ac0fc5a7360b382f2c373b9211d12aaffbff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - Python
  - R (kruskal.test)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# kruskal-wallis-hypothesis-testing-scrna-seq

## Summary

Per-gene Kruskal-Wallis rank-sum testing of single-cell RNA-seq expression matrices grouped by SSRE-inferred cluster labels, used to identify differentially expressed genes as demonstrated on the Yan dataset, where markers such as NLRP11, GDF9, OTX2, and TUBB8 were recovered at P < 0.01.

## When to use

Use when you have a scRNA-seq gene expression matrix plus non-parametric cluster labels (e.g., from SSRE clustering) and need to detect differentially expressed genes across those clusters without assuming normality.

## When NOT to use

- No group labels are available for cells — the Kruskal-Wallis test requires a grouping factor (e.g., SSRE-inferred labels)
- Expression data violate the independence assumptions or contain unaligned cells between the expression matrix and the label table without alignment
- Goal is clustering itself rather than downstream differential expression — use SSRE/spectral clustering instead

## Inputs

- scRNA-seq gene expression matrix (e.g., Yan dataset, cells x genes)
- SSRE-inferred cluster/cell type labels per cell (aligned to the expression matrix rows)

## Outputs

- Per-gene Kruskal-Wallis p-values and test statistics
- Top-50 differentially expressed gene list with p-values and group assignments (top50_de_genes_yan.csv)
- Validation report comparing recovered markers against reported markers at P < 0.01

## How to apply

Load the scRNA-seq gene expression matrix and the SSRE-inferred cell type labels, aligning cells between the two tables. For each gene, run the Kruskal-Wallis rank-sum test in R (kruskal.test) with the SSRE-inferred labels as the grouping factor, yielding one p-value per gene. Apply the significance threshold P < 0.01 and rank significant genes by test statistic/p-value to extract the top-50 differentially expressed genes, writing them with p-values and group assignments to top50_de_genes_yan.csv. Validate by checking the recovered marker set against previously reported markers (NLRP11, NLRP4, CLEC10A, H1FOO, GDF9, OTX2, ACCSL, TUBB8, TUBB4Q from Figure S2 of the SSRE paper) and report any markers not recovered at P < 0.01.

## Related tools

- **R (kruskal.test)** (Runs the per-gene Kruskal-Wallis rank-sum test with SSRE-inferred labels as the grouping factor)
- **MATLAB** (Used to produce the SSRE-inferred cluster labels that feed the DEG test) — https://github.com/CSUBioGroup/SSRE
- **Python** (SSRE implementation used to generate clustering labels for downstream DEG analysis) — https://github.com/CSUBioGroup/SSRE

## Examples

```
kw <- apply(expr_matrix, 1, function(g) kruskal.test(g, factor(ssre_labels))); padj <- p.adjust(kw$p.value, "BH"); top50 <- names(sort(which(kw$p.value < 0.01))[1:50])
```

## Evaluation signals

- Recovered marker set includes previously reported markers NLRP11, NLRP4, CLEC10A, H1FOO, GDF9, OTX2, ACCSL, TUBB8, and TUBB4Q (Figure S2 of the SSRE paper)
- All reported genes pass the P < 0.01 significance threshold
- Cells in the expression matrix and label table are correctly aligned (one label per cell)
- Exactly one p-value per gene, ranked by test statistic/p-value, with top-50 written to top50_de_genes_yan.csv

## Limitations

- Markers not recovered at P < 0.01 must be explicitly reported — the test may miss some reported markers
- SSRE itself is relatively time-consuming on large-scale datasets, which affects upstream label generation for large scRNA-seq data
- Kruskal-Wallis is a rank-based test on cluster groupings; results depend on the quality of the upstream SSRE-inferred labels

## Evidence

- [other] SSRE's DEG analysis on the Yan dataset identified previously reported marker genes NLRP11, NLRP4, CLEC10A, H1FOO, GDF9, OTX2, ACCSL, TUBB8, and TUBB4Q: [evidence span withheld — anchor status: not_found]
- [abstract] In addition, SSRE can be extended to visualization of scRNA-seq data and identification of differentially expressed genes.: "In addition, SSRE can be extended to visualization of scRNA-seq data and identification of differentially expressed genes."
- [other] For each gene, run the Kruskal-Wallis rank-sum test in R (kruskal.test) with the SSRE-inferred labels as the grouping factor: [evidence span withheld — anchor status: not_found]
- [supplementary] The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE.: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
