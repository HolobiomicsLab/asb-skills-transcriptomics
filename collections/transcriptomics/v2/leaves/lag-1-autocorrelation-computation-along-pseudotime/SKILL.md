---
name: lag-1-autocorrelation-computation-along-pseudotime
description: Use when comparing pseudotime orderings produced by multiple trajectory-inference
  algorithms on the same scRNA-seq dataset (e.g., GSE72857 myeloid progenitors or
  Splatter simulations) and you need a quantitative smoothness score per algorithm.
when_to_use_negative:
- When the goal is to reconstruct a trajectory itself (e.g., subpopulation-aware flow-network
  reconstruction with CellRouter) rather than to score the smoothness of orderings
  already produced by one or more algorithms.
- When cells lack a single pseudotime ordering (e.g., unconnected subpopulations or
  no trajectory spanning the cell states of interest), since lag-1 autocorrelation
  requires a successive ordering of cells.
- When the input is bulk RNA-seq or otherwise lacks per-cell expression and per-cell
  pseudotime values.
tools:
- name: CellRouter
  role: Source of the benchmark trajectories being scored; its orderings are compared
    against baselines using lag-1 autocorrelation and correlated-gene counts
  repo: https://github.com/edroaldo/cellrouter
- name: Monocle 2
  role: Baseline trajectory algorithm whose pseudotime ordering is scored with the
    autocorrelation metric
- name: Diffusion Pseudotime (DPT)
  role: Baseline algorithm (requires a priori number of branches) whose pseudotime
    ordering is scored
- name: Wishbone
  role: Baseline algorithm (single branch point) whose pseudotime ordering is scored
- name: Waterfall
  role: Baseline algorithm (linear trajectories) whose pseudotime ordering is scored
- name: Splatter
  role: Generator of the synthetic single-cell count matrices on which the metric
    is evaluated (Supplementary Fig. 4g,h)
- name: R
  role: Implementation environment for computing the metric and assembling the comparison
    table/figures
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:67670c633ec8bd3e72b56696c77e4662b9ad02d5316563ddc542a5a749f4adef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - Diffusion Pseudotime (DPT)
  - Monocle 1
  - Monocle 2
  - R
  - Splatter
  - Waterfall
  - Wishbone
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Lag-1 autocorrelation computation along pseudotime

## Summary

A trajectory-smoothness metric that correlates gene expression between successive cells along a reconstructed pseudotime ordering, used in the CellRouter paper to quantify whether trajectories (e.g., CellRouter vs. Monocle 2, DPT, Wishbone, Waterfall) have smoother, more autocorrelated gene expression dynamics.

## When to use

Use when comparing pseudotime orderings produced by multiple trajectory-inference algorithms on the same scRNA-seq dataset (e.g., GSE72857 myeloid progenitors or Splatter simulations) and you need a quantitative smoothness score per algorithm.

## When NOT to use

- When the goal is to reconstruct a trajectory itself (e.g., subpopulation-aware flow-network reconstruction with CellRouter) rather than to score the smoothness of orderings already produced by one or more algorithms.
- When cells lack a single pseudotime ordering (e.g., unconnected subpopulations or no trajectory spanning the cell states of interest), since lag-1 autocorrelation requires a successive ordering of cells.
- When the input is bulk RNA-seq or otherwise lacks per-cell expression and per-cell pseudotime values.

## Inputs

- Pseudotime orderings of the same cells produced by each trajectory algorithm (CellRouter, Monocle 2, Diffusion Pseudotime, Wishbone, Waterfall, plus CellRouter with Paul et al. cluster labels)
- Normalized scRNA-seq expression matrix (GSE72857 Paul et al. mouse myeloid progenitor data or Splatter synthetic counts)
- Branch marker gene list: Klf1, Car2, Cited4 (erythroid); Mpo, Prtn3, Ctsg (myeloid)

## Outputs

- Lag-1 autocorrelation values of branch-marker expression along each algorithm's pseudotime
- Comparison table algorithm_comparison.csv with one row per algorithm variant and columns for significantly correlated gene counts and lag-1 autocorrelations
- Side-by-side plot of correlated-gene counts and lag-1 autocorrelation distributions per method (supplementary_fig4gh_comparison.png)

