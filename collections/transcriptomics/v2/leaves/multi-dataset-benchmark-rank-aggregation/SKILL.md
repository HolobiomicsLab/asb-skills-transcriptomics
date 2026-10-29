---
name: multi-dataset-benchmark-rank-aggregation
description: Use when a dimension reduction method must be quantitatively benchmarked
  against baselines across 20 scRNA-seq datasets, scoring 2D embeddings via k-means
  NMI/ARI and aggregating per-method mean ranks.
when_to_use_negative:
- The task is single-dataset exploratory visualization rather than a multi-method,
  multi-dataset performance comparison
- No ground-truth cell-type labels exist for the datasets, so NMI/ARI against true
  labels cannot be computed
- Only raw high-dimensional expression data is available and no 2D representations
  from VASC or the baselines (PCA, t-SNE, ZIFA, SIMLR) have been generated yet
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VASC
  role: 'Method under test: deep variational autoencoder producing the 2D latent representations
    that are clustered and scored; models dropout events with the Gumbel distribution'
  repo: github.com/wang-research/VASC
- name: sklearn
  role: Provides the baselines' k-means clustering, PCA, t-SNE, and the sklearn.metrics
    NMI/ARI scoring used for each 2D representation
- name: tensorflow
  role: Backend runtime required to execute the VASC model during representation generation
- name: keras
  role: API layer used to build and run the VASC autoencoder (version 2.0.6 pinned
    in the paper's environment)
  repo: keras-team/keras
- name: h5py
  role: Loads the h5 expression matrices and labels for the 20 datasets prior to benchmarking
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:b67aaa9dabd7891796bf1aeef2dd007323de9ee7cbb463b3e179c283d67e6bff
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - VASC
  - h5py
  - keras
  - numpy
  - sklearn
  - tensorflow
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# multi-dataset-benchmark-rank-aggregation

## Summary

A benchmarking protocol that compares a dimension reduction method (e.g., VASC) against baselines (PCA, t-SNE, ZIFA, SIMLR) across many scRNA-seq datasets by scoring k-means clustering of 2D embeddings with NMI and ARI, then aggregating per-method mean ranks across datasets.

## When to use

Use when a dimension reduction method must be quantitatively benchmarked against baselines across 20 scRNA-seq datasets, scoring 2D embeddings via k-means NMI/ARI and aggregating per-method mean ranks.

## When NOT to use

- The task is single-dataset exploratory visualization rather than a multi-method, multi-dataset performance comparison
- No ground-truth cell-type labels exist for the datasets, so NMI/ARI against true labels cannot be computed
- Only raw high-dimensional expression data is available and no 2D representations from VASC or the baselines (PCA, t-SNE, ZIFA, SIMLR) have been generated yet

## Inputs

- Expression matrices for ~20 Hemberg-group scRNA-seq datasets (h5 files loaded via h5py)
- Ground-truth cell-type labels per dataset
- Known number of cell types per dataset (for setting k in k-means)
- 2D representations from VASC (vasc.py deep variational autoencoder) and from PCA, t-SNE, ZIFA, SIMLR baselines

## Outputs

- Per-dataset NMI and ARI scores for each method on each 2D embedding
- Mean rank per method aggregated across all 20 datasets
- Comparison table (e.g., nmi_ari_benchmark.csv) reproducing the paper's Figure 3

## How to apply

Load each dataset's expression matrix and true cell labels (Hemberg-group datasets, read with h5py/numpy). For each dataset, produce a 2D latent representation with the method under test (vasc.py, class VASC / function vase, running on tensorflow 1.1.0 + keras 2.0.6) and comparable 2D representations from each baseline (PCA, t-SNE, ZIFA, SIMLR; sklearn 0.18.1 for PCA/t-SNE). Apply k-means to every 2D embedding using the known number of cell types per dataset as k. Compute NMI and ARI between cluster assignments and true labels with sklearn.metrics, then tabulate per-dataset scores and compute mean ranks per method. A result like VASC being best on 15/20 datasets for NMI and 17/20 for ARI while always ranking in the top two (reproducing Figure 3 as nmi_ari_benchmark.csv) indicates the benchmark was run correctly.

## Related tools

- **VASC** (Method under test: deep variational autoencoder producing the 2D latent representations that are clustered and scored; models dropout events with the Gumbel distribution) — github.com/wang-research/VASC
- **sklearn** (Provides the baselines' k-means clustering, PCA, t-SNE, and the sklearn.metrics NMI/ARI scoring used for each 2D representation)
- **tensorflow** (Backend runtime required to execute the VASC model during representation generation)
- **keras** (API layer used to build and run the VASC autoencoder (version 2.0.6 pinned in the paper's environment)) — keras-team/keras
- **h5py** (Loads the h5 expression matrices and labels for the 20 datasets prior to benchmarking)

## Examples

```
from sklearn.metrics import normalized_mutual_info_score, adjusted_rand_score; nmi = normalized_mutual_info_score(labels_true, kmeans.labels_); ari = adjusted_rand_score(labels_true, kmeans.labels_)
```

## Evaluation signals

- Each dataset's k-means k equals the known number of true cell types for that dataset
- NMI and ARI are computed only between k-means assignments and ground-truth labels on 2D (not higher-dimensional) representations
- The output table covers all 20 datasets x all 5 methods (VASC, PCA, t-SNE, ZIFA, SIMLR) with both NMI and ARI columns
- Expected pattern per the source finding: the method under test best on ~15/20 datasets for NMI and ~17/20 for ARI, and always within the top two ranks
- Per-dataset scores are in [0, 1] for both NMI and ARI; the compiled CSV matches the aggregate of per-dataset results

## Limitations

- Stochastic optimization in VASC model learning introduces run-to-run variation in the dimension reduction; repeated runs are recommended for consensus, so single-run NMI/ARI scores may fluctuate
- Determination of the optimal latent dimension is tricky without prior knowledge; fixed-dimension (2D) choices constrain the benchmark
- When sample size is small, the co-variance matrix should be fixed, which affects reproducibility of embeddings on small datasets (e.g., Biase-style demonstration data)
- VASC model learning costs several hours on a single-GPU desktop for datasets with several thousand cells, limiting how easily the full 5-method x 20-dataset grid can be repeated

## Evidence

- [intro] We systematically compared VASC with several state-of-the-art dimension reduction methods on 20 datasets.: "We systematically compared VASC with several state-of-the-art dimension reduction methods on 20 datasets."
- [readme] vasc.py: contains a class VASC and a function vase: "vasc.py: contains a class VASC and a function vase"
- [discussion] The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction. Repeated runs are thus recommended: "The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction. Repeated runs are thus recommended"
- [abstract] Tested on over 20 datasets, VASC shows superior performances in most cases and exhibits broader dataset compatibility: "Tested on over 20 datasets, VASC shows superior performances in most cases and exhibits broader dataset compatibility"
