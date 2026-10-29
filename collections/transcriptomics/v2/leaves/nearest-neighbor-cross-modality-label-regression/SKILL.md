---
name: nearest-neighbor-cross-modality-label-regression
description: Use when you have aligned latent embeddings of two single-cell modalities
  and need a quantitative downstream benchmark of alignment quality via cross-modality
  value prediction (Spearman correlations), complementing Leiden-clustering ARI.
when_to_use_negative:
- The two modalities share overlapping features or cells — current integration methods
  built on that overlap are sufficient, and the unsupervised alignment/skill pipeline
  is designed for non-overlapping settings
- The embeddings have not yet been aligned (e.g. raw counts straight from the Data
  folder) — NN regression must operate on a common latent space from run_scTopoGAN
  or a benchmarked alignment method
- The goal is standalone evaluation of individual-modality low-dimensional representations
  (e.g. TopoAE vs UMAP benchmarking via TopoAE_benchmark.py) rather than cross-modality
  prediction
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scTopoGAN
  role: Produces the topology-guided GAN-aligned embeddings being scored by the 50-NN
    cross-modality regression; the method expected to outperform UnionCom and MMD-MA
    in the benchmark ordering
  repo: github.com/AkashCiel/scTopoGAN
- name: Topological Autoencoder
  role: Obtains the low-dimensional manifold of each dataset separately via get_TopoAE_Embeddings
    before GAN alignment, providing the per-modality latent spaces used downstream
  repo: github.com/AkashCiel/scTopoGAN
- name: UnionCom
  role: Benchmarked alignment baseline applied via the UnionCom.py script; its aligned
    embeddings are scored with the same 50-NN regression and Spearman evaluation
  repo: github.com/AkashCiel/scTopoGAN
- name: MMD-MA
  role: Benchmarked alignment baseline applied via the manifold_align_mmd_pytorch.py
    script; its aligned embeddings are scored with the same 50-NN regression and Spearman
    evaluation
  repo: github.com/AkashCIEL/scTopoGAN
- name: UMAP
  role: Included as an alignment/representation baseline whose embeddings are scored
    with the same regression benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:bd9fa4df357fbb5b7ee32c5b0def4e921d6f3f9a45f6b63f2dcc9497b64ddae6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MMD-MA
  - Topological Autoencoder
  - UMAP
  - UnionCom
  - scTopoGAN
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# nearest-neighbor-cross-modality-label-regression

## Summary

Evaluates the biological utility of aligned single-cell embeddings by using 50-nearest-neighbor weighted regression to predict one modality's feature values from another (ATAC from RNA, RNA from ATAC on Full PBMC; ADT from RNA, RNA from ADT on BM), scoring each method by the median Spearman correlation between predicted and observed values.

## When to use

Use when you have aligned latent embeddings of two single-cell modalities and need a quantitative downstream benchmark of alignment quality via cross-modality value prediction (Spearman correlations), complementing Leiden-clustering ARI.

## When NOT to use

- The two modalities share overlapping features or cells — current integration methods built on that overlap are sufficient, and the unsupervised alignment/skill pipeline is designed for non-overlapping settings
- The embeddings have not yet been aligned (e.g. raw counts straight from the Data folder) — NN regression must operate on a common latent space from run_scTopoGAN or a benchmarked alignment method
- The goal is standalone evaluation of individual-modality low-dimensional representations (e.g. TopoAE vs UMAP benchmarking via TopoAE_benchmark.py) rather than cross-modality prediction

## Inputs

- Aligned latent embeddings per method (scTopoGAN, topoAE, Autoencoder, VAE, UMAP, UnionCom, MMD-MA) for Full PBMC and BM datasets
- Pre-processed single-modality feature matrices (RNA, ATAC, ADT) from the Data folder
- Known cell-type labels for each dataset

## Outputs

- spearman_table.csv with per-method median Spearman correlations for cross-modality prediction on Full PBMC (ATAC↔RNA) and BM (ADT↔RNA)
- ari_table.csv (companion Leiden-clustering ARI benchmark, Table 3 equivalent)

## How to apply

