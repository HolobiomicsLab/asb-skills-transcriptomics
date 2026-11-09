---
name: cell-type-clustering-unknown-subtypes
description: Use when raw scRNA-seq read count matrices from multiple batches contain
  cell types that are unknown a priori, and clustering plus batch-effect correction
  must be performed jointly without preliminary normalization.
when_to_use_negative:
- Cell-type labels for every cell are already known a priori — methods like ComBat
  or SVA designed for bulk experiments with subtype information suffice
- The experimental design is completely confounded (e.g. batch 1 measures cell types
  1 and 2 while batch 2 measures cell types 3 and 4) — no method is applicable
- All subtypes are known to be present in each batch and a simpler method such as
  MetaSparseKmeans would apply; BUSseq is aimed at unknown subtypes and designs allowing
  cell types to be missing from some batches
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BUSseq
  role: Bayesian hierarchical model that simultaneously corrects batch effects and
    clusters unknown cell types via MCMC inference
  repo: https://github.com/songfd2018/BUSseq
- name: BUSseq-Rpackage
  role: R package providing BUSseq_MCMC, BIC-based K selection, corrected_read_counts,
    imputed_read_counts, intrinsic_genes_BUSseq, and heatmap_data_BUSseq
  repo: https://github.com/songfd2018/BUSseq-Rpackage
- name: BUSseq-1.0
  role: C++ source with demo pipeline (simulate_data.R, run_MCMC.R) and command-line
    BUSseq/BUSseq_inference executables with -K, -i, -b, -s, -c parameters
  repo: https://github.com/songfd2018/BUSseq-1.0
- name: BUSseq_gpu
  role: GPU (CUDA C) implementation for faster execution of the BUSseq model
  repo: https://github.com/Anguscgm/BUSseq_gpu
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:a00f8035a03c8cb90b1c1b82a3ed80e7c4b7a2fd9d668134def9e5af2d95f411
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq
  tools:
  - BUSseq
  - BUSseq-1.0
  - BUSseq-Rpackage
  - BUSseq_gpu
  - LIGER
  - MNN
  - Scanorama
  - Seurat
  - ZINB-WaVE
  - scVI
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# cell-type-clustering-unknown-subtypes

## Summary

Simultaneously cluster scRNA-seq cells into unknown cell types and correct batch effects by fitting BUSseq, an interpretable Bayesian hierarchical model, via multi-core MCMC with K selected by BIC comparison.

## When to use

Use when raw scRNA-seq read count matrices from multiple batches contain cell types that are unknown a priori, and clustering plus batch-effect correction must be performed jointly without preliminary normalization.

## When NOT to use

- Cell-type labels for every cell are already known a priori — methods like ComBat or SVA designed for bulk experiments with subtype information suffice
- The experimental design is completely confounded (e.g. batch 1 measures cell types 1 and 2 while batch 2 measures cell types 3 and 4) — no method is applicable
- All subtypes are known to be present in each batch and a simpler method such as MetaSparseKmeans would apply; BUSseq is aimed at unknown subtypes and designs allowing cell types to be missing from some batches

## Inputs

- Raw G-by-N scRNA-seq read count matrix (genes x cells), batched across experiments
- Dimension file listing total cells, genes, number of batches, and per-batch cell counts (e.g. 450 / 1000 / 3 / 150 / 150 / 150)
- Metadata file with batch labels per cell
- Candidate range of cell-type numbers K (e.g. K = 3 to 6) for BIC comparison

## Outputs

- Per-cell cell-type cluster assignments at the selected optimal K
- Corrected values usable for downstream single-batch-style analysis
- BIC values per candidate K for optimal K selection
- Imputed read counts (dropout-imputed) and identified intrinsic genes
- Posterior sampling (MCMC_sampling_K*) and inference (Inference_K*) output folders

## How to apply

