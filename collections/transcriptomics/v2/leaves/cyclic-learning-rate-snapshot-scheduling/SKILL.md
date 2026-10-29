---
name: cyclic-learning-rate-snapshot-scheduling
description: Use when training VAE encoders on multiple single-cell omics modalities
  (e.g. CITE-seq RNA and ADT) and you need a set of diverse embedding snapshots —
  one per learning-rate cycle — to feed per-snapshot clusterers and produce consensus
  cell labels.
when_to_use_negative:
- The modalities are not measured across the same cells — SnapCCESS expects a list
  of modalities measured across the same cells, so unmatched cell sets must be harmonized
  first
- You want out-of-the-box optimal hyperparameters — the tutorial only explains package
  usage and does not recommend best parameters for your datasets
- The goal is a single trained embedding/model rather than an ensemble of snapshot
  embeddings for consensus clustering
edam_operation: http://edamontology.org/operation_3432
tools:
- name: snapccess
  role: Python package whose VAE training routine implements the shifted-cosine cyclic
    learning-rate scheduler and emits one embedding snapshot per cycle
  repo: https://github.com/PYangLab/SnapCCESS
- name: SnapCCESS (R wrapper)
  role: R interface installed via remotes::install_github that calls the Python package
    through reticulate for the same snapshot-embedding workflow
  repo: https://github.com/yulijia/SnapCCESS
- name: PyTorch
  role: Dependency powering the variational autoencoder training; GPU acceleration
    is optional via the local PyTorch/CUDA or PyTorch/ROCm installation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad382
    title: Ensemble deep learning of embeddings for clustering multimodal single-cell
      omics data
  source_content_hash: sha256:854dc956798e7552183ac95d40ccfbb4635a7a33245a5178399e477b9d95d9bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PYangLab/SnapCCESS
  tools:
  - PyTorch
  - Python
  - SnapCCESS (R wrapper)
  - snapccess
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cyclic-learning-rate-snapshot-scheduling

## Summary

A training scheme from SnapCCESS that runs modality-specific variational autoencoder encoders under a shifted-cosine cyclic learning-rate scheduler so that one embedding snapshot is emitted per learning-rate cycle, providing the diverse embeddings needed for ensemble consensus clustering of multimodal single-cell omics data.

## When to use

Use when training VAE encoders on multiple single-cell omics modalities (e.g. CITE-seq RNA and ADT) and you need a set of diverse embedding snapshots — one per learning-rate cycle — to feed per-snapshot clusterers and produce consensus cell labels.

## When NOT to use

- The modalities are not measured across the same cells — SnapCCESS expects a list of modalities measured across the same cells, so unmatched cell sets must be harmonized first
- You want out-of-the-box optimal hyperparameters — the tutorial only explains package usage and does not recommend best parameters for your datasets
- The goal is a single trained embedding/model rather than an ensemble of snapshot embeddings for consensus clustering

## Inputs

- List of modality-specific count matrices (e.g. CITE-seq RNA and ADT), arranged features-by-cells for the same set of cells
- Combined PyTorch data loader expected by the snapccess Python training routine
- Tutorial data in tutorials/in/ (small CITE-seq RNA, ADT, and cell type files) for running the examples

## Outputs

- One multimodality embedding snapshot per learning-rate cycle (e.g. embeddings.npy)
- Per-snapshot cell clusterings
- Consensus cluster labels, exactly one label per cell (e.g. consensus_labels.csv)

## How to apply

Install the snapccess Python package ('pip install snapccess --index-url https://pypi.org/simple') or the R wrapper via remotes, then arrange each modality as features-by-cells matrices measured across the same cells and feed them through the SnapCCESS tutorial workflow (an_example_of_generate_embedding_using_SnapCCESS_python_version.ipynb). Train modality-specific variational autoencoder encoders joined at a concatenated latent space (the Matilda component) under a shifted-cosine cyclic learning-rate scheduler, saving one embedding snapshot at the end of each learning-rate cycle (e.g. embeddings.npy). Because the schedule revisits low learning rates repeatedly, each cycle converges to a distinct embedding rather than a single optimum, which is what supplies ensemble diversity. Couple each snapshot embedding to a clustering algorithm to obtain one clustering per embedding, then combine the per-embedding clusterings with the clue cl_consensus fixed-point loop to produce consensus cluster labels (e.g. consensus_labels.csv). Hyperparameters are dataset-dependent: the tutorial does not recommend best parameters, so take them from the Bioinformatics btad382 paper for the published datasets and tune from there.

## Related tools

- **snapccess** (Python package whose VAE training routine implements the shifted-cosine cyclic learning-rate scheduler and emits one embedding snapshot per cycle) — https://github.com/PYangLab/SnapCCESS
- **SnapCCESS (R wrapper)** (R interface installed via remotes::install_github that calls the Python package through reticulate for the same snapshot-embedding workflow) — https://github.com/yulijia/SnapCCESS
- **PyTorch** (Dependency powering the variational autoencoder training; GPU acceleration is optional via the local PyTorch/CUDA or PyTorch/ROCm installation)

## Examples

```
pip install snapccess --index-url https://pypi.org/simple
```

## Evaluation signals

- The training loop emits exactly one embedding snapshot per learning-rate cycle of the shifted-cosine scheduler
- The Python tutorial notebook/script runs to completion without error
- Every cell in the input dataset receives exactly one consensus cluster label after the cl_consensus fixed-point loop
- Snapshot embeddings are saved in the expected artifact form (e.g. embeddings.npy) and labels as consensus_labels.csv
- The consensus clustering is consistent with the paper's reported effective clustering of multimodal single-cell data (better than state-of-the-art multimodal embedding baselines)

## Limitations

- The tutorial only explains how to use the package; it does not recommend the best parameters for your datasets — for the published datasets, parameters are listed in the paper
- GPU acceleration is optional and depends on the local PyTorch/CUDA or PyTorch/ROCm installation
- No discussion-section content or changelog is available in the extracted article, so failure modes beyond hyperparameter tuning are not documented

## Evidence

- [intro] snapshot approach: "By creating snapshots of embeddings of multimodality using variational autoencoders, SnapCCESS can be coupled with various clustering algorithms for generating consensus clustering of cells."
- [methods] install: "pip install snapccess  --index-url https://pypi.org/simple"
- [readme] tutorial caveat: [evidence span withheld — anchor status: external_doc]
- [readme] input expectation: [evidence span withheld — anchor status: external_doc]
