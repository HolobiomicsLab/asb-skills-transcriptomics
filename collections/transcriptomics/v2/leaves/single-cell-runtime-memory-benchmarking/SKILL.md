---
name: single-cell-runtime-memory-benchmarking
description: Use when comparing runtime, memory usage, and clustering performance
  (ARI) of scRNA-seq integration/clustering methods across increasing dataset sizes,
  or quantifying device speed-up (CPU vs GPU) of a deep-learning single-cell model.
when_to_use_negative:
- When the goal is biological interpretation of learned topics or pathway enrichment
  rather than scalability benchmarking
- When datasets are small (< a few thousand cells) and runtime differences between
  methods are negligible; scETM convergence instead depends on reaching at least ~6k
  gradient updates
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: scETM
  role: Method under benchmark; trained with UnsupervisedTrainer (n_epochs=12000,
    eval_every=1000, n_samplers=4) and timed on CPU and GPU
  repo: https://github.com/hui2000ji/scETM
- name: Harmony
  role: Competing integration method benchmarked for runtime, memory, and ARI
  repo: https://github.com/immunogenomics/harmony
- name: Scanorama
  role: Competing integration method benchmarked for runtime, memory, and ARI
  repo: https://github.com/brianhie/scanorama
- name: scVAE-GM
  role: Competing deep-learning method benchmarked for runtime, memory, and ARI
  repo: https://github.com/scvae/scvae
- name: scVI
  role: Competing deep-learning method benchmarked for runtime, memory, and ARI
  repo: https://github.com/scverse/scvi-tools
- name: LIGER
  role: Competing integrative NMF method benchmarked for runtime, memory, and ARI
  repo: https://github.com/welch-lab/liger
- name: anndata
  role: Reads and merges the AD, MDD, and MousePancreas .h5ad inputs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:61a4e1d3cb28546ad74177b9b8a11d1c7ffcb1382664d0d794e1e4e8cb5133c5
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
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# single-cell-runtime-memory-benchmarking

## Summary

Benchmark the scalability of single-cell integration methods (scETM vs Harmony, Scanorama, Seurat, scVAE-GM, scVI, scVI-LD, LIGER) by recording wall-clock runtime, peak memory, and Adjusted Rand Index across subsampled merged AD+MDD datasets, plus CPU-vs-GPU timing of scETM on Mouse Pancreas. Reproduces Fig 3a-c of the scETM paper.

## When to use

Use when comparing runtime, memory usage, and clustering performance (ARI) of scRNA-seq integration/clustering methods across increasing dataset sizes, or quantifying device speed-up (CPU vs GPU) of a deep-learning single-cell model.

## When NOT to use

- When the goal is biological interpretation of learned topics or pathway enrichment rather than scalability benchmarking
- When datasets are small (< a few thousand cells) and runtime differences between methods are negligible; scETM convergence instead depends on reaching at least ~6k gradient updates

## Inputs

- AD and MDD scRNA-seq AnnData (.h5ad) objects with cell type annotations
- MousePancreas.h5ad AnnData object (1886 cells) for CPU-vs-GPU timing
- Commands from the scETM repository's benchmarking scripts folder

## Outputs

- benchmark_results.csv with runtime, peak memory, and ARI per method/subsample pair
- Trend plot of runtime/memory/ARI vs dataset size reproducing Fig 3a-c
- CPU vs GPU timing table for 1000 scETM training steps on Mouse Pancreas

## How to apply

Load the AD and MDD scRNA-seq AnnData objects, merge them keeping only shared/homologous genes, and select the 3000 most variable genes as the feature space. Subsample cells without replacement to four sizes (14,000, 28,000, 70,000, and 148,247 cells) for the scalability benchmark. Train scETM with UnsupervisedTrainer (n_epochs=12000, eval_every=1000, n_samplers=4) and run competing methods (Harmony, Scanorama, Seurat, scVAE-GM, scVI, scVI-LD, LIGER) using the commands in the scETM repository's benchmarking scripts folder. For every method/subsample pair, record wall-clock runtime, peak memory, and the ARI of cell clustering against annotated cell type labels. Separately, time scETM for 1000 training steps on the Mouse Pancreas dataset (1886 cells) on CPU and on GPU to quantify the device speed-up; compile everything into benchmark_results.csv and a trend plot matching Fig 3.

## Related tools

- **scETM** (Method under benchmark; trained with UnsupervisedTrainer (n_epochs=12000, eval_every=1000, n_samplers=4) and timed on CPU and GPU) — https://github.com/hui2000ji/scETM
- **Harmony** (Competing integration method benchmarked for runtime, memory, and ARI) — https://github.com/immunogenomics/harmony
- **Scanorama** (Competing integration method benchmarked for runtime, memory, and ARI) — https://github.com/brianhie/scanorama
- **scVAE-GM** (Competing deep-learning method benchmarked for runtime, memory, and ARI) — https://github.com/scvae/scvae
- **scVI** (Competing deep-learning method benchmarked for runtime, memory, and ARI) — https://github.com/scverse/scvi-tools
- **LIGER** (Competing integrative NMF method benchmarked for runtime, memory, and ARI) — https://github.com/welch-lab/liger
- **anndata** (Reads and merges the AD, MDD, and MousePancreas .h5ad inputs)

## Examples

```
from scETM import scETM, UnsupervisedTrainer; model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True); UnsupervisedTrainer(model, mp).train(n_epochs=12000, eval_every=1000, n_samplers=4)
```

## Evaluation signals

- On the largest subsample (148,247 cells), scETM runs 3–4x faster than scVI and scVI-LD and over 10x faster than scVAE-GM, with the shortest runtime among deep-learning based models
- Mouse Pancreas CPU-vs-GPU timing for 1000 training steps approximates the reported 13x speed-up (~650 s CPU vs ~50 s GPU)
- Every method/subsample pair has non-null wall-clock runtime and peak memory entries in benchmark_results.csv
- ARI is computed against annotated cell type labels for all methods at all four subsample sizes (14,000 / 28,000 / 70,000 / 148,247 cells)
- Merged dataset retains only shared/homologous genes restricted to the 3000 most variable genes

## Limitations

- To enable GPU computing, PyTorch with GPU support must be installed before installing scETM
- scETM is trained via batched gradient descent: for small datasets with large mini-batch sizes, one epoch yields one gradient step, so epochs must be set to give at least 6k gradient updates or the model may not converge
- Benchmark results depend on matching hardware and the specific versions of the 7 competing state-of-the-art methods; scETM's scalability claim is demonstrated to over 10^6 cells in the paper but this skill benchmarks up to 148,247 cells

## Evidence

- [results] On the largest benchmark dataset (148,247 cells), scETM ran 3–4 times faster than scVI and scVI-LD, and over 10 times faster than scVAE-GM: [evidence span withheld — anchor status: not_found]
- [abstract] scETM is scalable to over 10^6 cells: "scETM is scalable to over 10 6 cells and confers remarkable cross-tissue and cross-species zero-shot transfer-learning performance."
- [readme] scETM requires a cells-by-genes matrix as AnnData input: "scETM requires a cells-by-genes matrix `adata` as input, in the format of an AnnData object"
- [readme] Recommended training loop with UnsupervisedTrainer: "trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4)"
- [readme] Install PyTorch with GPU support before scETM: "To enable GPU computing (which significantly boosts the performance), please install [PyTorch](https://pytorch.org/) with GPU support **before** installing scETM."
- [readme] Gradient-update guidance for small datasets: "it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model. Otherwise, the model may not have converged when training stops."
