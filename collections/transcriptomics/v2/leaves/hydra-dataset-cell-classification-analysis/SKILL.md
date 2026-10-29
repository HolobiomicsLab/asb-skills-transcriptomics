---
name: hydra-dataset-cell-classification-analysis
description: Use when reproducing scDEED dubious/trustworthy embedding percentages
  on a scRNA-seq dataset (e.g., Hydra) at a known original versus optimized t-SNE
  perplexity, or when validating hyperparameter optimization by counting cell classifications.
when_to_use_negative:
- Input is not a raw single-cell count matrix requiring the Seurat normalization pipeline
  (e.g., already an embedding or a feature table)
- The goal is optimizing hyperparameters rather than verifying reported dubious/trustworthy
  percentages at fixed perplexity values
- The embedding method is not t-SNE or UMAP as supported by the scDEED package
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDEED
  role: Classifies each cell embedding as dubious, trustworthy, or intermediate via
    reliability scores against a Permuted-data null distribution, and reports full_results
    per perplexity
  repo: github.com/JSB-UCLA/scDEED
- name: Seurat
  role: Builds the Seurat object (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures,
    RunPCA, RunTSNE) that scDEED consumes
  repo: satijalab/seurat-wrappers
- name: R
  role: Execution environment; scDEED is installed from GitHub via devtools::install_github('JSB-UCLA/scDEED')
- name: t-SNE
  role: 2D embedding method whose perplexity (40 vs 230) is varied to compare cell
    classification percentages
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:2b20971a07ba8a3e4f1deaab937d90e391f2f37c4f98de3bffffe0b6b122e628
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - scDEED
  - t-SNE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# hydra-dataset-cell-classification-analysis

## Summary

Reproduce and verify scDEED's classification of Hydra dataset cell embeddings as dubious or trustworthy at the original t-SNE perplexity (40) versus the scDEED-optimized perplexity (230), checking that dubious cells drop from 4.77% to 0.51% and trustworthy cells rise from 92.84% to 99.1%.

## When to use

Use when reproducing scDEED dubious/trustworthy embedding percentages on a scRNA-seq dataset (e.g., Hydra) at a known original versus optimized t-SNE perplexity, or when validating hyperparameter optimization by counting cell classifications.

## When NOT to use

- Input is not a raw single-cell count matrix requiring the Seurat normalization pipeline (e.g., already an embedding or a feature table)
- The goal is optimizing hyperparameters rather than verifying reported dubious/trustworthy percentages at fixed perplexity values
- The embedding method is not t-SNE or UMAP as supported by the scDEED package

## Inputs

- Hydra dataset count matrix (input_counts.rda) from Zenodo deposit 8371425
- Seurat object after NormalizeData, ScaleData, FindVariableFeatures, RunPCA with K=31 PCs
- scDEED result object (result$full_results) at perplexity 40 and perplexity 230

## Outputs

- Percentage of dubious and trustworthy cell embeddings at perplexity 40 (expected 4.77% / 92.84% of 24,985 cells)
- Percentage of dubious and trustworthy cell embeddings at perplexity 230 (expected 0.51% / 99.1% of 24,985 cells)
- scdeed_verification_results.csv with percentages for both perplexity settings

## How to apply

Load the Hydra count matrix (input_counts.rda) from the Zenodo deposit and build a Seurat object with CreateSeuratObject, then run NormalizeData, ScaleData, FindVariableFeatures, and RunPCA, using K=31 PCs as the pre-embedding dimension. Run scDEED with reduction.method='tsne' at the original perplexity=40 (rerun=F, Seurat defaults; null distribution built from the Permuted data with similarity_percent=0.5, dubious_cutoff=0.05, trustworthy_cutoff=0.95), then rerun scDEED at the optimized perplexity=230 (t-SNE with seed.use=100) on the same PCA space. Extract dubious_cells and trustworthy_cells from result$full_results for each perplexity and compute percentages out of 24,985 cells. Accept the reproduction if the percentages match 4.77%/92.84% at perplexity 40 and 0.51%/99.1% at perplexity 230 within numerical tolerance.

## Related tools

- **scDEED** (Classifies each cell embedding as dubious, trustworthy, or intermediate via reliability scores against a Permuted-data null distribution, and reports full_results per perplexity) — github.com/JSB-UCLA/scDEED
- **Seurat** (Builds the Seurat object (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunTSNE) that scDEED consumes) — satijalab/seurat-wrappers
- **R** (Execution environment; scDEED is installed from GitHub via devtools::install_github('JSB-UCLA/scDEED'))
- **t-SNE** (2D embedding method whose perplexity (40 vs 230) is varied to compare cell classification percentages)

## Examples

```
result = scDEED(data, K = 31, reduction.method = 'tsne', perplexity = 230, rerun = F)
```

## Evaluation signals

- Dubious percentage at perplexity 40 matches 4.77% of 24,985 cells within numerical reproduction tolerance
- Trustworthy percentage at perplexity 40 matches 92.84% within tolerance
- Dubious percentage at optimized perplexity 230 matches 0.51% and trustworthy matches 99.1% within tolerance
- Optimized perplexity (230) has fewer dubious embeddings than the original (40), consistent with scDEED's minimization-of-dubious-embeddings optimization criterion
- scdeed_verification_results.csv is written with percentages for both perplexity settings

## Limitations

- The scDEED tutorial's input_counts contains 10000 randomly selected cells from the Hydra dataset, whereas this verification uses the full 24,985 cells, so setups differ in scale
- Seurat version differences (5.2.1 vs 5.1.0) reportedly affect installation and potentially reproducibility
- scDEED currently supports t-SNE and UMAP implemented through Seurat; other dimension reduction methods require adapting the package
- Under optimized hyperparameters, differences between dubious and trustworthy embeddings may be more due to randomness than biological signals

## Evidence

- [results] At the scDEED-optimized t-SNE perplexity of 230 on the Hydra dataset, dubious cell embeddings decreased from 4.77% to 0.51%: "at a low t-SNE perplexity, cells with dubious embeddings tend to exhibit higher sparsity compared to cells with trustworthy embeddings"
- [methods] scDEED runs at Seurat default thresholds: "result = scDEED(data, K = K, reduction.method = 'tsne', perplexity = 40, rerun = F)"
- [methods] Null distribution construction via permuted data: "data.permuted = Permuted(data,  default_assay = 'RNA', slot = 'scale.data', K)"
- [readme] Cutoff semantics for classification: "For an embedding to be considered dubious, the cell's similarity score must be lower than the 5th percentile (or `dubious_cutoff` percentile) of cell embedding scores from the null distribution."
- [readme] Preprocessing pipeline on Hydra counts: "This input_counts contains 10000 randomly selected cells from the Hydra dataset."
