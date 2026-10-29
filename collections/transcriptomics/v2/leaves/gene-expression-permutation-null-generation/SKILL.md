---
name: gene-expression-permutation-null-generation
description: Use when a scDEED-style reliability analysis needs a null distribution
  of cell-embedding similarity scores to classify 2D t-SNE/UMAP embeddings as dubious
  or trustworthy.
when_to_use_negative:
- The goal is embedding stability assessment rather than a signalless null test —
  DynamicViz-style stability evaluation does not use gene permutation
- Input is not a Seurat object with a scaled gene-by-cell matrix (scale.data slot);
  Permuted expects this structure per the scDEED methods
- Comparing embeddings across two different real datasets rather than testing one
  dataset against a gene-permuted null
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scDEED
  role: Provides the Permuted function that shuffles each gene row of the Seurat scale.data
    slot and the downstream similarity/classification machinery for the null test
  repo: https://github.com/JSB-UCLA/scDEED
- name: Seurat
  role: Provides the Seurat object, scale.data slot, and PCA preprocessing (RunPCA)
    applied identically to original and permuted data
  repo: https://github.com/satijalab/seurat-wrappers
- name: R
  role: Runtime environment; scDEED is installed from GitHub via devtools
- name: devtools
  role: Installs the scDEED R package from GitHub
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:9562b7de2bc5d6973d236143163ded78729759fbd6768589712313f52b29a0c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scDEED
  tools:
  - R
  - Seurat
  - devtools
  - distances
  - scDEED
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Gene-expression permutation null generation

## Summary

Generates a signalless null dataset for scDEED by permuting (shuffling) each gene row of the Seurat scale.data slot with a fixed seed, then re-running PCA to obtain permuted pre-embedding distances against which per-cell reliability scores are tested.

## When to use

Use when a scDEED-style reliability analysis needs a null distribution of cell-embedding similarity scores to classify 2D t-SNE/UMAP embeddings as dubious or trustworthy.

## When NOT to use

- The goal is embedding stability assessment rather than a signalless null test — DynamicViz-style stability evaluation does not use gene permutation
- Input is not a Seurat object with a scaled gene-by-cell matrix (scale.data slot); Permuted expects this structure per the scDEED methods
- Comparing embeddings across two different real datasets rather than testing one dataset against a gene-permuted null

## Inputs

- Seurat object with normalized and scaled single-cell count matrix (scale.data slot), e.g., input_counts.rda from 10000 randomly selected Hydra dataset cells
- Number of PCs K for the pre-embedding space (e.g., K = 8)
- Random seed for permutation reproducibility (e.g., seed 1000)

## Outputs

- Permuted Seurat object with shuffled scale.data slot and recomputed PCA embedding
- Permuted pre-embedding and reduced-dimension nearest-neighbor distance matrices
- Null distribution of per-cell Pearson correlation similarity scores (rho_permuted) used for 5th/95th-percentile dubious/trustworthy classification

## How to apply

After running the Seurat chain (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA) on the input_counts Hydra dataset, apply the Permuted function to shuffle each row (gene) of the scale.data slot using a fixed seed (e.g., seed 1000 as in the vignette permutation code) for reproducibility. Recompute PCA (and the 2D embedding across the perplexity grid, e.g., 20–200 for t-SNE) on the permuted matrix so that permuted cells undergo the identical preprocessing path as original cells. Compute cell-by-cell ordered nearest-neighbor distances in both the original and permuted pre-embedding (PCA) and 2D spaces, then form per-cell Pearson correlation similarity scores with similarity_percent = 0.5. Cells whose similarity scores fall below the 5th percentile of the permuted null score distribution are labeled dubious and those above the 95th percentile trustworthy; minimizing the number of dubious embeddings guides hyperparameter choice (e.g., optimized t-SNE perplexity of 200 on the Hydra data).

## Related tools

- **scDEED** (Provides the Permuted function that shuffles each gene row of the Seurat scale.data slot and the downstream similarity/classification machinery for the null test) — https://github.com/JSB-UCLA/scDEED
- **Seurat** (Provides the Seurat object, scale.data slot, and PCA preprocessing (RunPCA) applied identically to original and permuted data) — https://github.com/satijalab/seurat-wrappers
- **R** (Runtime environment; scDEED is installed from GitHub via devtools)
- **devtools** (Installs the scDEED R package from GitHub)

## Examples

```
library(devtools); devtools::install_github("JSB-UCLA/scDEED"); data.permuted = Permuted(data, default_assay = 'RNA', slot = 'scale.data', K)
```

## Evaluation signals

- Permuted PCA is computed from a matrix whose each gene row is a shuffle of the original scale.data row — row-wise gene means/variances should match the original per gene
- Percentile semantics verified against the package's tsne_table reference: dubious cells = scores below the 5th-percentile cutoff of permuted scores, trustworthy = above the 95th percentile
- Dubious-cell counts per perplexity reproduce the reported Hydra results, with a minimum of 15 dubious cells at the optimized perplexity of 200
- Fixed seed (e.g., 1000) yields reproducible permuted scores across reruns
- similarity_percent = 0.5 (50% of neighbors) is used consistently for both original and permuted similarity scores

## Limitations

- Difference between dubious and trustworthy embeddings may be more due to randomness than biological signals under optimized hyperparameters
- Users of the scDEED R package may need to increase or decrease the percentile threshold based on knowledge and preference; defaults are dubious_cutoff = 0.05 and trustworthy_cutoff = 0.95
- Seurat version differences (5.2.1 vs 5.1.0) can affect installation and reproducibility of the preprocessing chain

## Evidence

- [methods] Permuted function shuffles each gene row of the scale.data slot to build the null: "data.permuted = Permuted(data,  default_assay = 'RNA', slot = 'scale.data', K)"
- [readme] Default cutoff semantics for the null distribution classification: "`dubious_cutoff`: (default = 0.05) This sets the cutoff for classifying cells."
- [abstract] Cell classification compares similarity scores to percentiles of the permuted null distribution: "scDEED identifies the cell embeddings with low reliability scores as dubious and those with high reliability scores as trustworthy"
- [readme] Similarity score is the Pearson correlation of ordered pre-embedding vs 2D-embedding neighbor distances: "We measure this by comparing the pearson correlation of ordered distances between a cell and its nearest neighbors in the pre-embedding space and its nearest neighbors in the post-embedding space."
- [readme] Preprocessing chain applied to the Hydra input before permutation: "data = CreateSeuratObject(input_counts); data = NormalizeData(data); data = ScaleData(data); data = FindVariableFeatures(data); data = RunPCA(data)"
