---
name: spearman-correlation-benchmark-scoring
description: Use when comparing alignment methods (e.g. scTopoGAN vs UnionCom, MMD-MA,
  SCOT) on their ability to predict one modality from another — ATAC↔RNA on Full PBMC
  or ADT↔RNA on BM — from unsupervised aligned embeddings.
when_to_use_negative:
- When the datasets share overlapping cells or features — current integration methods
  that rely on overlapping anchors are applicable instead of the fully unsupervised
  alignment + cross-modality prediction benchmark
- When the goal is assessing clustering quality of aligned embeddings — use the complementary
  Leiden clustering + ARI benchmark (Table 3) rather than cross-modality Spearman
  prediction
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scTopoGAN
  role: Produces the topology-guided GAN aligned embeddings being scored on cross-modality
    prediction
  repo: github.com/AkashCiel/scTopoGAN
- name: UnionCom
  role: Benchmarked competitor alignment method whose embeddings are scored via median
    Spearman correlation
  repo: github.com/AkashCiel/scTopoGAN
- name: MMD-MA
  role: Benchmarked competitor alignment method applied via manifold_align_mmd_pytorch.py
  repo: github.com/AkashCiel/scTopoGAN
- name: Topological Autoencoder
  role: Generates the per-modality latent representations (get_TopoAE_Embeddings)
    fed into scTopoGAN before scoring
  repo: github.com/AkashCiel/scTopoGAN
- name: UMAP
  role: Baseline manifold projection method included in the benchmark comparison
  repo: github.com/AkashCiel/scTopoGAN
- name: Python
  role: Implementation language for the alignment, regression, and Spearman correlation
    scoring scripts
  repo: github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:888f8efec665edb707d72ce515b5e137981c038cd3fe7f8c3f0a388e7a0efb02
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MMD-MA
  - Python
  - Topological Autoencoder
  - UMAP
  - UnionCom
  - scTopoGAN
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# spearman-correlation-benchmark-scoring

## Summary

Scores manifold-alignment methods on cross-modality prediction by running 50-nearest-neighbor weighted regression on each method's aligned single-cell embeddings and computing the median Spearman correlation between predicted and observed values per modality pair and dataset, as done in the scTopoGAN benchmark.

## When to use

Use when comparing alignment methods (e.g. scTopoGAN vs UnionCom, MMD-MA, SCOT) on their ability to predict one modality from another — ATAC↔RNA on Full PBMC or ADT↔RNA on BM — from unsupervised aligned embeddings.

## When NOT to use

- When the datasets share overlapping cells or features — current integration methods that rely on overlapping anchors are applicable instead of the fully unsupervised alignment + cross-modality prediction benchmark
- When the goal is assessing clustering quality of aligned embeddings — use the complementary Leiden clustering + ARI benchmark (Table 3) rather than cross-modality Spearman prediction

## Inputs

- Aligned low-dimensional embeddings for each method and dataset (Full PBMC and BM) produced by scTopoGAN, topoAE, Autoencoder, VAE, UMAP, UnionCom, or MMD-MA
- Pre-processed single-cell multi-omic data (RNA, ATAC, ADT) from the Data folder, including observed cross-modality feature values

## Outputs

- spearman_table.csv with per-method median Spearman correlation rows for each dataset and prediction direction (ATAC-from-RNA / RNA-from-ATAC on Full PBMC; ADT-from-RNA / RNA-from-ADT on BM)

## How to apply

Take the aligned embeddings produced by each benchmarked method (scTopoGAN, topoAE, Autoencoder, VAE, UMAP, UnionCom, MMD-MA) on the pre-processed Full PBMC and BM data from the Data folder. For each method and direction, perform 50-nearest-neighbor weighted-regression label prediction: predict ATAC from RNA and RNA from ATAC on Full PBMC, and ADT from RNA and RNA from ADT on BM. For each method/dataset/direction compute the Spearman correlation between predicted and observed cross-modality values, then take the median across cells and write the rows to spearman_table.csv (Supplementary Fig. S3 equivalent). Judge success by whether the reported ordering — scTopoGAN outperforming UnionCom and MMD-MA — is reproduced; in the paper scTopoGAN reached a median Spearman correlation of 0.044 predicting ATAC from RNA on Full PBMC versus 0.005 (UnionCom), 0.001 (MMD-MA), and −0.024 (SCOT), so near-zero values are expected in this fully unsupervised setting and negative medians indicate poor alignment.

## Related tools

- **scTopoGAN** (Produces the topology-guided GAN aligned embeddings being scored on cross-modality prediction) — github.com/AkashCiel/scTopoGAN
- **UnionCom** (Benchmarked competitor alignment method whose embeddings are scored via median Spearman correlation) — github.com/AkashCiel/scTopoGAN
- **MMD-MA** (Benchmarked competitor alignment method applied via manifold_align_mmd_pytorch.py) — github.com/AkashCiel/scTopoGAN
- **Topological Autoencoder** (Generates the per-modality latent representations (get_TopoAE_Embeddings) fed into scTopoGAN before scoring) — github.com/AkashCiel/scTopoGAN
- **UMAP** (Baseline manifold projection method included in the benchmark comparison) — github.com/AkashCiel/scTopoGAN
- **Python** (Implementation language for the alignment, regression, and Spearman correlation scoring scripts) — github.com/AkashCiel/scTopoGAN

## Examples

```
from scipy.stats import spearmanr, median; rho = spearmanr(pred_atac, obs_atac).statistic; med = median(rho)  # per method/dataset direction on aligned embeddings
```

## Evaluation signals

- spearman_table.csv exists and contains one median-Spearman row per method for all four dataset/direction combinations (Full PBMC ATAC↔RNA and BM ADT↔RNA)
- Reproduced scTopoGAN median Spearman for ATAC-from-RNA on Full PBMC is ≈ 0.044, exceeding UnionCom (0.005), MMD-MA (0.001), and SCOT (−0.024)
- The benchmark ordering scTopoGAN > UnionCom and MMD-MA holds across the scored datasets
- Median correlations are expected to be small (near zero) in this complete unsupervised, non-overlapping setting; negative medians flag alignment failure rather than a scoring bug

## Limitations

- Correlations depend on the quality of upstream embeddings from Topological Autoencoder and the GAN; weak per-modality low-dimensional structure propagates into low scores
- Spearman-based cross-modality prediction values are near zero for all methods in the fully unsupervised, non-overlapping setting, so absolute differences are small and ranking, not magnitude, is the meaningful signal
- No changelog found in the repository, so script behavior may not be traceable across commits

## Evidence

- [abstract] We use topological autoencoders (topoAE) to obtain latent representations of each modality separately.: "We use topological autoencoders (topoAE) to obtain latent representations of each modality separately."
- [methods] Both methods were used to benchmark the performance of scTopoGAN.: "Both methods were used to benchmark the performance of scTopoGAN."
- [methods] The ```UnionCom.py``` and ```manifold_align_mmd_pytorch.py``` scripts are used to apply UnionCom and MMD-MA, respectively.: "The ```UnionCom.py``` and ```manifold_align_mmd_pytorch.py``` scripts are used to apply UnionCom and MMD-MA, respectively."
- [abstract] We show that scTopoGAN outperforms state-of-the-art manifold alignment methods in complete unsupervised settings.: "We show that scTopoGAN outperforms state-of-the-art manifold alignment methods in complete unsupervised settings."
- [readme] This repository contains the Python implementation of the scTopoGAN method.: "This repository contains the Python implementation of the scTopoGAN method."
