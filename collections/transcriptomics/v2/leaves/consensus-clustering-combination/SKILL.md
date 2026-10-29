---
name: consensus-clustering-combination
description: Use when you have multiple cell embeddings (e.g. VAE snapshot embeddings
  from CITE-seq or SHARE-seq/SNARE-seq modalities) or clusterings over the same cells
  and need a single consensus cluster label per cell.
when_to_use_negative:
- When you have only a single embedding or a single precomputed clustering of the
  cells — consensus combination requires multiple clusterings to reconcile
- When cells differ across embeddings/clustering runs (e.g. different cell subsets),
  since the consensus loop assumes every clustering covers the same cell set
- 'When you need dataset-optimal hyperparameters for a new dataset: the tutorial ''only
  explains how to use this package; it doesn''t recommend the best parameters for
  your datasets'' — parameters must come from analogous published work, not the consensus
  step itself'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: SnapCCESS
  role: Unsupervised ensemble deep learning framework that creates snapshot embeddings
    of multimodality via variational autoencoders to be coupled with clustering algorithms;
    implemented as a Python (snapccess) package and R wrapper
  repo: https://github.com/PYangLab/SnapCCESS
- name: snapccess
  role: PyPI Python package used to generate the snapshot embeddings from multimodal
    single-cell omics inputs before consensus clustering
  repo: https://github.com/yulijia/SnapCCESS/tree/main/snapccess-py
- name: clue
  role: R package whose cl_consensus fixed-point loop combines per-embedding clusterings
    into the final consensus cluster labels
- name: remotes
  role: R package used to install the SnapCCESS R wrapper from GitHub
  repo: https://github.com/yulijia/SnapCCESS
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad382
    title: Ensemble deep learning of embeddings for clustering multimodal single-cell
      omics data
  source_content_hash: sha256:496fa5939702c852bef61a14a13fc00a26b887ef64ff2d7cfc0b0678e72d3cbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PYangLab/SnapCCESS
  tools:
  - Python
  - SnapCCESS
  - clue
  - remotes
  - snapccess
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# consensus-clustering-combination

## Summary

Combines per-embedding clusterings of multimodal single-cell omics data into a single set of consensus cell labels via SnapCCESS's ensemble snapshot approach and the clue cl_consensus fixed-point loop, avoiding the cost of conventional ensemble deep learning.

## When to use

Use when you have multiple cell embeddings (e.g. VAE snapshot embeddings from CITE-seq or SHARE-seq/SNARE-seq modalities) or clusterings over the same cells and need a single consensus cluster label per cell.

## When NOT to use

- When you have only a single embedding or a single precomputed clustering of the cells — consensus combination requires multiple clusterings to reconcile
- When cells differ across embeddings/clustering runs (e.g. different cell subsets), since the consensus loop assumes every clustering covers the same cell set
- When you need dataset-optimal hyperparameters for a new dataset: the tutorial 'only explains how to use this package; it doesn't recommend the best parameters for your datasets' — parameters must come from analogous published work, not the consensus step itself

## Inputs

- Multiple cell embeddings over the same cells (e.g. embeddings.npy snapshot embeddings from SnapCCESS variational autoencoders)
- Per-embedding cluster assignments of cells (from k-means, SIMLR, or spectral clustering)
- Multimodal single-cell omics count matrices (RNA/ADT from CITE-seq, RNA/ATAC from SHARE-seq or SNARE-seq) arranged as features by cells

## Outputs

- A single consensus cluster label per cell (e.g. consensus_labels.csv)
- Snapshot embeddings (e.g. embeddings.npy) that produced the per-embedding clusterings

## How to apply

First generate multiple embeddings of the same cells — in SnapCCESS, train modality-specific variational autoencoder encoders joined at a concatenated latent space under a shifted-cosine cyclic learning-rate scheduler, emitting one embedding snapshot per learning-rate cycle. Couple each snapshot embedding to a clustering algorithm (k-means, SIMLR, or spectral clustering in the paper's benchmark) to obtain a per-embedding clustering of cells. Then feed the per-embedding clusterings into the clue package's cl_consensus fixed-point loop to combine them into one consensus clustering. Every cell in the input dataset must receive exactly one consensus cluster label; SnapCCESS can be coupled with various clustering algorithms, and the tutorial notebook an_example_of_generate_embedding_using_SnapCCESS_python_version.ipynb walks through embedding generation. Take dataset-specific hyperparameters from the published paper (doi:10.1093/bioinformatics/btad382), not the tutorial defaults.

## Related tools

- **SnapCCESS** (Unsupervised ensemble deep learning framework that creates snapshot embeddings of multimodality via variational autoencoders to be coupled with clustering algorithms; implemented as a Python (snapccess) package and R wrapper) — https://github.com/PYangLab/SnapCCESS
- **snapccess** (PyPI Python package used to generate the snapshot embeddings from multimodal single-cell omics inputs before consensus clustering) — https://github.com/yulijia/SnapCCESS/tree/main/snapccess-py
- **clue** (R package whose cl_consensus fixed-point loop combines per-embedding clusterings into the final consensus cluster labels)
- **remotes** (R package used to install the SnapCCESS R wrapper from GitHub) — https://github.com/yulijia/SnapCCESS

## Examples

```
pip install snapccess --index-url https://pypi.org/simple; R: remotes::install_github(repo='yulijia/SnapCCESS',branch='main',subdir='snapccess-r/SnapCCESS')
```

## Evaluation signals

- Every cell in the input dataset receives exactly one consensus cluster label after the cl_consensus fixed-point loop terminates
- The pipeline runs end-to-end on the tutorial data in tutorials/in/ (small CITE-seq RNA, ADT, and cell type files)
- Consensus clustering scores (ARI and NMI against reference cell-type labels) are competitive with, or better than, totalVI on CITE-seq datasets and MultiVI on SHARE-seq and SNARE-seq datasets, replicating the Figure 5 benchmark
- SnapCCESS consensus clustering is more efficient (faster) than conventional ensemble deep learning-based clustering methods at comparable quality

## Limitations

- The tutorial does not recommend best parameters for your datasets; parameters for the paper's datasets are listed in the paper itself (doi:10.1093/bioinformatics/btad382)
- The R wrapper depends on reticulate and a correctly configured Python environment (conda, virtualenv, or cluster module); GPU acceleration is optional and depends on the local PyTorch/CUDA or PyTorch/ROCm installation
- Benchmark performance is dataset- and method-pairing-specific (e.g. 'characteristically poor SIMLR-on-Jvis-learn result'), so a poor clustering score may reflect the clustering-embedding pairing rather than the consensus step

## Evidence

- [intro] SnapCCESS can be coupled with various clustering algorithms for generating consensus clustering of cells.: "By creating snapshots of embeddings of multimodality using variational autoencoders, SnapCCESS can be coupled with various clustering algorithms for generating consensus clustering of cells."
- [methods] Installation and pip command: "pip install snapccess  --index-url https://pypi.org/simple"
- [readme] Tutorial parameters note: "This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets."
- [readme] Tutorial data contents: [evidence span withheld — anchor status: external_doc]
- [abstract] Abstract efficacy claim: "Our results demonstrate that SnapCCESS is effective and more efficient than conventional ensemble deep learning-based clustering methods and outperforms other state-of-the-art multimodal embedding"
