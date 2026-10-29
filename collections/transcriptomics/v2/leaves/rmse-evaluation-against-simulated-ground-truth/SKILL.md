---
name: rmse-evaluation-against-simulated-ground-truth
description: Use when benchmarking deconvolution methods (STdeconvolve, SPOTlight,
  RCTD, spatialDWLS) on simulated multi-cellular pixel-resolution ST data with known
  ground-truth cell-type proportions and a significance test for RMSE differences
  is required.
when_to_use_negative:
- Real ST datasets with no simulated ground-truth proportions — RMSE against truth
  cannot be computed; use qualitative agreement or annotation-based validation instead.
- Sub-cellular pixel-resolution ST data — the article states STdeconvolve 'may not
  be best suited for analysis of such sub-cellular resolution ST data', so this simulated-pixel
  benchmark does not transfer.
- Datasets where cell-type proportions are homogeneous across pixels or extremely
  sparse with high stochastic drop-outs — LDA-based STdeconvolve may fail, invalidating
  the RMSE comparison baseline.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: STdeconvolve
  role: Reference-free (LDA-based) deconvolution baseline whose estimated proportions
    (theta) are compared to simulated ground truth via RMSE
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: SPOTlight
  role: 'Supervised reference-based deconvolution method evaluated: marker genes from
    an scRNA-seq reference seed non-negative matrix factorization'
- name: RCTD
  role: 'Supervised reference-based deconvolution method evaluated: builds a probabilistic
    model from cell-type-specific mean marker expression of the scRNA-seq reference'
- name: spatialDWLS
  role: 'Supervised reference-based deconvolution method evaluated: uses signature
    genes from the scRNA-seq reference to enrich likely cell types per pixel then
    fits DWLS'
- name: BiocManager
  role: Used to install the STdeconvolve R package from Bioconductor for running the
    reference-free baseline
- name: remotes
  role: Used to install STdeconvolve from GitHub (remotes::install_github('JEFworks-Lab/STdeconvolve'))
  repo: https://github.com/JEFworks-Lab/STdeconvolve
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:b9d6c8ea9a1aa5044c553f2c2c0f6b83f4bb017c3bec12eb380956838e371121
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - RCTD
  - SPOTlight
  - STdeconvolve
  - remotes
  - spatialDWLS
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# rmse-evaluation-against-simulated-ground-truth

## Summary

Benchmark cell-type deconvolution of multi-cellular pixel-resolution spatial transcriptomics (ST) data by comparing estimated pixel-level cell-type proportions against simulated ground-truth proportions using RMSE, supplemented with one-sided Diebold-Mariano tests to establish significance (as in the STdeconvolve paper, DOI 10.1038/s41467-022-30033-z).

## When to use

Use when benchmarking deconvolution methods (STdeconvolve, SPOTlight, RCTD, spatialDWLS) on simulated multi-cellular pixel-resolution ST data with known ground-truth cell-type proportions and a significance test for RMSE differences is required.

## When NOT to use

- Real ST datasets with no simulated ground-truth proportions — RMSE against truth cannot be computed; use qualitative agreement or annotation-based validation instead.
- Sub-cellular pixel-resolution ST data — the article states STdeconvolve 'may not be best suited for analysis of such sub-cellular resolution ST data', so this simulated-pixel benchmark does not transfer.
- Datasets where cell-type proportions are homogeneous across pixels or extremely sparse with high stochastic drop-outs — LDA-based STdeconvolve may fail, invalidating the RMSE comparison baseline.

## Inputs

- Simulated multi-cellular pixel-resolution ST count matrix (e.g., simulated MPOA ST pixels)
- Ground-truth simulated per-pixel cell-type proportion matrix
- scRNA-seq reference(s) for supervised methods (e.g., MPOA scRNA-seq reference with neurons removed; mouse brain scRNA-seq atlas)
- Pixel spatial coordinates for STdeconvolve preprocessing (cleanCounts/restrictCorpus)

## Outputs

- Per-pixel estimated cell-type proportion matrices (theta) for each method
- RMSE values per method versus simulated ground truth
- Diebold-Mariano test p-values (expected p < 2.2e-16 for unsuitable references)
- Comparison table such as supervised_deconvolution_rmse_dm.csv

## How to apply

