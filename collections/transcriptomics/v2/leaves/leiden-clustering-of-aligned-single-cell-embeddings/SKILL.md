---
name: leiden-clustering-of-aligned-single-cell-embeddings
description: Use when you have embeddings from single-cell manifold-alignment methods
  on datasets with known cell-type labels (e.g. Full PBMC RNA/ATAC, BM RNA/ADT) and
  need to quantify unsupervised structure recovery via ARI.
when_to_use_negative:
- The data are raw expression/count matrices that have not yet been aligned into a
  common embedding space — this skill presupposes post-alignment embeddings from methods
  such as scTopoGAN, UnionCom, or MMD-MA.
- The datasets have no known cell-type labels, so the ARI between Leiden assignments
  and ground-truth labels cannot be computed.
- The goal is alignment method development (e.g. obtaining embeddings via get_TopoAE_Embeddings/run_scTopoGAN)
  rather than downstream evaluation of existing aligned embeddings.
edam_operation: http://edamontology.org/operation_3432
tools:
- name: scTopoGAN
  role: Produces the topology-guided GAN-aligned embeddings whose Leiden clustering
    ARI is evaluated and compared against benchmarks
  repo: github.com/AkashCiel/scTopoGAN
- name: Topological Autoencoder
  role: Obtains the low-dimensional manifold of each modality separately (get_TopoAE_Embeddings)
    before alignment; its embeddings are one of the evaluated inputs
  repo: github.com/AkashCiel/scTopoGAN
- name: UnionCom
  role: Benchmark alignment method whose aligned embeddings are clustered and scored
    with ARI for comparison against scTopoGAN
  repo: github.com/AkashCiel/scTopoGAN
- name: MMD-MA
  role: Benchmark alignment method (manifold_align_mmd_pytorch.py) whose aligned embeddings
    are clustered and scored with ARI for comparison against scTopoGAN
  repo: github.com/AkashCiel/scTopoGAN
- name: UMAP
  role: Manifold-projection baseline whose low-dimensional representations are included
    in the embedding comparison
  repo: github.com/AkashCiel/scTopoGAN
- name: Python
  role: Implementation language for the clustering/ARI evaluation scripts in the scTopoGAN
    repository
  repo: github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:13b0d048214291d6170efcd2d93d61272e60fd87d8a7312e7073e30feafe27ea
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# leiden-clustering-of-aligned-single-cell-embeddings

## Summary

Evaluates whether aligned single-cell embeddings (e.g. from scTopoGAN, topoAE, UnionCom, MMD-MA, UMAP) preserve biological structure by running Leiden clustering at a resolution matched to the number of known cell classes and scoring cluster-label agreement with the Adjusted Rand Index (ARI).

## When to use

Use when you have embeddings from single-cell manifold-alignment methods on datasets with known cell-type labels (e.g. Full PBMC RNA/ATAC, BM RNA/ADT) and need to quantify unsupervised structure recovery via ARI.

## When NOT to use

- The data are raw expression/count matrices that have not yet been aligned into a common embedding space — this skill presupposes post-alignment embeddings from methods such as scTopoGAN, UnionCom, or MMD-MA.
- The datasets have no known cell-type labels, so the ARI between Leiden assignments and ground-truth labels cannot be computed.
- The goal is alignment method development (e.g. obtaining embeddings via get_TopoAE_Embeddings/run_scTopoGAN) rather than downstream evaluation of existing aligned embeddings.

## Inputs

- Aligned embeddings for each method (scTopoGAN, topoAE, Autoencoder, VAE, UMAP, UnionCom, MMD-MA) on Full PBMC (RNA/ATAC) and BM (RNA/ADT)
- Known cell-type labels from the pre-processed data in the Data folder

## Outputs

- ari_table.csv with per-method ARI rows for all datasets (Table 3 equivalent)
- spearman_table.csv with per-method median Spearman correlations for cross-modality prediction (Supplementary Fig. S3 equivalent)

## How to apply

Load the aligned embeddings produced by each benchmarked method (scTopoGAN, topoAE, Autoencoder, VAE, UMAP, UnionCom, MMD-MA) together with the known cell-type labels from the pre-processed data in the Data folder. Run Leiden clustering on each aligned embedding with the clustering resolution matched to the number of known cell classes per dataset — this is the key decision point, since it makes cluster counts comparable across methods and datasets. Compute the Adjusted Rand Index (ARI) between Leiden cluster assignments and known cell labels for every method and dataset, and write the results to ari_table.csv (the Table 3 equivalent). As a complementary downstream check, perform 50-nearest-neighbor weighted-regression cross-modality label prediction (ATAC from RNA and RNA from ATAC on Full PBMC; ADT from RNA and RNA from ADT on BM) and record median Spearman correlations per method in spearman_table.csv (Supplementary Fig. S3 equivalent). Judge success by whether the benchmark ordering holds: scTopoGAN outperforms UnionCom and MMD-MA on both unsupervised clustering (ARI) and cross-modality prediction (median Spearman).

## Related tools

- **scTopoGAN** (Produces the topology-guided GAN-aligned embeddings whose Leiden clustering ARI is evaluated and compared against benchmarks) — github.com/AkashCiel/scTopoGAN
- **Topological Autoencoder** (Obtains the low-dimensional manifold of each modality separately (get_TopoAE_Embeddings) before alignment; its embeddings are one of the evaluated inputs) — github.com/AkashCiel/scTopoGAN
- **UnionCom** (Benchmark alignment method whose aligned embeddings are clustered and scored with ARI for comparison against scTopoGAN) — github.com/AkashCiel/scTopoGAN
- **MMD-MA** (Benchmark alignment method (manifold_align_mmd_pytorch.py) whose aligned embeddings are clustered and scored with ARI for comparison against scTopoGAN) — github.com/AkashCiel/scTopoGAN
- **UMAP** (Manifold-projection baseline whose low-dimensional representations are included in the embedding comparison) — github.com/AkashCiel/scTopoGAN
- **Python** (Implementation language for the clustering/ARI evaluation scripts in the scTopoGAN repository) — github.com/AkashCiel/scTopoGAN

## Evaluation signals

- ari_table.csv exists and contains per-method ARI rows for every method and dataset (Full PBMC, BM), matching the Table 3 schema
- Leiden clustering resolution was set to the number of known cell classes per dataset, ensuring cluster counts are comparable across methods
- spearman_table.csv exists with median Spearman correlations from 50-NN weighted-regression cross-modality prediction for all method/dataset pairs
- The benchmark ordering is reproduced: scTopoGAN outperforms UnionCom and MMD-MA (e.g. median Spearman 0.044 vs 0.005 and 0.001 for ATAC-from-RNA on Full PBMC)

## Limitations

- Leiden resolution must be matched to the number of known cell classes; using a mismatched resolution makes ARI values incomparable across methods and datasets.
- ARI requires known cell-type labels, so the evaluation is not applicable in fully unlabeled settings even though the alignment itself (scTopoGAN) is unsupervised.
- Absolute cross-modality prediction scores are low even for the best method (median Spearman 0.044 for ATAC-from-RNA on Full PBMC), so results should be interpreted as relative ordering between methods rather than strong absolute predictive performance.

## Evidence

- [other] Resolution matched to known classes: [evidence span withheld — anchor status: not_found]
- [other] ARI computation and output: [evidence span withheld — anchor status: not_found]
- [abstract] scTopoGAN benchmark claim: "We show that scTopoGAN outperforms state-of-the-art manifold alignment methods in complete unsupervised settings."
- [readme] Reproducibility script: "The scTopoGAN_results.py script reproduce the alignment results of scTopoGAN using different datasets."
