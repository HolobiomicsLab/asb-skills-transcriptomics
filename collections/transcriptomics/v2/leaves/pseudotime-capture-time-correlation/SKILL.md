---
name: pseudotime-capture-time-correlation
description: Use when a pseudotime method has been fit on scRNA-seq data whose cells
  have known physical capture/sampling times, and you need an external check that
  the latent ordering recapitulates real time progression.
when_to_use_negative:
- The dataset has no external per-cell time labels (e.g. purely cross-sectional single-sample
  data), so no physical capture-time validation is possible
- Capture times were included as model covariates during fitting, making the correlation
  a circular, non-independent validation
- The biological process is expected to be uncoupled from sampling time (e.g. in vitro
  activation dynamics not aligned with capture order), making capture-time correlation
  a misleading quality metric
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3524
tools:
- name: PhenoPath
  role: Fits the pseudotimes under validation via phenopath() with CAVI variational
    inference; pseudotimes extracted with trajectory(fit)
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: Monocle 2
  role: Baseline pseudotime method against which capture-time correlation of PhenoPath
    is compared
  repo: https://bioconductor.org/packages/release/bioc/html/monocle.html
- name: R
  role: Language in which the correlation analysis, CSV export and baseline methods
    are run
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:38feb8e0a9d2bc6673d277f7248cb0727812f591a100ac57a202492812736ada
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle 2
  - PhenoPath
  - R
  - SummarizedExperiment
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# pseudotime-capture-time-correlation

## Summary

Validates inferred single-cell pseudotimes by computing the Pearson correlation (and R2) between the fitted per-cell pseudotimes and externally known physical capture times, as done for PhenoPath on the Shalek LPS/PAM dendritic-cell scRNA-seq data.

## When to use

Use when a pseudotime method has been fit on scRNA-seq data whose cells have known physical capture/sampling times, and you need an external check that the latent ordering recapitulates real time progression.

## When NOT to use

- The dataset has no external per-cell time labels (e.g. purely cross-sectional single-sample data), so no physical capture-time validation is possible
- Capture times were included as model covariates during fitting, making the correlation a circular, non-independent validation
- The biological process is expected to be uncoupled from sampling time (e.g. in vitro activation dynamics not aligned with capture order), making capture-time correlation a misleading quality metric

## Inputs

- Per-cell inferred pseudotimes (e.g. posterior means fit$m_z from a PhenoPath fit)
- Per-cell physical capture times held out from model fitting
- Expression matrix used for fitting (e.g. 820 dendritic cells x 7500 most variable genes, log2(TPM+1)-scale, Salmon re-quantified GSE48963)

## Outputs

- Pearson correlation and R2 between inferred pseudotimes and capture times (expected R2 = 0.68 for PhenoPath on GSE48963)
- Comparison table of correlations vs capture time across gene-set sizes and methods (PhenoPath vs Monocle 2, DPT, TSCAN)
- CSV of per-cell pseudotimes for reproducibility

## How to apply

After fitting the trajectory model (e.g. PhenoPath via phenopath() with coordinate ascent mean-field variational inference, initialised on the first principal component since the posterior is multi-modal), extract the posterior-mean pseudotimes per cell with trajectory(fit), which retrieves fit$m_z. Crucially, capture times must be deliberately excluded from the model input — they are used only for validation, never as a covariate. Compute the Pearson correlation of the fitted pseudotimes against the physical capture times and report R2; for the Shalek dendritic-cell data (820 cells, GSE48963, 7500 most variable genes on a log2(TPM+1)-comparable scale) the expected value is R2 = 0.68, exceeding Monocle 2, DPT and TSCAN. Report the correlation for every input gene-set size so methods can be compared fairly across feature-selection choices. Write the per-cell pseudotimes to a CSV file alongside capture times to make the comparison reproducible.

## Related tools

- **PhenoPath** (Fits the pseudotimes under validation via phenopath() with CAVI variational inference; pseudotimes extracted with trajectory(fit)) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **Monocle 2** (Baseline pseudotime method against which capture-time correlation of PhenoPath is compared) — https://bioconductor.org/packages/release/bioc/html/monocle.html
- **R** (Language in which the correlation analysis, CSV export and baseline methods are run) — https://bioconductor.org/packages/release/bioc/html/phenopath.html

## Examples

```
fit <- phenopath(Y, x, z_init = 1); pt <- trajectory(fit); cor(pt, capture_times); write.csv(data.frame(cell = colnames(Y), pseudotime = pt), "pseudotimes.csv", row.names = FALSE)
```

## Evaluation signals

- Pearson correlation between trajectory(fit) pseudotimes and capture times reaches approximately R2 = 0.68 on the Shalek dendritic-cell data
- Correlation with capture time is higher than Monocle 2, DPT and TSCAN for every input gene set size tested
- ELBO trace from plot_elbo(fit) is approximately flat, confirming CAVI convergence before pseudotimes are extracted
- fit$m_z contains one posterior-mean pseudotime per cell (820 values for GSE48963) and matches the per-cell CSV written out

## Limitations

- The validation assumes the biological process is monotonic in physical capture time; pseudotime methods recover relative progression, not absolute clock time
- Capture times are deliberately excluded from model input, so this is external validation only and cannot improve the fit itself
- PhenoPath's underlying assumption that features evolve linearly with pseudotime may bias correlations if nonlinear dynamics dominate

## Evidence

- [methods] Capture times excluded from model fitting and used only for validation: [evidence span withheld — anchor status: not_found]
- [methods] Pseudotime extraction via trajectory(fit): [evidence span withheld — anchor status: not_found]
- [results] Expected validation result R2 = 0.68 outperforming baselines: [evidence span withheld — anchor status: not_found]
- [results] Higher capture-time correlation than competing methods at all gene-set sizes: [evidence span withheld — anchor status: not_found]
- [readme] CAVI inference and Rcpp implementation grounding the fitting step: [evidence span withheld — anchor status: external_doc]
