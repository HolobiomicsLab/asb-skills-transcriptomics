---
name: multimodal-single-cell-embedding-generation
description: Use when the input is multimodal single-cell omics count data (RNA plus
  ADT or ATAC measured on the same cells) and the goal is an unsupervised shared latent
  space for clustering cells.
when_to_use_negative:
- The tutorial only explains package usage and does not recommend best parameters,
  so do not rely on tutorial defaults when reproducing paper benchmarks — use dataset-specific
  parameters from the paper instead
- Data are unimodal (a single feature table with no second modality); the framework
  concatenates modality-specific encoders and expects a list of modalities across
  the same cells
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: snapccess
  role: Python package that trains multimodality VAEs and emits embedding snapshots
    for consensus clustering
  repo: https://github.com/PYangLab/SnapCCESS
- name: SnapCCESS (R wrapper)
  role: R interface that installs/calls the Python package through reticulate; expects
    features-by-cells modality matrices before preprocessing
  repo: https://github.com/yulijia/SnapCCESS
- name: Python
  role: Runtime for the snapccess package and tutorial notebook
  repo: https://github.com/PYangLab/SnapCCESS
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_003
  - task_008
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad382
    title: Ensemble deep learning of embeddings for clustering multimodal single-cell
      omics data
  source_content_hash: sha256:4afac7631c22fa8970447f20360f034163e17ba1459a8612471e4c9efa8f6eaa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PYangLab/SnapCCESS
  tools:
  - Python
  - SnapCCESS (R wrapper)
  - snapccess
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# multimodal-single-cell-embedding-generation

## Summary

Generates cell embeddings from multimodal single-cell omics data (e.g. CITE-seq RNA/ADT, SHARE-seq/SNARE-seq RNA/ATAC) by creating snapshots of multimodality variational autoencoder embeddings, which are then coupled with clustering algorithms to produce consensus cell clusterings.

## When to use

Use when the input is multimodal single-cell omics count data (RNA plus ADT or ATAC measured on the same cells) and the goal is an unsupervised shared latent space for clustering cells.

## When NOT to use

- The tutorial only explains package usage and does not recommend best parameters, so do not rely on tutorial defaults when reproducing paper benchmarks — use dataset-specific parameters from the paper instead
- Data are unimodal (a single feature table with no second modality); the framework concatenates modality-specific encoders and expects a list of modalities across the same cells

## Inputs

- Multimodal single-cell omics count matrices (e.g. CITE-seq RNA and ADT, SHARE-seq/SNARE-seq RNA and ATAC), arranged features by cells for the same set of cells
- Combined PyTorch data loader consumed by the snapccess training routine
- Tutorial example data in tutorials/in/ (small CITE-seq RNA, ADT, and cell type files)

## Outputs

- Snapshot cell embeddings from the multimodality VAE ensemble (e.g. embeddings.npy)
- Consensus cluster labels for every cell (e.g. consensus_labels.csv)
- ARI/NMI benchmark tables (e.g. ari_nmi_ensemble_benchmark.csv, figure5_ari_nmi_benchmark.csv)

## How to apply

Install the Python package with `pip install snapccess --index-url https://pypi.org/simple` (or the R wrapper via `remotes::install_github(repo='yulijia/SnapCCESS',branch='main',subdir='snapccess-r/SnapCCESS')`). Arrange each modality as a features-by-cells matrix measured across the same cells, and follow the tutorial notebook `an_example_of_generate_embedding_using_SnapCCESS_python_version.ipynb` to build the combined PyTorch data loader expected by the training routine. Train modality-specific variational autoencoder encoders joined at a concatenated latent space — a fully connected layer with 100 neurons performs joint learning — under a shifted-cosine cyclic learning-rate scheduler that emits one embedding snapshot per learning-rate cycle; sweep ensemble sizes up to 100 with per-model budgets of 1, 2, and 10 epochs (SnapCCESS plateaus near ensemble size ~50 and performs comparably to a conventional VAE ensemble with just one epoch). Couple each snapshot embedding with clustering algorithms (k-means, SIMLR, or spectral clustering) and combine the per-embedding clusterings with the clue cl_consensus fixed-point loop to obtain consensus cluster labels. Because the tutorial does not recommend parameters, take dataset-specific settings from the published paper (doi:10.1093/bioinformatics/btad382).

## Related tools

- **snapccess** (Python package that trains multimodality VAEs and emits embedding snapshots for consensus clustering) — https://github.com/PYangLab/SnapCCESS
- **SnapCCESS (R wrapper)** (R interface that installs/calls the Python package through reticulate; expects features-by-cells modality matrices before preprocessing) — https://github.com/yulijia/SnapCCESS
- **Python** (Runtime for the snapccess package and tutorial notebook) — https://github.com/PYangLab/SnapCCESS

## Examples

```
pip install snapccess --index-url https://pypi.org/simple
```

## Evaluation signals

- Every cell in the input dataset receives exactly one consensus cluster label
- SnapCCESS clustering performance is comparable to the conventional VAE ensemble at ensemble sizes of 50 and 100 with only one training epoch (ARI/NMI vs ground-truth cell-type labels)
- Performance plateaus at ensemble size ~50, as reported in Figure 2a / Supplementary Fig. S1
- Clustering ARI/NMI is competitive with totalVI, MultiVI, MOFA+, and Jvis-learn embeddings, and typically significantly better than totalVI on CITE-seq and MultiVI on SHARE-seq/SNARE-seq datasets
- Default initial learning rate of 0.02 yields high ARI on TEA-seq; a 0.2 learning rate significantly reduces performance (Supplementary Fig. S2a trend)

## Limitations

- The tutorial does not recommend best parameters for user datasets; dataset-specific parameters must be taken from the paper
- Large initial learning rates (e.g. 0.2 vs the default 0.02) significantly reduce clustering performance on TEA-seq data
- GPU acceleration is optional and depends on the local PyTorch/CUDA or PyTorch/ROCm installation
- For the published datasets the repository provides no changelog or discussion-section content in the documentation

## Evidence

- [readme] SnapCCESS is an unsupervised ensemble deep learning framework for clustering multimodal single-cell omics data. It creates snapshots of multimodal variational autoencoder embeddings: [evidence span withheld — anchor status: external_doc]
- [readme] features-by-cells input layout: [evidence span withheld — anchor status: external_doc]
- [methods] install command: "pip install snapccess  --index-url https://pypi.org/simple"
- [abstract] efficiency finding: "SnapCCESS is effective and more efficient than conventional ensemble deep learning-based clustering methods and outperforms other state-of-the-art multimodal embedding"
- [intro] shared latent space architecture: [evidence span withheld — anchor status: not_found]
- [readme] parameter caveat: "This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets."
