---
name: multi-platform-scrna-seq-data-integration
description: Use when raw read count matrices from multiple scRNA-seq batches or platforms
  must be integrated while cell-type labels are unknown, and batch effects, overdispersion,
  dropout events, and cell-specific sequencing depth need joint correction.
when_to_use_negative:
- The experimental design is completely confounded (e.g., batch 1 measures only cell
  types 1 and 2, batch 2 only cell types 3 and 4) — the article states no method,
  including BUSseq, is applicable
- The design violates the conditions of the completely randomized, reference panel,
  or chain-type designs under which biological variability can be separated from batch
  effects
- Input is already normalized/transformed expression data — BUSseq takes raw read
  counts and models count nature, overdispersion, and dropouts itself
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BUSseq
  role: 'Primary method: Bayesian hierarchical model fit via parallel MCMC to simultaneously
    correct batch effects, cluster cell types, impute dropouts, and detect intrinsic
    genes across batches/platforms'
  repo: https://github.com/songfd2018/BUSseq
- name: BUSseq-Rpackage
  role: R package providing BUSseq_MCMC, corrected_read_counts, imputed_read_counts,
    intrinsic_genes_BUSseq, and heatmap_data_BUSseq functions
  repo: https://github.com/songfd2018/BUSseq-Rpackage
- name: BUSseq-1.0
  role: C++ source implementation; compiled with make and run via BUSseq and BUSseq_inference
    executables with -K, -i, -b, -s, -c flags
  repo: https://github.com/songfd2018/BUSseq-1.0
- name: BUSseq-1.1_implementation
  role: Reproduction code for the manuscript's human pancreas and mouse hematopoietic
    studies; RawCountData folders are placed into HumanPancreas/MouseHematopoietic
    folders
  repo: https://github.com/songfd2018/BUSseq-1.1_implementation
- name: BUSseq_gpu
  role: GPU (CUDA C) version of BUSseq compiled with nvcc for accelerated execution
  repo: https://github.com/Anguscgm/BUSseq_gpu
- name: MNN
  role: Competing batch-correction baseline; mutual-nearest-neighbor approach requiring
    batch effects almost orthogonal to biological subspaces
- name: Scanorama
  role: Competing MNN-based integration baseline benchmarked on the same preprocessed
    data
- name: Seurat
  role: Competing baseline that attempts to identify shared variation across batches
- name: LIGER
  role: Competing baseline that attempts to identify shared variation across batches
- name: scVI
  role: Competing deep-learning integration baseline in the pancreas ARI benchmark
    (ARI 0.282)
- name: ZINB-WaVE
  role: Competing baseline in the pancreas ARI benchmark (ARI 0.380)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:6939ff9c971014e599568cf6e6536cd6dcb5c5af02e065c06f503e133e56d790
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq
  tools:
  - BUSseq
  - BUSseq-1.0
  - BUSseq-1.1_implementation
  - BUSseq-Rpackage
  - BUSseq_gpu
  - LIGER
  - MNN
  - Scanorama
  - Seurat
  - ZINB-WaVE
  - scVI
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# multi-platform-scrna-seq-data-integration

## Summary

Integrate read count matrices from multiple scRNA-seq batches or platforms (e.g., CEL-seq2, Drop-seq, 10X Chromium) using BUSseq, a Bayesian hierarchical model (Batch Effects Correction with Unknown Subtypes) that simultaneously corrects batch effects, clusters cell types, imputes dropouts, and identifies intrinsic genes without preliminary normalization.

## When to use

Use when raw read count matrices from multiple scRNA-seq batches or platforms must be integrated while cell-type labels are unknown, and batch effects, overdispersion, dropout events, and cell-specific sequencing depth need joint correction.

## When NOT to use

- The experimental design is completely confounded (e.g., batch 1 measures only cell types 1 and 2, batch 2 only cell types 3 and 4) — the article states no method, including BUSseq, is applicable
- The design violates the conditions of the completely randomized, reference panel, or chain-type designs under which biological variability can be separated from batch effects
- Input is already normalized/transformed expression data — BUSseq takes raw read counts and models count nature, overdispersion, and dropouts itself

## Inputs

- Gene-by-cell raw read count matrices from each scRNA-seq batch/platform (e.g., count_data_[study]_v[version].txt)
- Dimension file with total cells, total genes, number of batches, and per-batch cell counts (dim_[study]_v[version].txt)
- Metadata file with batch labels per cell (metadata_[study]_v[version].txt)
- Optional known cell-type labels for benchmarking (e.g., ARI computation on the four human pancreas datasets GSE81076, GSE85241, GSE86473, E-MATB-5061)

## Outputs

- Fitted BUSseqfits object (BUSseqfits_example) with posterior clustering and batch-effect estimates
- Corrected read counts (corrected_read_counts) usable for downstream analysis as a single-batch dataset
- Imputed read counts (imputed_read_counts) for dropout-caused missing data
- Intrinsic gene set (intrinsic_genes_BUSseq) differentiating cell types
- Heatmaps of islet/non-islet or cell-type separation via heatmap_data_BUSseq

## How to apply

