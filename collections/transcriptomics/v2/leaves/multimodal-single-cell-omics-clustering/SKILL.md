---
name: multimodal-single-cell-omics-clustering
description: Use when you have multimodal single-cell omics data (e.g. CITE-seq RNA
  + ADT matrices over the same cells) with no labels, and the goal is unsupervised
  consensus cell-type clustering via deep-learning embeddings.
when_to_use_negative:
- Data are single-modality only, or modalities are not measured across the same cells
  (SnapCCESS expects a list of modalities over the same cells, e.g. RNA + ADT)
- 'A supervised cell-type classifier is needed: SnapCCESS is an unsupervised framework
  for consensus clustering, not label-based classification'
- You need dataset-specific optimal hyperparameters from the package alone — the tutorial
  explicitly does not recommend best parameters; those must come from the paper
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SnapCCESS (snapccess)
  role: Python package that creates snapshots of multimodality embeddings using variational
    autoencoders for ensemble clustering
  repo: https://github.com/PYangLab/SnapCCESS
- name: SnapCCESS R wrapper
  role: R interface installed via remotes::install_github that calls the Python package
    through reticulate; expects each modality arranged features by cells before preprocessing
  repo: https://github.com/yulijia/SnapCCESS
- name: Python
  role: Implementation language; snapccess depends on PyTorch, with optional GPU acceleration
- name: k-means (scikit-learn), CiteFuse spectral clustering, SIMLR
  role: Clustering algorithms coupled to SnapCCESS embeddings to generate consensus
    cell clusterings, benchmarked with ARI and NMI
provenance:
  source_task_ids:
  - task_006
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad382
    title: Ensemble deep learning of embeddings for clustering multimodal single-cell
      omics data
  source_content_hash: sha256:b399c6c5dd918f762012082cd13ee5f095f02e829d649ef8c1520cd2130e5470
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PYangLab/SnapCCESS
  tools:
  - CiteFuse spectral clustering
  - Python
  - SIMLR
  - SnapCCESS (snapccess)
  - SnapCCESS R wrapper
  - k-means (scikit-learn), CiteFuse spectral clustering, SIMLR
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# multimodal-single-cell-omics-clustering

## Summary

Unsupervised ensemble deep learning (SnapCCESS) for clustering multimodal single-cell omics data: it creates snapshots of multimodality variational autoencoder embeddings and couples them with clustering algorithms to generate consensus clusters, achieving performance comparable to conventional VAE ensembles at far lower training cost.

## When to use

Use when you have multimodal single-cell omics data (e.g. CITE-seq RNA + ADT matrices over the same cells) with no labels, and the goal is unsupervised consensus cell-type clustering via deep-learning embeddings.

## When NOT to use

- Data are single-modality only, or modalities are not measured across the same cells (SnapCCESS expects a list of modalities over the same cells, e.g. RNA + ADT)
- A supervised cell-type classifier is needed: SnapCCESS is an unsupervised framework for consensus clustering, not label-based classification
- You need dataset-specific optimal hyperparameters from the package alone — the tutorial explicitly does not recommend best parameters; those must come from the paper

## Inputs

- List of multimodal single-cell omics matrices measured across the same cells (e.g. CITE-seq RNA and ADT features-by-cells matrices)
- Optional ground-truth cell-type labels for ARI/NMI concordance evaluation
- Tutorial data files (small CITE-seq RNA, ADT, and cell type files from tutorials/in/)

## Outputs

- SnapCCESS ensemble embeddings (snapshots of multimodality VAE embeddings)
- Consensus cell clusterings from coupled algorithms (k-means, spectral clustering, SIMLR)
- ARI/NMI benchmark tables versus epochs and ensemble size (e.g. ari_nmi_ensemble_benchmark.csv, algorithm_comparison_ari_nmi.csv, computation_time.csv)

## How to apply

Install the snapccess Python package (pip install snapccess --index-url https://pypi.org/simple) or the SnapCCESS R wrapper via remotes::install_github(repo='yulijia/SnapCCESS',branch='main',subdir='snapccess-r/SnapCCESS'). Arrange each modality as features by cells (e.g. RNA and ADT matrices from CITE-seq) and feed them as a list of modalities measured across the same cells; the Python API expects the combined PyTorch data loader used by the training routine. Generate snapshots of multimodality embeddings with variational autoencoders, sweeping per-model training budgets (e.g. 1, 2, 10 epochs) and ensemble sizes up to 100 members, since clustering performance improves with ensemble size and plateaus around an ensemble size of 50. Couple each embedding set with clustering algorithms (k-means from scikit-learn, CiteFuse spectral clustering, or SIMLR) to generate consensus clusterings. Assess quality with concordance metrics against reference cell-type labels, using Adjusted Rand Index (ARI) and Normalized Mutual Information (NMI), tabulating ARI/NMI per algorithm, epoch, and ensemble size.

## Related tools

- **SnapCCESS (snapccess)** (Python package that creates snapshots of multimodality embeddings using variational autoencoders for ensemble clustering) — https://github.com/PYangLab/SnapCCESS
- **SnapCCESS R wrapper** (R interface installed via remotes::install_github that calls the Python package through reticulate; expects each modality arranged features by cells before preprocessing) — https://github.com/yulijia/SnapCCESS
- **Python** (Implementation language; snapccess depends on PyTorch, with optional GPU acceleration)
- **k-means (scikit-learn), CiteFuse spectral clustering, SIMLR** (Clustering algorithms coupled to SnapCCESS embeddings to generate consensus cell clusterings, benchmarked with ARI and NMI)

## Examples

```
pip install snapccess --index-url https://pypi.org/simple; then follow tutorials/src/an_example_of_generate_embedding_using_SnapCCESS_python_version.ipynb to generate embeddings
```

## Evaluation signals

- ARI and NMI of consensus clusterings against ground-truth cell-type labels should increase with ensemble size regardless of clustering algorithm, peaking around an ensemble size of 50
- At ensemble sizes of 50 and 100, SnapCCESS with 1 training epoch should match the ARI/NMI of the conventional VAE ensemble, which requires many epochs for comparable results
- Timed network building + model training should show SnapCCESS more efficient (lower training time) than conventional ensemble deep learning-based clustering, with similar times at equal epoch counts
- ARI/NMI results should reproduce Figure 2a / Supplementary Fig. S1 (epochs x ensemble size) and Figure 4b / Supplementary Fig. S5 (algorithm comparison) patterns

## Limitations

- The tutorial does not recommend the best parameters for your datasets; dataset-specific parameters must be taken from the published paper
- GPU acceleration is optional and depends on the local PyTorch/CUDA or PyTorch/ROCm installation
- The repository README notes no changelog; discussion-level guidance is absent, so parameter choices rely on the paper

## Evidence

- [intro] SnapCCESS can be coupled with various clustering algorithms for generating consensus clustering of cells.: "By creating snapshots of embeddings of multimodality using variational autoencoders, SnapCCESS can be coupled with various clustering algorithms for generating consensus clustering of cells."
- [methods] pip install command from methods: "pip install snapccess  --index-url https://pypi.org/simple"
- [results] improvement peaks around ensemble size 50: [evidence span withheld — anchor status: not_found]
- [abstract] SnapCCESS efficient vs conventional ensemble: "Our results demonstrate that SnapCCESS is effective and more efficient than conventional ensemble deep learning-based clustering methods and outperforms other state-of-the-art multimodal embedding"
- [readme] README input data description: [evidence span withheld — anchor status: external_doc]
- [readme] README tutorial parameters caveat: "This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets. For the datasets used in the published paper associated with this package, the"
