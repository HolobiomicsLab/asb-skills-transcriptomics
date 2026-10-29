---
name: zero-inflated-poisson-count-downsampling
description: Use when you need to synthetically downsample scRNA-seq count profiles
  to create a degraded input (with a known ground-truth full-depth counterpart) for
  benchmarking denoising methods such as scPRINT, MAGIC, or KNNsmoothing2.
when_to_use_negative:
- When the goal is gene network inference rather than denoising benchmarking — scPRINT's
  GN inference benchmarks (BeeLine, MCalla et al., gwps K562) do not use this downsampling
  step
- When you already hold matched low-depth and high-depth measurements of the same
  cells — an empirical degradation control is preferable to simulated downsampling
- When the input data are not raw integer counts (e.g. normalized or log-transformed
  matrices) — the zero-inflated Poisson sampling of reads per gene assumes raw counts
edam_topics:
- http://edamontology.org/topic_3934
tools:
- name: scPRINT
  role: Provides the zero-inflated Poisson downsampling procedure used in its multi-task
    pretraining denoising task, and the pretrained model/Denoiser used to recover
    the downsampled profiles
  repo: https://github.com/cantinilab/scPRINT
- name: scdataloader
  role: Preprocesses the cellxgene test datasets (raw counts, gene format checks)
    before downsampling
  repo: https://github.com/jkobject/scDataLoader
- name: MAGIC
  role: Comparison denoising method applied to the downsampled profiles (Scanpy implementation,
    approximate solver, K=5 for reduced cell sets)
- name: KNNsmoothing2
  role: Comparison denoising method applied to the downsampled profiles with K=10
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:e3bab7d02d8c2e04aa087c1298e135b63b061075664ad6fd651701305ea9a1ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/scPRINT
  tools:
  - KNNsmoothing2
  - MAGIC
  - Python
  - Scanpy
  - scPRINT
  - scdataloader
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# zero-inflated-poisson-count-downsampling

## Summary

A controlled transcript-count perturbation procedure from the scPRINT paper that removes a defined fraction of counts per cell (e.g. 70%) using a zero-inflated Poisson model, splitting drops between Poisson-sampled reads per gene and complete gene dropouts. It provides a reproducible degradation step for evaluating denoising methods against true full-depth profiles.

## When to use

Use when you need to synthetically downsample scRNA-seq count profiles to create a degraded input (with a known ground-truth full-depth counterpart) for benchmarking denoising methods such as scPRINT, MAGIC, or KNNsmoothing2.

## When NOT to use

- When the goal is gene network inference rather than denoising benchmarking — scPRINT's GN inference benchmarks (BeeLine, MCalla et al., gwps K562) do not use this downsampling step
- When you already hold matched low-depth and high-depth measurements of the same cells — an empirical degradation control is preferable to simulated downsampling
- When the input data are not raw integer counts (e.g. normalized or log-transformed matrices) — the zero-inflated Poisson sampling of reads per gene assumes raw counts

## Inputs

- Raw single-cell RNA-seq count matrices (e.g. cellxgene datasets such as ciliary body, colon, retina), preprocessed with scdataloader's Preprocessor
- Downsampling fraction (e.g. 70% of transcript counts removed per cell)
- A seeded random number generator state for reproducible Poisson sampling and gene dropout

## Outputs

- Downsampled count profiles per cell (70% of transcript counts removed; half via Poisson sampling of reads per gene, half via complete gene dropout)
- Paired ground-truth full-depth profiles retained for Spearman correlation evaluation
- Per-cell correlation-increase metrics (e.g. denoising_benchmark_results.csv) over initially expressed genes, including rare cell-state subsets

## How to apply

Load the target scRNA-seq datasets (e.g. the ciliary body, colon, and retina cellxgene test sets) and preprocess them with scdataloader's Preprocessor so profiles contain raw counts with validated gene format. Apply scPRINT's zero-inflated Poisson downsampling to remove 70% of transcript counts per cell: half of the drops are performed by Poisson sampling of reads per gene and half by complete gene dropout, mirroring scPRINT's pretraining denoising task design. Keep the original full-depth profiles as ground truth. Downstream, denoise the downsampled profiles (e.g. with the pretrained scPRINT model loaded via scPrint.load_from_checkpoint(precpt_gene_emb=None) and the Denoiser on 10,000 cells and the 5000 most variable genes, or with MAGIC/Scanpy K=5 and KNNsmoothing2 K=10) and compute average Spearman correlation of denoised vs true profiles over genes initially expressed in each cell, reporting the increase over the downsampled baseline.

## Related tools

- **scPRINT** (Provides the zero-inflated Poisson downsampling procedure used in its multi-task pretraining denoising task, and the pretrained model/Denoiser used to recover the downsampled profiles) — https://github.com/cantinilab/scPRINT
- **scdataloader** (Preprocesses the cellxgene test datasets (raw counts, gene format checks) before downsampling) — https://github.com/jkobject/scDataLoader
- **MAGIC** (Comparison denoising method applied to the downsampled profiles (Scanpy implementation, approximate solver, K=5 for reduced cell sets))
- **KNNsmoothing2** (Comparison denoising method applied to the downsampled profiles with K=10)

## Evaluation signals

- The downsampled profile has exactly the intended fraction removed (70% of transcript counts per cell), with drops split between Poisson-sampled reads per gene and complete gene dropout
- Denoising the downsampled profiles should yield an average Spearman correlation increase over the downsampled baseline toward the true profiles, computable for every method
- Correlations are computed only over genes initially expressed in each cell, restricted to the 5000 most variable genes per dataset
- On rare cell populations (pericytes, microfold cells of epithelium of small intestine, microglial cells), scPRINT's Spearman correlation increase should exceed that of MAGIC and KNNsmoothing2 because it denoises each cell independently
- Results for all three methods across overall and rare-population subsets are saved to denoising_benchmark_results.csv

## Limitations

- The synthetic degradation assumes a zero-inflated Poisson model of the data; real low-depth artifacts may deviate from this model
- Denoising evaluation used a representative subset of 10,000 cells and the 5000 most variable genes per dataset, not the full datasets
- The paper notes that scPRINT's denoising advantage on rare cell states comes from denoising each cell independently rather than aggregating over neighbors, so neighbor-aggregating methods may behave differently on abundant populations

## Evidence

- [methods] Zero-inflated Poisson downsampling is the denoising-task degradation model used in scPRINT's pretraining.: "We downsample an expression profile using a zero-inflated Poisson model of the data."
- [methods] The denoising task is one of the three multi-task pretraining objectives, motivating the downsampling procedure.: "The three tasks of the multi-task pretraining are the denoising task, the classification task, and the bottleneck learning task."
- [abstract] The pretrained model confers zero-shot denoising ability that the downsampling benchmark evaluates.: "competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction"
