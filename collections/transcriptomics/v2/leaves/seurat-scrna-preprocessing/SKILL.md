---
name: seurat-scrna-preprocessing
description: Use when a raw scRNA-seq count matrix (e.g. Hydra dataset counts, Tabula
  Muris marrow counts, PBMC GSE132044 matrices) must be converted into a normalized,
  scaled, PCA-reduced Seurat object before running t-SNE/UMAP hyperparameter optimization
  or embedding-quality assessment.
when_to_use_negative:
- Input is already a normalized/processed Seurat object or a pre-computed expression
  matrix rather than a raw count matrix — re-running the full chain may be redundant
- Data are not single-cell expression counts (e.g. bulk RNA-seq or non-count data),
  where the Seurat NormalizeData/ScaleData assumptions do not apply
- Analysis requires a non-PCA pre-embedding space; scDEED's default is pre_embedding
  = 'pca', and changing it requires the separate 'Changing Input Space' procedure
  rather than this default pipeline
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: R framework providing CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures,
    RunPCA, ElbowPlot, RunTSNE, and RunUMAP; scDEED consumes Seurat objects produced
    by this pipeline
- name: scDEED
  role: Downstream consumer of the preprocessed Seurat object and its PCA space; uses
    K and the scale.data slot to compute pre-embedding distances for dubious/trustworthy
    embedding classification
  repo: github.com/JSB-UCLA/scDEED
- name: devtools
  role: Used to install the scDEED R package from GitHub before running it on the
    preprocessed object
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:3afaaa8aa3ce823efcab15db47adf2441ec37c3e64f5a2f644b549084d35e563
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMBEDR
  - R
  - Seurat
  - devtools
  - scDEED
  - scDEED R package
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# seurat-scrna-preprocessing

## Summary

Standard Seurat scRNA-seq preprocessing pipeline (CreateSeuratObject → NormalizeData → ScaleData → FindVariableFeatures → RunPCA) that produces the Seurat object and PCA pre-embedding space required by downstream embedding-quality tools such as scDEED, with the input dimension K chosen via Seurat::ElbowPlot.

## When to use

Use when a raw scRNA-seq count matrix (e.g. Hydra dataset counts, Tabula Muris marrow counts, PBMC GSE132044 matrices) must be converted into a normalized, scaled, PCA-reduced Seurat object before running t-SNE/UMAP hyperparameter optimization or embedding-quality assessment.

## When NOT to use

- Input is already a normalized/processed Seurat object or a pre-computed expression matrix rather than a raw count matrix — re-running the full chain may be redundant
- Data are not single-cell expression counts (e.g. bulk RNA-seq or non-count data), where the Seurat NormalizeData/ScaleData assumptions do not apply
- Analysis requires a non-PCA pre-embedding space; scDEED's default is pre_embedding = 'pca', and changing it requires the separate 'Changing Input Space' procedure rather than this default pipeline

## Inputs

- Raw scRNA-seq count matrix (e.g. input_counts.rda, 10000 randomly selected Hydra dataset cells)
- Tabula Muris Consortium marrow count matrix (4821 cells)
- Human PBMC count matrices from Dropseq, inDrops, and SeqWell (GSE132044)
- scDesign3-simulated count datasets

## Outputs

- Processed Seurat object with normalized, scaled data, variable features, and PCA reduction
- PCA pre-embedding space used for neighbor-distance calculations
- Chosen input dimension K from ElbowPlot inspection

## How to apply

