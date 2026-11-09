---
name: single-cell-multiomic-alignment-benchmark-evaluation
description: Use when you have aligned single-cell embeddings from multiple alignment
  methods (e.g. scTopoGAN, UnionCom, MMD-MA) on paired-modality datasets (RNA/ATAC
  PBMC, RNA/ADT BM) and must rank them on clustering and cross-modality prediction
  performance.
when_to_use_negative:
- The data has overlapping cells or features between modalities — standard integration
  methods that exploit overlaps apply, and the unsupervised alignment benchmark setting
  here is for non-overlapping modalities.
- You only need the per-modality low-dimensional representations themselves (the TopoAE-vs-Autoencoder/VAE/UMAP
  benchmark via TopoAE_benchmark.py), not a comparison of full alignment methods on
  downstream tasks.
- Inputs are raw or unaligned single-cell matrices rather than method-produced aligned
  embeddings — run the alignment step (e.g. run_scTopoGAN) first.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3361
tools:
- name: scTopoGAN
  role: Method under benchmark; its aligned embeddings are scored by Leiden clustering
    ARI and cross-modality Spearman correlation, and it is the reference method expected
    to outperform UnionCom and MMD-MA
  repo: https://github.com/AkashCiel/scTopoGAN
- name: Topological Autoencoder
  role: Produces the per-modality low-dimensional manifolds (via get_TopoAE_Embeddings)
    whose alignment quality is evaluated in this benchmark
  repo: https://github.com/AkashCiel/scTopoGAN
- name: UnionCom
  role: Benchmarked competitor method applied via UnionCom.py; its aligned embeddings
    are evaluated on the same downstream tasks
  repo: https://github.com/AkashCiel/scTopoGAN
- name: MMD-MA
  role: Benchmarked competitor method applied via manifold_align_mmd_pytorch.py; its
    aligned embeddings are evaluated on the same downstream tasks
  repo: https://github.com/AkashCiel/scTopoGAN
- name: UMAP
  role: Manifold projection baseline compared against topoAE in the TopoAE_benchmark.py
    comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:c52eb84391b8a9cf9d26a2db307eabc088b39cbe475bbb441236e4706b3cf1c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AkashCiel/scTopoGAN
  tools:
  - MMD-MA
  - Topological Autoencoder
  - UMAP
  - UnionCom
  - scTopoGAN
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# single-cell-multiomic-alignment-benchmark-evaluation

## Summary

Benchmark evaluation of unsupervised single-cell manifold alignment methods (scTopoGAN, topoAE, Autoencoder, VAE, UMAP, UnionCom, MMD-MA) by scoring their aligned embeddings on two downstream tasks: Leiden clustering agreement with known cell labels (Adjusted Rand Index) and 50-nearest-neighbor weighted-regression cross-modality prediction scored by median Spearman correlation.

## When to use

Use when you have aligned single-cell embeddings from multiple alignment methods (e.g. scTopoGAN, UnionCom, MMD-MA) on paired-modality datasets (RNA/ATAC PBMC, RNA/ADT BM) and must rank them on clustering and cross-modality prediction performance.

## When NOT to use

- The data has overlapping cells or features between modalities — standard integration methods that exploit overlaps apply, and the unsupervised alignment benchmark setting here is for non-overlapping modalities.
- You only need the per-modality low-dimensional representations themselves (the TopoAE-vs-Autoencoder/VAE/UMAP benchmark via TopoAE_benchmark.py), not a comparison of full alignment methods on downstream tasks.
- Inputs are raw or unaligned single-cell matrices rather than method-produced aligned embeddings — run the alignment step (e.g. run_scTopoGAN) first.

## Inputs

- Aligned low-dimensional embeddings for each modality produced by each benchmarked alignment method (scTopoGAN, topoAE, Autoencoder, VAE, UMAP, UnionCom, MMD-MA)
- Pre-processed single-cell data from the Data folder of the scTopoGAN repository, including known cell-type labels
- Full PBMC dataset (RNA + ATAC modalities) and BM dataset (RNA + ADT modalities)

## Outputs

- ari_table.csv — per-method ARI values from Leiden clustering vs known cell labels for every dataset (Table 3 equivalent)
- spearman_table.csv — per-method median Spearman correlations between predicted and observed cross-modality values (Supplementary Fig. S3 equivalent)

## How to apply