Compile the G-by-N read count matrices with a dimension file (N, G, number of batches, per-batch cell counts) and a metadata file, as in the BUSseq demo format (e.g. count_data_demo_v1.txt with dim_demo_v1.txt listing 450 cells, 1000 genes, 3 batches of 150 cells). Since the true number of cell types K is unknown, run BUSseq_MCMC for candidate K values (the vignettes enumerate K from 3 to 6) and select the optimal K by comparing BIC values, choosing the K with the minimum BIC. Run MCMC sampling with a fixed seed and multiple cores for parallelism (most MCMC steps are parallelizable), then run posterior inference with a chosen number of burn-in iterations. BUSseq simultaneously corrects batch effects, clusters cell types, imputes dropout-caused missing data, and identifies intrinsic genes, handling the count data nature, overdispersion, dropouts, and cell-specific sequencing depth without prior cell-type labels or normalization. Downstream analysis (e.g. t-SNE colored by batch and cell type, heatmap_data_BUSseq with the intrinsic gene set) is performed on the corrected values as if all cells were sequenced in a single batch.

## Related tools

- **BUSseq** (Bayesian hierarchical model that simultaneously corrects batch effects and clusters unknown cell types via MCMC inference) — https://github.com/songfd2018/BUSseq
- **BUSseq-Rpackage** (R package providing BUSseq_MCMC, BIC-based K selection, corrected_read_counts, imputed_read_counts, intrinsic_genes_BUSseq, and heatmap_data_BUSseq) — https://github.com/songfd2018/BUSseq-Rpackage
- **BUSseq-1.0** (C++ source with demo pipeline (simulate_data.R, run_MCMC.R) and command-line BUSseq/BUSseq_inference executables with -K, -i, -b, -s, -c parameters) — https://github.com/songfd2018/BUSseq-1.0
- **BUSseq_gpu** (GPU (CUDA C) implementation for faster execution of the BUSseq model) — https://github.com/Anguscgm/BUSseq_gpu

## Examples

```
BUSseq -d./ -r/your/count/data/directory/ -p mystudy -v 1 -K 6 -i 1000 -o 10 -s 1 -c 4 && BUSseq_inference -d./ -r/your/count/data/directory/ -p mystudy -v 1 -K 6 -i 1000 -b 200 -c 4
```

## Evaluation signals

- BIC vs K scatter shows a clear minimum at the selected optimal K (vignettes enumerate K from 3 to 6)
- t-SNE plots of the corrected values show cells separating by cell type rather than by batch
- Clustering accuracy against known labels is high (e.g. ARI 0.608 on the four human pancreas datasets, the highest among benchmarked methods LIGER 0.542, MNN 0.279, Scanorama 0.527, scVI 0.282, Seurat 0.287, ZINB-WaVE 0.380)
- Inference_K* and MCMC_sampling_K* output folders are populated and the MCMC run used a fixed reproducible seed and burn-in
- Heatmap of corrected values with the intrinsic gene set (heatmap_data_BUSseq) shows coherent islet vs non-islet separation on the pancreas data

## Limitations

- MCMC algorithms are well-known for heavy computation load, although the computational complexity of BUSseq is mitigated because most MCMC steps are parallelizable
- Experimental designs where batches are completely confounded with cell types admit no applicable method
- Cell type labels must remain unknown at inference time — the model is designed for the presence of unknown cell types, not supervised classification
- C++ source codes support Linux operating systems (tested on Ubuntu 18.04 and CentOS 7); the GPU version additionally requires a working GPU and nvcc

## Evidence

- [abstract] BUSseq can simultaneously correct batch effects, cluster cell types, impute dropout-caused missing data, and detect differentially expressed genes without preliminary normalization: "BUSseq can simultaneously correct batch effects, cluster cell types, impute missing data caused by dropout events, and detect differentially expressed genes without requiring a preliminary"
- [methods] K selection by BIC comparison enumerating K from 3 to 6: "To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6."
- [methods] Corrected values support downstream analysis as if a single batch: "the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch"
- [discussion] MCMC steps are parallelizable: "most steps of the MCMC algorithm for BUSseq are parallelizable (Supplementary Note 12)."
- [readme] Demo data format and BIC-based K selection in README: [evidence span withheld — anchor status: external_doc]
