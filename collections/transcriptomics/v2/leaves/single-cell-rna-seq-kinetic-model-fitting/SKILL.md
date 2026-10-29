---
name: single-cell-rna-seq-kinetic-model-fitting
description: Use when raw UMI or non-UMI scRNA-seq count matrices must be converted
  into gene-level transcription kinetic parameter distributions (kon, koff, s) for
  simulation or bursting analysis.
when_to_use_negative:
- Input is already imputed or normalized expression values rather than raw counts
  from which kinetics are fitted — fitting is defined on raw counts after a deliberate
  imputation step (scVI/MAGIC)
- Cells are an unclustered heterogeneous mixture, since the model assumes all cells
  share the same kinetic parameters within a fit
- The goal is to fit simulation parameters by statistical matching (BestMatchParams
  grid search) rather than to estimate transcription kinetics
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: RJAGS
  role: Provides the Gibbs sampler used to fit the two-state kinetic model (kon, koff,
    s) via MCMC with 2000 iterations
- name: scVI
  role: Imputes UMI-based counts prior to kinetic parameter fitting (applicable only
    to large datasets)
- name: MAGIC
  role: Imputes counts (both UMI and non-UMI) prior to kinetic parameter fitting
- name: SymSim
  role: Supplies the real count data (extdata/cortex_counts.txt) and consumes the
    fitted kinetic parameter distributions to simulate datasets
  repo: https://github.com/YosefLab/SymSim
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:87f7f1f8187b84b0b0d8f24313c75aaec97ee63d1f3eff4f044b01fbe073c965
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - MAGIC
  - RJAGS
  - SymSim
  - scVI
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-rna-seq-kinetic-model-fitting

## Summary

Fits a two-state transcription kinetic model (kon, koff, s) to single-cell RNA-seq count data via MCMC, producing posterior distributions of kinetic parameters usable to parameterize SymSim simulations.

## When to use

Use when raw UMI or non-UMI scRNA-seq count matrices must be converted into gene-level transcription kinetic parameter distributions (kon, koff, s) for simulation or bursting analysis.

## When NOT to use

- Input is already imputed or normalized expression values rather than raw counts from which kinetics are fitted — fitting is defined on raw counts after a deliberate imputation step (scVI/MAGIC)
- Cells are an unclustered heterogeneous mixture, since the model assumes all cells share the same kinetic parameters within a fit
- The goal is to fit simulation parameters by statistical matching (BestMatchParams grid search) rather than to estimate transcription kinetics

## Inputs

- Raw scRNA-seq count matrix (e.g., UMI-based Zeisel cortex counts, cortex_counts.txt)
- Cell cluster assignments satisfying the shared-kinetics assumption
- Imputed count matrices from scVI and/or MAGIC

## Outputs

- Posterior distributions of kon, koff, and s per gene per cluster per imputation method
- Summary table of fitted kinetic parameters (e.g., kinetic_params_zeisel_umi.csv) mirroring Supplementary Tables 2-5
- Parameter distributions feeding SymSim's param_realdata kinetic simulation (e.g., Fig. 2b)

## How to apply

Load the raw count matrix (e.g., extdata/cortex_counts.txt from the SymSim package via read.table(header=FALSE)) and cluster cells first, because the fitting model assumes all cells in a cluster share the same kinetic parameters (the Zeisel cortex data were split into oligodendrocyte, pyramidal CA1, and pyramidal S1 subpopulations). Impute the counts before fitting — scVI and MAGIC for UMI data, MAGIC only for non-UMI data since scVI is only applicable to large datasets — because raw zero-inflated counts are unsuitable. For each cluster-by-imputation combination, fit the two-state model with the Gibbs sampler implemented in RJAGS, setting the number of MCMC iterations to 2000. Aggregate posterior estimates of kon, koff, and s per cluster and imputation method into a summary table (mirroring Supplementary Tables 2-5) for use as SymSim kinetic parameter distributions.

## Related tools

- **RJAGS** (Provides the Gibbs sampler used to fit the two-state kinetic model (kon, koff, s) via MCMC with 2000 iterations)
- **scVI** (Imputes UMI-based counts prior to kinetic parameter fitting (applicable only to large datasets))
- **MAGIC** (Imputes counts (both UMI and non-UMI) prior to kinetic parameter fitting)
- **SymSim** (Supplies the real count data (extdata/cortex_counts.txt) and consumes the fitted kinetic parameter distributions to simulate datasets) — https://github.com/YosefLab/SymSim

## Examples

```
cortex_counts <- read.table(system.file("extdata", "cortex_counts.txt", package = "SymSim"), header = F)
```

## Evaluation signals

- Output file (e.g., kinetic_params_zeisel_umi.csv) exists and contains kon/koff/s estimates for every cluster-by-imputation-method combination
- Estimated per-cluster parameter distributions reproduce the patterns reported in Fig. 2b and Supplementary Tables 2-5
- MCMC runs completed the specified 2000 iterations without chain failure
- Fitting was performed on imputed counts (scVI/MAGIC), not raw zero-inflated counts
- Each fit corresponds to a single cell cluster, satisfying the shared-kinetics assumption

## Limitations

- The kinetic parameter fitting assumes all cells in a cluster share the same kinetic parameters, so heterogeneous or poorly resolved clusters bias the estimates
- Imputation method choice affects the fitted kinetics; scVI is not applicable to small non-UMI datasets (e.g., the 130-cell Th17 data, for which only MAGIC was used)
- Supplementary Tables 2-5 with the reported fit parameters are referenced but not present in the provided text, complicating direct numerical validation

## Evidence

- [discussion] We fit this model to the experimental data using the Gibbs sampler implemented in RJAGS.: "We fit this model to the experimental data using the Gibbs sampler implemented in RJAGS."
- [discussion] We estimated kinetic parameters from experimental data using an MCMC approach... The number of iterations is set to 2000.: [evidence span withheld — anchor status: not_found]
- [discussion] To meet the assumption that all cells share the same kinetic parameters we divide cells by clustering: "To meet the assumption that all cells share the same kinetic parameters we divide cells by clustering"
- [discussion] We use imputed read counts, rather than the raw read counts.: "We use imputed read counts, rather than the raw read counts."
- [methods] cortex_counts <- read.table(system.file("extdata", "cortex_counts.txt", package = "SymSim"), header = F: "cortex_counts <- read.table(system.file("extdata", "cortex_counts.txt", package = "SymSim"), header = F"
- [discussion] the UMI-based data were imputed using scVI4 and MAGIC18, and the non-UMI data were imputed using MAGIC18 (as scVI is only applicable to large datasets): "the UMI-based data were imputed using scVI4 and MAGIC18, and the non-UMI data were imputed using MAGIC18 (as scVI is only applicable to large datasets)"
- [readme] devtools::install_github("YosefLab/SymSim"): "devtools::install_github("YosefLab/SymSim")"
