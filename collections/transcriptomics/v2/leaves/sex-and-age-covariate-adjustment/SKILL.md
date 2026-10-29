---
name: sex-and-age-covariate-adjustment
description: Use when a replicated single-cell dataset is analyzed with Scanpro and
  donor demographics such as sex or age may confound condition-associated cell-type
  proportion changes.
when_to_use_negative:
- The dataset lacks demographic metadata (sex/age) per sample, so no covariate design
  can be specified
- Clusters supported by less than 2% of all cells — benchmarking showed such clusters
  produce unstable results, so covariate effects on them are unreliable
- You need definitive conclusions rather than indications — bootstrapping-based pseudo-replicate
  results should be interpreted with caution and require additional experimental validation
edam_operation: http://edamontology.org/operation_3664
tools:
- name: Scanpro
  role: Performs the linear-model (ANOVA) proportion analysis with added covariates;
    run_stats/anova for statistical testing and sim_scanpro for bootstrapped pseudo-replicates
  repo: https://github.com/loosolab/scanpro
- name: Scanpy
  role: Python framework into which Scanpro integrates for handling the single-cell
    workflow
- name: AnnData
  role: Input format natively handled by Scanpro for the covariate-adjusted runs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:c69f67f808e9c43fd91f3addd936e3eb92da0b348ba0c0fae28b317ad9223bbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/loosolab/scanpro
  tools:
  - AnnData
  - Python
  - Scanpro
  - Scanpy
  edam_operation: http://edamontology.org/operation_3664
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# sex-and-age-covariate-adjustment

## Summary

Extends Scanpro's linear-model-based proportion analysis of single-cell AnnData objects with biological covariates (sex in a human myocardial-infarction dataset, age in a PBMC dataset) to test whether significant cell-type proportion shifts are confounded by donor demographics.

## When to use

Use when a replicated single-cell dataset is analyzed with Scanpro and donor demographics such as sex or age may confound condition-associated cell-type proportion changes.

## When NOT to use

- The dataset lacks demographic metadata (sex/age) per sample, so no covariate design can be specified
- Clusters supported by less than 2% of all cells — benchmarking showed such clusters produce unstable results, so covariate effects on them are unreliable
- You need definitive conclusions rather than indications — bootstrapping-based pseudo-replicate results should be interpreted with caution and require additional experimental validation

## Inputs

- Single-cell dataset stored as an AnnData (.h5ad) object with sample, cluster, and condition columns plus demographic covariate columns (sex, age)
- Pre-computed cell-type annotations from prior clustering (e.g. cardiomyocytes, myeloid cells, cycling cells in MI; monocytes, NK cells in PBMC)
- Scanpro installed via 'pip install scanpro'

## Outputs

- Per-cell-type statistics table from out.results for each covariate-adjusted run
- Assembled covariate-adjusted significance results for both datasets (scanpro_covariate_results.csv)
- Optional result plots via out.plot()

## How to apply

Load the single-cell data (e.g. MI and PBMC datasets) as AnnData objects and run the scanpro() function with samples_col, clusters_col, and conds_col, using ANOVA-based linear-model testing (run_stats/anova) with biological replicates. Re-run the same design with a demographic covariate added (sex for the MI dataset, then age added on top of sex for PBMC) and compare per-cell-type statistics from out.results with and without the covariate. Repeat the covariate-adjusted analysis on bootstrapped pseudo-replicates via sim_scanpro to confirm that attenuated effects are not artifacts of the replication strategy. In the MI dataset, expect cardiomyocyte and myeloid proportion changes to be partially driven by sex while cycling cells are unaffected; in PBMC, monocytes should remain significant for age and NK cells should become significant only when age is included. Write the covariate-adjusted significance results for both datasets to a CSV such as scanpro_covariate_results.csv.

## Related tools

- **Scanpro** (Performs the linear-model (ANOVA) proportion analysis with added covariates; run_stats/anova for statistical testing and sim_scanpro for bootstrapped pseudo-replicates) — https://github.com/loosolab/scanpro
- **Scanpy** (Python framework into which Scanpro integrates for handling the single-cell workflow)
- **AnnData** (Input format natively handled by Scanpro for the covariate-adjusted runs)

## Examples

```
from scanpro import scanpro; out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='condition'); out.results
```

## Evaluation signals

- With sex added to the MI dataset, p-values for cardiomyocyte and myeloid cells attenuate compared with the unadjusted run, while cycling-cell significance is unchanged
- In PBMC, monocytes remain significant after adding age, and NK cells become significant only when age is included in the design
- Covariate-adjusted conclusions agree in direction between the biological-replicate run (scanpro) and the pseudo-replicate run (sim_scanpro)
- Each per-cell-type result row is retrievable from out.results for every tested cluster
- All analyzed clusters contain at least ~2% of all cells, per the benchmarking threshold for stable results

## Limitations

- Bootstrapped pseudo-replicate results are an indication of proportional shifts and should be interpreted with caution and validated experimentally
- Bootstrapping has limited benefits when handling rare cell types
- Clusters supported by less than 2% of all cells produce unstable results
- Lack of replicates limits drawing definitive conclusions about variability; propeller-style approaches require replicates for all conditions investigated

## Evidence

- [readme] Scanpro is a modular tool for proportion analysis, seamlessly integrating into widely accepted frameworks in the python environment.: "Scanpro is a modular tool for proportion analysis, seamlessly integrating into widely accepted frameworks in the python environment."
- [methods] out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'): "out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')"
- [methods] - `run_stats`
- `anova`
- `t_test`
- `sim_scanpro`: "- `run_stats`
- `anova`
- `t_test`
- `sim_scanpro`"
- [intro] Due to technical (e.g. sequencing depth, bias in the cell extracting method and different cell decay rate) and biological (e.g. differences in age) effects, raw cell counts cannot be compared: "Due to technical (e.g. sequencing depth, bias in the cell extracting method and different cell decay rate) and biological (e.g. differences in age) effects, raw cell counts cannot be compared"
- [discussion] the results should be seen as an indication of cell types that have a proportional shift and should be interpreted with caution and additional experimental validation: "the results should be seen as an indication of cell types that have a proportional shift and should be interpreted with caution and additional experimental validation"
- [other] the significant proportion changes in cardiomyocytes and myeloid cells were found to be partially driven by the sex covariate, while cycling cell proportions were not affected by sex: [evidence span withheld — anchor status: not_found]
- [readme] If samples_col is not given or set to None, the dataset is assumed to be not replicated and scanpro will run the bootstrapping method.: [evidence span withheld — anchor status: external_doc]