Load the aligned embeddings produced by each benchmarked alignment method (scTopoGAN, topoAE, Autoencoder, VAE, UMAP, UnionCom, MMD-MA) together with the pre-processed data from the Data folder containing the observed per-modality feature matrices. For each method and dataset, perform 50-nearest-neighbor weighted regression: for each cell, use its 50 nearest neighbors in the source modality's aligned embedding to compute a weighted prediction of the target modality's feature values (ATAC from RNA and RNA from ATAC on Full PBMC; ADT from RNA and RNA from ADT on BM). Compute the Spearman correlation between predicted and observed values per feature, then take the median across features as the per-method/per-dataset score, writing rows to spearman_table.csv. Run this alongside Leiden clustering with resolution matched to the number of known cell classes and report ARI in ari_table.csv, so clustering and prediction benchmarks are directly comparable. Rationale: an alignment that preserves cell identity and modality structure should enable accurate cross-modality prediction; scTopoGAN achieved a median Spearman correlation of 0.044 predicting ATAC from RNA on Full PBMC versus 0.005, 0.001 and −0.024 for UnionCom, MMD-MA and SCOT.

## Related tools

- **scTopoGAN** (Produces the topology-guided GAN-aligned embeddings being scored by the 50-NN cross-modality regression; the method expected to outperform UnionCom and MMD-MA in the benchmark ordering) — github.com/AkashCiel/scTopoGAN
- **Topological Autoencoder** (Obtains the low-dimensional manifold of each dataset separately via get_TopoAE_Embeddings before GAN alignment, providing the per-modality latent spaces used downstream) — github.com/AkashCiel/scTopoGAN
- **UnionCom** (Benchmarked alignment baseline applied via the UnionCom.py script; its aligned embeddings are scored with the same 50-NN regression and Spearman evaluation) — github.com/AkashCiel/scTopoGAN
- **MMD-MA** (Benchmarked alignment baseline applied via the manifold_align_mmd_pytorch.py script; its aligned embeddings are scored with the same 50-NN regression and Spearman evaluation) — github.com/AkashCIEL/scTopoGAN
- **UMAP** (Included as an alignment/representation baseline whose embeddings are scored with the same regression benchmark)

## Examples

```
python scTopoGAN_results.py  # produce aligned embeddings; then 50-NN weighted regression on Full PBMC/BM embeddings to fill spearman_table.csv
```

## Evaluation signals

- spearman_table.csv exists and contains one median-Spearman row per method for each prediction direction (ATAC from RNA and RNA from ATAC on Full PBMC; ADT from RNA and RNA from ADT on BM)
- scTopoGAN's median Spearman correlation for ATAC-from-RNA on Full PBMC is ~0.044, with UnionCom ~0.005, MMD-MA ~0.001 and SCOT ~−0.024
- Reported benchmark ordering is reproduced: scTopoGAN outperforms UnionCom and MMD-MA in the complete unsupervised setting
- ari_table.csv companion results use Leiden clustering with resolution matched to the number of known cell classes per dataset, and both CSVs cover all benchmarked methods
- Negative or near-zero median Spearman values are retained (e.g. SCOT's −0.024) rather than clipped, so method comparison stays comparable

## Limitations

- Median Spearman correlations are low in absolute terms even for the best method (0.044), so the benchmark is primarily a relative ranking of alignment methods rather than a demonstration of accurate per-feature prediction
- Only four datasets/settings are covered (Full PBMC ATAC↔RNA and BM ADT↔RNA), so generalization to other tissue/modalities is untested
- Evaluation requires pre-processed data from the Data folder as input; results are not reproducible from raw sequencing output

## Evidence

- [abstract] We use topological autoencoders (topoAE) to obtain latent representations of each modality separately.: "We use topological autoencoders (topoAE) to obtain latent representations of each modality separately."
- [abstract] We show that scTopoGAN outperforms state-of-the-art manifold alignment methods in complete unsupervised settings.: "We show that scTopoGAN outperforms state-of-the-art manifold alignment methods in complete unsupervised settings."
- [methods] run_scTopoGAN aligns the source manifold to the target manifold using the topology-guided GAN model.: "run_scTopoGAN which aligns the source manifold to the target manifold using the topology-guided GAN model."
- [methods] this script takes as input the pre-processed data provided in the Data folder: "this script takes as input the pre-processed data provided in the Data folder"
- [methods] Both methods were used to benchmark the performance of scTopoGAN.: "Both methods were used to benchmark the performance of scTopoGAN."
- [readme] The scTopoGAN_results.py script reproduce the alignment results of scTopoGAN using different datasets.: "The scTopoGAN_results.py script reproduce the alignment results of scTopoGAN using different datasets."
