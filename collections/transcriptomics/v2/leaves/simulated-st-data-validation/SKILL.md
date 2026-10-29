---
name: simulated-st-data-validation
description: Use when a deconvolution method for multi-cellular pixel-resolution ST
  data must be quantitatively benchmarked against ground-truth cell-type proportions,
  especially when a matched scRNA-seq/MERFISH reference is or is not available.
when_to_use_negative:
- Data is real ST data without known ground-truth proportions — RMSE cannot be computed
  and simulation-based validation is inappropriate
- ST data are at single-cell or sub-cellular resolution — STdeconvolve and this multi-cellular-pixel
  benchmarking design do not apply
- The single-cell reference is mismatched (batch effects, missing cell types, different
  perturbations) — supervised comparators will be unfairly degraded and the benchmark
  will not reflect their suitable-reference performance
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STdeconvolve
  role: Reference-free LDA-based deconvolution applied to the simulated ST pixels
    to produce proportions theta and transcriptional profiles beta for benchmarking
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: SPOTlight
  role: Supervised comparator that uses reference-derived cell-type marker genes to
    seed non-negative matrix factorization for per-pixel proportion estimation
- name: RCTD
  role: Supervised comparator that builds a probabilistic model from cell-type-specific
    mean expression of reference marker genes
- name: spatialDWLS
  role: Supervised comparator that enriches likely cell types per pixel via reference
    signature genes, then applies dampened weighted least squares
- name: R
  role: Language in which STdeconvolve and the benchmarking pipeline are implemented
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: remotes
  role: Used to install STdeconvolve from GitHub (remotes::install_github('JEFworks-Lab/STdeconvolve'))
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Alternative installer for the STdeconvolve Bioconductor package (R >= 4.2
    required for release 3.15)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:318fbb9b32439173d1782f18a51f9769897676d31859faa6e4ba7d168e708869
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# simulated-st-data-validation

## Summary

Validate spatial transcriptomics deconvolution methods on simulated multi-cellular pixel-resolution ST data with known ground-truth cell-type proportions, benchmarking reference-free STdeconvolve against supervised methods (SPOTlight, RCTD, spatialDWLS) via RMSE of deconvolved proportions.

## When to use

Use when a deconvolution method for multi-cellular pixel-resolution ST data must be quantitatively benchmarked against ground-truth cell-type proportions, especially when a matched scRNA-seq/MERFISH reference is or is not available.

## When NOT to use

- Data is real ST data without known ground-truth proportions — RMSE cannot be computed and simulation-based validation is inappropriate
- ST data are at single-cell or sub-cellular resolution — STdeconvolve and this multi-cellular-pixel benchmarking design do not apply
- The single-cell reference is mismatched (batch effects, missing cell types, different perturbations) — supervised comparators will be unfairly degraded and the benchmark will not reflect their suitable-reference performance

## Inputs

- Simulated multi-cellular pixel-resolution ST count matrix with known ground-truth per-pixel cell-type proportions (e.g. 100 µm² MPOA simulated ST pixels)
- Ground-truth cell-type proportion matrix per pixel
- Matched single-cell reference (e.g. MERFISH scRNA-seq reference from mouse preoptic area) with cell-type labels for supervised methods
- Pixel spatial coordinates for visualization

## Outputs

- Per-pixel estimated cell-type proportion matrices from each method (STdeconvolve theta, SPOTlight, RCTD, spatialDWLS outputs)
- RMSE comparison table of deconvolution accuracy vs ground truth (e.g. rmse_comparison_table.csv) reproducing Fig. 2e, f

## How to apply

