---
name: binomial-detection-probability-modeling
description: Use when planning a single-cell RNA-seq experiment to determine the cell
  number N needed to detect a rare population (e.g., 5% prevalence) while accounting
  for within-population variability and capture efficiency.
when_to_use_negative:
- The goal is benchmarking clustering, DE, or trajectory-inference method accuracy
  rather than sizing an experiment to detect a rare population.
- The population of interest is abundant (not rare), where theoretical binomial sizing
  alone is adequate without simulating variability.
- The sequencing protocol and biological system differ substantially from the fitted
  reference (e.g., non-UMI data where scVI is not applicable to small datasets), so
  fitted Sigma/alpha parameters would not transfer.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: SymSim
  role: R package supplying SimulateTrueCounts, True2ObservedCounts, BestMatchParams,
    Phyla5, and gene_len_pool to simulate rare-population datasets with matched variability
    parameters
  repo: https://github.com/YosefLab/SymSim
- name: Rtsne
  role: t-SNE embedding of simulated observed counts used as input to k-means clustering
    for one of the evaluated detection methods
- name: SIMLR
  role: Dimensionality reduction/embedding method evaluated as one of the four clustering
    approaches for rare-population detection
- name: scVI
  role: Generative-model embedding method evaluated as one of the four clustering
    approaches for rare-population detection
- name: PCA
  role: Dimensionality reduction of simulated counts feeding k-means clustering for
    one of the evaluated detection methods
- name: ape
  role: Phylogenetic tree support used to construct the Phyla5 population tree whose
    rare leaf holds the target subpopulation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:b6680483361596bcecd8b29703ec53c601c256abfc87e4224633206216eb4410
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - BestMatchParams
  - PCA
  - Rtsne
  - SIMLR
  - SimulateTrueCounts
  - SymSim
  - True2ObservedCounts
  - ape
  - scVI
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# binomial-detection-probability-modeling

## Summary

Uses SymSim-based simulation to estimate how many cells must be sequenced to detect a rare subpopulation, comparing empirical detection probabilities across clustering methods with a closed-form binomial theoretical detection curve.

## When to use

Use when planning a single-cell RNA-seq experiment to determine the cell number N needed to detect a rare population (e.g., 5% prevalence) while accounting for within-population variability and capture efficiency.

## When NOT to use

- The goal is benchmarking clustering, DE, or trajectory-inference method accuracy rather than sizing an experiment to detect a rare population.
- The population of interest is abundant (not rare), where theoretical binomial sizing alone is adequate without simulating variability.
- The sequencing protocol and biological system differ substantially from the fitted reference (e.g., non-UMI data where scVI is not applicable to small datasets), so fitted Sigma/alpha parameters would not transfer.

## Inputs

- UMI-based reference count matrix (e.g., SymSim packaged cortex_counts.txt, 3005 Zeisel cortex cells)
- Candidate total cell numbers N (e.g., 600 to 7000)
- Grid of within-population variability (Sigma) and capture-efficiency (alpha) settings from BestMatchParams
- Population tree with a rare leaf (e.g., Phyla5) and rare-population size (e.g., min_popsize=50)
- Gene length pool (gene_len_pool) for transcript-length sampling

## Outputs

- Empirical detection probability curves P(N, Sigma, alpha, method) per clustering approach
- Smoothed empirical upper detection bound across methods
- Closed-form binomial theoretical detection curve
- Heatmap of required cell number over the Sigma x alpha grid (Fig. 8a-e panels, figure8_rare_population_detection.png)
- Table of detection probabilities (detection_probabilities.csv)

## How to apply

