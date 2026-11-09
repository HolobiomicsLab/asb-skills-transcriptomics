---
name: variational-autoencoder-training
description: Use when clustering cells from multimodal single-cell omics data (e.g.
  CITE-seq RNA+ADT feature-by-cell matrices) requires generating multiple latent-space
  embedding snapshots via variational autoencoders for ensemble consensus clustering.
when_to_use_negative:
- The goal is embedding generation for a single modality only — SnapCCESS is designed
  to integrate multiple data modalities measured across the same cells
- Supervised cell-type annotation is required — SnapCCESS is an unsupervised ensemble
  deep learning framework for clustering
- You need dataset-specific optimal hyperparameters — the tutorials explicitly do
  not recommend best parameters; those are listed only in the paper for the datasets
  it used
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: snapccess
  role: Python package that trains the multimodal VAE encoders and emits embedding
    snapshots; installed via pip from PyPI
  repo: https://github.com/PYangLab/SnapCCESS
- name: SnapCCESS (R wrapper)
  role: R wrapper that calls the Python package through reticulate; installs the Python
    package into a reticulate environment via SnapCCESS::install_SnapCCESS(envname
    = "SnapCCESS", method = "conda")
  repo: https://github.com/yulijia/SnapCCESS
- name: Python
  role: Language in which SnapCCESS is implemented and in which the embedding-generation
    tutorial notebook runs
- name: remotes
  role: Used to install the SnapCCESS R wrapper from GitHub (remotes::install_github
    with subdir='snapccess-r/SnapCCESS')
provenance:
  source_task_ids:
  - task_008
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad382
    title: Ensemble deep learning of embeddings for clustering multimodal single-cell
      omics data
  source_content_hash: sha256:df04bf1e798099eefc0362895d22d61d5d8587bf828fa946b7db496805c1bf3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PYangLab/SnapCCESS
  tools:
  - Python
  - SnapCCESS (R wrapper)
  - remotes
  - snapccess
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# variational-autoencoder-training

## Summary

Trains variational autoencoders to create snapshots of multimodality embeddings from multimodal single-cell omics data (e.g. CITE-seq RNA/ADT), where each learning-rate cycle emits one embedding snapshot that feeds ensemble consensus clustering. In the SnapCCESS framework, brief per-model training (as little as 1 epoch) at large ensemble sizes matches the performance of conventionally fully trained VAE ensembles.

## When to use

Use when clustering cells from multimodal single-cell omics data (e.g. CITE-seq RNA+ADT feature-by-cell matrices) requires generating multiple latent-space embedding snapshots via variational autoencoders for ensemble consensus clustering.

## When NOT to use

- The goal is embedding generation for a single modality only — SnapCCESS is designed to integrate multiple data modalities measured across the same cells
- Supervised cell-type annotation is required — SnapCCESS is an unsupervised ensemble deep learning framework for clustering
- You need dataset-specific optimal hyperparameters — the tutorials explicitly do not recommend best parameters; those are listed only in the paper for the datasets it used

## Inputs

- List of modality-specific count matrices measured across the same cells, arranged as features by cells (e.g. CITE-seq RNA and ADT matrices)
- Tutorial data files in tutorials/in/ (small CITE-seq RNA, ADT, and cell type files)
- Optional ground-truth cell-type label files for computing ARI/NMI

## Outputs

- Snapshot embeddings of the shared multimodality latent space (e.g. embeddings.npy)
- Per-embedding clusterings of cells combined into consensus cluster labels (e.g. consensus_labels.csv)
- ARI/NMI benchmark tables versus epochs and ensemble size (e.g. ari_nmi_ensemble_benchmark.csv) and timing tables (e.g. computation_time.csv)

## How to apply

