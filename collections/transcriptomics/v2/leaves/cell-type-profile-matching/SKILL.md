---
name: cell-type-profile-matching
description: Use when ST pixels are cell mixtures and you need to assign reference-free
  deconvolved cell-type transcriptional profiles to known cell populations and compare
  them across conditions (e.g., aged vs young) for perturbation-driven gene changes.
when_to_use_negative:
- Cell-type proportions are homogeneous/uniform across pixels so genes lack variability
  across pixels — STdeconvolve may fail to deconvolve distinct cell types
- ST data are too sparse with high stochastic drop-out rates, which the LDA model
  may struggle to separate into co-expressed gene groups
- The tissue covers only a few pixels (e.g., tissue slivers or thin structures), which
  challenges deconvolution
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STdeconvolve
  role: 'Reference-free LDA deconvolution: feature-selects overdispersed genes, fits
    LDA with fixed K, and extracts beta (cell-type transcriptional profiles) and theta
    (proportions) used for profile matching and log2FC computation'
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Analysis environment in which STdeconvolve is run and the log2 fold-change
    table is computed and exported
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: remotes
  role: Used to install STdeconvolve from GitHub (remotes::install_github('JEFworks-Lab/STdeconvolve'))
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Alternative installer for the Bioconductor distribution of STdeconvolve (requires
    R >=4.2 for Bioconductor release 3.15)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:26323c5c3d91f4b194c36f13d67f8bdb2b88b18d131c5ca7a56de6fef9fb8ac2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - STdeconvolve
  - remotes
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cell-type-profile-matching

## Summary

Matches reference-free deconvolved cell-type transcriptional profiles (beta matrices from STdeconvolve's LDA) to known cell populations, then computes gene-wise log2 fold-changes between deconvolved profiles across conditions (e.g., aged vs young macrophages) to recover perturbation-driven expression changes without a single-cell reference.

## When to use

Use when ST pixels are cell mixtures and you need to assign reference-free deconvolved cell-type transcriptional profiles to known cell populations and compare them across conditions (e.g., aged vs young) for perturbation-driven gene changes.

## When NOT to use

- Cell-type proportions are homogeneous/uniform across pixels so genes lack variability across pixels — STdeconvolve may fail to deconvolve distinct cell types
- ST data are too sparse with high stochastic drop-out rates, which the LDA model may struggle to separate into co-expressed gene groups
- The tissue covers only a few pixels (e.g., tissue slivers or thin structures), which challenges deconvolution

## Inputs

- Condition-specific scRNA-seq single-cell matrices (e.g., aged and young mouse mammary gland) aggregated into pseudo-pixel count mixtures
- Multi-cellular pixel-resolution ST count matrix (pixels x genes)
- Chosen number of cell types K for LDA (e.g., K=2 for aged/young mixtures)

## Outputs

- Deconvolved cell-type transcriptional profiles (beta) and pixel proportions (theta) from STdeconvolve
- Gene-by-gene log2 fold-change table between matched cell-type profiles across conditions (e.g., aged_vs_young_macrophage_log2fc.csv)

## How to apply

Aggregate single cells from each condition (e.g., aged and young mouse mammary gland scRNA-seq) into pseudo-pixel mixtures simulating ST data, then run STdeconvolve's LDA with K fixed at the expected number of cell types (K=2 in the aged/young mammary case), after overdispersed-gene feature selection via restrictCorpus. Extract the beta transcriptional profiles and theta proportions with getBetaTheta, then match each deconvolved cell-type topic to a known population (e.g., the macrophage topic) using profile correspondence rather than external references, since STdeconvolve is reference-free. Compute gene-by-gene log2 fold-change between the matched cell-type profiles from the two conditions and export as a table (e.g., aged_vs_young_macrophage_log2fc.csv). Rationale: STdeconvolve's LDA infers putative cell-type transcriptional profiles and pixel-level proportions without scRNA-seq references, avoiding batch effects and reference-perturbation mismatches that confound reference-based methods.

## Related tools

- **STdeconvolve** (Reference-free LDA deconvolution: feature-selects overdispersed genes, fits LDA with fixed K, and extracts beta (cell-type transcriptional profiles) and theta (proportions) used for profile matching and log2FC computation) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Analysis environment in which STdeconvolve is run and the log2 fold-change table is computed and exported) — https://github.com/JEFworks-Lab/STdeconvolve
- **remotes** (Used to install STdeconvolve from GitHub (remotes::install_github('JEFworks-Lab/STdeconvolve'))) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Alternative installer for the Bioconductor distribution of STdeconvolve (requires R >=4.2 for Bioconductor release 3.15))

## Examples

```
results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000); deconGexp <- results$beta
```

## Evaluation signals

- Log2 fold-change table shows positive values for Cd274 and Clec4d and negative values for Coro1a in aged vs young macrophage profiles, reproducing the directionality of Fig. 3c of the paper
- The recovered aged/young macrophage deconvolved profiles are directionally consistent with the original scRNA-seq study's age-associated macrophage changes
- The optimal LDA model is selected at the a priori K (K=2) using STdeconvolve's data-driven model-selection metrics
- Overdispersed-gene feature selection (restrictCorpus) yields a restricted corpus before LDA, and beta rows map one-to-one onto the expected number of matched cell types

## Limitations

- LDA performance depends on dataset size in terms of number of pixels and number of genes, and may struggle when its assumptions are not satisfied
- Sparse ST data with high stochastic drop-out rates can prevent the LDA model from identifying distinct groups of co-expressed genes
- STdeconvolve may not be suited for sub-cellular pixel-resolution ST data; methods specific to that resolution are anticipated to be needed

## Evidence

- [intro] STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [intro] overdispersed gene feature selection: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels14"
- [abstract] reference-free comparability: "STdeconvolve provides comparable performance to existing reference-based methods when suitable single-cell references are available"
- [readme] extraction of beta and theta from the fitted LDA model: [evidence span withheld — anchor status: external_doc]
