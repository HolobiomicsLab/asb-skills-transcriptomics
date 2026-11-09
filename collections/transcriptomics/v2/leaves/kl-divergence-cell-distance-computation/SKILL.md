---
name: kl-divergence-cell-distance-computation
description: Use when a single-cell expression matrix (e.g. GO-subset, dispersion-filtered)
  must be converted into cell-to-cell edge weights for redPATH's cHMT pseudotime or
  Hamiltonian path trajectory reconstruction.
when_to_use_negative:
- The input matrix has not been GO-subset and dispersion-filtered — raw noisy expression
  inflates KL divergence between same-type cells
- The goal is clustering/cell-type assignment rather than ordering cells along a linear
  or branching developmental trajectory
- Pseudotime results perform poorly (per the article, downstream approaches that skip
  cell-type labeling may fail in this case)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: redPATH
  role: Provides the KL-divergence edge-weight computation and the redpath() driver
    that turns those edge weights into Hamiltonian path solutions
  repo: https://github.com/tinglabs/redPATH
- name: Rcpp
  role: C++ backend dependency enabling the fast KL divergence distance computation
    in the redPATH package
  repo: https://github.com/tinglabs/redPATH
- name: RcppArmadillo
  role: Linear algebra backend for the Rcpp-implemented KL distance and Hamiltonian
    path routines
  repo: https://github.com/tinglabs/redPATH
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:ed4a062a45b1585268b0972fee4fe27d4c659c4fba7a75dad1d5daadd2a84919
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglabs/redPATH
  tools:
  - Rcpp
  - RcppArmadillo
  - doParallel
  - mclust
  - redPATH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# kl-divergence-cell-distance-computation

## Summary

Compute pairwise cell-to-cell edge weights for redPATH's Hamiltonian path formulation using the symmetric Kullback-Leibler divergence, implemented in C++ via Rcpp/RcppArmadillo. This probabilistic distance matrix is the input edge weighting from which multiple noisy Hamiltonian path solutions are derived.

## When to use

Use when a single-cell expression matrix (e.g. GO-subset, dispersion-filtered) must be converted into cell-to-cell edge weights for redPATH's cHMT pseudotime or Hamiltonian path trajectory reconstruction.

## When NOT to use

- The input matrix has not been GO-subset and dispersion-filtered — raw noisy expression inflates KL divergence between same-type cells
- The goal is clustering/cell-type assignment rather than ordering cells along a linear or branching developmental trajectory
- Pseudotime results perform poorly (per the article, downstream approaches that skip cell-type labeling may fail in this case)

## Inputs

- m genes x n cells expression matrix (rows = genes, columns = cells), e.g. llorens_exprs (145 neural stem cells)
- GO-subset, dispersion-filtered expression matrix (get_GO_exp + filter_exp with dispersion_threshold = 10)

## Outputs

- Symmetric KL-divergence-based cell-to-cell edge weight (distance) matrix consumed by the Hamiltonian path heuristic
- redpath_pseudotime: N-by-1 per-cell pseudotime vector (length = number of input cells, e.g. 145)
- norm_res_new: multiple Hamiltonian path solutions saved to the global environment for transition-matrix trajectory visualization

## How to apply

First prepare the expression matrix by GO-based gene subsetting (get_GO_exp over Cell Development/Morphology/Differentiation/Fate/Maturation terms) and dispersion filtering (filter_exp with dispersion_threshold = 10). Then invoke the redpath() call (e.g. threadnum = 4, base_path_range = c(4:8)), which internally computes the asymmetric cell-to-cell edge weights via the symmetric Kullback-Leibler divergence implemented in Rcpp/RcppArmadillo. The KL edge weights feed the spectral-style clustering and the modified arbitrary-insertion Hamiltonian path heuristic (hamiltonian_path_as / hamiltonian_path_force) run at each k in the base path range. The divergence is symmetric so edge weights can be compared consistently, while the path formulation treats the distances asymmetrically when ordering cells. Note the redpath() run also deposits norm_res_new (the multiple Hamiltonian path solutions) in the global environment for downstream trajectory visualization.

## Related tools

- **redPATH** (Provides the KL-divergence edge-weight computation and the redpath() driver that turns those edge weights into Hamiltonian path solutions) — https://github.com/tinglabs/redPATH
- **Rcpp** (C++ backend dependency enabling the fast KL divergence distance computation in the redPATH package) — https://github.com/tinglabs/redPATH
- **RcppArmadillo** (Linear algebra backend for the Rcpp-implemented KL distance and Hamiltonian path routines) — https://github.com/tinglabs/redPATH

## Examples

```
redpath_pseudotime <- redpath(filtered_GO_data, threadnum = 4, base_path_range = c(4:8))
```

## Evaluation signals

- Output pseudotime vector length equals the number of input cells (e.g. 145 for llorens_exprs, 252 for MGH107)
- Cell-type labels (e.g. llorens_labels) progress monotonically along the inferred pseudotime ordering
- The distance matrix is N x N and symmetric (symmetric KL divergence), with finite non-negative values
- norm_res_new appears in the global environment after redpath() runs, containing multiple Hamiltonian path solutions
- Downstream calc_correlation (dcor/mic) on the full expression matrix against redpath_pseudotime recovers significant genes at threshold = 0.5

## Limitations

- This approach may fail when the pseudotime results perform poorly, since cell type labeling is not required
- Performance depends on prior GO subsetting and dispersion filtering; unfiltered cycling or low-dispersion genes degrade the KL distances
- Hamiltonian path solutions are noisy and must be consensus-merged (cHMT with base_path_range = c(4:8)) to yield a stable pseudotime

## Evidence

- [methods] asymmetric edge weights via symmetric KL: "we developed a computational method, called redPATH, to reconstruct the pseudo developmental time of cell lineages using a consensus asymmetric Hamiltonian path algorithm"
- [readme] Hamiltonian path formulation: "It formulates the problem of pseudo temporal ordering into a Hamiltonian path problem and attempts to recover the pseudo development time in single-cell RNA-seq datasets."
- [readme] preprocessing thresholds: "filtered_GO_data <- filter_exp(subset_GO_data, dispersion_threshold = 10, threads = 2)"
- [readme] driver invocation parameters: "redpath_pseudotime <- redpath(filtered_GO_data, threadnum = 4, base_path_range = c(4:8))"