Simulate or load a multi-cellular pixel-resolution ST dataset (e.g. the paper's simulated 100 µm² MPOA ST pixel count matrix) for which the per-pixel ground-truth cell-type proportions are known by construction. Run the unsupervised method (STdeconvolve: cleanCounts to remove low-count pixels, restrictCorpus to feature-select overdispersed genes, fitLDA over a K range with data-driven metrics to choose K, then getBetaTheta to extract proportions theta and profiles beta). Run the supervised comparators (SPOTlight, RCTD, spatialDWLS) using the matched MERFISH single-cell reference from the mouse preoptic area, so each exploits cell-type marker genes or mean expression profiles. Compute root-mean-square error (RMSE) between each method's estimated per-pixel proportions and the known simulated ground truth, tabulate for all methods (e.g. to rmse_comparison_table.csv reproducing Fig. 2e, f), and judge success when the reference-free method's RMSE is comparable to reference-based methods when a suitable reference exists.

## Related tools

- **STdeconvolve** (Reference-free LDA-based deconvolution applied to the simulated ST pixels to produce proportions theta and transcriptional profiles beta for benchmarking) — https://github.com/JEFworks-Lab/STdeconvolve
- **SPOTlight** (Supervised comparator that uses reference-derived cell-type marker genes to seed non-negative matrix factorization for per-pixel proportion estimation)
- **RCTD** (Supervised comparator that builds a probabilistic model from cell-type-specific mean expression of reference marker genes)
- **spatialDWLS** (Supervised comparator that enriches likely cell types per pixel via reference signature genes, then applies dampened weighted least squares)
- **R** (Language in which STdeconvolve and the benchmarking pipeline are implemented) — https://github.com/JEFworks-Lab/STdeconvolve
- **remotes** (Used to install STdeconvolve from GitHub (remotes::install_github('JEFworks-Lab/STdeconvolve'))) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Alternative installer for the STdeconvolve Bioconductor package (R >= 4.2 required for release 3.15))

## Examples

```
require(remotes); remotes::install_github('JEFworks-Lab/STdeconvolve'); ldas <- fitLDA(t(as.matrix(corpus)), Ks = seq(2, 9, by = 1)); results <- getBetaTheta(optimalModel(models = ldas, opt = "min"), perc.filt = 0.05, betaScale = 1000)
```

## Evaluation signals

- The RMSE table contains rows for all four methods: SPOTlight, RCTD, SpatialDWLS, and STdeconvolve
- STdeconvolve's RMSE is comparable to SPOTlight, RCTD, and spatialDWLS when the ideal matched MERFISH MPOA reference is used, consistent with Fig. 2e, f
- All estimated per-pixel proportion matrices are non-negative and rows (pixels) sum to 1, matching ground-truth proportion format
- STdeconvolve chose a number of cell types K using the paper's data-driven metrics (e.g. optimalModel with opt='min') rather than an arbitrary value
- Deconvolved cell-type proportions can be spatially visualized with vizAllTopics against pixel coordinates and annotations without formatting errors

## Limitations

- LDA performance depends on dataset size in terms of number of pixels and number of genes; performance may degrade when assumptions are unsatisfied
- If genes lack variability across pixels (homogeneous/uniform cell-type proportions), STdeconvolve may fail to deconvolve distinct cell types
- Sparse ST data with high stochastic drop-out rates can hinder the LDA model from identifying distinct groups of co-expressed genes
- ST profiling of thin structures covering only a few pixels presents challenges for deconvolution
- STdeconvolve is not best suited for sub-cellular pixel-resolution ST data

## Evidence

- [abstract] STdeconvolve provides comparable performance to existing reference-based methods when suitable single-cell references are available: "STdeconvolve provides comparable performance to existing reference-based methods when suitable single-cell references are available"
- [intro] SPOTlight mechanism: "SPOTlight3 uses cell-type marker genes derived from a single-cell RNA-sequencing (scRNA-seq) reference to seed a non-negative matrix factorization."
- [intro] RCTD and spatialDWLS mechanisms: "RCTD4 uses the cell-type-specific mean expression of marker genes derived from a scRNA-seq reference to build a probabilistic model"
- [intro] STdeconvolve LDA outputs: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [readme] Installation command: "remotes::install_github('JEFworks-Lab/STdeconvolve')"
- [intro] Overdispersed gene feature selection: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels"
