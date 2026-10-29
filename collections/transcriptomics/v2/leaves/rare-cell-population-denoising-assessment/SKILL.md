---
name: rare-cell-population-denoising-assessment
description: Use when comparing denoising performance of single-cell methods on downsampled
  count data, especially when rare cell states (e.g., pericytes, microfold cells,
  microglial cells) are present and neighbor-smoothing may bias results.
when_to_use_negative:
- When the task is gene network inference rather than denoising quality assessment
  (scPRINT's primary benchmark is GN inference, which uses BenGRN/GRnnData instead).
- When ground truth profiles are unavailable or datasets are so sparse that the true-profile
  Spearman baseline cannot be computed.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scPRINT
  role: Denoises each cell independently with the pretrained model (Denoiser) and
    provides the zero-inflated Poisson downsampling procedure
  repo: https://github.com/cantinilab/scPRINT
- name: MAGIC
  role: Baseline denoising method applied via the Scanpy implementation with the approximate
    solver (K=5)
- name: KNNsmoothing2
  role: Baseline KNN-based denoising method applied with default parameters and K=10
- name: Scanpy
  role: Provides the MAGIC implementation used for the comparison
- name: Python
  role: Runtime environment for the benchmarking workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:3c8a7934f7cb5275864b6a0120715ea0f14bfc2362d8611c6c7025bdb645738e
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# rare-cell-population-denoising-assessment

## Summary

A benchmarking protocol that evaluates how well single-cell denoising methods (scPRINT, MAGIC, KNNsmoothing2) recover true expression profiles after heavy transcript downsampling, with a dedicated analysis of rare cell populations where neighbor-aggregation methods fail.

## When to use

Use when comparing denoising performance of single-cell methods on downsampled count data, especially when rare cell states (e.g., pericytes, microfold cells, microglial cells) are present and neighbor-smoothing may bias results.

## When NOT to use

- When the task is gene network inference rather than denoising quality assessment (scPRINT's primary benchmark is GN inference, which uses BenGRN/GRnnData instead).
- When ground truth profiles are unavailable or datasets are so sparse that the true-profile Spearman baseline cannot be computed.

## Inputs

- cellxgene test datasets (ciliary body, colon, retina) as raw count matrices
- Pretrained scPRINT model checkpoint (precpt_gene_emb=None)
- Rare cell population labels (pericytes, microfold cells of epithelium of small intestine, microglial cells)

## Outputs

- denoising_benchmark_results.csv with overall and rare-population Spearman correlation increases for scPRINT, MAGIC, and KNNsmoothing2
- Per-cell average Spearman correlations between denoised and true profiles over initially expressed genes

## How to apply

Load test datasets (ciliary body, colon, retina) from cellxgene and preprocess with the scdataloader Preprocessor (raw counts, gene format checks). Remove 70% of transcript counts per cell using scPRINT's zero-inflated Poisson downsampling, split between Poisson sampling of reads per gene and complete gene dropout. Denoise with the pretrained scPRINT model (scPrint.load_from_checkpoint with precpt_gene_emb=None) via the Denoiser on 10,000 cells and the 5000 most variable genes per dataset; denoise in parallel with MAGIC (Scanpy implementation, approximate solver, K=5 on reduced cell sets) and KNNsmoothing2 (K=10). Compute the average Spearman correlation between denoised and true profiles over genes initially expressed in each cell, derive the correlation increase over the downsampled baseline, and repeat restricted to rare cell populations. The rationale: scPRINT denoises each cell independently without aggregating profiles over neighboring cells, so it should outperform neighborhood-based methods on rare cell states.

## Related tools

- **scPRINT** (Denoises each cell independently with the pretrained model (Denoiser) and provides the zero-inflated Poisson downsampling procedure) — https://github.com/cantinilab/scPRINT
- **MAGIC** (Baseline denoising method applied via the Scanpy implementation with the approximate solver (K=5))
- **KNNsmoothing2** (Baseline KNN-based denoising method applied with default parameters and K=10)
- **Scanpy** (Provides the MAGIC implementation used for the comparison)
- **Python** (Runtime environment for the benchmarking workflow)

## Examples

```
model = scPrint.load_from_checkpoint('model.ckpt', precpt_gene_emb=None); denoised = Denoiser(model)(adata[:, top_5000_hvg], downsample=0.7)
```

## Evaluation signals

- scPRINT's average Spearman correlation increase to true profiles exceeds MAGIC and KNNsmoothing2 on the full datasets.
- The performance gap in favor of scPRINT is larger on rare cell subsets (pericytes, microfold cells, microglial cells) than on the full populations.
- Downsampling consistently removes 70% of transcript counts per cell, split between Poisson read sampling (half) and complete gene dropout (half).
- Spearman correlations are computed only over genes initially expressed in each cell, as specified in the protocol.
- denoising_benchmark_results.csv contains overall and rare-population correlation increases for all three methods.

## Limitations

- Denoising is performed on a representative subset of 10,000 cells and only the 5000 most variable genes per dataset, not the full data.
- MAGIC required K=5 on reduced cell sets (vs default) and KNNsmoothing2 used K=10, so baselines depend on these smoothing parameter choices.
- Neighborhood-aggregation methods like MAGIC and KNNsmoothing2 may perform poorly on rare cell states simply because few similar neighbors exist.
- The article notes that no perfect ground truths exist for benchmarking in related inference tasks, so 'true' profiles are the full (undownsamped) measurements.

## Evidence

- [other] Because scPRINT denoises each cell independently without aggregating profiles over neighboring cells, it outperforms MAGIC and KNNsmoothing2 on rare cell states: [evidence span withheld — anchor status: not_found]
- [methods] zero-inflated Poisson downsampling: "We downsample an expression profile using a zero-inflated Poisson model of the data."
- [abstract] zero-shot denoising ability: "competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction"
- [methods] multi-task pretraining includes denoising: "The three tasks of the multi-task pretraining are the denoising task, the classification task, and the bottleneck learning task."
