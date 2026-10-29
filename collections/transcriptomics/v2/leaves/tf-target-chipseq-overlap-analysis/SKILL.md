---
name: tf-target-chipseq-overlap-analysis
description: Use when evaluating whether single-cell co-expression network edges are
  biologically supported, by overlapping inferred TF–gene edges against known ChIP-seq
  TF–target interactions across cell types.
when_to_use_negative:
- When gold-standard TF–target edges are available or the dataset is simulated with
  a known true network — direct accuracy metrics are preferable to ChIP-seq overlap
- For cell types with fewer than 100 cells, which fall outside the Tabula Muris analysis
  scope
- When the goal is to assign directionality or edge type (regulatory vs co-regulatory)
  — scLink edges are undirected statistical dependencies
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: scLink
  role: Normalizes counts (sclink_norm) and infers sparse co-expression networks (sclink_net)
    whose TF–gene edges are tested for ChIP-seq overlap
  repo: https://github.com/Vivianstats/scLink
- name: R
  role: Implementation environment for the scLink R package and the overlap computation
  repo: https://github.com/Vivianstats/scLink
- name: PIDC
  role: Comparison network inference method whose TF–gene edge overlap proportions
    are benchmarked against scLink
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:4fffa6cce3bc9b1aa95f9000749f39a8e53cc10b1fe862e3f6dbd0d96412bfe0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - PIDC
  - R
  - scLink
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# tf-target-chipseq-overlap-analysis

## Summary

Validates cell type-specific gene networks inferred by scLink on Tabula Muris Smart-seq2 data by computing the proportion of inferred TF–gene edges that match a pooled ChIP-seq TF–target reference built from ENCODE, ChEA, and Harmonizome.

## When to use

Use when evaluating whether single-cell co-expression network edges are biologically supported, by overlapping inferred TF–gene edges against known ChIP-seq TF–target interactions across cell types.

## When NOT to use

- When gold-standard TF–target edges are available or the dataset is simulated with a known true network — direct accuracy metrics are preferable to ChIP-seq overlap
- For cell types with fewer than 100 cells, which fall outside the Tabula Muris analysis scope
- When the goal is to assign directionality or edge type (regulatory vs co-regulatory) — scLink edges are undirected statistical dependencies

## Inputs

- Tabula Muris Smart-seq2 cell-type count matrices (59 cell types, >=100 cells each; 53,760 cells total, 20 tissues, 8 mice)
- Pooled ChIP-seq TF–target reference set from ENCODE, ChEA, and Harmonizome covering 310 transcription factors
- Networks inferred by comparison methods: PIDC, Pearson, Spearman, glasso-f, glasso-r

## Outputs

- tf_edge_overlap_tabula_muris.csv: 59-row table (one row per cell type) with per-method TF–gene overlap proportions
- Per-cell-type scLink networks (sparse co-expression edges selected by minimum BIC, capped at 5% of possible edges)

## How to apply

Load each Tabula Muris Smart-seq2 cell-type count matrix (59 types with >=100 cells from 53,760 cells, 20 tissues, 8 mice), normalize with sclink_norm (scale.factor=1e6, filter.genes=TRUE, n=500 to keep the top 500 highly expressed genes), then run sclink_net over the regularization grid lda=seq(0.5, 0.1, -0.05), selecting the network by minimum BIC and restricting to at most 5% of all possible edges. Build a pooled TF–gene reference from 310 transcription factors combining ChIP-seq targets from ENCODE, ChEA, and Harmonizome. For each cell type and each method (scLink, PIDC, Pearson, Spearman, glasso-f, glasso-r), compute the proportion of inferred edges that are TF–gene pairs present in the pooled reference, and assemble a 59-row overlap table. Rationale: since true networks are unknown for real data, overlap with experimentally validated ChIP-seq edges serves as an external correctness signal for the inferred networks.

## Related tools

- **scLink** (Normalizes counts (sclink_norm) and infers sparse co-expression networks (sclink_net) whose TF–gene edges are tested for ChIP-seq overlap) — https://github.com/Vivianstats/scLink
- **R** (Implementation environment for the scLink R package and the overlap computation) — https://github.com/Vivianstats/scLink
- **PIDC** (Comparison network inference method whose TF–gene edge overlap proportions are benchmarked against scLink)

## Examples

```
dat_norm <- sclink_norm(counts, scale.factor = 1e6, filter.genes = TRUE, n = 500); net <- sclink_net(dat_norm, lda = seq(0.5, 0.1, -0.05), model.type = "BIC")
```

## Evaluation signals

- Output tf_edge_overlap_tabula_muris.csv has exactly 59 rows (one per Tabula Muris cell type)
- scLink's median TF–gene overlap proportion is approximately 59.3%, with range 15.6%–89.5% across cell types
- scLink and PIDC rows show higher overlap proportions than Pearson, Spearman, glasso-f, and glasso-r rows
- Each scLink network retains at most 5% of all possible edges after BIC-based model selection
- Each input cell-type matrix was normalized with scale.factor=1e6 and filtered to the top 500 highly expressed genes

## Limitations

- TF–gene overlap ranges widely across cell types (15.6%–89.5%), so low overlap in a single cell type is not by itself conclusive
- Edges from scLink are based on statistical dependencies and have no direction, so regulatory vs co-regulatory relationships cannot be distinguished from expression data alone
- Overlap with ChIP-seq references is an indirect validation since true gene networks are unknown for real data
- Imputation-based alternatives can be time-consuming depending on cell number, and imputed single-cell data may remain sparser than bulk data, degrading bulk-data network methods

## Evidence

- [results] ChIP-seq overlap proportions definition: [evidence span withheld — anchor status: not_found]
- [methods] sclink_norm preprocessing parameters: "normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis"
- [methods] sclink_net role: "use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink"
- [discussion] Edges undirected limitation: "the edges identified by scLink are based on statistical dependencies and do not have directions"
- [readme] Package availability and functions: [evidence span withheld — anchor status: not_found]
