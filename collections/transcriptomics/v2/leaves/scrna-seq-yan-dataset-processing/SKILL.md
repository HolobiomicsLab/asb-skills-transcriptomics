---
name: scrna-seq-yan-dataset-processing
description: Use when a Yan (or similar scRNA-seq) expression matrix and SSRE-inferred
  cluster labels are available and the goal is validating that the clustering supports
  downstream DEG detection via Kruskal-Wallis testing.
when_to_use_negative:
- Cluster labels come from a different clustering method or unvalidated similarity
  matrix — the skill's purpose is specifically validating that SSRE-inferred labels
  support DEG detection
- The goal is differential expression with replicate-level statistical modeling (e.g.
  pseudobulk DESeq2), not nonparametric per-gene rank testing across SSRE clusters
- Large-scale scRNA-seq datasets where the SSRE similarity-learning framework is known
  to be time-consuming and the paper defers acceleration to future parallel computing
  work
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SSRE
  role: Produces the SSRE-inferred cell type labels on the Yan dataset via adaptive
    gene selection, sparse subspace similarity learning, and spectral clustering,
    which serve as the grouping factor for DEG testing
  repo: https://github.com/CSUBioGroup/SSRE
- name: R (kruskal.test)
  role: Runs the per-gene Kruskal-Wallis rank-sum test with SSRE-inferred labels as
    the grouping factor to compute p-values for the DEG ranking
- name: MATLAB
  role: Implementation environment for running the SSRE pipeline that generates the
    cluster labels used in the DEG analysis
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: Alternative implementation environment for the SSRE pipeline that generates
    the cluster labels used in the DEG analysis
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:38c25c16db2120ea2c3a2465164901ca6d0e9adf8f888c36695898c98e1e7cae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - Python
  - R (kruskal.test)
  - SSRE
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# scrna-seq-yan-dataset-processing

## Summary

Reproduce differential expression analysis downstream of SSRE clustering on the Yan scRNA-seq dataset: apply the Kruskal-Wallis rank-sum test per gene using SSRE-inferred cell labels, threshold at P < 0.01, and rank the top-50 DE genes for comparison against the paper's Figure S2 marker set.

## When to use

Use when a Yan (or similar scRNA-seq) expression matrix and SSRE-inferred cluster labels are available and the goal is validating that the clustering supports downstream DEG detection via Kruskal-Wallis testing.

## When NOT to use

- Cluster labels come from a different clustering method or unvalidated similarity matrix — the skill's purpose is specifically validating that SSRE-inferred labels support DEG detection
- The goal is differential expression with replicate-level statistical modeling (e.g. pseudobulk DESeq2), not nonparametric per-gene rank testing across SSRE clusters
- Large-scale scRNA-seq datasets where the SSRE similarity-learning framework is known to be time-consuming and the paper defers acceleration to future parallel computing work

## Inputs

- Yan scRNA-seq gene expression matrix (genes x cells or cells x genes table)
- SSRE-inferred cell type labels (one label per cell, aligned to expression matrix columns/rows)

## Outputs

- top50_de_genes_yan.csv: top-50 differentially expressed genes with Kruskal-Wallis p-values and group assignments
- Validation report of recovered Figure S2 marker genes (NLRP11, NLRP4, CLEC10A, H1FOO, GDF9, OTX2, ACCSL, TUBB8, TUBB4Q)

## How to apply

Load the Yan scRNA-seq gene expression matrix and the SSRE-inferred cell type labels, aligning cells between the two tables. For each gene, run the Kruskal-Wallis rank-sum test in R (kruskal.test) with the SSRE-inferred labels as the grouping factor, yielding one p-value per gene. Adjust or apply the significance threshold P < 0.01 and rank significant genes by test statistic/p-value to extract the top-50 differentially expressed genes, writing them with p-values and group assignments to top50_de_genes_yan.csv. Rationale: SSRE was shown to be extensible to identification of differentially expressed genes, so a successful run confirms the learned similarities and spectral clustering recovered biologically meaningful blocks. Validate that the recovered marker set includes the previously reported markers NLRP11, NLRP4, CLEC10A, H1FOO, GDF9, OTX2, ACCSL, TUBB8, and TUBB4Q, and report any markers not recovered at P < 0.01.

## Related tools

- **SSRE** (Produces the SSRE-inferred cell type labels on the Yan dataset via adaptive gene selection, sparse subspace similarity learning, and spectral clustering, which serve as the grouping factor for DEG testing) — https://github.com/CSUBioGroup/SSRE
- **R (kruskal.test)** (Runs the per-gene Kruskal-Wallis rank-sum test with SSRE-inferred labels as the grouping factor to compute p-values for the DEG ranking)
- **MATLAB** (Implementation environment for running the SSRE pipeline that generates the cluster labels used in the DEG analysis) — https://github.com/CSUBioGroup/SSRE
- **Python** (Alternative implementation environment for the SSRE pipeline that generates the cluster labels used in the DEG analysis) — https://github.com/CSUBioGroup/SSRE

## Examples

```
kw <- apply(expr_matrix, 1, function(g) kruskal.test(g, factor(ssre_labels))$p.value); top50 <- names(sort(kw[kw < 0.01]))[1:50]; write.csv(data.frame(gene=top50, p=kw[top50]), 'top50_de_genes_yan.csv')
```

## Evaluation signals

- The top-50 DE gene list at P < 0.01 recovers the Figure S2 marker set: NLRP11, NLRP4, CLEC10A, H1FOO, GDF9, OTX2, ACCSL, TUBB8, and TUBB4Q
- Each gene has exactly one Kruskal-Wallis p-value, and the number of genes tested equals the number of genes in the aligned expression matrix
- Every cell in the expression matrix maps to exactly one SSRE-inferred label with no unmatched cells after alignment
- top50_de_genes_yan.csv is written with columns for gene, p-value, and group assignment, and contains exactly 50 rows all satisfying P < 0.01
- Any paper-reported marker absent from the recovered set at P < 0.01 is explicitly reported rather than silently dropped

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; the paper notes parallel computing is a possible future acceleration strategy
- Markers not recovered at the P < 0.01 threshold must be reported rather than assumed — the DEG list depends on the quality of the SSRE-inferred labels
- The Kruskal-Wallis test treats cells as independent samples per cluster, so it does not model dropouts or per-cell uncertainty inherent to scRNA-seq data

## Evidence

- [abstract] SSRE can be extended to visualization of scRNA-seq data and identification of differentially expressed genes: "In addition, SSRE can be extended to visualization of scRNA-seq data and identification of differentially expressed genes."
- [intro] The experimental results show that when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably.: "The experimental results show that when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably."
- [discussion] One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework: "One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework"
- [supplementary] The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE.: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
