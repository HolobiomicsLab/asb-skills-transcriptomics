---
name: differential-gene-expression-analysis
description: Use when cells have already been assigned to clusters by SINCERA cluster.assignment
  and you need per-cluster differentially expressed genes to characterize cell types
  or build gene signatures.
when_to_use_negative:
- Cells have not yet been clustered — cluster.diffgenes requires cluster assignments
  from cluster.assignment; differential testing on unclustered single cells is out
  of scope
- The analysis targets single-cell differential expression across conditions (e.g.
  disease vs control) rather than between cell clusters; tools like SCDE or SAMstrt
  are referenced for that setting
- Genes have not passed cluster.geneSelection (specificity threshold 0.7) — unselectively
  expressed housekeeping genes should be dropped first to avoid non-discriminative
  signatures
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SINCERA
  role: R S4 pipeline whose cluster.diffgenes function performs the per-cluster differential
    expression testing (one-tailed Welch's t test or Wilcoxon, p < 0.05)
  repo: https://github.com/xu-lab/SINCERA
- name: SCDE
  role: Alternative tool mentioned in the article for single-cell differential expression
    analysis outside the SINCERA clustering workflow
- name: SAMstrt
  role: Alternative tool mentioned in the article for single-cell differential expression
    analysis outside the SINCERA clustering workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:98089474a6c051377297c7499465804fb40b5116191d841211abd94b1c678abb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xu-lab/SINCERA
  tools:
  - SAMstrt
  - SCDE
  - SINCERA
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# differential-gene-expression-analysis

## Summary

Identifies differentially expressed genes per cell cluster in SINCERA after clustering, using one-tailed Welch's t test or Wilcoxon rank-sum test with a p-value threshold of 0.05, feeding downstream cell type enrichment analysis.

## When to use

Use when cells have already been assigned to clusters by SINCERA cluster.assignment and you need per-cluster differentially expressed genes to characterize cell types or build gene signatures.

## When NOT to use

- Cells have not yet been clustered — cluster.diffgenes requires cluster assignments from cluster.assignment; differential testing on unclustered single cells is out of scope
- The analysis targets single-cell differential expression across conditions (e.g. disease vs control) rather than between cell clusters; tools like SCDE or SAMstrt are referenced for that setting
- Genes have not passed cluster.geneSelection (specificity threshold 0.7) — unselectively expressed housekeeping genes should be dropped first to avoid non-discriminative signatures

## Inputs

- SINCERA S4 object built via construct from a genes x cells expression matrix (exprfile, FPKM/TPM) and per-cell sample information (samplefile), preprocessed with filterLowQualityCells (min.genes = 500), prefilterGenes, expr.minimum (0.01 FPKM/TPM), normalization.zscore, and cluster.geneSelection (specificity threshold 0.7)
- Per-cluster cell assignments produced by cluster.assignment

## Outputs

- Differentially expressed genes per cluster (p < 0.05, one-tailed Welch's t test or Wilcoxon test)
- Cell type-specific gene signature lists usable as input to celltype.enrichment and celltype.validation

## How to apply

After cluster.assignment assigns cells to initial clusters (hierarchical clustering with average linkage on Pearson correlation distance of z-score transformed expression), run cluster.diffgenes on the SINCERA S4 object to find differentially expressed genes per cluster. Choose a one-tailed Welch's t test or a Wilcoxon rank-sum test as the statistical test, and apply a p-value threshold of 0.05 to call cluster-specific genes. The resulting per-cluster gene lists can then be passed to celltype.enrichment, which performs a one-tailed Fisher's exact test against the precompiled EBI Expression Atlas cell type-gene table to assign predicted cell types. Validate assignments with celltype.validation, a rank-aggregation-based check that confirms predicted cell types are consistent with selectively expressed known marker genes.

## Related tools

- **SINCERA** (R S4 pipeline whose cluster.diffgenes function performs the per-cluster differential expression testing (one-tailed Welch's t test or Wilcoxon, p < 0.05)) — https://github.com/xu-lab/SINCERA
- **SCDE** (Alternative tool mentioned in the article for single-cell differential expression analysis outside the SINCERA clustering workflow)
- **SAMstrt** (Alternative tool mentioned in the article for single-cell differential expression analysis outside the SINCERA clustering workflow)

## Examples

```
library(SINCERA); sc <- cluster.diffgenes(sc, cluster = '1', test = 'welch', pvalue = 0.05)
```

## Evaluation signals

- Each cluster returns a non-empty set of differentially expressed genes at p < 0.05 with the chosen test (Welch's t test or Wilcoxon)
- Per-cluster DE gene lists overlap substantially with the selectively expressed genes retained by cluster.geneSelection (specificity threshold 0.7)
- celltype.enrichment (one-tailed Fisher's exact test against the EBI Expression Atlas table) yields a predicted cell type for each cluster
- celltype.validation rank-aggregation check confirms each predicted cell type is consistent with selectively expressed known marker genes
- plotMarkers visualizations show the called DE genes are cluster-selective rather than housekeeping

## Limitations

- Differential calls are per-cluster, not per-condition: they identify cluster-specific markers, not disease-vs-control DE between samples such as IPF vs normal
- Quality depends on upstream preprocessing: cells must pass filterLowQualityCells (min.expression = 1 FPKM/TPM, min.genes = 500) and genes prefilterGenes/expr.minimum (0.01 FPKM/TPM) before testing
- Run on a standard desktop/laptop computer; SINCERA targets datasets of moderate scale rather than massive atlases
- Statistical model is simple (one-tailed t test / Wilcoxon at p < 0.05); no explicit modeling of dropouts or zero inflation is described

## Evidence

- [methods] Run cluster.diffgenes (one-tailed Welch's t test or Wilcoxon test, p-value threshold 0.05) to find differentially expressed genes per cluster: [evidence span withheld — anchor status: not_found]
