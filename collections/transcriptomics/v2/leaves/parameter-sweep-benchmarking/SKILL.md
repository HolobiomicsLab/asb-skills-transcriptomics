---
name: parameter-sweep-benchmarking
description: Use when evaluating how technical or biological simulation parameters
  (e.g., capture efficiency alpha, variability sigma) affect the relative performance
  of multiple methods that need simulated gold standards for scoring.
when_to_use_negative:
- The task is a single fixed-parameter run rather than a sweep over simulation settings
  such as Sigma or alpha
- Real experimental data with verified gold standards are available, making simulation-based
  ground truth unnecessary
- The methods under comparison require UMI-based input; this workflow uses True2ObservedCounts
  with protocol='nonUMI'
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SymSim
  role: Generates ground-truth and observed-count datasets at each grid point (SimulateTrueCounts,
    True2ObservedCounts, getTrajectoryGenes, Phyla5)
  repo: github.com/YosefLab/SymSim
- name: Monocle
  role: Trajectory inference method benchmarked at each Sigma/alpha setting (version
    2.6.4)
- name: Slingshot
  role: Trajectory inference method benchmarked at each Sigma/alpha setting (version
    0.99.12)
- name: dynmethods
  role: Provides the minimum-spanning-tree trajectory inference implementation wrapped
    via dynwrap 0.1.0 for the benchmark
  repo: github.com/dynverse/dynmethods
- name: dynwrap
  role: Common trajectory model used to wrap method inputs/outputs in the dynverse
    benchmarking framework (version 0.1.0)
  repo: github.com/dynverse/dynwrap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:8d2c047f0f9358e2993d47d404386ad5408bad66fa71a67796e5b7ed35a490fd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle
  - Slingshot
  - SymSim
  - dynmethods
  - dynwrap
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# parameter-sweep-benchmarking

## Summary

Benchmark computational methods across a controlled parameter grid (e.g., SymSim's Sigma within-population variability and alpha capture efficiency) by simulating gold-standard datasets, running competing methods on each grid point, and scoring against simulated ground truth.

## When to use

Use when evaluating how technical or biological simulation parameters (e.g., capture efficiency alpha, variability sigma) affect the relative performance of multiple methods that need simulated gold standards for scoring.

## When NOT to use

- The task is a single fixed-parameter run rather than a sweep over simulation settings such as Sigma or alpha
- Real experimental data with verified gold standards are available, making simulation-based ground truth unnecessary
- The methods under comparison require UMI-based input; this workflow uses True2ObservedCounts with protocol='nonUMI'

## Inputs

- Parameter grid definition (e.g., Sigma and alpha values)
- Five-leaf reference tree from SymSim Phyla5()
- Human transcript-length pool (gene_len_pool) for assigning gene lengths
- Fixed randseed for reproducible simulation
- Trajectory inference methods to benchmark (Monocle 2.6.4, Slingshot 0.99.12, dynmethods MST)

## Outputs

- Observed-count matrices (one per grid point) with paired gold-standard pseudotime and branch labels
- Per-method score table of cor_pseudotime (Spearman) and cal_KNNP (KNN purity) across the Sigma/alpha grid
- Trend/magnitude comparison against reference plots (e.g., Fig. 7a-b)

## How to apply

Define a grid over the parameters of interest — here Sigma (within-population variability) and alpha (capture efficiency) — and for each cell generate a ground-truth dataset with SymSim's SimulateTrueCounts (evf_type='continuous', ncells_total=500, ngenes=500, nevf=20, n_de_evf=12, vary='s', phyla=Phyla5(), fixed randseed), then convert to observed non-UMI counts with True2ObservedCounts using gene lengths sampled from gene_len_pool. Extract gold-standard pseudotime (distance from root) and branch labels via getTrajectoryGenes. Run every competing method — Monocle 2.6.4, Slingshot 0.99.12, and the dynmethods (dynwrap 0.1.0) minimum-spanning-tree implementation — on each observed-count dataset, then score with cor_pseudotime (Spearman correlation between inferred and true pseudotime) and cal_KNNP (k-nearest-neighbor purity). Compare the metric trends across the grid (Fig. 7a-b) to identify both parameter sensitivity and the best-performing method; in this benchmark Slingshot clearly outperformed Monocle and the MST method.

## Related tools

- **SymSim** (Generates ground-truth and observed-count datasets at each grid point (SimulateTrueCounts, True2ObservedCounts, getTrajectoryGenes, Phyla5)) — github.com/YosefLab/SymSim
- **Monocle** (Trajectory inference method benchmarked at each Sigma/alpha setting (version 2.6.4))
- **Slingshot** (Trajectory inference method benchmarked at each Sigma/alpha setting (version 0.99.12))
- **dynmethods** (Provides the minimum-spanning-tree trajectory inference implementation wrapped via dynwrap 0.1.0 for the benchmark) — github.com/dynverse/dynmethods
- **dynwrap** (Common trajectory model used to wrap method inputs/outputs in the dynverse benchmarking framework (version 0.1.0)) — github.com/dynverse/dynwrap

## Examples

```
phyla <- Phyla5(); true_counts_res <- SimulateTrueCounts(ncells_total=500, ngenes=500, nevf=20, n_de_evf=12, sigma=0.6, vary='s', evf_type='continuous', phyla=phyla, randseed=1); obs <- True2ObservedCounts(true_counts=true_counts_res[[1]], meta=true_counts_res[[3]], protocol='nonUMI', alpha_mean=0.1, gene_len_pool=gene_len_pool)
```

## Evaluation signals

- Spearman correlation (cor_pseudotime) between inferred and gold-standard pseudotime is computed for every grid cell and method
- KNN purity (cal_KNNP) reproduces the known trend: metrics decrease as Sigma increases (alpha=0.1) and increase with alpha (sigma=0.6)
- The method ranking (Slingshot > Monocle and MST) matches the magnitudes reported in Fig. 7a-b
- All grid points use the same randseed and Phyla5() tree so differences are attributable to Sigma/alpha, not simulation noise
- Each observed-count dataset is paired with gold-standard pseudotime and branch labels from getTrajectoryGenes before scoring

## Limitations

- The simulation pipeline omits several steps, including reverse transcription and library cleanup, so observed-count properties only approximate real protocols
- Results are only as faithful as SymSim's generative model; parameter effects (Sigma, alpha) are simulated, not empirically measured
- Conclusions may depend on the fixed simulation configuration (ncells_total=500, ngenes=500, nevf=20, n_de_evf=12, five-leaf tree) and may not transfer to other tree topologies or dataset sizes

## Evidence

- [abstract] SymSim can be used to benchmark clustering, differential expression, and trajectory inference methods and to examine parameter effects on their performance: "We demonstrate how SymSim can be used for benchmarking methods for clustering, differential expression and trajectory inference, and for examining the effects of various parameters on their"
- [methods] SimulateTrueCounts and True2ObservedCounts are the core generation functions: "**SimulateTrueCounts( )** and **True2ObservedCounts( )** are the two major functions to generate datasets."
- [methods] getTrajectoryGenes returns simulated trajectory information used as gold standard: "Function **getTrajectoryGenes( )** returns the simulated trajectory information of cells."
- [discussion] dynverse versions used for the MST method wrapper: "We use the R packages dynwrap (https://github.com/dynverse/dynwrap, version 0.1.0)"
- [readme] dynmethods wraps trajectory inference methods for comparison: "The output of each method is
transformed into a common trajectory model using
[dynwrap](https://www.github.com/dynverse/dynwrap), which allows easy
[visualisation and comparison]"
