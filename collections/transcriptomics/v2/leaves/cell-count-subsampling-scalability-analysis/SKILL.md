---
name: cell-count-subsampling-scalability-analysis
description: Use when you must quantify the runtime, memory, and clustering-performance
  scaling of an integration/clustering method across increasing numbers of cells and
  compare it with alternatives.
when_to_use_negative:
- The goal is biological interpretation of learned topics/pathway associations (p-scETM,
  GSEA) rather than computational scalability measurement.
- The task is cross-dataset transfer learning (zero-shot transfer or fine-tuning to
  a target dataset) rather than same-data speed/accuracy benchmarking.
- The dataset is too small to support meaningful cell-count subsampling (e.g. Mouse
  Pancreas' 1886 cells alone); use it only for the CPU-vs-GPU timing arm.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scETM
  role: Method under test; trained with UnsupervisedTrainer on each cell-count subsample
    and timed on CPU vs GPU
  repo: https://github.com/hui2000ji/scETM
- name: Harmony
  role: Competing integration method run on each subsample via the scETM benchmarking
    scripts
  repo: https://github.com/immunogenomics/harmony
- name: Scanorama
  role: Competing integration method run on each subsample via the scETM benchmarking
    scripts
  repo: https://github.com/brianhie/scanorama
- name: scVI
  role: Deep-learning competitor whose runtime scETM should beat by 3-4x at 148,247
    cells; scVI-LD is its low-dimension variant
  repo: https://github.com/scverse/scvi-tools
- name: scVAE-GM
  role: Deep-learning competitor expected to run over 10x slower than scETM at the
    largest size
  repo: https://github.com/scvae/scvae
- name: LIGER
  role: Integrative-NMF competitor run on each subsample via the scETM benchmarking
    scripts
  repo: https://github.com/welch-lab/liger
- name: anndata
  role: Reads the .h5ad inputs and holds merged/subsampled cells-by-genes matrices
    for all methods
  repo: https://github.com/hui2000ji/scETM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:9be84071b6cd33ee09ec24be38039e9929b70ccb18fb5a15881d3cd4e9e42464
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hui2000ji/scETM
  tools:
  - Harmony
  - LIGER
  - Scanorama
  - Seurat
  - anndata
  - scETM
  - scVAE-GM
  - scVI
  - scVI-LD
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# cell-count-subsampling-scalability-analysis

## Summary

Benchmarks how scETM's wall-clock runtime, peak memory, and clustering accuracy (ARI) scale with cell count against competing single-cell integration methods on merged AD+MDD scRNA-seq data, plus CPU-vs-GPU timing on Mouse Pancreas.

## When to use

Use when you must quantify the runtime, memory, and clustering-performance scaling of an integration/clustering method across increasing numbers of cells and compare it with state-of-the-art alternatives.

## When NOT to use

- The goal is biological interpretation of learned topics/pathway associations (p-scETM, GSEA) rather than computational scalability measurement.
- The task is cross-dataset transfer learning (zero-shot transfer or fine-tuning to a target dataset) rather than same-data speed/accuracy benchmarking.
- The dataset is too small to support meaningful cell-count subsampling (e.g. Mouse Pancreas' 1886 cells alone); use it only for the CPU-vs-GPU timing arm.

## Inputs

- AD scRNA-seq AnnData (.h5ad) with cell type annotations
- MDD scRNA-seq AnnData (.h5ad) with cell type annotations
- MousePancreas.h5ad AnnData for CPU-vs-GPU timing
- Annotated cell type labels for ARI computation
- Benchmarking scripts from the scETM repository (scripts folder)

## Outputs

- benchmark_results.csv with runtime, peak memory, and ARI for every method/subsample pair
- CPU vs GPU timing table for 1000 scETM training steps on Mouse Pancreas
- Scalability trend plots matching Fig 3a-c

## How to apply

Load the AD and MDD AnnData objects, merge them keeping only shared/homologous genes, and select the 3000 most variable genes as the feature space. Subsample cells without replacement to four sizes (14,000, 28,000, 70,000, and 148,247 cells) so runtime trends can be fit across an order of magnitude. For each subsample, train scETM with UnsupervisedTrainer (n_epochs=12000, eval_every=1000, n_samplers=4) and run the competing methods (Harmony, Scanorama, Seurat, scVAE-GM, scVI, scVI-LD, LIGER) using the scripts provided in the scETM repository. Record wall-clock runtime and peak memory for every method/subsample pair, and compute the Adjusted Rand Index of each method's cell clustering against annotated cell type labels. Separately time scETM for 1000 training steps on Mouse Pancreas on CPU and GPU to quantify the device speed-up, then compile everything into benchmark_results.csv and reproduce the Fig 3 trend plots.

## Related tools

- **scETM** (Method under test; trained with UnsupervisedTrainer on each cell-count subsample and timed on CPU vs GPU) — https://github.com/hui2000ji/scETM
- **Harmony** (Competing integration method run on each subsample via the scETM benchmarking scripts) — https://github.com/immunogenomics/harmony
- **Scanorama** (Competing integration method run on each subsample via the scETM benchmarking scripts) — https://github.com/brianhie/scanorama
- **scVI** (Deep-learning competitor whose runtime scETM should beat by 3-4x at 148,247 cells; scVI-LD is its low-dimension variant) — https://github.com/scverse/scvi-tools
- **scVAE-GM** (Deep-learning competitor expected to run over 10x slower than scETM at the largest size) — https://github.com/scvae/scvae
- **LIGER** (Integrative-NMF competitor run on each subsample via the scETM benchmarking scripts) — https://github.com/welch-lab/liger
- **anndata** (Reads the .h5ad inputs and holds merged/subsampled cells-by-genes matrices for all methods) — https://github.com/hui2000ji/scETM

## Examples

```
from scETM import scETM, UnsupervisedTrainer, evaluate; import anndata; mp = anndata.read_h5ad("MousePancreas.h5ad"); model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True); UnsupervisedTrainer(model, mp, ckpt_dir="../results").train(n_epochs=12000, eval_every=1000, n_samplers=4)
```

## Evaluation signals

- On the 148,247-cell subsample, scETM's wall-clock runtime is 3-4x lower than scVI/scVI-LD and more than 10x lower than scVAE-GM (shortest among deep-learning models).
- Mouse Pancreas CPU-vs-GPU timing for 1000 training steps approximates the reported ~13x speed-up (about 650 s on CPU vs ~50 s on GPU).
- The scaling curve covers exactly four subsample sizes: 14,000, 28,000, 70,000, and 148,247 cells.
- benchmark_results.csv contains runtime, peak memory, and ARI rows for every method/subsample pair, with ARI computed against the annotated cell type labels.
- scETM was trained with n_epochs=12000, eval_every=1000, n_samplers=4 for every subsample (i.e. at least ~6k gradient updates were reached).

## Limitations

- scETM handles only a single categorical batch variable; multi-batch-variable correction is future work, which constrains which merged datasets can be integrated.
- GPU acceleration requires installing GPU-enabled PyTorch before scETM; without it, CPU-only timings will not match the reported speed-ups.
- For small datasets and large mini-batch sizes, one epoch is one gradient step, so epochs must be set to yield at least 6k gradient updates or the model may not converge when timing stops.

## Evidence

- [abstract] scETM is scalable to over 10^6 cells: "scETM is scalable to over 10 6 cells and confers remarkable cross-tissue and cross-species zero-shot transfer-learning performance."
- [readme] Training parameters for scalability runs: "trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4)"
- [readme] Input format requirement: "scETM requires a cells-by-genes matrix `adata` as input, in the format of an AnnData object"
- [readme] Epoch-to-gradient-update caveat for timing: "it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model. Otherwise, the model may not have converged when training stops."
- [other] Benchmarking scope against 7 methods: "This repository includes detailed instructions for installation and requirements, demos, and scripts used for the benchmarking of 7 other state-of-art methods."
