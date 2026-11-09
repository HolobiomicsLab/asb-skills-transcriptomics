---
name: batch-effects-correction-single-cell-rna-seq
description: Use when integrating raw scRNA-seq read count matrices across multiple
  batches or platforms (e.g., GEO datasets like GSE85241 or GSE86473) where cell-type
  labels are unknown and batch effects and dropout events must be corrected before
  clustering.
when_to_use_negative:
- The experimental design is completely confounded (e.g., batch 1 measures cell types
  1 and 2, batch 2 measures cell types 3 and 4) — no method, including BUSseq, is
  applicable.
- Cell-type/subtype labels are already known a priori and the data are bulk rather
  than single-cell — classic methods like ComBat or SVA designed for bulk experiments
  suffice.
- All subtypes are guaranteed present in every batch and the goal is only subtype
  identification — MetaSparseKmeans-style joint clustering may be considered instead.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BUSseq
  role: Primary Bayesian hierarchical model fitted with BUSseq_MCMC to correct batch
    effects and cluster unknown cell types; optimal K chosen by BIC
  repo: https://github.com/songfd2018/BUSseq-Rpackage
- name: BUSseq-1.0
  role: Compiled C++ MCMC implementation run with flags -d -r -p -v -K -i -o -s -c
    for parallel sampling and BUSseq_inference
  repo: https://github.com/songfd2018/BUSseq-1.0
- name: BUSseq_gpu
  role: CUDA C GPU version of BUSseq compiled with nvcc and run with -B (batches),
    -N (dimension file), -G (genes) arguments
  repo: https://github.com/Anguscgm/BUSseq_gpu
- name: Scanorama
  role: MNN-based batch correction competitor benchmarked via ARI (0.527 pancreas,
    0.518 hematopoietic)
- name: Seurat
  role: Shared-variation integration competitor benchmarked via ARI (0.287 pancreas,
    0.266 hematopoietic)
- name: LIGER
  role: Shared-variation integration competitor benchmarked via ARI (0.542 pancreas,
    0.307 hematopoietic)
- name: MNN
  role: Mutual-nearest-neighbor competitor benchmarked via ARI (0.279 pancreas, 0.575
    hematopoietic)
- name: ZINB-WaVE
  role: Zero-inflated factor-model competitor benchmarked via ARI (0.380 pancreas,
    0.348 hematopoietic)
- name: ComBat
  role: Bulk-design batch correction baseline noted in the intro; used as a comparison
    method in the hematopoietic benchmark
merged_aliases:
- single-cell-rna-seq-batch-effect-correction
merged_alias_records:
- alias: single-cell-rna-seq-batch-effects-correction
  slug: single-cell-rna-seq-batch-effects-correction
  jaccard_score: 1.0
  method: token-set-jaccard
  decision: auto
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:c26f81253f051977c1d9948ef0705816a2329028b85b8e9aed93443707261d17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq-Rpackage
  tools:
  - BUSseq
  - BUSseq-1.0
  - BUSseq_gpu
  - ComBat
  - ComBat-seq
  - Harmony
  - LIGER
  - MNN
  - R
  - RUV
  - SVA
  - Scanorama
  - Seurat
  - Seurat v5
  - ZINB-WaVE
  - gplots
  - scMerge
  - scVI
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# batch-effects-correction-single-cell-rna-seq

## Summary

Apply BUSseq, an interpretable Bayesian hierarchical model, to simultaneously correct batch effects, cluster unknown cell types, impute dropout-caused missing data, and detect differentially expressed genes in scRNA-seq count matrices, and benchmark the corrected clustering against competing methods using adjusted Rand index and silhouette coefficients.

## When to use

Use when integrating raw scRNA-seq read count matrices across multiple batches or platforms (e.g., GEO datasets like GSE85241 or GSE86473) where cell-type labels are unknown and batch effects and dropout events must be corrected before clustering.

## When NOT to use

- The experimental design is completely confounded (e.g., batch 1 measures cell types 1 and 2, batch 2 measures cell types 3 and 4) — no method, including BUSseq, is applicable.
- Cell-type/subtype labels are already known a priori and the data are bulk rather than single-cell — classic methods like ComBat or SVA designed for bulk experiments suffice.
- All subtypes are guaranteed present in every batch and the goal is only subtype identification — MetaSparseKmeans-style joint clustering may be considered instead.

## Inputs

- G-by-N raw scRNA-seq read count matrix (e.g., count_data_[study]_v[version].txt)
- dim_[study]_v[version].txt dimension file (N cells, G genes, B batches, per-batch cell counts)
- metadata file with batch and (if known) cell-type labels
- Preprocessed matrix of highly variable genes (e.g., 7095 cells x 2480 HVGs for pancreas; 3470 HVGs for hematopoietic data)

## Outputs

- BUSseqfits object (sce_BUSseqfit) with fitted cell-type clusters and BIC per candidate K
- corrected_read_counts(sce_BUSseqfit) batch-corrected count matrix
- imputed_read_counts(sce_BUSseqfit) dropout-imputed counts
- intrinsic_genes_BUSseq(sce_BUSseqfit) intrinsic gene set
- Per-method ARI and silhouette comparison table (e.g., batch_correction_benchmark.csv)
- t-SNE plots colored by batch and cell type, and heatmap_data_BUSseq islet/non-islet heatmap

## How to apply

