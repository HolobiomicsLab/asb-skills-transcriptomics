---
name: seurat-dimensionality-reduction-processing
description: Use when raw single-cell counts need to be converted into a Seurat object
  with normalized, scaled data, variable features, PCA, and a t-SNE or UMAP 2D embedding
  — e.g. before detecting dubious embeddings or optimizing hyperparameters with scDEED.
when_to_use_negative:
- Input is already a processed Seurat object with the required PCA and 2D reduction
  — re-running the full pipeline would discard prior normalization choices
- The desired embedding method is not t-SNE or UMAP via Seurat (scDEED supports these
  natively; other methods require adapting the pipeline)
- Data are not single-cell expression counts (e.g. bulk RNA-seq or feature tables),
  where the Seurat single-cell normalization workflow does not apply
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Provides the object model and pipeline functions (CreateSeuratObject, NormalizeData,
    ScaleData, FindVariableFeatures, RunPCA, RunTSNE, RunUMAP); scDEED consumes Seurat
    objects
  repo: https://github.com/satijalab/seurat-wrappers
- name: devtools
  role: Used to install the scDEED R package from GitHub into the R environment that
    runs the Seurat pipeline
  repo: https://github.com/JSB-UCLA/scDEED
- name: scDEED R package
  role: Downstream consumer of the processed Seurat object; classifies embeddings
    and optimizes t-SNE/UMAP hyperparameters
  repo: https://github.com/JSB-UCLA/scDEED
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:65cadb060079d5ef14a9b6a40f34646700dfb0f49ef37d1f259746fcd66a78ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satijalab/seurat-wrappers
  tools:
  - R
  - Seurat
  - devtools
  - scDEED R package
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# seurat-dimensionality-reduction-processing

## Summary

Build and process a Seurat object through the standard scRNA-seq dimensionality-reduction pipeline (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA, then RunTSNE or RunUMAP) so that a pre-embedding space (PCA) and a 2D embedding are available for downstream evaluation, e.g. with scDEED.

## When to use

Use when raw single-cell counts need to be converted into a Seurat object with normalized, scaled data, variable features, PCA, and a t-SNE or UMAP 2D embedding — e.g. before detecting dubious embeddings or optimizing hyperparameters with scDEED.

## When NOT to use

- Input is already a processed Seurat object with the required PCA and 2D reduction — re-running the full pipeline would discard prior normalization choices
- The desired embedding method is not t-SNE or UMAP via Seurat (scDEED supports these natively; other methods require adapting the pipeline)
- Data are not single-cell expression counts (e.g. bulk RNA-seq or feature tables), where the Seurat single-cell normalization workflow does not apply

## Inputs

- Raw single-cell count matrix (e.g. input_counts.rda from the Hydra dataset)
- Seurat object with normalized/scaled data and computed PCA (e.g. GSE125881 CAR-T CD8+ Seurat object with 15 PCs)
- User-specified number of PCs K for the pre-embedding space

## Outputs

- Processed Seurat object containing PCA and t-SNE or UMAP reductions
- ElbowPlot used to select K
- 2D cell embeddings ready for downstream classification (dubious/trustworthy/intermediate)

## How to apply

Load a raw count matrix (e.g. input_counts.rda) and construct the object with CreateSeuratObject(input_counts), then run NormalizeData, ScaleData, FindVariableFeatures, and RunPCA in sequence. Choose the number of PCs (K) for the pre-embedding space using Seurat::ElbowPlot. Run RunTSNE (with a chosen perplexity) or RunUMAP (with min.dist and n.neighbors, e.g. seed.use = 100 for reproducibility) on dims = 1:K to obtain the 2D embedding. The resulting Seurat object is the direct input to scDEED's scDEED(), Permuted(), and Distances functions, which compare pre-embedding (PCA) neighbor distances against 2D-embedding neighbor distances using Pearson correlation of ordered distances. Default slot is 'scale.data' and default pre_embedding space is 'pca'; both can be changed if a different normalization pipeline is used.

## Related tools

- **Seurat** (Provides the object model and pipeline functions (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunTSNE, RunUMAP); scDEED consumes Seurat objects) — https://github.com/satijalab/seurat-wrappers
- **devtools** (Used to install the scDEED R package from GitHub into the R environment that runs the Seurat pipeline) — https://github.com/JSB-UCLA/scDEED
- **scDEED R package** (Downstream consumer of the processed Seurat object; classifies embeddings and optimizes t-SNE/UMAP hyperparameters) — https://github.com/JSB-UCLA/scDEED

## Examples

```
library(devtools); devtools::install_github("JSB-UCLA/scDEED"); data = CreateSeuratObject(input_counts); data = NormalizeData(data); data = ScaleData(data); data = FindVariableFeatures(data); data = RunPCA(data); data = RunTSNE(data)
```

## Evaluation signals

- ElbowPlot shows a clear elbow justifying the chosen K for the PCA pre-embedding space
- Seurat object contains both a 'pca' reduction (dims 1:K) and the requested 2D reduction ('tsne' or 'umap')
- scDEED downstream runs return non-empty full_results rows with dubious/trustworthy/intermediate cell classifications at each hyperparameter setting
- For the CAR-T dataset, computed percentages reproduce the reported values (~1.81% dubious / 83.99% trustworthy at perplexity 30; ~0.09% dubious / 95.87% trustworthy at perplexity 750) within tolerance
- UMAP runs use seed.use = 100 so embeddings are reproducible

## Limitations

- Seurat version differences affect behavior/installation: version 5.2.1 was reportedly different from version 5.1.0 when installing scDEED dependencies
- The scDEED grid search for UMAP (two hyperparameters, min.dist and n.neighbors) can be computationally expensive
- If the default normalization pipeline is not used, users must replace the scale.data slot or add normalized data as another layer and specify the slot name

## Evidence

- [methods] scDEED uses Seurat objects.: "scDEED uses Seurat objects."
- [readme] Seurat pipeline steps: "data = CreateSeuratObject(input_counts)
data = NormalizeData(data)
data = ScaleData(data)
data = FindVariableFeatures(data)
data = RunPCA(data)"
- [readme] K chosen via elbow plot: "The user must specify the input dimension for the high dimensional space (`K`). There are many ways the users can decide how many dimensions to use. A quick method may be to use an elbow plot."
- [methods] UMAP reproducibility seed: "data_opt = RunUMAP(data_opt, dims = 1:K, min.dist = m, n.neighbors = n, seed.use = 100)"
- [readme] default slot and pre-embedding space: "`slot`: (default = 'scale.data', the default slot in Seurat) the input space for PCA."
- [readme] Seurat version caveat: "its version 5.2.1 was reportedly different from version 5.1.0"
