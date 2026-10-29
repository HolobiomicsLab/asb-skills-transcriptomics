---
name: statistical-distribution-fitting
description: Use when you need to find simulation parameters that reproduce the statistical
  properties (gene CV/mean distributions, zero fraction, SD) of a real scRNA-seq dataset,
  for either UMI or non-UMI protocols.
when_to_use_negative:
- The goal is forward simulation with known mechanistic parameters (use SimulateTrueCounts/True2ObservedCounts
  directly) rather than fitting parameters to an existing real dataset
- The input dataset is small and non-UMI such that generative-model alternatives are
  required; note scVI is only applicable to large datasets in the article's imputation
  context
- You need kinetic parameter estimation via MCMC (kon, koff, s with RJAGS Gibbs sampling,
  2000 iterations) rather than grid-based best-match search
edam_operation: http://edamontology.org/operation_3575
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: SymSim
  role: Provides BestMatchParams( ) to grid-search simulated datasets and return parameters
    matching a real dataset's statistics, plus the pre-computed simulation database
    and extdata example datasets
  repo: https://github.com/YosefLab/SymSim
- name: ggplot2
  role: Used to render Q-Q plots of gene mean, percent non-zero, and SD comparing
    real vs. top-ranked simulated data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:b9a0b6fd354a13ccbc1aa167aa74d64e82c91e4105d15cf98ac53dbd25a987a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - SymSim
  - ggplot2
  edam_operation: http://edamontology.org/operation_3575
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# statistical-distribution-fitting

## Summary

Fit simulation parameters to real scRNA-seq data by grid-searching a database of SymSim-simulated datasets and matching statistical distributions (gene mean, CV, percent non-zero, SD), then validating the fit with Q-Q plots against identity lines and comparing against Splatter and powsimR fits.

## When to use

Use when you need to find simulation parameters that reproduce the statistical properties (gene CV/mean distributions, zero fraction, SD) of a real scRNA-seq dataset, for either UMI or non-UMI protocols.

## When NOT to use

- The goal is forward simulation with known mechanistic parameters (use SimulateTrueCounts/True2ObservedCounts directly) rather than fitting parameters to an existing real dataset
- The input dataset is small and non-UMI such that generative-model alternatives are required; note scVI is only applicable to large datasets in the article's imputation context
- You need kinetic parameter estimation via MCMC (kon, koff, s with RJAGS Gibbs sampling, 2000 iterations) rather than grid-based best-match search

## Inputs

- Real scRNA-seq count matrix, e.g., cortex_counts.txt (UMI cortex, Zeisel et al., 3005 cells) read from SymSim extdata
- Non-UMI scRNA-seq count matrix, e.g., ss2_130cells_counts from Th17_data.RData (Gaublomme et al., 130 IL17-expressing T helper cells), filtered to genes expressed in >10 cells
- Pre-computed grid database of SymSim-simulated datasets over a wide range of parameters
- Technology specification: 'UMI' or 'nonUMI'; depth_range bounds; n_optimal (e.g., 5)

## Outputs

- Top-n parameter configurations (e.g., 5) with statistical properties matching the real dataset
- Q-Q plot PDFs (best_params.umi.qqplot.pdf, best_params.nonUMI.qqplot.pdf) comparing mean, percent non-zero, and SD of genes between real and simulated data
- CSV table compiling best-matching parameters for both datasets and distribution-fit quality (CV/mean relationship)

## How to apply

Load the real count matrix (e.g., the UMI cortex dataset cortex_counts.txt or the non-UMI Th17 dataset ss2_130cells_counts from SymSim's extdata, filtering Th17 counts to genes expressed in more than 10 cells). Run SymSim's BestMatchParams( ) with tech='UMI' or tech='nonUMI' over a pre-computed grid of simulated datasets, restricting the sequencing depth with depth_range (e.g., c(200e3, Inf) for cortex UMI, c(1e6, 2e6) for Th17 non-UMI) and requesting n_optimal=5 top configurations. This approach fits parameters by finding the simulated dataset whose distributions of statistics are most similar to the real data, rather than fitting abstract distributional models as Splatter and powsimR do. Generate Q-Q plots of gene mean, percent non-zero, and standard deviation between the real dataset and the top-ranking simulation, and compile the best-matching parameter configurations into a table for comparison with Supplementary Fig. 5c-e.

## Related tools

- **SymSim** (Provides BestMatchParams( ) to grid-search simulated datasets and return parameters matching a real dataset's statistics, plus the pre-computed simulation database and extdata example datasets) — https://github.com/YosefLab/SymSim
- **ggplot2** (Used to render Q-Q plots of gene mean, percent non-zero, and SD comparing real vs. top-ranked simulated data)

## Examples

```
best_params <- BestMatchParams(counts, tech='UMI', depth_range=c(200e3,Inf), n_optimal=5)
```

## Evaluation signals

- Both Q-Q plot PDFs (best_params.umi.qqplot.pdf and best_params.nonUMI.qqplot.pdf) exist after the run
- Q-Q points for mean expression, percent non-zero, and SD of genes fall close to the identity line for the top-ranked simulation
- BestMatchParams returns exactly n_optimal (5) parameter configurations per dataset
- The Q-Q fit of gene CV and mean distributions is overall better than Splatter and powsimR fits on the same datasets (as in Supplementary Fig. 5c-e)
- The Th17 counts were filtered to genes expressed in more than 10 cells before fitting

## Limitations

- The grid search is limited to the pre-computed database of simulated datasets over a fixed parameter grid, so the best match is approximate rather than a continuous maximum-likelihood fit
- The SymSim pipeline omits several steps, including reverse transcription and library cleaning up
- Supplementary Tables 2-5 with fit parameters and Supplementary Note 3 fragmentation details are referenced but not included in the main text

## Evidence

- [methods] For a single population, BestMatchParams( ) returns parameters to generate datasets with similar statistical properties as a given input real dataset.: "For a single population, **BestMatchParams( )** returns parameters to generate datasets with similar statistical properties as a given input real dataset."
- [discussion] Fitting by matching simulated database statistics to real-data statistics: [evidence span withheld — anchor status: not_found]
- [intro] Existing simulation strategies rely on fitting distributional models with abstract parameters: "their parameters are often abstract and do not directly correspond to the actual processes that gave rise to the observations"
- [intro] SymSim recapitulates real-data properties without forcing them in a distributional model: "SymSim recapitulates properties of the data (e.g., high abundance of zeros or increased noise in non-UMI protocols) without the need to explicitly force them as factors"
- [readme] SymSim installation via devtools from GitHub: "devtools::install_github("YosefLab/SymSim")"