Compile per-batch read count matrices into a single gene-by-cell count matrix with dimension and metadata files (as in the BUSseq demo: count_data, dim, and metadata .txt files listing cell counts, gene counts, batch count, and per-batch cell counts). Fit BUSseq with BUSseq_MCMC(ObservedData, n.celltypes, seed, n.cores, n.iterations, n.burnin, ...) using multi-core parallel MCMC, enumerating candidate cell-type numbers K (e.g., K = 3 to 6) and selecting the K with the minimum BIC. Extract corrected_read_counts or imputed_read_counts from the fitted BUSseqfits object for downstream analysis as if all cells were sequenced in a single batch, and identify intrinsic genes with intrinsic_genes_BUSseq. Validate by computing adjusted Rand index of clusters against known labels and median silhouette coefficients, and by checking that a method's design does not completely confound batch with cell type (no method is applicable to fully confounded designs).

## Related tools

- **BUSseq** (Primary method: Bayesian hierarchical model fit via parallel MCMC to simultaneously correct batch effects, cluster cell types, impute dropouts, and detect intrinsic genes across batches/platforms) — https://github.com/songfd2018/BUSseq
- **BUSseq-Rpackage** (R package providing BUSseq_MCMC, corrected_read_counts, imputed_read_counts, intrinsic_genes_BUSseq, and heatmap_data_BUSseq functions) — https://github.com/songfd2018/BUSseq-Rpackage
- **BUSseq-1.0** (C++ source implementation; compiled with make and run via BUSseq and BUSseq_inference executables with -K, -i, -b, -s, -c flags) — https://github.com/songfd2018/BUSseq-1.0
- **BUSseq-1.1_implementation** (Reproduction code for the manuscript's human pancreas and mouse hematopoietic studies; RawCountData folders are placed into HumanPancreas/MouseHematopoietic folders) — https://github.com/songfd2018/BUSseq-1.1_implementation
- **BUSseq_gpu** (GPU (CUDA C) version of BUSseq compiled with nvcc for accelerated execution) — https://github.com/Anguscgm/BUSseq_gpu
- **MNN** (Competing batch-correction baseline; mutual-nearest-neighbor approach requiring batch effects almost orthogonal to biological subspaces)
- **Scanorama** (Competing MNN-based integration baseline benchmarked on the same preprocessed data)
- **Seurat** (Competing baseline that attempts to identify shared variation across batches)
- **LIGER** (Competing baseline that attempts to identify shared variation across batches)
- **scVI** (Competing deep-learning integration baseline in the pancreas ARI benchmark (ARI 0.282))
- **ZINB-WaVE** (Competing baseline in the pancreas ARI benchmark (ARI 0.380))

## Examples

```
devtools::install_github("songfd2018/BUSseq-Rpackage"); fit <- BUSseq_MCMC(ObservedData, n.celltypes = K, seed = 123, n.cores = 4, n.iterations = 2000, n.burnin = 1000); corrected <- corrected_read_counts(fit)
```

## Evaluation signals

- BIC decreases to a minimum at the selected number of cell types K when enumerating candidate K values (e.g., K from 3 to 6)
- Adjusted Rand index of BUSseq clusters against known cell-type labels is highest among methods (0.608 on pancreas data vs. LIGER 0.542, Scanorama 0.527, ZINB-WaVE 0.380, Seurat 0.287, scVI 0.282, MNN 0.279)
- Median silhouette coefficient computed on corrected values is highest among all benchmarked methods
- Corrected values support single-batch-style downstream analysis: heatmap of intrinsic genes (heatmap_data_BUSseq) shows clean islet vs. non-islet separation
- MCMC reproducibility: runs use an explicit seed (-s flag) and the reported per-K clustering is stable across burn-in iterations

## Limitations

- MCMC inference is computationally heavy; most steps are parallelizable (n.cores) and a GPU version exists, but runtime remains a constraint (demo: ~12 mins for 4 K values on 4 cores)
- C++ sources are tested only on Linux (Ubuntu 18.04, CentOS 7) and require installing RngStreams before compilation
- BUSseq cannot handle completely confounded designs where some cell types appear exclusively in single batches
- Multi-stage approaches that cluster before correcting ignore uncertainties in previous stages, which can lead to false positives and false negatives

## Evidence

- [readme] BUSseq can integrate read count matrices obtained from different scRNA-seq platforms and allow cell types to be measured in some but not all of the batches as long as the experimental design fulfills the conditions.: "BUSseq can integrate read count matrices obtained from different scRNA-seq platforms and allow cell types to be measured in some but not all of the batches"
- [methods] MCMC function signature and BIC-based K selection.: "`BUSseq_MCMC(ObservedData, n.celltypes, seed, n.cores, n.iterations, n.burnin, n.unchanged, n.output, working_dir, Drop_ind, fdr, hyper_pi, hyper_gamma0, hyper_gamma1, hyper_alpha, tau1sq, hyper_p,`"
- [methods] Optimal K chosen by comparing BIC values across candidates.: "To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6."
- [methods] Corrected values permit downstream analysis as a single batch.: "the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch"
- [intro] Completely confounded designs admit no applicable method.: "for completely confounded designs—for example one in which batch 1 measures cell type 1 and 2, whereas batch 2 measures cell type 3 and 4—no method is applicable"
- [readme] Raw count data for the pancreas reproduction must be downloaded and placed in study folders.: [evidence span withheld — anchor status: external_doc]