Load the raw count matrix and build a Seurat object with CreateSeuratObject, then run the canonical chain: NormalizeData, ScaleData, FindVariableFeatures, and RunPCA; scDEED operates directly on this Seurat object and reads PCs from the scale.data slot by default (default_assay = 'RNA', slot = 'scale.data'). Determine the input dimension K (number of PCs for the pre-embedding space) by inspecting Seurat::ElbowPlot, since K must be explicitly passed to downstream functions such as scDEED(data, K = K, reduction.method = 'tsne', perplexity = 40). In benchmark settings (e.g. the 4821-cell Tabula Muris marrow matrix and 20 scDesign3-simulated datasets, or the Dropseq/inDrops/SeqWell PBMC matrices), apply the identical chain per dataset so embeddings are comparable across methods and conditions. The resulting PCA space serves as the 'pre_embedding' space (default 'pca') from which scDEED computes pre-embedding neighbor distances against which 2D-embedding neighbors are correlated. For reproducibility of optimized embeddings, seed downstream calls explicitly (e.g. seed.use = 100 in RunTSNE/RunUMAP).

## Related tools

- **Seurat** (R framework providing CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA, ElbowPlot, RunTSNE, and RunUMAP; scDEED consumes Seurat objects produced by this pipeline)
- **scDEED** (Downstream consumer of the preprocessed Seurat object and its PCA space; uses K and the scale.data slot to compute pre-embedding distances for dubious/trustworthy embedding classification) — github.com/JSB-UCLA/scDEED
- **devtools** (Used to install the scDEED R package from GitHub before running it on the preprocessed object)

## Examples

```
data = CreateSeuratObject(input_counts); data = NormalizeData(data); data = ScaleData(data); data = FindVariableFeatures(data); data = RunPCA(data); Seurat::ElbowPlot(data); result = scDEED(data, K = 8, reduction.method = 'tsne', perplexity = 40, rerun = F)
```

## Evaluation signals

- The processed object contains all four preprocessing steps applied (normalized data, scale.data slot populated, variable features identified, PCA reduction present)
- K is explicitly determined and documented via Seurat::ElbowPlot and passed to downstream calls (e.g. scDEED(data, K = K, reduction.method = 'tsne', perplexity = 40))
- The same preprocessing chain is applied identically across all datasets in a benchmark (e.g. real marrow data and all 20 scDesign3 simulations) so per-dataset comparisons are valid
- Pre-embedding distances used by scDEED are computed from the PCA space (results.PCA via Distances.pre_embedding with pre_embedding = 'pca')
- Optimized embedding regeneration is seeded for reproducibility (RunTSNE(data, perplexity = optimized, seed.use = 100); RunUMAP(..., seed.use = 100))

## Limitations

- Seurat version differences affect setup: version 5.2.1 was reportedly different from 5.1.0; the scDEED README advises against updating all packages during installation (do not select option 1 when asked which packages to update)
- K (number of PCs) must be chosen by the user; ElbowPlot is described only as a 'quick method', so K selection carries analyst subjectivity
- scDEED assumes the default input space is PCA from the scale.data slot; deviating from this normalization pipeline requires replacing the scale.data slot or adding the normalized data as another layer with a named slot

## Evidence

- [readme] data = CreateSeuratObject(input_counts); data = NormalizeData(data); data = ScaleData(data); data = FindVariableFeatures(data); data = RunPCA(data): "data = CreateSeuratObject(input_counts)
data = NormalizeData(data)
data = ScaleData(data)
data = FindVariableFeatures(data)
data = RunPCA(data)"
- [readme] K selection via ElbowPlot: "The user must specify the input dimension for the high dimensional space (`K`). There are many ways the users can decide how many dimensions to use. A quick method may be to use an elbow plot."
- [methods] scDEED uses Seurat objects; default slot is scale.data: "`slot`: (default = 'scale.data', the default slot in Seurat) the input space for PCA."
- [readme] Seurat version difference affects installation: "If you do not want to update all packages (especially Seurat, e.g., its version 5.2.1 was reportedly different from version 5.1.0), then please do not selection option 1 when asked which packages you"
- [methods] Seeded optimized embedding regeneration: "data_opt = RunUMAP(data_opt, dims = 1:K, min.dist = m, n.neighbors = n, seed.use = 100)"
