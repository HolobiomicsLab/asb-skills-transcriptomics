---
name: k-means-clustering-of-embedded-representations
description: Use when comparing dimension reduction methods on labeled single-cell
  RNA-seq datasets and you need reproducible quantitative clustering scores (NMI/ARI)
  from low-dimensional (e.g., 2D) representations.
when_to_use_negative:
- The data lack ground-truth cell-type labels or a known number of cell types, since
  k and the NMI/ARI reference both depend on true labels
- The goal is visualization quality assessment rather than quantitative clustering
  benchmarking
- The representation is not a fixed low-dimensional embedding comparable across methods
  (e.g., raw high-dimensional expression matrices)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: VASC
  role: Produces the 2D latent representations whose k-means clustering quality is
    benchmarked (deep variational autoencoder modeling dropout with the Gumbel distribution)
  repo: github.com/wang-research/VASC
- name: sklearn
  role: Provides k-means clustering, PCA, t-SNE, and the NMI/ARI metrics (sklearn.metrics)
    used for scoring
- name: h5py
  role: Loads the expression matrices and true cell labels for the 20 scRNA-seq datasets
- name: keras
  role: Backend framework for the VASC model that generates the embedded representations
    (keras 2.0.6)
  repo: keras-team/keras
- name: tensorflow
  role: Backend for the VASC deep model (tensorflow 1.1.0)
- name: numpy
  role: Array handling for expression matrices and label vectors during scoring
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:f70f4e84fb5310d29b1770864f529a9fd208eb709e46901d27e6d6935d7b2f47
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# k-means-clustering-of-embedded-representations

## Summary

Quantitatively benchmark 2D dimension-reduction embeddings of scRNA-seq data by applying k-means clustering with the known number of cell types and scoring cluster assignments against true labels with NMI and ARI, as used to show VASC outperforms PCA, t-SNE, ZIFA, and SIMLR.

## When to use

Use when comparing dimension reduction methods on labeled single-cell RNA-seq datasets and you need reproducible quantitative clustering scores (NMI/ARI) from low-dimensional (e.g., 2D) representations.

## When NOT to use

- The data lack ground-truth cell-type labels or a known number of cell types, since k and the NMI/ARI reference both depend on true labels
- The goal is visualization quality assessment rather than quantitative clustering benchmarking
- The representation is not a fixed low-dimensional embedding comparable across methods (e.g., raw high-dimensional expression matrices)

## Inputs

- 2D latent/embedded representations per method (e.g., VASC 2D output from vasc.py, PCA/t-SNE 2D coordinates)
- True cell-type label vector for each of the 20 Hemberg-group scRNA-seq datasets
- Known number of cell types k per dataset

## Outputs

- Per-dataset NMI and ARI scores for each method (PCA, t-SNE, ZIFA, SIMLR, VASC)
- Mean ranks per method across the 20 datasets
- Aggregated benchmark comparison table, e.g., nmi_ari_benchmark.csv (reproducing Figure 3)

## How to apply

For each dataset, take the 2D latent representation produced by the dimension reduction method (e.g., the VASC variational autoencoder from vasc.py, or PCA/t-SNE via sklearn 0.18.1). Apply k-means clustering (sklearn) using the known number of cell types in that Hemberg-group dataset as k, so all methods are evaluated under identical cluster counts. Compute Normalized Mutual Information (NMI) and Adjusted Rand Index (ARI) between k-means assignments and true cell labels using sklearn.metrics. Aggregate per-dataset scores across the 20 datasets, compute mean ranks per method, and compile the comparison table (e.g., nmi_ari_benchmark.csv reproducing Figure 3). Because VASC's stochastic optimization introduces run-to-run variation, repeated runs are recommended to reach consensus scores.

## Related tools

- **VASC** (Produces the 2D latent representations whose k-means clustering quality is benchmarked (deep variational autoencoder modeling dropout with the Gumbel distribution)) — github.com/wang-research/VASC
- **sklearn** (Provides k-means clustering, PCA, t-SNE, and the NMI/ARI metrics (sklearn.metrics) used for scoring)
- **h5py** (Loads the expression matrices and true cell labels for the 20 scRNA-seq datasets)
- **keras** (Backend framework for the VASC model that generates the embedded representations (keras 2.0.6)) — keras-team/keras
- **tensorflow** (Backend for the VASC deep model (tensorflow 1.1.0))
- **numpy** (Array handling for expression matrices and label vectors during scoring)

## Examples

```
from sklearn.cluster import KMeans; from sklearn.metrics import normalized_mutual_info_score as nmi, adjusted_rand_score as ari; labels = KMeans(n_clusters=k, n_init=10).fit_predict(Z2d)
```

## Evaluation signals

- VASC achieves best NMI on 15 of the 20 datasets and best ARI on 17 of the 20 datasets when k-means is applied to its 2D representation
- VASC is always ranked in the top two methods by mean rank across all tested datasets
- k is set to the known number of cell types per dataset, so cluster counts match the true label cardinality
- NMI and ARI computed with sklearn.metrics return values in [0, 1] for these label sets; scores near 0 indicate label-shuffling or alignment bugs
- Repeated runs converge to consistent rankings despite stochastic-optimization variation

## Limitations

- Stochastic optimization in VASC model learning introduces run-to-run variation in the dimension reduction; repeated runs are recommended to reach a consensus
- VASC model learning can take several hours on a single-GPU desktop for datasets with several thousand cells
- Determining the optimal embedding dimension is difficult without prior knowledge; the benchmark fixes 2D for visualization-comparable evaluation

## Evidence

- [results] VASC outperformed the other four methods in NMI and ARI on most of the 20 datasets, achieving the best performance on 15 datasets for NMI and 17 datasets for ARI: "Tested on over 20 datasets, VASC shows superior performances in most cases and exhibits broader dataset compatibility compared to four state-of-the-art dimension reduction and visualization methods"
- [methods] sklearn versions used for clustering and metrics: "vasc.py: contains a class VASC and a function vase"
- [discussion] Stochastic variation requires repeated runs: "The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction. Repeated runs are thus recommended"
- [intro] Systematic comparison on 20 datasets motivated the benchmark: "We systematically compared VASC with several state-of-the-art dimension reduction methods on 20 datasets. Results show that VASC has superior performance in most cases"
- [readme] Demo entry points in the VASC repository: "We gave a demo.py and config.py to demonstrate the use of VASC."