Install the snapccess Python package with pip ('pip install snapccess --index-url https://pypi.org/simple') or the R wrapper via remotes::install_github, and prepare one input matrix per modality (features by cells, e.g. RNA and ADT from CITE-seq) following the tutorial notebook an_example_of_generate_embedding_using_SnapCCESS_python_version.ipynb. Train modality-specific VAE encoders joined at a concatenated latent space — SnapCCESS concatenates encoder outputs and performs joint learning through a fully connected layer with 100 neurons, producing the shared multimodality latent space — under a shifted-cosine cyclic learning-rate scheduler, so each learning-rate cycle emits one embedding snapshot. Sweep per-model training budgets of 1, 2, and 10 epochs and ensemble sizes up to 100 members; the paper shows performance plateaus at ensemble size ~50 and that 1 epoch at ensemble sizes of 50 and 100 is sufficient to match a conventional fully trained VAE ensemble. Couple each snapshot embedding to a clustering algorithm (via the R clue package's cl_consensus fixed-point loop) to obtain consensus cluster labels, and save embeddings (e.g. embeddings.npy) and labels (e.g. consensus_labels.csv). Time only network building and model training when benchmarking against the conventional VAE-ensemble baseline.

## Related tools

- **snapccess** (Python package that trains the multimodal VAE encoders and emits embedding snapshots; installed via pip from PyPI) — https://github.com/PYangLab/SnapCCESS
- **SnapCCESS (R wrapper)** (R wrapper that calls the Python package through reticulate; installs the Python package into a reticulate environment via SnapCCESS::install_SnapCCESS(envname = "SnapCCESS", method = "conda")) — https://github.com/yulijia/SnapCCESS
- **Python** (Language in which SnapCCESS is implemented and in which the embedding-generation tutorial notebook runs)
- **remotes** (Used to install the SnapCCESS R wrapper from GitHub (remotes::install_github with subdir='snapccess-r/SnapCCESS'))

## Examples

```
pip install snapccess --index-url https://pypi.org/simple
```

## Evaluation signals

- Every cell in the input dataset receives exactly one consensus cluster label after the cl_consensus fixed-point loop
- ARI/NMI against ground-truth labels plateaus at ensemble size ~50, consistent with Figure 2a / Supplementary Fig. S1
- With 1 training epoch at ensemble sizes of 50 and 100, ARI/NMI is comparable to the conventional fully trained VAE-ensemble baseline
- Computation time under the same number of epochs is very similar between SnapCCESS and the conventional VAE ensemble, and fewer epochs yield significantly faster runs, especially at large ensemble sizes
- The tutorial notebook/script runs to completion and produces saved artifacts (embeddings.npy, consensus_labels.csv)

## Limitations

- The tutorial only explains how to use the package and does not recommend best parameters for new datasets; for the published datasets the parameters are listed in the paper (Bioinformatics, btad382)
- GPU acceleration is optional and depends on the local PyTorch/CUDA or PyTorch/ROCm installation
- The R wrapper requires configuring reticulate to use an environment (conda, virtualenv, or cluster module) where the Python package is installed
- SnapCCESS expects all modalities to be measured across the same cells (e.g. RNA and ADT from CITE-seq)

## Evidence

- [methods] SnapCCESS concatenates the output from the encoder trained on each data modality and performs joint learning through a fully connected layer with 100 neurons: [evidence span withheld — anchor status: not_found]
- [intro] By creating snapshots of embeddings of multimodality using variational autoencoders, SnapCCESS can be coupled with various clustering algorithms for generating consensus clustering of cells: "By creating snapshots of embeddings of multimodality using variational autoencoders, SnapCCESS can be coupled with various clustering algorithms for generating consensus clustering of cells."
- [methods] pip install snapccess --index-url https://pypi.org/simple: "pip install snapccess  --index-url https://pypi.org/simple"
- [readme] SnapCCESS expects a list of modalities measured across the same cells, for example RNA and ADT matrices from CITE-seq. Each modality should be arranged as features by cells before preprocessing: [evidence span withheld — anchor status: external_doc]
- [readme] This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets: "This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets."
- [readme] The Python package depends on PyTorch. GPU acceleration is optional and depends on the local PyTorch/CUDA or PyTorch/ROCm installation.: [evidence span withheld — anchor status: external_doc]
