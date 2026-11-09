---
name: goodness-of-fit-r2-evaluation-per-gene
description: Use when validating RNA velocity model fits (UniTVelo with spliced/unspliced
  AnnData) by scoring per-gene R2 goodness-of-fit, e.g. to check that velocity fields
  and latent time are supported by dynamically informed genes.
when_to_use_negative:
- No spliced/unspliced counts are available (e.g. metabolic-labeling information is
  the required input modality or only a feature table exists) — UniTVelo's phase-portrait-based
  R2 is undefined
- The goal is comparing velocity methods against ground-truth cluster edges with directional
  metrics (e.g. Cross Boundary Direction Correctness via utv.evaluate) rather than
  per-gene fit quality
- Genes were not dynamically informed by the fitted model — R2 goodness-of-fit is
  meaningful for dynamically informed genes, not all genes
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: UniTVelo
  role: Fits the RNA velocity model and supplies per-gene dynamical fits whose phase
    portraits are scored with R2; provides R2_ADJUST configuration and plotting utilities
    (utv.pl.plot_range)
  repo: https://github.com/StatBiomed/UniTVelo
- name: scVelo
  role: Used for downstream embedding and visualization of fitted results (velocity_embedding_stream,
    latent_time scatter) while UniTVelo supplies the per-gene R2 fit metric
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:a146a3655ab27860f2aecd8222945459f8b96ef2908cbedc7f3e10d284a1b5a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/StatBiomed/UniTVelo
  tools:
  - Jupyter
  - Python 3
  - UniTVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Goodness-of-fit R2 evaluation per gene

## Summary

Computes the coefficient of determination R2 per gene as a goodness-of-fit indicator for UniTVelo RNA velocity fits, reproducing the Fig. 4c histogram on the scEU-seq intestinal organoid dataset to validate dynamical model quality.

## When to use

Use when validating RNA velocity model fits (UniTVelo with spliced/unspliced AnnData) by scoring per-gene R2 goodness-of-fit, e.g. to check that velocity fields and latent time are supported by dynamically informed genes.

## When NOT to use

- No spliced/unspliced counts are available (e.g. metabolic-labeling information is the required input modality or only a feature table exists) — UniTVelo's phase-portrait-based R2 is undefined
- The goal is comparing velocity methods against ground-truth cluster edges with directional metrics (e.g. Cross Boundary Direction Correctness via utv.evaluate) rather than per-gene fit quality
- Genes were not dynamically informed by the fitted model — R2 goodness-of-fit is meaningful for dynamically informed genes, not all genes

## Inputs

- UniTVelo-fitted AnnData object (spliced/unspliced mRNA counts, e.g. scEU-seq intestinal organoid) with velocity and latent_time in .obs
- Trained UniTVelo model object (velo) returned/configured via utv.config.Configuration
- Cell-type label column name in adata.obs

## Outputs

- Per-gene coefficient of determination R2 values (goodness-of-fit indicator)
- Histogram of per-gene R2 distribution (reproducing Fig. 4c)
- Optionally, example gene phase portraits via utv.pl.plot_range

## How to apply

After running utv.run_model on the scEU-seq intestinal organoid AnnData with velo.R2_ADJUST=True, velo.IROOT=None, velo.FIT_OPTION='1' (Unified-time mode) and velo.GPU=0 (or -1 for CPU), compute the coefficient of determination R2 for each dynamically informed gene to quantify how well the fitted phase portrait explains its spliced/unspliced kinetics. Set R2_ADJUST to True so the regression R-squared is computed on the full data rather than only the extreme quantile, as recommended in the UniTVelo README. Aggregate per-gene R2 values into a histogram corresponding to Fig. 4c and inspect example phase portraits via utv.pl.plot_range(gene_name, adata, velo, show_ax=True, time_metric='latent_time'). Genes with high R2 are well explained by the unified latent time; the article notes that stably and monotonically changed genes show strong dynamical patterns on spliced RNAs with limited deviation from un/spliced balance.

## Related tools

- **UniTVelo** (Fits the RNA velocity model and supplies per-gene dynamical fits whose phase portraits are scored with R2; provides R2_ADJUST configuration and plotting utilities (utv.pl.plot_range)) — https://github.com/StatBiomed/UniTVelo
- **scVelo** (Used for downstream embedding and visualization of fitted results (velocity_embedding_stream, latent_time scatter) while UniTVelo supplies the per-gene R2 fit metric)

## Examples

```
velo = utv.config.Configuration(); velo.R2_ADJUST = True; velo.FIT_OPTION = '1'; velo.GPU = 0; adata = utv.run_model(path_to_adata, label, config_file=velo); utv.pl.plot_range(gene_name, adata, velo, show_ax=True, time_metric='latent_time')
```

## Evaluation signals

- Per-gene R2 histogram matches the reported Fig. 4c goodness-of-fit distribution on the intestinal organoid dataset
- Velocity embedding stream shows the expected two-branch organoid structure (secretory, enterocyte) and local paneth-to-goblet direction before trusting R2 values
- R2_ADJUST=True yields R-squared computed on full data rather than only the extreme quantile — confirm config was applied
- Example phase portraits (utv.pl.plot_range) visually align with fitted kinetics along latent_time for high-R2 genes

## Limitations

- R2 goodness-of-fit is informative only for dynamically informed genes; there is no unsupervised method to select them, stated as an open challenge in the article
- Possible over-counting in unspliced reads can distort per-gene kinetics and thus cell transition metrics
- UniTVelo needs a GPU with ~12GB memory for datasets larger than 50,000 cells (e.g. NVIDIA GeForce RTX 2080 Ti); otherwise fitting is slow (around 8x more time on the human erythroid dataset)

## Evidence

- [discussion] The coefficient of determination R2 can serve as an informative indicator of goodness-of-fit for dynamically informed genes: "we illustrated that the coefficient of determination R2 could be an informative indicator of goodness-of-fit"
- [readme] R2_ADJUST controls full-data vs extreme-quantile regression: "velo.R2_ADJUST` (bool), linear regression R-squared on extreme quantile (default) or full data (adjusted)"
- [discussion] Stably and monotonically changed genes show strong spliced dynamics near un/spliced balance: "we observed a set of genes that have strong dynamical patterns on spliced RNAs but limited deviation from the balance of unspliced and spliced RNAs"
- [methods] Phase portraits per gene via plot_range: "utv.pl.plot_range(gene_name, adata, velo, show_ax=True, time_metric='latent_time')"
- [discussion] No unsupervised selection of dynamically informed genes: "it remains an open challenge to select dynamically informed genes for projecting cell transitions from the velocity vector, particularly in an unsupervised manner"
