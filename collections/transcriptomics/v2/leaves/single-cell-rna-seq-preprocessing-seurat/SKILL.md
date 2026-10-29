---
name: single-cell-rna-seq-preprocessing-seurat
description: Use when a raw single-cell RNA-seq count matrix (e.g., GEO-supplied counts
  such as GSE141259 or the Hydra input_counts matrix) must be normalized, scaled,
  variable-feature-selected, and PCA-reduced before running scDEED or t-SNE/UMAP embedding.
when_to_use_negative:
- Input is already a processed Seurat object with normalization, scaling, and PCA
  completed — rerunning this pipeline can alter the scale.data slot that scDEED uses
  as PCA input
- Data are not single-cell expression counts (e.g., bulk RNA-seq, spatial transcriptomics
  not in Seurat-compatible format)
- You only need clustering or cell-type annotation without 2D embedding quality assessment
  — this pipeline alone does not perform dubious-embedding detection
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: Seurat
  role: R framework used to create the Seurat object and run NormalizeData, ScaleData,
    FindVariableFeatures, RunPCA, and ElbowPlot prior to scDEED
- name: scDEED
  role: Downstream package consuming the preprocessed Seurat object to detect dubious
    cell embeddings and optimize t-SNE/UMAP hyperparameters
  repo: github.com/JSB-UCLA/scDEED
- name: devtools
  role: Used to install scDEED from GitHub (devtools::install_github('JSB-UCLA/scDEED'))
- name: doParallel
  role: Registers parallel cores (e.g., cores=6) to speed up scDEED grid searches
    over hyperparameters
- name: R
  role: Execution environment for the Seurat/scDEED pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:0ebddefcb6d8e5c18d8f176912d3b97ba190f9c8116fc29a29a8cc570795ceab
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - devtools
  - doParallel
  - scDEED
  - scDEED R package
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-rna-seq-preprocessing-seurat

## Summary

Standard Seurat scRNA-seq preprocessing pipeline (CreateSeuratObject → NormalizeData → ScaleData → FindVariableFeatures → RunPCA) that prepares a raw count matrix as a PCA-reduced Seurat object suitable for downstream 2D embedding and hyperparameter optimization with scDEED.

## When to use

Use when a raw single-cell RNA-seq count matrix (e.g., GEO-supplied counts such as GSE141259 or the Hydra input_counts matrix) must be normalized, scaled, variable-feature-selected, and PCA-reduced before running scDEED or t-SNE/UMAP embedding.

## When NOT to use

- Input is already a processed Seurat object with normalization, scaling, and PCA completed — rerunning this pipeline can alter the scale.data slot that scDEED uses as PCA input
- Data are not single-cell expression counts (e.g., bulk RNA-seq, spatial transcriptomics not in Seurat-compatible format)
- You only need clustering or cell-type annotation without 2D embedding quality assessment — this pipeline alone does not perform dubious-embedding detection

## Inputs

- Raw scRNA-seq count matrix (e.g., .rda file such as input_counts.rda or GEO GSE141259 counts for 29,297 alveolar cells)
- User-specified number of PCs K (selected via ElbowPlot)

## Outputs

- Processed Seurat object with normalized counts, scaled data, variable features, and PCA embedding
- ElbowPlot used to select the input dimension K
- Seurat object ready for scDEED analysis (reduction.method='tsne' or 'umap')

## How to apply

Load the count matrix and construct a Seurat object with CreateSeuratObject(input_counts), then apply NormalizeData, ScaleData (which fills the default 'scale.data' slot used for PCA), FindVariableFeatures, and RunPCA. Choose the number of PCs K with Seurat::ElbowPlot(data), since K is a required user input to scDEED and defines the pre-embedding space (default pre_embedding = 'pca'). The resulting Seurat object is the direct input to scDEED, which supports reduction.method='tsne' (perplexity sweeps) or reduction.method='umap' (min.dist and n_neighbors sweeps) with seed.use = 100 for reproducibility. Rationale: scDEED requires a Seurat object and computes reliability scores by comparing Pearson correlations of cell-neighbor distances in the PCA pre-embedding space versus the 2D embedding, so the preprocessing quality directly determines the dubious/trustworthy classifications at the default dubious_cutoff=0.05 and trustworthy_cutoff=0.95 percentiles of the null distribution.

## Related tools

- **Seurat** (R framework used to create the Seurat object and run NormalizeData, ScaleData, FindVariableFeatures, RunPCA, and ElbowPlot prior to scDEED)
- **scDEED** (Downstream package consuming the preprocessed Seurat object to detect dubious cell embeddings and optimize t-SNE/UMAP hyperparameters) — github.com/JSB-UCLA/scDEED
- **devtools** (Used to install scDEED from GitHub (devtools::install_github('JSB-UCLA/scDEED')))
- **doParallel** (Registers parallel cores (e.g., cores=6) to speed up scDEED grid searches over hyperparameters)
- **R** (Execution environment for the Seurat/scDEED pipeline)

## Examples

```
data = CreateSeuratObject(input_counts); data = NormalizeData(data); data = ScaleData(data); data = FindVariableFeatures(data); data = RunPCA(data); result = scDEED(data, K = K, reduction.method = 'umap')
```

## Evaluation signals

- The Seurat object contains populated scale.data slot, variable feature set, and a PCA reduction; ElbowPlot should show an inflection justifying the chosen K
- Preprocessed object loads into scDEED without error (scDEED uses Seurat objects and defaults to pre_embedding='pca' with slot='scale.data')
- Downstream scDEED output num_dubious data.frame should be non-empty and contain hyperparameter settings with dubious cell counts
- Reproducibility check: downstream RunUMAP with seed.use=100 should regenerate consistent embeddings across reruns
- When applied to GSE141259 (29,297 cells), the pipeline should support reproducing the reported optima (marginal n.neighbors=5 with 2.28% dubious, marginal min.dist=0.6 with 2.45% dubious, joint min.dist=0.7/n.neighbors=6 with 2.04% dubious and 58.84% trustworthy)

## Limitations

- scDEED only supports t-SNE and UMAP reductions implemented through Seurat by default; adapting to other dimension reduction methods requires extra work described in the package tutorial
- The two-hyperparameter UMAP grid search over (min.dist, n_neighbors) can be computationally expensive depending on user resources
- Seurat version differences matter: version 5.2.1 was reportedly different from version 5.1.0, so users should avoid updating all packages during scDEED installation
- The number of dubious embeddings may show a complex trend as a hyperparameter value increases, indicating complex cell topology and requiring caution in interpreting 2D visualization

## Evidence

- [readme] Standard Seurat preprocessing chain: "data = CreateSeuratObject(input_counts)
data = NormalizeData(data)
data = ScaleData(data)
data = FindVariableFeatures(data)
data = RunPCA(data)"
- [readme] K selection via ElbowPlot: "The user must specify the input dimension for the high dimensional space (K). There are many ways the users can decide how many dimensions to use. A quick method may be to use an elbow plot."
- [methods] scDEED input requirements: "scDEED uses Seurat objects."
- [readme] PCA pre-embedding space and slot: "By default, the pre-embedding space is the PCA space, although this can also be changed if the user wishes"
- [readme] UMAP grid search cost: "The grid search (due to two hyperparameters, rather than one) can be computationally expensive."
