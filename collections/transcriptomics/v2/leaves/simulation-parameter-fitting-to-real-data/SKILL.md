---
name: simulation-parameter-fitting-to-real-data
description: Use when a simulation must be calibrated to a specific real scRNA-seq
  dataset (e.g., Zeisel UMI cortex counts) so downstream benchmarks or experimental-design
  curves reflect realistic depth, dropout, and variability.
when_to_use_negative:
- The analysis goal is purely theoretical and does not need calibration to any real
  dataset (e.g., a closed-form binomial bound on detecting a rare population)
- The real dataset's protocol or statistics have no counterpart in the pre-computed
  simulated grid (BestMatchParams relies on grid-search database coverage)
- Fitting kinetic parameters when cells are not dividable into clusters sharing the
  same kinetic parameters
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: SymSim
  role: Provides BestMatchParams and the grid-search simulation framework to match
    simulated data statistics to a real dataset; also the simulation engine whose
    parameters are fitted
  repo: github.com/YosefLab/SymSim
- name: RJAGS
  role: Gibbs sampler used to fit kinetic (kon, koff, s) parameters from real experimental
    data via MCMC with 2000 iterations
- name: scVI
  role: Imputation of UMI-based counts before kinetic parameter fitting (applicable
    only to large datasets)
- name: MAGIC
  role: Imputation of counts (with scVI for UMI data; alone for non-UMI data) prior
    to kinetic parameter fitting
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:53667f9979e5e3b56e7faae87dd6a4bc932c063f08901ef02b7cf79e0bc87688
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BestMatchParams
  - MAGIC
  - RJAGS
  - Rtsne
  - SimulateTrueCounts
  - SymSim
  - True2ObservedCounts
  - ape
  - scVI
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# simulation-parameter-fitting-to-real-data

## Summary

Fit SymSim simulation parameters (Sigma, capture efficiency alpha, sequencing depth, and kinetic kon/koff/s parameters) so that simulated datasets reproduce the statistical properties of a given real scRNA-seq dataset, via BestMatchParams grid-search matching or MCMC kinetic fitting with RJAGS.

## When to use

Use when a simulation must be calibrated to a specific real scRNA-seq dataset (e.g., Zeisel UMI cortex counts) so downstream benchmarks or experimental-design curves reflect realistic depth, dropout, and variability.

## When NOT to use

- The analysis goal is purely theoretical and does not need calibration to any real dataset (e.g., a closed-form binomial bound on detecting a rare population)
- The real dataset's protocol or statistics have no counterpart in the pre-computed simulated grid (BestMatchParams relies on grid-search database coverage)
- Fitting kinetic parameters when cells are not dividable into clusters sharing the same kinetic parameters

## Inputs

- Real scRNA-seq count matrix (e.g., UMI cortex_counts.txt packaged with SymSim; or Th17 ss2_130cells_counts)
- Pre-computed grid-search database of simulated datasets with statistics over Sigma, alpha, and depth
- Gene length pool (gene_len_pool) sampled from human transcript lengths
- Cell cluster assignments used to satisfy the shared-kinetics assumption
- Imputed counts (scVI/MAGIC) for kinetic parameter fitting

## Outputs

- Best-matching parameter configurations (Sigma, capture efficiency alpha/alpha_sd, depth), e.g. top n_optimal=5 settings
- Fitted kinetic parameters (kon, koff, s) per cluster
- A calibrated simulation recipe for SimulateTrueCounts / True2ObservedCounts (e.g., param_realdata='zeisel.imputed')

## How to apply

First choose the simulation scenario matching the real data protocol ('UMI' or 'non-UMI'). For single-population matching, load the real count matrix (e.g., the packaged cortex_counts.txt, 3005 Zeisel cortex cells) and call BestMatchParams against a pre-computed grid-search database of simulated datasets, retaining the top configurations (e.g., n_optimal=5) with depth_range filtered to the observed library-size regime (e.g., c(200e3, Inf)); the returned configurations specify Sigma, capture-efficiency alpha (and its spread alpha_sd), and sequencing depth. For kinetic parameters, cluster the real cells so all cells in a cluster share kinetics, use imputed counts (scVI/MAGIC for UMI data; MAGIC for non-UMI data, since scVI is only applicable to large datasets), and fit the transcriptional bursting model (kon, koff, s) with the Gibbs sampler implemented in RJAGS over 2000 iterations. Pass the fitted parameters forward (e.g., param_realdata='zeisel.imputed') into SimulateTrueCounts and True2ObservedCounts so simulated data carry realistic zeros, depth distributions, and noise levels.

## Related tools

- **SymSim** (Provides BestMatchParams and the grid-search simulation framework to match simulated data statistics to a real dataset; also the simulation engine whose parameters are fitted) — github.com/YosefLab/SymSim
- **RJAGS** (Gibbs sampler used to fit kinetic (kon, koff, s) parameters from real experimental data via MCMC with 2000 iterations)
- **scVI** (Imputation of UMI-based counts before kinetic parameter fitting (applicable only to large datasets))
- **MAGIC** (Imputation of counts (with scVI for UMI data; alone for non-UMI data) prior to kinetic parameter fitting)

## Examples

```
library(SymSim); counts <- read.table(system.file("extdata", "cortex_counts.txt", package="SymSim")); params <- BestMatchParams('UMI', counts, 'best_params.umi.qqplot.pdf', depth_range=c(200e3, Inf), n_optimal=5)
```

## Evaluation signals

- BestMatchParams output reproduces the real dataset's distributions of summary statistics (e.g., library-size/depth distribution within depth_range such as c(200e3, Inf))
- Simulated observed counts from the fitted parameters recapitulate real-data properties (zero abundance, noise level) without explicitly forcing them
- Multiple top parameter sets (n_optimal=5) are inspected, since the grid search can yield several near-equally-matching configurations
- Kinetic fitting is performed on imputed counts with cells divided by clustering, so within-cluster cells plausibly share the same kinetic parameters
- Fitted settings (e.g., zeisel.imputed) transfer coherently to downstream SimulateTrueCounts/True2ObservedCounts calls without re-tuning

## Limitations

- The grid-search approach can only return parameters represented in the pre-computed simulated database; unusual real-data statistics may fall outside the grid
- Kinetic parameter fitting assumes all cells within a cluster share the same kinetic parameters, which is an approximation
- SymSim's pipeline omits several steps, including reverse transcription and library cleanup
- scVI-based imputation is only applicable to large datasets, so non-UMI data are imputed with MAGIC alone
- Supplementary Tables 2-5 with the fitted parameters (e.g., capture efficiency per dataset) are not included in the main text

## Evidence

- [methods] For a single population, BestMatchParams( ) returns parameters to generate datasets with similar statistical properties as a given input real dataset.: "For a single population, **BestMatchParams( )** returns parameters to generate datasets with similar statistical properties as a given input real dataset."
- [discussion] Grid search over simulated database to match real data statistics.: [evidence span withheld — anchor status: not_found]
- [discussion] RJAGS Gibbs sampler MCMC estimation of kinetic parameters with 2000 iterations.: [evidence span withheld — anchor status: not_found]
- [discussion] Imputed counts are used instead of raw read counts for fitting.: "We use imputed read counts, rather than the raw read counts."
- [intro] SymSim recapitulates properties of real data without explicitly forcing them.: "SymSim recapitulates properties of the data (e.g., high abundance of zeros or increased noise in non-UMI protocols) without the need to explicitly force them as factors in a distributional model"