## How to apply

For each algorithm's pseudotime ordering over the same cells, compute the lag-1 autocorrelation of expression along the ordered trajectory — for the Paul et al. benchmark this is done specifically for the erythroid branch markers Klf1, Car2, Cited4 and the myeloid branch markers Mpo, Prtn3, Ctsg. Pair this metric with the count of genes whose expression is significantly correlated with pseudotime. Assemble one row per algorithm variant (five baselines plus the CellRouter-with-Paul-cluster-labels variant) with both metric columns into algorithm_comparison.csv, or plot correlated-gene counts and lag-1 autocorrelation distributions side by side (Supplementary Fig. 4g,h layout). The rationale is that a biologically coherent trajectory should show highly autocorrelated expression of lineage markers between adjacent cells along the path.

## Related tools

- **CellRouter** (Source of the benchmark trajectories being scored; its orderings are compared against baselines using lag-1 autocorrelation and correlated-gene counts) — https://github.com/edroaldo/cellrouter
- **Monocle 2** (Baseline trajectory algorithm whose pseudotime ordering is scored with the autocorrelation metric)
- **Diffusion Pseudotime (DPT)** (Baseline algorithm (requires a priori number of branches) whose pseudotime ordering is scored)
- **Wishbone** (Baseline algorithm (single branch point) whose pseudotime ordering is scored)
- **Waterfall** (Baseline algorithm (linear trajectories) whose pseudotime ordering is scored)
- **Splatter** (Generator of the synthetic single-cell count matrices on which the metric is evaluated (Supplementary Fig. 4g,h))
- **R** (Implementation environment for computing the metric and assembling the comparison table/figures)

## Examples

```
source('path/to/CellRouter_Class.R'); libdir <- 'path/to/CellRouter/'
```

## Evaluation signals

- algorithm_comparison.csv exists, contains a row for each of the six algorithm variants, and has both metric columns (significantly correlated gene counts and lag-1 autocorrelations) populated.
- The reproduced table/figure shows the paper's pattern: CellRouter trajectories are smoother, with more significantly correlated genes and higher lag-1 autocorrelation of branch markers than Monocle 2, DPT, Wishbone, and Waterfall.
- The synthetic-data figure (supplementary_fig4gh_comparison.png) qualitatively matches Supplementary Fig. 4g,h, with CellRouter recovering at least as many correlated genes and comparable lag-1 autocorrelation as the baselines.
- Marker-specific checks: erythroid markers (Klf1, Car2, Cited4) and myeloid markers (Mpo, Prtn3, Ctsg) each have lag-1 autocorrelation computed on the correct branch of each ordering.
- All methods are scored on the same cells/ordering inputs, so metric differences reflect trajectory quality rather than cell-set differences.

## Limitations

- Trajectories identified by CellRouter were reported as smoother than compared algorithms specifically for the correlation and autocorrelation metrics used; other smoothness criteria might rank methods differently.
- Existing algorithms are less robust in reconstructing trajectories from early states towards intermediate or transitory cell states, which can bias orderings for baselines on such paths.
- The original CellRouter was tested in Unix-based systems only (not Windows), so the R environment must run accordingly.
- On simulated data the validation was qualitative (figure matches Supplementary Fig. 4g,h), not a strict quantitative threshold.

## Evidence

- [methods] High-resolution multi-state trajectories are identified from any subpopulation to any other, including intermediate and mature cell types: "High-resolution multi-state trajectories are identified from any subpopulation to any other, including intermediate and mature cell types"
- [methods] Analysis of mouse myeloid progenitor single-cell RNA-seq data generated by Paul et al.: "Analysis of mouse myeloid progenitor single-cell RNA-seq data generated by Paul et al.(Cell 2015)"
- [results] Trajectories identified by CellRouter are smoother than those of the compared algorithms: [evidence span withheld — anchor status: not_found]
- [intro] Current trajectory algorithms are less robust from early states: "they are less robust in reconstructing trajectories from early states towards intermediate or transitory cell states"
- [readme] README: source CellRouter class and set libdir: "source the CellRouter class and make sure you provide the correct directory for the Java libraries inside the folder "CellRouter", as specified by the variable 'libdir'."
