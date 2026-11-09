---
name: scib-metric-computation
description: Use when multiple data-integration or label-transfer methods have been
  run on the same reference-query split and you need a ranked, metric-based comparison
  of their embeddings (e.g., on a pan-cancer tumor-infiltrating myeloid AnnData).
when_to_use_negative:
- When a method cannot produce a cell embedding on the given batch structure (e.g.,
  Seurat-based methods with >85 batches) — it should be excluded, not force-scored
- When the goal is only qualitative inspection of a single embedding (e.g., UMAP)
  rather than cross-method benchmarking
- When the input is raw gene expression rather than a method output embedding/annotation
  that scIB metrics can be computed on
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TOSICA
  role: Generates the attention-matrix embedding and predicted labels that are library-size-normalized
    (size factor 10,000) and evaluated with scIB metrics; ranked second of 11 applicable
    methods
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Loads the reference and query AnnData objects and supports library-size normalization
    of the attention matrix prior to metric computation
- name: torch
  role: Runtime for TOSICA model training and prediction whose outputs feed the scIB
    benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:47dc008216a75e42f292b3361a17713aff211580fae622ca4a77632118e99431
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# scib-metric-computation

## Summary

Benchmark single-cell data integration embeddings (e.g., TOSICA's library-size-normalized attention matrix) against comparison methods by computing 8 scIB metrics and combining them into an overall score of 0.6 × biology conservation + 0.4 × batch removal, then ranking methods.

## When to use

Use when multiple data-integration or label-transfer methods have been run on the same reference-query split and you need a ranked, metric-based comparison of their embeddings (e.g., on a pan-cancer tumor-infiltrating myeloid AnnData).

## When NOT to use

- When a method cannot produce a cell embedding on the given batch structure (e.g., Seurat-based methods with >85 batches) — it should be excluded, not force-scored
- When the goal is only qualitative inspection of a single embedding (e.g., UMAP) rather than cross-method benchmarking
- When the input is raw gene expression rather than a method output embedding/annotation that scIB metrics can be computed on

## Inputs

- Cell embeddings from each integration/label-transfer method computed on the same reference-query split (e.g., TOSICA attention matrix new_adata.X after TOSICA.pre)
- Reference AnnData (KIDNEY/UCEC/ESCA pan-cancer tumor-infiltrating myeloid, 71,159 cells) and six-cancer-type query AnnData loaded with scanpy
- Predicted cell-type labels from each method for label-based scIB metrics

## Outputs

- scIB metric values (8 metrics) per method
- Combined overall score per method (0.6 × biology conservation + 0.4 × batch removal)
- Ranked method table (e.g., scib_ranking.csv with one row per applicable method)

## How to apply

Run each integration/label-transfer method (TOSICA plus comparison methods) on the identical reference-query split, keeping the KIDNEY/UCEC/ESCA pan-cancer myeloid reference (71,159 cells) and six-cancer-type query separate. For TOSICA specifically, take the attention matrix from TOSICA.pre (new_adata.X) and normalize it by library-size correction using the default size factor of 10,000, mirroring the paper's pipeline. Compute the 8 scIB metrics for every method's embedding and combine them as overall score = 0.6 × biology conservation + 0.4 × batch removal. Rank all applicable methods by the combined score and write the ranking table (e.g., scib_ranking.csv). Note applicability constraints: Seurat-based integration methods are inapplicable when the number of batches exceeds 85, so they must be excluded from the ranking rather than scored.

## Related tools

- **TOSICA** (Generates the attention-matrix embedding and predicted labels that are library-size-normalized (size factor 10,000) and evaluated with scIB metrics; ranked second of 11 applicable methods) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Loads the reference and query AnnData objects and supports library-size normalization of the attention matrix prior to metric computation)
- **torch** (Runtime for TOSICA model training and prediction whose outputs feed the scIB benchmark)

## Examples

```
new_adata = TOSICA.pre(query_adata[:, ref_adata.var_names], project='human_reactome_20xxxxxx')
```

## Evaluation signals

- The ranking table (e.g., scib_ranking.csv) contains exactly 11 method rows for the pan-cancer myeloid benchmark
- TOSICA is placed second in the combined-score ranking (overall score = 0.6 × biology conservation + 0.4 × batch removal)
- Each embedding was normalized by library-size correction with the default size factor of 10,000 before metric computation
- All 11 methods were evaluated on the same reference-query split and gene ordering (query_adata[:, ref_adata.var_names])
- Inapplicable methods (Seurat-based with >85 batches) are excluded from the table rather than assigned scores

## Limitations

- Seurat-based integration methods are inapplicable for >85 batches, restricting the comparable method pool on highly batched datasets
- Number of attention heads, learning rate, batch size, and random seed are not reported in the TOSICA methods, complicating exact reproduction of the embedding being scored
- The overall score's 0.6/0.4 weighting of biology conservation vs. batch removal is a fixed choice that determines ranking outcomes

## Evidence

- [results] TOSICA ranked second out of all 11 applicable data integration methods on the pan-cancer myeloid dataset when evaluated with 8 scIB metrics combined (overall score = 0.6 × biology conservation + 0.4 × batch removal): [evidence span withheld — anchor status: not_found]
- [discussion] Normalize attention matrix by library-size correction using default size factor 10,000: "the matrix is normalized by library-size correction using default size factor 10,000"
- [readme] the var_names (genes) of the ref_adata and query_adata must be consistent and in the same order: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [readme] new_adata.X : Attention matrix; new_adata.obs['Prediction'] : Predicted labels: "+ `new_adata.X` : Attention matrix
+ `new_adata.obs['Prediction']` : Predicted labels
+ `new_adata.obs['Probability']` : Probability of the prediction"