Load the aligned embeddings produced by each method together with the pre-processed data in the Data folder, which carries the known cell-type labels. Run Leiden clustering on each aligned embedding with the clustering resolution matched to the number of known cell classes per dataset, and compute the Adjusted Rand Index (ARI) between cluster assignments and known labels per method and dataset, writing results to ari_table.csv (Table 3 equivalent). Then perform 50-nearest-neighbor weighted-regression cross-modality label prediction — ATAC from RNA and RNA from ATAC on Full PBMC, and ADT from RNA and RNA from ADT on BM — using each method's aligned embeddings, and compute the median Spearman correlation between predicted and observed cross-modality values per method and dataset, writing to spearman_table.csv (Supplementary Fig. S3 equivalent). Judge correctness by checking that both CSVs contain per-method rows for all datasets and that values reproduce the reported ordering (e.g. scTopoGAN median Spearman 0.044 vs 0.005, 0.001, −0.024 for UnionCom, MMD-MA, SCOT when predicting ATAC from RNA on Full PBMC).

## Related tools

- **scTopoGAN** (Method under benchmark; its aligned embeddings are scored by Leiden clustering ARI and cross-modality Spearman correlation, and it is the reference method expected to outperform UnionCom and MMD-MA) — https://github.com/AkashCiel/scTopoGAN
- **Topological Autoencoder** (Produces the per-modality low-dimensional manifolds (via get_TopoAE_Embeddings) whose alignment quality is evaluated in this benchmark) — https://github.com/AkashCiel/scTopoGAN
- **UnionCom** (Benchmarked competitor method applied via UnionCom.py; its aligned embeddings are evaluated on the same downstream tasks) — https://github.com/AkashCiel/scTopoGAN
- **MMD-MA** (Benchmarked competitor method applied via manifold_align_mmd_pytorch.py; its aligned embeddings are evaluated on the same downstream tasks) — https://github.com/AkashCiel/scTopoGAN
- **UMAP** (Manifold projection baseline compared against topoAE in the TopoAE_benchmark.py comparison)

## Examples

```
python scTopoGAN_results.py
```

## Evaluation signals

- ari_table.csv exists with per-method ARI rows for every method–dataset combination, with Leiden resolution matched to the number of known cell classes per dataset
- spearman_table.csv exists with median Spearman correlations for all four prediction tasks (ATAC from RNA, RNA from ATAC on Full PBMC; ADT from RNA, RNA from ADT on BM)
- scTopoGAN attains a median Spearman correlation of 0.044 for ATAC-from-RNA prediction on Full PBMC, outperforming UnionCom (0.005), MMD-MA (0.001), and SCOT (−0.024)
- Values reproduce the reported benchmark ordering in which scTopoGAN outperforms UnionCom and MMD-MA in complete unsupervised settings

## Limitations

- The benchmark is restricted to complete unsupervised settings with non-overlapping cells or features; current integration methods relying on overlapping features or cells are out of scope.
- Cross-modality prediction correlations are low in absolute terms (median Spearman ~0.044 for the best method), so results should be interpreted comparatively rather than as strong absolute predictive performance.
- The evaluation requires pre-processed input data as provided in the Data folder; it was not validated on arbitrary new datasets in this workflow.

## Evidence

- [results] On the downstream cross-modality prediction task, scTopoGAN outperformed all other methods when predicting ATAC from RNA on the Full PBMC dataset, with a median Spearman correlation of 0.044: [evidence span withheld — anchor status: not_found]
- [full_text] Run Leiden clustering on each aligned embedding with the clustering resolution matched to the number of known cell classes per dataset: [evidence span withheld — anchor status: not_found]
- [methods] The ```UnionCom.py``` and ```manifold_align_mmd_pytorch.py``` scripts are used to apply UnionCom and MMD-MA, respectively. Both methods were used to benchmark the performance of scTopoGAN.: "Both methods were used to benchmark the performance of scTopoGAN."
- [readme] The ```scTopoGAN_results.py``` script reproduce the alignment results of scTopoGAN using different datasets.: "The ```scTopoGAN_results.py``` script reproduce the alignment results of scTopoGAN using different datasets."
- [abstract] Current integration methods rely on overlapping features or cells, limiting application to experiments where modalities are profiled in different subsets of cells: "Current integration methods rely on overlapping features or cells to link datasets measuring different modalities"
