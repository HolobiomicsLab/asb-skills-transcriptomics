---
name: biological-conservation-scoring
description: Use when comparing single-cell batch-correction or integration methods
  on scRNA-seq or spatial (e.g., MERFISH–scRNA-seq) embeddings and you must quantify
  whether cell types and biological structure survive correction, not just whether
  batches mix.
when_to_use_negative:
- The input is raw count data that has not yet been integrated or batch-corrected
  — scoring biological conservation presupposes an embedding from an integration method
- The dataset lacks cell-type annotations, since ARI, NMI, and CellType ASW compare
  clusters against known biological labels
- You only need to assess batch mixing without any claim about preserved biology —
  the over-correction risk scCobra targets is invisible to batch-mixing metrics alone
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scIB
  role: Pipeline used to apply Leiden clustering across resolutions 0.1–2.0 and compute
    batch-removal (Batch ASW) and biological-conservation (ARI, NMI, CellType ASW,
    Graph Connectivity) metrics on each embedding
- name: Scanpy
  role: Preprocessing and clustering backbone (filtering, log1p normalization, Leiden
    clustering) installed together with scIB per the README
  repo: https://github.com/mcgilldinglab/scCobra
- name: scCobra
  role: Produces the contrastive cell embeddings whose biological conservation is
    scored, alongside six benchmark methods run with default parameters
  repo: https://github.com/mcgilldinglab/scCobra
- name: Seurat v3.2.3
  role: Reference implementation whose 'vst' method is used to select the 2000 highly
    variable genes fed into the evaluated embeddings
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:e77a037b6d448347244ab61c579f6d0b50fe30a58685413a2b4fe6454863665d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scCobra
  tools:
  - Harmony
  - PyTorch
  - Scanorama
  - Scanpy
  - Seurat
  - Seurat v3.2.3
  - scCobra
  - scDML
  - scDREAMER
  - scIB
  - scVI
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# biological-conservation-scoring

## Summary

Evaluates whether a single-cell data integration (e.g., scCobra vs. Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER embeddings) preserves biological information, using the scIB pipeline with ARI, NMI, CellType ASW, and Graph Connectivity alongside batch-removal metrics (Batch ASW).

## When to use

Use when comparing single-cell batch-correction or integration methods on scRNA-seq or spatial (e.g., MERFISH–scRNA-seq) embeddings and you must quantify whether cell types and biological structure survive correction, not just whether batches mix.

## When NOT to use

- The input is raw count data that has not yet been integrated or batch-corrected — scoring biological conservation presupposes an embedding from an integration method
- The dataset lacks cell-type annotations, since ARI, NMI, and CellType ASW compare clusters against known biological labels
- You only need to assess batch mixing without any claim about preserved biology — the over-correction risk scCobra targets is invisible to batch-mixing metrics alone

## Inputs

- Corrected/integrated cell embeddings per method (e.g., scCobra contrastive embeddings, or outputs of Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) with cell-level batch and cell-type annotations
- Preprocessed expression matrices (log1p-normalized, top 2000 'vst' HVGs) as h5ad/AnnData objects
- Known cell-type labels (e.g., 14 cell types from 9 batches in the pancreas dataset, 16 cell types from 10 batches in the immune dataset, 17 cell types from 16 batches in the lung atlas dataset)

## Outputs

- Per-dataset, per-method metric table (ARI, NMI, CellType ASW, Graph Connectivity, Batch ASW) — e.g., benchmark_metrics.csv or hypothalamus_integration_benchmark.csv
- Fig. 2c–f / Fig. 4e–f style comparison plots of batch-removal vs. biological conservation scores
- Leiden cluster assignments across resolutions 0.1–2.0 used for metric computation

## How to apply

Take each method's corrected embedding as input and run the scIB pipeline over it. Apply Leiden clustering across a sweep of resolutions from 0.1 to 2.0 in increments of 0.1 so that metric values are not biased by a single clustering granularity. Compute batch-removal metrics (Batch ASW / batch mixing) and summed biological conservation metrics — ARI, NMI, CellType ASW, and Graph Connectivity — matching the scCobra benchmark design. The biological metrics require cluster labels that are compared against known cell-type annotations (e.g., 14–17 cell types in the pancreas, immune, and lung atlas datasets), so preprocessed inputs should retain cell-type labels; preprocessing per the scCobra protocol filters cells with fewer than 200 genes, removes genes detected in fewer than 3 cells, drops cells with mitochondrial fraction above 5%, applies log1p normalization, and selects 2000 highly variable genes using Seurat v3.2.3's 'vst' method. Rank methods on each metric and inspect whether the expected cell populations (e.g., newly formed OD and OD Immature cells in the mouse hypothalamus MERFISH integration) remain adjacent in the embedding.

## Related tools

- **scIB** (Pipeline used to apply Leiden clustering across resolutions 0.1–2.0 and compute batch-removal (Batch ASW) and biological-conservation (ARI, NMI, CellType ASW, Graph Connectivity) metrics on each embedding)
- **Scanpy** (Preprocessing and clustering backbone (filtering, log1p normalization, Leiden clustering) installed together with scIB per the README) — https://github.com/mcgilldinglab/scCobra
- **scCobra** (Produces the contrastive cell embeddings whose biological conservation is scored, alongside six benchmark methods run with default parameters) — https://github.com/mcgilldinglab/scCobra
- **Seurat v3.2.3** (Reference implementation whose 'vst' method is used to select the 2000 highly variable genes fed into the evaluated embeddings)

## Examples

```
pip install scanpy scib
```

## Evaluation signals

- Metric values are computed after Leiden clustering swept over resolutions 0.1 to 2.0 in increments of 0.1, not a single fixed resolution
- Averaged over the lung atlas, immune, and pancreas benchmarks, scCobra should attain the highest ARI, NMI, CellType ASW, and Graph Connectivity of the seven methods
- In the MERFISH–scRNA-seq hypothalamus integration, scCobra ranks first on both Batch ASW (Fig. 4e) and summed biological conservation (Fig. 4f); Seurat, Harmony, Scanorama, scDML, and scVI should perform similarly to each other
- Biologically expected neighborhoods are preserved: newly formed OD and OD Immature cells remain correctly adjacent in the integrated embedding
- Metric table is complete across all 7 methods × all evaluated datasets and saves cleanly to the CSV output (e.g., benchmark_metrics.csv)

## Limitations

- Over-correction scores were derived from two-dimensional UMAP embeddings, following the SCALEX methodology, introducing dependence on the UMAP visualization
- Benchmarking methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) were run with default parameters, which may not reflect their tuned performance
- Evaluation of current batch-correction methods risks inaccuracy because simulated batch-affected data may not resemble real scRNA-seq data

## Evidence

- [discussion] scIB Leiden sweep: "The scIB pipeline applied Leiden clustering across a range of resolutions from 0.1 to 2.0 (in increments of 0.1)"
- [results] biological conservation metrics: [evidence span withheld — anchor status: not_found]
- [discussion] preprocessing thresholds: "we removed cells where mitochondrial genes accounted for more than 5% of the total gene expression"
- [discussion] UMAP dependence of scores: "The scores were derived from two-dimensional UMAP embeddings, following the methodology described in the original SCALEX26 paper"
- [readme] README environment: "pip install scanpy scib"
