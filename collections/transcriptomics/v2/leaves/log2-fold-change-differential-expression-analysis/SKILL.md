---
name: log2-fold-change-differential-expression-analysis
description: Use when two deconvolved or reference-free cell-type transcriptional
  profiles (beta matrices) from different biological conditions (e.g., aged vs young
  pseudo-pixels) must be compared gene-by-gene to validate recovered perturbation
  signatures.
when_to_use_negative:
- Input is raw pixel-level ST counts rather than deconvolved, cell-type-matched transcriptional
  profiles — the comparison requires condition-matched beta profiles, not mixtures
- The ST data are too sparse with high stochastic drop-out rates or cell-type proportions
  are homogeneous across pixels, so STdeconvolve itself fails to recover distinct
  cell-type profiles to compare
- Sub-cellular pixel-resolution ST data, for which the paper states STdeconvolve may
  not be best suited
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: STdeconvolve
  role: Reference-free LDA-based deconvolution producing the cell-type beta transcriptional
    profiles whose aged-vs-young log2 fold-changes are computed
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Implementation environment for the deconvolution and log2 fold-change computation
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Installs STdeconvolve from Bioconductor (release 3.15 requires R >= 4.2)
- name: remotes
  role: Alternative installer for STdeconvolve from GitHub
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:234cc09be1ac705f178e620ed858dfb1637b19aa099f1c60727073b9a90d3e56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - STdeconvolve
  - remotes
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# log2-fold-change-differential-expression-analysis

## Summary

Computes gene-by-gene log2 fold-changes between two condition-matched transcriptional profiles — here, STdeconvolve LDA-derived cell-type beta profiles (e.g., aged vs young macrophages) — to recover perturbation-driven, cell-type-specific expression changes from deconvolved spatial transcriptomics data.

## When to use

Use when two deconvolved or reference-free cell-type transcriptional profiles (beta matrices) from different biological conditions (e.g., aged vs young pseudo-pixels) must be compared gene-by-gene to validate recovered perturbation signatures.

## When NOT to use

- Input is raw pixel-level ST counts rather than deconvolved, cell-type-matched transcriptional profiles — the comparison requires condition-matched beta profiles, not mixtures
- The ST data are too sparse with high stochastic drop-out rates or cell-type proportions are homogeneous across pixels, so STdeconvolve itself fails to recover distinct cell-type profiles to compare
- Sub-cellular pixel-resolution ST data, for which the paper states STdeconvolve may not be best suited

## Inputs

- Deconvolved cell-type transcriptional profiles (beta matrices) from STdeconvolve LDA fit on aged and young simulated ST pixel corpora
- Simulated ST pixel count matrices aggregated from aged and young mouse mammary gland scRNA-seq data
- Gene symbol annotations for aligning profiles gene-by-gene

## Outputs

- aged_vs_young_macrophage_log2fc.csv — gene-by-gene log2 fold-change table (aged vs young macrophage deconvolved profiles)

## How to apply

After fitting STdeconvolve's LDA model with K fixed at the number of simulated cell types (K=2 for the aged/young mammary mixture), extract the deconvolved cell-type transcriptional profiles (beta) for the condition of interest using getBetaTheta and match the deconvolved profiles to the cell-type of interest (macrophages) in each condition. Compute the log2 fold-change of the aged vs young macrophage profiles gene-by-gene and write the result to a CSV table (aged_vs_young_macrophage_log2fc.csv). Validate directionality against the known biology: Cd274 and Clec4d must show positive log2 fold-change and Coro1a negative, reproducing Fig. 3c of the paper. Beta profiles are scaled (betaScale = 1000 in the reference workflow), so apply fold-change consistently on matched, normalized profiles.

## Related tools

- **STdeconvolve** (Reference-free LDA-based deconvolution producing the cell-type beta transcriptional profiles whose aged-vs-young log2 fold-changes are computed) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Implementation environment for the deconvolution and log2 fold-change computation) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Installs STdeconvolve from Bioconductor (release 3.15 requires R >= 4.2))
- **remotes** (Alternative installer for STdeconvolve from GitHub)

## Examples

```
results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000); log2fc <- log2(results$beta$aged / results$beta$young); write.csv(log2fc, 'aged_vs_young_macrophage_log2fc.csv')
```

## Evaluation signals

- aged_vs_young_macrophage_log2fc.csv shows positive log2 fold-change for Cd274
- aged_vs_young_macrophage_log2fc.csv shows positive log2 fold-change for Clec4d
- aged_vs_young_macrophage_log2fc.csv shows negative log2 fold-change for Coro1a
- Directionality matches Fig. 3c of the original scRNA-seq study (Cd274/Clec4d up, Coro1a down in aged macrophages)
- The correct macrophage-matched beta column was extracted from each condition (K=2 profiles correctly assigned to macrophages, not the other cell type)

## Limitations

- LDA performance depends on dataset size in terms of number of pixels and number of genes; deconvolution may be inaccurate when assumptions are not satisfied
- If gene expression is too sparse with high stochastic drop-out rates, the LDA model may struggle to identify distinct groups of co-expressed genes
- If genes lack variability across pixels due to homogeneous cell-type proportions, STdeconvolve may fail to deconvolve, invalidating the downstream fold-change comparison

## Evidence

- [intro] STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel.: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each"
- [readme] results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000); deconGexp <- results$beta: "results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000)
deconProp <- results$theta
deconGexp <- results$beta"
- [discussion] if the gene expression in the ST data is too sparse with high rates of stochastic drop-outs40, then the LDA model may struggle to identify distinct groups of co-expressed genes: "if the gene expression in the ST data is too sparse with high rates of stochastic drop-outs40, then the LDA model may struggle to identify distinct groups of co-expressed genes"
- [discussion] the performance of LDA in accurately deconvolving cell types depends on the size of the dataset with respect to the number of pixels and the number of genes: "the performance of LDA in accurately deconvolving cell types depends on the size of the dataset with respect to the number of pixels and the number of genes"