Load the simulated multi-cellular pixel-resolution ST dataset (e.g., simulated MPOA data) together with its known ground-truth per-pixel cell-type proportions. Run the supervised reference-based deconvolution methods — SPOTlight (marker-gene-seeded NMF), RCTD (probabilistic model on cell-type mean marker expression), and spatialDWLS (signature-gene enrichment followed by DWLS) — using the specified scRNA-seq reference (e.g., MPOA reference missing excitatory/inhibitory neurons, or a mouse brain scRNA-seq atlas), and run the reference-free STdeconvolve baseline (LDA on significantly overdispersed genes) on the same simulated pixels. For each method, compute RMSE between its estimated pixel-level proportions (theta) and the ground-truth simulated proportions. Perform one-sided Diebold-Mariano tests comparing each supervised method's per-pixel errors against STdeconvolve's errors; in the paper this yields p < 2.2 × 10−16 when unsuitable references (neurons removed or mouse brain atlas) are used. Assemble RMSE values and p-values into a results table (e.g., supervised_deconvolution_rmse_dm.csv) and visualize (Fig. 2g, h style).

## Related tools

- **STdeconvolve** (Reference-free (LDA-based) deconvolution baseline whose estimated proportions (theta) are compared to simulated ground truth via RMSE) — https://github.com/JEFworks-Lab/STdeconvolve
- **SPOTlight** (Supervised reference-based deconvolution method evaluated: marker genes from an scRNA-seq reference seed non-negative matrix factorization)
- **RCTD** (Supervised reference-based deconvolution method evaluated: builds a probabilistic model from cell-type-specific mean marker expression of the scRNA-seq reference)
- **spatialDWLS** (Supervised reference-based deconvolution method evaluated: uses signature genes from the scRNA-seq reference to enrich likely cell types per pixel then fits DWLS)
- **BiocManager** (Used to install the STdeconvolve R package from Bioconductor for running the reference-free baseline)
- **remotes** (Used to install STdeconvolve from GitHub (remotes::install_github('JEFworks-Lab/STdeconvolve'))) — https://github.com/JEFworks-Lab/STdeconvolve

## Examples

```
results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000); deconProp <- results$theta; rmse <- sqrt(mean((deconProp - ground_truth_proportions)^2))
```

## Evaluation signals

- RMSE values are non-negative and computed strictly between per-pixel estimated proportions (rows summing to 1) and ground-truth simulated proportions on the same set of pixels.
- Under unsuitable references (neuron-removed MPOA reference or mouse brain atlas), every supervised method (SPOTlight, RCTD, spatialDWLS) should show higher RMSE than STdeconvolve, with one-sided Diebold-Mariano p < 2.2 × 10−16.
- Under a suitable reference, supervised method RMSE should be comparable to STdeconvolve (no significant DM advantage), matching the paper's Fig. 2g, h pattern.
- STdeconvolve pipeline outputs valid theta/beta objects (getBetaTheta with perc.filt = 0.05, betaScale = 1000) whose theta dimensions match the simulated pixel x cell-type ground-truth matrix after aligning/deconvolved labels.
- Results are assembled into a structured CSV (e.g., supervised_deconvolution_rmse_dm.csv) containing method name, reference condition, RMSE, and DM p-value per comparison.

## Limitations

- LDA-based baseline performance depends on dataset size in pixels and genes; 'the performance of LDA in accurately deconvolving cell types depends on the size of the dataset with respect to the number of pixels and the number of genes'.
- STdeconvolve may fail when genes lack variability across pixels due to homogeneous cell-type proportions, making the reference-free baseline weak and RMSE comparisons uninformative.
- Sparse ST data with high stochastic drop-out rates can hinder LDA's ability to identify co-expressed gene groups, degrading the baseline.
- Ground truth exists only in simulation; on real ST data the RMSE-vs-truth evaluation is not applicable, so conclusions are conditional on the simulation design.

## Evidence

- [other] DM p-value expectation: [evidence span withheld — anchor status: not_found]
- [intro] Supervised methods rely on suitable reference: "As such, these approaches rely on the availability of a suitable single-cell reference, which may present limitations if such a reference does not exist"
- [intro] LDA recovers pixel-level proportions: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [abstract] Comparability claim motivating RMSE benchmark: "STdeconvolve provides comparable performance to existing reference-based methods when suitable single-cell references are available, as well as potentially superior performance when suitable"
- [readme] Concrete STdeconvolve invocation yielding theta: "results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000)
deconProp <- results$theta"
