---
name: spliced-unspliced-phase-portrait-analysis
description: Use when you have a UniTVelo-fitted AnnData with latent_time and velocity
  and need per-gene diagnostic phase portraits to check model fit or explain MURK/kinetic
  genes.
when_to_use_negative:
- When no UniTVelo model has been fitted yet and the AnnData lacks latent_time and
  velocity fields
- When you only need dataset-level trajectory summaries (velocity embedding streams
  or utv.evaluate metrics) rather than per-gene diagnostics
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: UniTVelo
  role: Provides utv.pl.plot_range to render gene-level phase portraits against latent
    time and the RBF-based fitted dynamics
  repo: https://github.com/StatBiomed/UniTVelo
- name: scVelo
  role: Preprocessing (spliced/unspliced matrices, moments) and AnnData container
    used by UniTVelo for plotting
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:9f3d7072089cb2c39645a36eb77f384d00ea50473c161b020c16a8e6ff9af749
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/StatBiomed/UniTVelo
  tools:
  - CellRank
  - Jupyter
  - Python 3
  - UniTVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spliced-unspliced-phase-portrait-analysis

## Summary

Generates and inspects gene-level spliced–unspliced phase portraits from a fitted UniTVelo model (e.g. MURK genes Abcg2, Smim1, Cnn3, Cyr61 on erythroid data) to verify how the RBF-based mean function and latent time explain each gene's spliced/unspliced dynamics.

## When to use

Use when you have a UniTVelo-fitted AnnData with latent_time and velocity and need per-gene diagnostic phase portraits to check model fit or explain MURK/kinetic genes.

## When NOT to use

- When no UniTVelo model has been fitted yet and the AnnData lacks latent_time and velocity fields
- When you only need dataset-level trajectory summaries (velocity embedding streams or utv.evaluate metrics) rather than per-gene diagnostics

## Inputs

- UniTVelo-annotated AnnData (.h5ad) with spliced/unspliced layers, velocity, and latent_time
- The utv.config.Configuration object used for the model fit
- Gene name(s) of interest (e.g. MURK genes)

## Outputs

- Per-gene spliced–unspliced phase portrait plots (utv.pl.plot_range figures)
- Gene-level goodness-of-fit assessment (R2) supporting dynamically informed gene selection

## How to apply

After running utv.run_model (with a utv.config.Configuration setting e.g. R2_ADJUST=True, FIT_OPTION='1' or '2'), call utv.pl.plot_range(gene_name, adata, velo, show_ax=True, time_metric='latent_time') for the genes of interest; this plots the fitted spliced-RNA trajectory (the RBF mean function, whose time derivative is the velocity) against observed spliced/unspliced counts along latent time. The rationale is that UniTVelo derives velocity as ds/dt of the designed RBF spliced profile rather than from deviation to a steady-state equilibrium, so the phase portrait reveals whether the concurrently updated latent time and kinetic parameters track the gene's induction/degradation behavior. Use the portraits together with the coefficient of determination R2 (with R2_ADJUST toggling extreme-quantile vs full-data regression) to decide whether a gene is dynamically informed. Portraits are especially informative for MURK genes (e.g. Abcg2, Smim1, Cnn3, Cyr61) whose dynamics violate steady-state assumptions and for verifying direction corrections in unified-time mode.

## Related tools

- **UniTVelo** (Provides utv.pl.plot_range to render gene-level phase portraits against latent time and the RBF-based fitted dynamics) — https://github.com/StatBiomed/UniTVelo
- **scVelo** (Preprocessing (spliced/unspliced matrices, moments) and AnnData container used by UniTVelo for plotting)

## Examples

```
import unitvelo as utv; utv.pl.plot_range('Abcg2', adata, velo, show_ax=True, time_metric='latent_time')
```

## Evaluation signals

- The phase portrait shows the fitted RBF spliced trajectory closely following observed spliced counts across latent time
- Gene R2 (goodness-of-fit indicator) supports the gene being dynamically informed, consistent with the chosen R2_ADJUST setting
- Portrait ordering follows the inferred latent time (color/scale from 'latent_time' metric), so direction matches the trajectory inferred in unified-time mode
- MURK genes (e.g. Abcg2, Smim1, Cnn3, Cyr61) show behavior explainable by the fitted profile despite violating steady-state assumptions

## Limitations

- The README labels phase portraits for individual genes as 'experimental', so plotting APIs may change
- No unsupervised method exists for selecting dynamically informed genes, so gene choice for portraits requires prior knowledge or R2 screening
- Possible over-counting in unspliced reads can distort portrait interpretation and cell transition metrics

## Evidence

- [readme] Phase portraits for individual genes (experimental): "# Phase portraits for individual genes (experimental)
    utv.pl.plot_range(gene_name, adata, velo, show_ax=True, time_metric='latent_time')"
- [intro] Velocity from derivative of RBF spliced profile: "developed UniTVelo, a statistical method that models the full dynamics of gene expression with a radial basis function (RBF) and quantifies RNA velocity in a top-down manner"
- [discussion] R2 as goodness-of-fit indicator: "we illustrated that the coefficient of determination R2 could be an informative indicator of goodness-of-fit"
- [methods] MURK genes plotted via plot_range: [evidence span withheld — anchor status: not_found]
