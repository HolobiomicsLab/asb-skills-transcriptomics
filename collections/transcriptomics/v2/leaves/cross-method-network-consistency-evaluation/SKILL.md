---
name: cross-method-network-consistency-evaluation
description: Use when true gene networks are unknown for real single-cell data and
  inferred TF–gene edges must be validated against ChIP-seq references, comparing
  scLink with PIDC, Pearson, Spearman, and glasso variants per cell type.
when_to_use_negative:
- Simulated data with known ground-truth networks — direct edge recovery (precision/recall)
  metrics are available and more informative than ChIP-seq overlap
- Organisms or cell types with little or no ChIP-seq TF–target coverage, where overlap
  proportions would be uninformative or biased toward zero
- When the goal is to determine edge direction or distinguish regulatory vs. co-regulatory
  vs. functionally coordinated relationships — scLink edges are undirected statistical
  dependencies
edam_operation: http://edamontology.org/operation_2492
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scLink
  role: 'Primary network inference method: sclink_norm normalizes counts (library
    size 10^6, top 500 genes) and sclink_net infers the sparse co-expression network
    via the lda regularization grid with BIC selection'
  repo: https://github.com/Vivianstats/scLink
- name: PIDC
  role: Comparison method whose inferred edges are scored with the same TF–gene ChIP-seq
    overlap proportion
- name: R
  role: Implementation language; scLink is distributed as an R package installed from
    CRAN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:10e6360e510050e17212360434e650d05863c28fda2a5e7224d7f3e2338dba03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - PIDC
  - R
  - scLink
  edam_operation: http://edamontology.org/operation_2492
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cross-method-network-consistency-evaluation

## Summary

Validates single-cell gene co-expression networks inferred from real data (where true networks are unknown) by measuring the proportion of inferred TF–gene edges that appear in pooled ChIP-seq TF–target references (ENCODE, ChEA, Harmonizome), benchmarking scLink against PIDC, Pearson, Spearman, glasso-f, and glasso-r per cell type.

## When to use

Use when true gene networks are unknown for real single-cell data and inferred TF–gene edges must be validated against ChIP-seq references, comparing scLink with PIDC, Pearson, Spearman, and glasso variants per cell type.

## When NOT to use

- Simulated data with known ground-truth networks — direct edge recovery (precision/recall) metrics are available and more informative than ChIP-seq overlap
- Organisms or cell types with little or no ChIP-seq TF–target coverage, where overlap proportions would be uninformative or biased toward zero
- When the goal is to determine edge direction or distinguish regulatory vs. co-regulatory vs. functionally coordinated relationships — scLink edges are undirected statistical dependencies

## Inputs

- Tabula Muris Smart-seq2 cell-type count matrices (>=100 cells per type; 53,760 cells, 20 tissues, 8 mice)
- ChIP-seq TF–target edge lists from ENCODE, ChEA, and Harmonizome
- Set of 310 transcription factors defining the pooled TF–gene reference
- Inferred edge lists from each compared method: scLink, PIDC, Pearson, Spearman, glasso-f, glasso-r

## Outputs

- tf_edge_overlap_tabula_muris.csv: 59-row overlap table (one row per cell type) with per-method TF–gene overlap proportions
- Per-cell-type scLink networks from sclink_net (selected by minimum BIC, restricted to <=5% of possible edges)

## How to apply

For each cell-type count matrix (≥100 cells), normalize with sclink_norm using scale.factor = 1e6 and filter.genes = TRUE with n = 500 to retain the top 500 highly expressed genes, then run sclink_net with the candidate regularization grid lda = seq(0.5, 0.1, -0.05), selecting the network by minimum BIC and restricting the final network to at most 5% of all possible edges. Build a pooled TF–gene reference set from 310 transcription factors by combining ChIP-seq target deposits from ENCODE, ChEA, and Harmonizome. For each cell type and each method (scLink, PIDC, Pearson, Spearman, glasso-f, glasso-r), compute the proportion of inferred edges that are TF–gene pairs present in the pooled reference. Rationale: since true gene networks are unknown for real data, overlap with experimentally determined ChIP-seq TF–target edges serves as an indirect consistency check, and scLink's penalized-likelihood edges are expected to recover more verified TF–gene pairs than correlation- or shrinkage-based baselines. Assemble the per-cell-type, per-method overlap proportions into a 59-row table (e.g. tf_edge_overlap_tabula_muris.csv).

## Related tools

- **scLink** (Primary network inference method: sclink_norm normalizes counts (library size 10^6, top 500 genes) and sclink_net infers the sparse co-expression network via the lda regularization grid with BIC selection) — https://github.com/Vivianstats/scLink
- **PIDC** (Comparison method whose inferred edges are scored with the same TF–gene ChIP-seq overlap proportion)
- **R** (Implementation language; scLink is distributed as an R package installed from CRAN)

## Examples

```
expr.norm <- sclink_norm(count.mat, scale.factor = 1e6, filter.genes = TRUE, n = 500)
```

## Evaluation signals

- Output table has exactly 59 rows, one per Tabula Muris Smart-seq2 cell type with >=100 cells
- scLink median TF–gene overlap proportion is approximately 59.3%, with range 15.6%–89.5% across cell types
- scLink and PIDC rows show higher overlap proportions than Pearson, Spearman, glasso-f, and glasso-r rows
- Every overlap proportion lies in [0, 1] and each method column is populated for all 59 cell types
- Each scLink network was selected at minimum BIC over lda = seq(0.5, 0.1, -0.05) and contains at most 5% of all possible edges

## Limitations

- Edges identified by scLink are based on statistical dependencies and do not have directions, so it is not feasible to directly distinguish regulatory, co-regulatory, and functionally coordinated edge types using only gene expression data
- ChIP-seq overlap is an indirect validation because true gene networks are unknown for real data; the reference is limited to 310 transcription factors
- Imputed data may still be much sparser than bulk data, so bulk-data network methods may perform poorly on imputed single-cell data, and imputation can be time-consuming depending on cell number

## Evidence

- [methods] normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis: "normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis"
- [methods] they can set `filter.genes=TRUE` to filter for the top $n$ genes with largest average expression values: "they can set `filter.genes=TRUE` to filter for the top $n$ genes with largest average expression values"
- [discussion] single-cell ChIP-seq experiments could be designed and prioritized based on scLink's identified TF–gene pairs [71]: [evidence span withheld — anchor status: not_found]
- [discussion] the edges identified by scLink are based on statistical dependencies and do not have directions: "the edges identified by scLink are based on statistical dependencies and do not have directions"
- [results] A substantial proportion of scLink's identified TF–gene edges were previously discovered in ChIP-seq experiments, ranging from 15.6% to 89.5% across cell types with a median of 59.3%.: [evidence span withheld — anchor status: not_found]
- [readme] sclink_net for constructing the gene co-expression network by scLink: [evidence span withheld — anchor status: not_found]
