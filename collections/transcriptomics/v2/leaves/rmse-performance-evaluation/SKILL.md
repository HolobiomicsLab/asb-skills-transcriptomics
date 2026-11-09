---
name: rmse-performance-evaluation
description: Use when comparing cell-type proportion estimation methods on simulated
  spatial transcriptomics data with known ground-truth pixel-level proportions, and
  per-pixel RMSE plus pairwise statistical accuracy tests are needed.
when_to_use_negative:
- Input is real ST data without simulated ground-truth cell-type proportions — per-pixel
  RMSE against truth cannot be computed
- Comparing single-cell-resolution or sub-cellular ST data, where multi-cellular pixel
  deconvolution benchmarks do not apply
- Evaluating only one method with no competitor — the Diebold-Mariano test is defined
  for pairwise accuracy comparison
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: STdeconvolve
  role: Reference-free LDA-based deconvolution method under evaluation; run on overdispersed-gene-filtered
    counts with topics set to the known K=3 cell types
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Implementation environment for simulation, RMSE computation, Diebold-Mariano
    testing, and figure rendering
- name: BiocManager
  role: Installer used to provision STdeconvolve and dependencies from Bioconductor
- name: remotes
  role: Installer used to install STdeconvolve from GitHub (JEFworks-Lab/STdeconvolve)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:2b4fb5a2a8964a15d41671a96ae5f658b168f8255b97fbb63338de37c33cd1b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - STdeconvolve
  - remotes
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# rmse-performance-evaluation

## Summary

Benchmark deconvolution accuracy by computing per-pixel RMSE between estimated and ground-truth cell-type proportions on simulated multi-cellular pixel-resolution ST data, and compare methods (STdeconvolve vs. transcriptional clustering and BayesSpace resolution-enhanced clustering) with Diebold-Mariano tests. This is the quantitative framework behind Fig. 3h/3l of the STdeconvolve paper.

## When to use

Use when comparing cell-type proportion estimation methods on simulated spatial transcriptomics data with known ground-truth pixel-level proportions, and per-pixel RMSE plus pairwise statistical accuracy tests are needed.

## When NOT to use

- Input is real ST data without simulated ground-truth cell-type proportions — per-pixel RMSE against truth cannot be computed
- Comparing single-cell-resolution or sub-cellular ST data, where multi-cellular pixel deconvolution benchmarks do not apply
- Evaluating only one method with no competitor — the Diebold-Mariano test is defined for pairwise accuracy comparison

## Inputs

- Simulated 3-cell-type multi-cellular pixel-resolution ST dataset with ground-truth per-pixel cell-type proportions
- Simulated 100 um2-pixel MERFISH mouse brain dataset (thalamus region subset, n=716 pixels) with ground-truth proportions
- Estimated per-pixel cell-type proportion matrices from STdeconvolve (theta), transcriptional clustering, and BayesSpace clustering

## Outputs

- Per-pixel RMSE values for each method on each simulated dataset
- Diebold-Mariano test statistics and p-values for pairwise method comparisons
- Comparison figure panels (Fig. 3h ST, Fig. 3l MERFISH equivalents)
- RMSE and DM statistics table (fig3_rmse_comparison.csv)

## How to apply

Simulate multi-cellular pixel-resolution ST datasets with known ground-truth cell-type proportions — a 3-cell-type ST dataset and a 100 um2-pixel MERFISH mouse brain dataset (thalamus region, n=716 pixels). Run each competing method on the same data: STdeconvolve after feature-selecting significantly overdispersed genes with the number of LDA topics set to the known K=3 cell types; transcriptional clustering; and BayesSpace resolution-enhanced clustering, mapping cluster assignments back to cell-type proportions per pixel. Compute per-pixel RMSE between estimated and ground-truth proportions for every method on both datasets, then apply the Diebold-Mariano test to compare per-pixel accuracies between method pairs. Report RMSE and DM statistics (e.g., to fig3_rmse_comparison.csv) and render comparison panels corresponding to Fig. 3h (ST) and Fig. 3l (MERFISH).

## Related tools

- **STdeconvolve** (Reference-free LDA-based deconvolution method under evaluation; run on overdispersed-gene-filtered counts with topics set to the known K=3 cell types) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Implementation environment for simulation, RMSE computation, Diebold-Mariano testing, and figure rendering)
- **BiocManager** (Installer used to provision STdeconvolve and dependencies from Bioconductor)
- **remotes** (Installer used to install STdeconvolve from GitHub (JEFworks-Lab/STdeconvolve))

## Examples

```
ldas <- fitLDA(t(as.matrix(corpus)), Ks = seq(2, 9, by = 1)); results <- getBetaTheta(optimalModel(models = ldas, opt = "min"), perc.filt = 0.05, betaScale = 1000)
```

## Evaluation signals

- STdeconvolve achieves the lowest per-pixel RMSE among the three methods on both simulated datasets
- Pairwise Diebold-Mariano comparisons of clustering (regular and BayesSpace-enhanced) vs. STdeconvolve yield p < 2.2e-16, as in Fig. 3h
- Estimated proportion matrices are per-pixel and sum to 1 (LDA theta), enabling direct RMSE against ground truth
- RMSE and DM statistics are written to fig3_rmse_comparison.csv with values matching the reproduced Fig. 3h/3l panels
- The thalamus-region inset comparison uses exactly n=716 pixels of the simulated MERFISH dataset

## Limitations

- Results depend on simulation design; clustering-based competitors require mapping cluster assignments to proportions, which inflates RMSE for pixels with mixed cell types
- LDA deconvolution performance depends on dataset size in pixels and genes, and can fail when cell-type proportions are homogeneous across pixels so genes lack variability
- Sparse ST data with high stochastic drop-out rates can prevent the LDA model from identifying distinct groups of co-expressed genes, biasing RMSE comparisons

## Evidence

- [results] Both regular transcriptional clustering and BayesSpace resolution-enhanced clustering showed significantly higher RMSE than STdeconvolve on the simulated 3-cell-type ST data (Diebold-Mariano p < 2.2e-16, Fig. 3h).: [evidence span withheld — anchor status: not_found]
- [intro] STdeconvolve applies LDA to infer cell-type proportions per pixel, which are the quantities scored by RMSE.: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [intro] Feature selection of overdispersed genes precedes LDA fitting in the evaluated workflow.: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels14"
- [readme] README example shows the fitLDA/optimalModel/getBetaTheta pipeline whose theta output is scored by RMSE.: "results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000)
deconProp <- results$theta
deconGexp <- results$beta"
- [discussion] Article discussion caveat about LDA dataset-size dependence relevant to benchmark interpretation.: "the performance of LDA in accurately deconvolving cell types depends on the size of the dataset with respect to the number of pixels and the number of genes"
