---
name: scrnaseq-reference-curation
description: Use when deciding whether an available scRNA-seq reference (e.g., one
  missing key cell types like neurons, or an atlas from a different tissue such as
  mouse brain) is fit for reference-based deconvolution of multi-cellular pixel-resolution
  ST data.
when_to_use_negative:
- A suitable scRNA-seq reference covering all cell types in the ST tissue is available
  — in that case supervised methods perform comparably to STdeconvolve and no curation
  stress test is needed
- The ST data are sub-cellular pixel-resolution, where STdeconvolve and this benchmark
  design are not well suited
- Cell-type proportions are homogeneous across pixels so overdispersed-gene feature
  selection yields no signal, undermining the reference-free baseline the comparison
  relies on
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: STdeconvolve
  role: Reference-free LDA-based deconvolution baseline used to benchmark supervised
    methods against unsuitable references
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: SPOTlight
  role: Supervised NMF-based deconvolution using marker genes from the candidate scRNA-seq
    reference
- name: RCTD
  role: Supervised probabilistic deconvolution using cell-type-specific mean marker
    expression from the candidate reference
- name: spatialDWLS
  role: Supervised deconvolution using reference-derived signature genes to enrich
    likely cell types per pixel, then DWLS
- name: R
  role: Implementation environment for all deconvolution methods and the Diebold-Mariano
    RMSE comparison
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: remotes
  role: Installs the STdeconvolve R package from GitHub
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Alternative installer for STdeconvolve via Bioconductor
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:6269176f49703e6f7ed8422f7ca8662a3d73e02598abce0da011d9896c4eafc4
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# scrnaseq-reference-curation

## Summary

A benchmark-driven approach for judging whether a candidate scRNA-seq reference is suitable for supervised ST deconvolution, by testing supervised methods (SPOTlight, RCTD, spatialDWLS) against the reference-free baseline STdeconvolve on simulated pixels with known ground truth and quantifying RMSE inflation with Diebold-Mariano tests.

## When to use

Use when deciding whether an available scRNA-seq reference (e.g., one missing key cell types like neurons, or an atlas from a different tissue such as mouse brain) is fit for reference-based deconvolution of multi-cellular pixel-resolution ST data.

## When NOT to use

- A suitable scRNA-seq reference covering all cell types in the ST tissue is available — in that case supervised methods perform comparably to STdeconvolve and no curation stress test is needed
- The ST data are sub-cellular pixel-resolution, where STdeconvolve and this benchmark design are not well suited
- Cell-type proportions are homogeneous across pixels so overdispersed-gene feature selection yields no signal, undermining the reference-free baseline the comparison relies on

## Inputs

- Simulated multi-cellular pixel-resolution ST dataset (e.g., simulated MPOA pixels) with known ground-truth cell-type proportions
- Candidate scRNA-seq reference with target cell types removed (e.g., MPOA reference lacking excitatory/inhibitory neurons)
- Mismatched scRNA-seq reference (e.g., mouse brain scRNA-seq atlas)
- R environment with STdeconvolve, SPOTlight, RCTD, and spatialDWLS installed (via remotes/BiocManager)

## Outputs

- Pixel-level estimated cell-type proportions from SPOTlight, RCTD, spatialDWLS, and STdeconvolve
- RMSE values between each method's estimates and the ground-truth simulated proportions
- Diebold-Mariano test p-values comparing supervised methods against STdeconvolve (reported p < 2.2e-16)
- Results table: supervised_deconvolution_rmse_dm.csv

## How to apply

Start from a simulated multi-cellular pixel-resolution ST dataset (e.g., simulated MPOA pixels) whose ground-truth cell-type proportions are known. Run the supervised deconvolution methods SPOTlight, RCTD, and spatialDWLS twice: once with the questionable reference (a scRNA-seq reference with excitatory/inhibitory neurons removed) and once with a clearly mismatched reference (a mouse brain scRNA-seq atlas). Run STdeconvolve — LDA on significantly overdispersed genes after cleanCounts and restrictCorpus feature selection — as the reference-free baseline on the same pixels. Compute per-pixel RMSE between each method's estimated proportions and the simulated ground truth, then apply one-sided Diebold-Mariano tests comparing each supervised method's errors against STdeconvolve's. In this paper, all supervised methods with unsuitable references gave significantly higher RMSE than STdeconvolve (DM p < 2.2 × 10−16; Fig. 2g, h), which is the signal that a reference should be rejected in favor of a reference-free approach.

## Related tools

- **STdeconvolve** (Reference-free LDA-based deconvolution baseline used to benchmark supervised methods against unsuitable references) — https://github.com/JEFworks-Lab/STdeconvolve
- **SPOTlight** (Supervised NMF-based deconvolution using marker genes from the candidate scRNA-seq reference)
- **RCTD** (Supervised probabilistic deconvolution using cell-type-specific mean marker expression from the candidate reference)
- **spatialDWLS** (Supervised deconvolution using reference-derived signature genes to enrich likely cell types per pixel, then DWLS)
- **R** (Implementation environment for all deconvolution methods and the Diebold-Mariano RMSE comparison) — https://github.com/JEFworks-Lab/STdeconvolve
- **remotes** (Installs the STdeconvolve R package from GitHub) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Alternative installer for STdeconvolve via Bioconductor)

## Examples

```
require(remotes); remotes::install_github('JEFworks-Lab/STdeconvolve'); ldas <- fitLDA(t(as.matrix(corpus)), Ks = seq(2, 9, by = 1)); results <- getBetaTheta(optimalModel(models = ldas, opt = "min"), perc.filt = 0.05, betaScale = 1000)
```

## Evaluation signals

- All three supervised methods run with a missing-cell-type or mismatched reference should show RMSE significantly higher than STdeconvolve (one-sided Diebold-Mariano p < 2.2e-16)
- STdeconvolve baseline estimates should be produced without any scRNA-seq reference, confirming the comparison isolates reference suitability
- Per-pixel proportion estimates must lie in [0,1] and be comparable against the simulated ground-truth proportions
- Aggregated RMSE and DM p-values should assemble into supervised_deconvolution_rmse_dm.csv matching Fig. 2g, h patterns

## Limitations

- This curation test requires simulated or ground-truth data with known pixel-level cell-type proportions; real ST data lack such truth
- STdeconvolve as the baseline fails when cell-type proportions are homogeneous across pixels, when data are too sparse with high stochastic drop-out, or when tissue covers only a few pixels
- LDA performance depends on dataset size (number of pixels and genes), so the baseline comparison may be unreliable for small datasets
- Unsuitable for sub-cellular pixel-resolution ST data

## Evidence

- [results] Unsuitable references inflate supervised-method RMSE: [evidence span withheld — anchor status: not_found]
- [intro] Supervised methods require a suitable reference: "As such, these approaches rely on the availability of a suitable single-cell reference, which may present limitations if such a reference does not exist"
- [intro] STdeconvolve feature selection and LDA baseline: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels14"
- [readme] Install STdeconvolve via remotes: "require(remotes)
remotes::install_github('JEFworks-Lab/STdeconvolve')"
- [readme] STdeconvolve preprocessing workflow: "counts <- cleanCounts(cd, min.lib.size = 100)
## feature select for genes
corpus <- restrictCorpus(counts, removeAbove=1.0, removeBelow = 0.05)"
