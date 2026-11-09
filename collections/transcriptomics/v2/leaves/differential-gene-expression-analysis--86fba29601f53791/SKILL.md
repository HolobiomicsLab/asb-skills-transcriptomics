---
name: differential-gene-expression-analysis
description: Use when you have a single-cell gene expression matrix plus cluster labels
  (e.g., SSRE-inferred cell type labels) and need to detect marker genes per cluster,
  expecting recovery of known markers.
when_to_use_negative:
- You only have raw scRNA-seq counts without any cluster or cell type labels — the
  test requires a pre-computed grouping factor such as SSRE-inferred labels.
- The goal is cell type detection itself rather than downstream marker discovery —
  use SSRE clustering (or SC3/SIMLR) first, then apply this skill to its labels.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SSRE
  role: Produces the cluster/cell-type labels used as the grouping factor for the
    per-gene Kruskal-Wallis DEG test
  repo: https://github.com/CSUBioGroup/SSRE
- name: R kruskal.test
  role: Runs the non-parametric Kruskal-Wallis rank-sum test per gene with cluster
    labels as the grouping factor
- name: MATLAB
  role: SSRE implementation environment used to generate the inferred labels from
    the expression matrix
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:849eee72e23ff20df17a75c533e2a4068271abb866b2be1088788578e447cb83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - Python
  - R kruskal.test
  - SSRE
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# differential-gene-expression-analysis

## Summary

Identifies significantly differentially expressed genes from scRNA-seq data by testing each gene across SSRE-inferred cluster labels, as demonstrated on the Yan dataset with the Kruskal-Wallis rank-sum test at P < 0.01. This extends SSRE's cell type detection output to downstream marker-gene discovery.

## When to use

Use when you have a single-cell gene expression matrix plus cluster labels (e.g., SSRE-inferred cell type labels) and need to detect marker genes per cluster, expecting recovery of known markers.

## When NOT to use

- You only have raw scRNA-seq counts without any cluster or cell type labels — the test requires a pre-computed grouping factor such as SSRE-inferred labels.
- The goal is cell type detection itself rather than downstream marker discovery — use SSRE clustering (or SC3/SIMLR) first, then apply this skill to its labels.

## Inputs

- scRNA-seq gene expression matrix (cells x genes), e.g., the Yan dataset expression table
- SSRE-inferred cell type / cluster labels aligned to the cells of the expression matrix

## Outputs

- Ranked table of top-50 differentially expressed genes with p-values and group assignments (e.g., top50_de_genes_yan.csv)
- Set of significant marker genes passing P < 0.01 per cluster grouping

## How to apply

Load the scRNA-seq gene expression matrix (e.g., the Yan dataset) and the SSRE-inferred cluster labels, aligning cells between the two tables so each cell has both an expression vector and a group assignment. For each gene, run the Kruskal-Wallis rank-sum test in R (kruskal.test) with the SSRE-inferred labels as the grouping factor, yielding one p-value per gene. Apply the significance threshold P < 0.01 and rank significant genes by test statistic or p-value to extract the top differentially expressed genes (top-50 in the Yan demonstration), writing them with p-values and group assignments to an output table such as top50_de_genes_yan.csv. The rationale is that SSRE's similarity-learning-based clustering (adaptive gene selection plus similarity enhancement) reliably recovers cell types, so its labels support meaningful downstream differential expression testing; validation checks that the recovered marker set includes the paper's reported markers.

## Related tools

- **SSRE** (Produces the cluster/cell-type labels used as the grouping factor for the per-gene Kruskal-Wallis DEG test) — https://github.com/CSUBioGroup/SSRE
- **R kruskal.test** (Runs the non-parametric Kruskal-Wallis rank-sum test per gene with cluster labels as the grouping factor)
- **MATLAB** (SSRE implementation environment used to generate the inferred labels from the expression matrix) — https://github.com/CSUBioGroup/SSRE

## Examples

```
kw_p <- apply(yan_expr, 1, function(g) kruskal.test(g ~ factor(ssre_labels))$p.value); top50 <- head(names(sort(kw_p[kw_p < 0.01]), decreasing=FALSE), 50)
```

## Evaluation signals

- Recovered marker set includes the paper-reported Yan markers: NLRP11, NLRP4, CLEC10A, H1FOO, GDF9, OTX2, ACCSL, TUBB8, and TUBB4Q at P < 0.01.
- Every gene in the expression matrix yields exactly one Kruskal-Wallis p-value (one test per gene with cluster labels as the factor).
- Output table contains exactly 50 ranked genes with p-values and group assignments when the top-50 cutoff is applied.
- Cells in the expression matrix and the label table are fully aligned (no unmatched cell IDs before testing).
- All reported genes meet the P < 0.01 significance threshold; any reported marker absent from the result is explicitly flagged.

## Limitations

- SSRE itself is relatively time-consuming on large-scale datasets, limiting throughput of the upstream labeling step that feeds this DEG analysis.
- SSRE's gene selection and similarity enhancement use three pairwise similarities by default; changing which similarities are used alters cluster labels and hence the DEG results.
- The workflow uses a fixed P < 0.01 threshold without explicit multiple-testing correction detail in the demonstrated Yan case; markers not recovered at this cutoff must be reported.

## Evidence

- [abstract] SSRE extends to DEG identification: "In addition, SSRE can be extended to visualization of scRNA-seq data and identification of differentially expressed genes."
- [discussion] Similarity choice affects learned representation: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
- [readme] Input data structure expected (README of related MATLAB clustering code): [evidence span withheld — anchor status: not_found]
- [methods] Per-gene Kruskal-Wallis procedure with P < 0.01 threshold and top-50 output: [evidence span withheld — anchor status: not_found]
