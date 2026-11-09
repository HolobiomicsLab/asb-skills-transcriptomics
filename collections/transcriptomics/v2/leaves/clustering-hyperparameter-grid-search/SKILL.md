---
name: clustering-hyperparameter-grid-search
description: Use when evaluating a pretrained cell-embedding (e.g., tGPT feature representations)
  by clustering, and you must sweep KNN/Leiden hyperparameters before computing NMI,
  ARI, and FMI against known cell-type annotations.
when_to_use_negative:
- No ground-truth cell-type labels are available — NMI, ARI, and FMI all require reference
  labels, so the grid search cannot be scored
- Input is already a cluster assignment or a metrics table — this skill operates on
  raw embedding matrices upstream of clustering
- The embedding being clustered is not a fixed feature representation (e.g., raw count
  matrix destined for Scanpy/scVI normalization pipelines, which the article benchmarks
  separately with HVG filtering)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: tGPT
  role: Provides the pretrained transformer embeddings whose clustering quality is
    being measured
  repo: https://github.com/deeplearningplus/tGPT
- name: Scanpy
  role: Builds the AnnData object, computes the KNN neighbor graph (sc.pp.neighbors),
    and runs Leiden clustering (sc.tl.leiden) over the parameter grid
  repo: https://github.com/theislab/scanpy
- name: transformers
  role: Loads the tGPT GPT2 checkpoint with output_hidden_states=True to extract per-cell
    features
- name: PyTorch
  role: Runs the GPT2 model forward pass in batches (batch_size=64) under torch.no_grad
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:f3b02960573867b8f1b93b4ce38f799bc528267a83562b40fd570ff46b4892e1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deeplearningplus/tGPT
  repo_url: https://github.com/deeplearningplus/tGPT
  tools:
  - NumPy
  - PyTorch
  - Scanpy
  - pandas
  - tGPT
  - transformers
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# clustering-hyperparameter-grid-search

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A grid-search protocol for tuning Scanpy KNN-neighbor-graph + Leiden clustering (n_neighbors and resolution) and scoring the resulting partitions against ground-truth cell labels with external clustering metrics (NMI, ARI, FMI), as used to quantify tGPT embedding quality across six single-cell and bulk tissue datasets.

## When to use

Use when evaluating a pretrained cell-embedding (e.g., tGPT feature representations) by clustering, and you must sweep KNN/Leiden hyperparameters before computing NMI, ARI, and FMI against known cell-type annotations.

## When NOT to use

- No ground-truth cell-type labels are available — NMI, ARI, and FMI all require reference labels, so the grid search cannot be scored
- Input is already a cluster assignment or a metrics table — this skill operates on raw embedding matrices upstream of clustering
- The embedding being clustered is not a fixed feature representation (e.g., raw count matrix destined for Scanpy/scVI normalization pipelines, which the article benchmarks separately with HVG filtering)

## Inputs

- Matrix of tGPT feature representations (mean-pooled hidden states, one vector per cell/tissue)
- Ground-truth cell-type annotation table (e.g., Muris_cell_labels.txt.gz, one label per cell)
- Gene-ranking sequence file (e.g., Muris_gene_rankings.txt.gz) used for tGPT feature extraction

## Outputs

- Best Leiden cluster assignments per dataset (optimal n_neighbors, resolution pair)
- clustering_metrics.csv with per-dataset NMI, ARI, and FMI for all six datasets

## How to apply

Mean-pool the tGPT hidden states (GPT2Model from checkpoint 'lixiangchun/transcriptome-gpt-1024-8-16-64', max_length=64, batch_size=64, excluding special tokens) to build a Scanpy AnnData of features. Compute the KNN graph with sc.pp.neighbors and run Leiden, but grid-search n_neighbors from 5 to 100 in steps of 5 and resolution from 0.1 to 2 in steps of 0.2, selecting the parameter combination per dataset rather than fixing defaults (README example uses n_neighbors=20, resolution=0.6). For each dataset, take the best Leiden assignment and compute normalized mutual information (NMI), adjusted Rand index (ARI), and Fowlkes-Mallows index (FMI) against ground-truth cell-type labels using scikit-learn external clustering metrics. Aggregate per-dataset metrics across the six evaluation datasets (single-cell: HCA, HCL, Tabula Muris, Mouse Cell Atlas; bulk: GTEx and TCGA tissue sets) into a single clustering_metrics.csv. Validate that recovered values fall within the published ranges before trusting the pipeline.

## Related tools

- **tGPT** (Provides the pretrained transformer embeddings whose clustering quality is being measured) — https://github.com/deeplearningplus/tGPT
- **Scanpy** (Builds the AnnData object, computes the KNN neighbor graph (sc.pp.neighbors), and runs Leiden clustering (sc.tl.leiden) over the parameter grid) — https://github.com/theislab/scanpy
- **transformers** (Loads the tGPT GPT2 checkpoint with output_hidden_states=True to extract per-cell features)
- **PyTorch** (Runs the GPT2 model forward pass in batches (batch_size=64) under torch.no_grad)

## Examples

```
sc.pp.neighbors(adata,n_neighbors=20); sc.tl.leiden(adata,resolution=0.6)
```

## Evaluation signals

- NMI across datasets falls within the reported range: 0.75 (HCA) to 0.90 (GTEx)
- ARI falls within 0.53 (HCL) to 0.84 (Tabula Muris)
- FMI falls within 0.55 (HCL) to 0.85 (Tabula Muris)
- The grid actually covered n_neighbors 5–100 (step 5) and resolution 0.1–2 (step 0.2) for every dataset
- clustering_metrics.csv contains exactly one scored row per dataset with all three metrics non-null

## Limitations

- The clustering-metric values depend on the tGPT embedding layer; the article notes that pretraining is not directly related to cell clustering and the last transformer layer is not guaranteed to give better clustering
- tGPT uses only top-expressing genes (62 or 126), so low-expression gene information is absent from the features being clustered; metrics from 62 genes are comparable to 126 genes
- Grid search over ~20 n_neighbors × 10 resolution values per dataset is computationally expensive on very large atlases (e.g., HCL with 586,135 cells)

## Evidence
