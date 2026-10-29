---
name: multimodal-embedding-snapshot-training
description: Use when you have multimodal single-cell omics data (RNA/ADT/ATAC measured
  on the same cells) and want to cluster cells by integrating modalities via ensemble
  VAE embeddings rather than clustering from any single modality alone.
when_to_use_negative:
- You only have a single modality — the integration benefit (better ARI/NMI than unimodality
  clustering) only arises when multiple modalities measured on the same cells are
  combined.
- You need a prescribed optimal hyperparameter set for a new dataset — the tutorial
  explicitly does not recommend best parameters; for datasets other than those in
  the paper, parameters must be determined separately.
- You need supervised cell-type annotation — SnapCCESS is an unsupervised clustering/embedding
  framework, not a classifier.
edam_operation: http://edamontology.org/operation_3686
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: snapccess
  role: Python package whose variational autoencoder backbone trains snapshots of
    multimodality embeddings; installed via pip
  repo: https://github.com/PYangLab/SnapCCESS
- name: SnapCCESS R wrapper
  role: R interface that calls the Python package through reticulate for the R example
    workflow
  repo: https://github.com/PYangLab/SnapCCESS
- name: remotes
  role: Used to install the SnapCCESS R wrapper from GitHub
  repo: https://github.com/yulijia/SnapCCESS
- name: Python
  role: Runtime for the snapccess package and the tutorial notebook workflow (PyTorch
    dependency; GPU optional)
  repo: https://github.com/PYangLab/SnapCCESS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad382
    title: Ensemble deep learning of embeddings for clustering multimodal single-cell
      omics data
  source_content_hash: sha256:540c45e16faa3f65951e71eb0a445ab8dd9325e1c68be33e26365cc486b8c898
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PYangLab/SnapCCESS
  tools:
  - Python
  - R
  - SnapCCESS R wrapper
  - remotes
  - snapccess
  edam_operation: http://edamontology.org/operation_3686
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# multimodal-embedding-snapshot-training

## Summary

Train SnapCCESS, an unsupervised ensemble deep learning framework that creates snapshots of multimodal variational autoencoder embeddings from single-cell omics data (e.g. RNA, ADT, ATAC), then couple the ensemble embedding with k-means to generate consensus cell clusters evaluated by ARI/NMI.

## When to use

Use when you have multimodal single-cell omics data (RNA/ADT/ATAC measured on the same cells) and want to cluster cells by integrating modalities via ensemble VAE embeddings rather than clustering from any single modality alone.

## When NOT to use

- You only have a single modality — the integration benefit (better ARI/NMI than unimodality clustering) only arises when multiple modalities measured on the same cells are combined.
- You need a prescribed optimal hyperparameter set for a new dataset — the tutorial explicitly does not recommend best parameters; for datasets other than those in the paper, parameters must be determined separately.
- You need supervised cell-type annotation — SnapCCESS is an unsupervised clustering/embedding framework, not a classifier.

## Inputs

- Multimodal single-cell omics data: list of modality matrices (RNA, ADT, ATAC) measured across the same cells, arranged features-by-cells
- Combined PyTorch data loader expected by the SnapCCESS Python training routine
- Tutorial CITE-seq RNA, ADT, and cell type files from tutorials/in/ for running the examples
- Epoch setting (epoch=1 for the published-paper benchmarks)

## Outputs

- Snapshots of multimodality VAE embeddings (ensemble embedding of cells)
- Consensus cell clusters from k-means on the ensemble embedding
- ARI and NMI metrics per modality condition, e.g. assembled into ari_nmi_modality_comparison.csv

## How to apply

Install the Python package with 'pip install snapccess --index-url https://pypi.org/simple' (or the R wrapper via remotes::install_github) and follow the tutorial notebook an_example_of_generate_embedding_using_SnapCCESS_python_version.ipynb as the reference workflow. Arrange each modality as features-by-cells and supply a list of modalities measured across the same cells (e.g. CITE-seq RNA and ADT matrices); the Python API takes the combined PyTorch data loader used by the training routine. Train SnapCCESS with its variational autoencoder backbone to create snapshots of multimodality embeddings — for the paper's benchmarks epoch=1 is used; note the tutorial explicitly does not recommend best parameters, so consult the paper's listed parameters per dataset. Run k-means on the resulting ensemble embedding to obtain consensus clusters and record clustering performance as ARI and NMI. To validate integration benefit, repeat training and k-means independently per unimodality (RNA only, ADT only, ATAC only) and compare — the paper reports integrated embeddings significantly outperform any unimodality (Fig. 4a, Supplementary Fig. S4).

## Related tools

- **snapccess** (Python package whose variational autoencoder backbone trains snapshots of multimodality embeddings; installed via pip) — https://github.com/PYangLab/SnapCCESS
- **SnapCCESS R wrapper** (R interface that calls the Python package through reticulate for the R example workflow) — https://github.com/PYangLab/SnapCCESS
- **remotes** (Used to install the SnapCCESS R wrapper from GitHub) — https://github.com/yulijia/SnapCCESS
- **Python** (Runtime for the snapccess package and the tutorial notebook workflow (PyTorch dependency; GPU optional)) — https://github.com/PYangLab/SnapCCESS

## Examples

```
pip install snapccess --index-url https://pypi.org/simple
```

## Evaluation signals

- Multimodal SnapCCESS embedding attains higher ARI and NMI than every single-modality (RNA-only, ADT-only, ATAC-only) embedding, reproducing the Fig. 4a / Supplementary Fig. S4 result.
- The ensemble embedding is generated by snapshots of the variational autoencoder trained for the paper-specified epoch count (epoch=1 in the benchmark), then clustered with k-means to yield consensus clusters.
- ARI/NMI comparison table (e.g. ari_nmi_modality_comparison.csv) contains one row per condition (multimodal plus each unimodality) with finite ARI and NMI values.
- Input check: every modality matrix covers the same cells and is arranged features-by-cells before preprocessing in the R wrapper.
- Training completes with the snapccess PyTorch backend (GPU acceleration optional); failures here indicate environment problems, not clustering quality.

## Limitations

- The tutorial only explains package usage and does not recommend best parameters for datasets; parameters for the paper's datasets are listed in the paper itself.
- GPU acceleration is optional and depends on the local PyTorch/CUDA or PyTorch/ROCm installation.
- If Python dependencies are managed through Conda, virtualenv, or cluster modules, reticulate must be configured to use that environment before loading the R wrapper.
- SnapCCESS expects modalities measured across the same cells (e.g. CITE-seq RNA and ADT); datasets without shared-cell multimodality do not match the input contract.

## Evidence

- [intro] We propose SnapCCESS for clustering cells by integrating data modalities in multimodal single-cell omics data using an unsupervised ensemble deep learning framework.: "We propose SnapCCESS for clustering cells by integrating data modalities in multimodal single-cell omics data using an unsupervised ensemble deep learning framework."
- [methods] pip install command: "pip install snapccess  --index-url https://pypi.org/simple"
- [methods] tutorial parameter caveat: "This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets."
- [readme] README input data description: [evidence span withheld — anchor status: external_doc]
- [readme] README installation/dependency note: [evidence span withheld — anchor status: external_doc]