Fit simulation parameters to a real UMI reference dataset (e.g., the packaged cortex_counts.txt with 3005 Zeisel cortex cells) using BestMatchParams('UMI', counts, depth_range = c(200e3, Inf), n_optimal = 5) to obtain Sigma, capture-efficiency alpha, and depth settings. For each candidate total cell number N in [600, 7000] and each Sigma/alpha combination, call SimulateTrueCounts with evf_type='discrete', a Phyla5 tree whose rare leaf holds min_popsize=50 cells, and param_realdata='zeisel.imputed', then convert true counts with True2ObservedCounts(protocol='UMI', alpha_mean=alpha, alpha_sd, gene lengths sampled from gene_len_pool). Cluster each replicate with k-means on PCA, t-SNE (Rtsne/PlotTsne), SIMLR, and scVI embeddings, scoring a replicate as 'detected' when the clustering recovers at least 50 rare-population cells with precision >= 70%. Aggregate replicates into empirical detection probability P(N, Sigma, alpha, method), overlay the closed-form binomial theoretical curve and the smoothed empirical upper bound, and summarize required cell counts as a heatmap over the Sigma x alpha grid. Rationale: the binomial bound ignores biological and technical variability, so comparing it against simulation-derived curves quantifies how much variability inflates the required N.

## Related tools

- **SymSim** (R package supplying SimulateTrueCounts, True2ObservedCounts, BestMatchParams, Phyla5, and gene_len_pool to simulate rare-population datasets with matched variability parameters) — https://github.com/YosefLab/SymSim
- **Rtsne** (t-SNE embedding of simulated observed counts used as input to k-means clustering for one of the evaluated detection methods)
- **SIMLR** (Dimensionality reduction/embedding method evaluated as one of the four clustering approaches for rare-population detection)
- **scVI** (Generative-model embedding method evaluated as one of the four clustering approaches for rare-population detection)
- **PCA** (Dimensionality reduction of simulated counts feeding k-means clustering for one of the evaluated detection methods)
- **ape** (Phylogenetic tree support used to construct the Phyla5 population tree whose rare leaf holds the target subpopulation)

## Examples

```
devtools::install_github('YosefLab/SymSim'); params <- BestMatchParams('UMI', cortex_counts, 'best_params.umi.qqplot.pdf', depth_range=c(200e3, Inf), n_optimal=5)
```

## Evaluation signals

- The fitted BestMatchParams output reproduces the statistical properties of the 3005-cell Zeisel cortex UMI dataset within depth_range = c(200e3, Inf) with n_optimal = 5 candidate configurations.
- Detected replicates strictly satisfy the dual criterion: at least 50 rare-population cells recovered AND precision >= 70%.
- Empirical P(N) curves are monotone increasing in N and lie at or below the theoretical binomial detection probability for the same rare-population fraction.
- Required cell counts from simulation substantially exceed the binomial bound (e.g., 3838 cells vs fewer than 1200 cells for a 5% population at 0.9 success under Sigma=0.7, alpha=0.01).
- detection_probabilities.csv contains P values for every (N, Sigma, alpha, method) combination and the heatmap covers the full Sigma x alpha grid.

## Limitations

- Required cell counts under realistic variability (3838 cells for a 5% population) far exceed the theoretical binomial bound of fewer than 1200 cells, so naive binomial sizing underestimates N.
- The simulation pipeline omits several steps, including reverse transcription and library cleaning up, which may affect transferability of fitted parameters.
- scVI is only applicable to large datasets; for smaller or non-UMI data the imputation/embedding strategy must change (non-UMI data were imputed using MAGIC).
- Kinetic parameter fitting assumes all cells in a cluster share the same kinetic parameters; cells are divided by clustering to meet this assumption.

## Evidence

- [abstract] SymSim can be used to evaluate the number of cells required to detect a rare population under various scenarios.: "We also show how SymSim can be used to evaluate the number of cells required to detect a rare population under various scenarios."
- [intro] Experimental-design framing of the skill: "In the second example, we use SymSim for the purpose of experimental design, focusing on the question of how many cells should one sequence to identify a certain subpopulation."
- [methods] Parameter matching to real data: "For a single population, **BestMatchParams( )** returns parameters to generate datasets with similar statistical properties as a given input real dataset."
- [methods] Core simulation-to-observation pipeline: "**SimulateTrueCounts( )** and **True2ObservedCounts( )** are the two major functions to generate datasets."
- [discussion] Reference dataset used for fitting: "we used a UMI-based dataset of 3005 cortex cells by Zeisel et al.38"
- [readme] SymSim installation command: "devtools::install_github("YosefLab/SymSim")"