Compile raw count matrices into a common G-by-N read count matrix with a dimension file listing total cells, genes, number of batches, and per-batch cell counts. Fit BUSseq via BUSseq_MCMC (or the compiled C++ BUSseq executable / CUDA version) enumerating candidate cell-type numbers K (e.g., K from 3 to 6) and selecting the optimal K by minimum BIC, using multi-core parallel MCMC (e.g., 500 iterations on four cores) with a fixed seed for reproducibility. Run each competing correction method (MNN, Scanorama, scVI, Seurat, LIGER, ZINB-WaVE, ComBat, SVA, RUV, scMerge) with its standard pipeline on the same preprocessed data. Compute the adjusted Rand index of clusters against known labels and median silhouette coefficients of corrected values, then extract corrected_read_counts / imputed_read_counts from the BUSseqfit object for downstream analysis as if all cells were sequenced in a single batch. Validate that BUSseq attains the top ARI (0.608 on pancreas, 0.582 on hematopoietic data) and visualize islet vs non-islet separation with heatmap_data_BUSseq using the intrinsic gene set.

## Related tools

- **BUSseq** (Primary Bayesian hierarchical model fitted with BUSseq_MCMC to correct batch effects and cluster unknown cell types; optimal K chosen by BIC) — https://github.com/songfd2018/BUSseq-Rpackage
- **BUSseq-1.0** (Compiled C++ MCMC implementation run with flags -d -r -p -v -K -i -o -s -c for parallel sampling and BUSseq_inference) — https://github.com/songfd2018/BUSseq-1.0
- **BUSseq_gpu** (CUDA C GPU version of BUSseq compiled with nvcc and run with -B (batches), -N (dimension file), -G (genes) arguments) — https://github.com/Anguscgm/BUSseq_gpu
- **Scanorama** (MNN-based batch correction competitor benchmarked via ARI (0.527 pancreas, 0.518 hematopoietic))
- **Seurat** (Shared-variation integration competitor benchmarked via ARI (0.287 pancreas, 0.266 hematopoietic))
- **LIGER** (Shared-variation integration competitor benchmarked via ARI (0.542 pancreas, 0.307 hematopoietic))
- **MNN** (Mutual-nearest-neighbor competitor benchmarked via ARI (0.279 pancreas, 0.575 hematopoietic))
- **ZINB-WaVE** (Zero-inflated factor-model competitor benchmarked via ARI (0.380 pancreas, 0.348 hematopoietic))
- **ComBat** (Bulk-design batch correction baseline noted in the intro; used as a comparison method in the hematopoietic benchmark)

## Examples

```
BUSseq_MCMC(ObservedData, n.celltypes=K, seed, n.cores, n.iterations, n.burnin, n.unchanged, n.output, working_dir, Drop_ind, fdr, hyper_pi, hyper_gamma0, hyper_gamma1, hyper_alpha, tau1sq, hyper_p)
```

## Evaluation signals

- BUSseq ARI is the maximum across all benchmarked methods: 0.608 on pancreas (vs LIGER 0.542, Scanorama 0.527, ZINB-WaVE 0.380, Seurat 0.287, scVI 0.282, MNN 0.279) and 0.582 on hematopoietic data (vs MNN 0.575, ZINB-WaVE 0.348, LIGER 0.307, Scanorama 0.518, Seurat 0.266, scVI 0.197).
- Optimal K is the one minimizing BIC across enumerated candidates (K = 3 to 6 in vignettes); the K-vs-BIC scatter plot should show a clear minimum at the chosen K.
- BUSseq also attains the highest median silhouette coefficient on corrected values among all seven methods, and islet vs non-islet cells separate cleanly in the heatmap_data_BUSseq heatmap over intrinsic genes.
- Corrected values can be used for downstream analysis as if all cells were sequenced in a single batch — no batch-driven separation should dominate t-SNE plots colored by batch label.
- Benchmark table (e.g., batch_correction_benchmark.csv) reproduces per-method ARI/silhouette values within tolerance.

## Limitations

- MCMC algorithms carry a heavy computational load; runtimes reported were on 4 Intel i7-6700HQ cores, and C++ codes support Linux only (tested on Ubuntu 18.04 and CentOS 7 with R 3.6.0).
- BUSseq requires the experimental design to fulfill identifiability conditions (completely randomized, reference panel, or chain-type designs); completely confounded designs admit no applicable method.
- Multi-stage pipelines that use clustering results in later stages ignore clustering uncertainties, which can lead to false positives and false negatives.
- Downloading raw count data for reproduction requires the authors' OneDrive share (password BUSseq2019) placed into HumanPancreas and MouseHematopoietic RawCountData folders.

## Evidence

- [discussion] BUSseq can simultaneously correct batch effects, cluster cell types, impute missing data, and identify intrinsic genes that differentiate cell types.: "BUSseq simultaneously corrects batch effects, clusters cell types, imputes missing data, and identifies intrinsic genes that differentiate cell types."
- [methods] To select the optimal K, we need to compare the BIC values for different Ks. In the vignettes, we enumerate K from 3 to 6.: "To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6."
- [methods] the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch: "the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch"
- [methods] install_github("songfd2018/BUSseq-Rpackage") # install BUSseq: "install_github("songfd2018/BUSseq-Rpackage") # install BUSseq"
- [intro] for completely confounded designs—for example one in which batch 1 measures cell type 1 and 2, whereas batch 2 measures cell type 3 and 4—no method is applicable: "for completely confounded designs—for example one in which batch 1 measures cell type 1 and 2, whereas batch 2 measures cell type 3 and 4—no method is applicable"
- [readme] most existing methods for batch effects correction, such as Combat and the surrogate variable analysis (SVA), are designed for bulk experiments: "most existing methods for batch effects correction, such as Combat and the surrogate variable analysis (SVA), are designed for bulk experiments"
