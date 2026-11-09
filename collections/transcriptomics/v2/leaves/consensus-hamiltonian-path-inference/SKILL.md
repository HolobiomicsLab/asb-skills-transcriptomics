---
name: consensus-hamiltonian-path-inference
description: Use when single-cell RNA-seq gene-by-cell expression data must be ordered
  along a linear pseudo-developmental time axis without cell type labels, e.g. neural
  stem cell differentiation.
when_to_use_negative:
- The developmental trajectory is expected to be branched rather than linear — use
  the transition-matrix PCA trajectory-detection approach instead of a single cHMT
  ordering
- Cell type labels are available and a supervised ordering would suffice, or the input
  is not a raw gene-by-cell scRNA-seq matrix
- The dataset requires removing cycling genes via traditional cell-cycle-effect removal
  — redPATH instead relies on GO subsetting and dispersion filtering
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: redPATH
  role: 'R package implementing the cHMT algorithm: KL divergence edge weights (Rcpp/RcppArmadillo),
    hamiltonian_path_as / hamiltonian_path_force heuristics, and the redpath() consensus
    merge'
  repo: https://github.com/tinglabs/redPATH
- name: doParallel
  role: parallel backend for threadnum = 4 execution of the Hamiltonian path heuristics
    and filtering
- name: Rcpp
  role: C++ backend for the symmetric Kullback-Leibler divergence computation
- name: RcppArmadillo
  role: linear algebra backend for the C++ KL-divergence distance calculations
- name: mclust
  role: spectral-style clustering of the distance matrix over the base path range
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:76383b75ba4cd1973ed44cdee9cf9f77fa67c1cf10091223c8edf9bbbb21d4c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglabs/redPATH
  tools:
  - Rcpp
  - RcppArmadillo
  - doParallel
  - ggplot2
  - mclust
  - plotly
  - redPATH
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# consensus-hamiltonian-path-inference

## Summary

redPATH's consensus Hamiltonian path (cHMT) algorithm reconstructs pseudotime for linear developmental trajectories by merging multiple noisy asymmetric Hamiltonian path solutions over a base path range, using symmetric KL divergence cell-to-cell distances computed via Rcpp.

## When to use

Use when single-cell RNA-seq gene-by-cell expression data must be ordered along a linear pseudo-developmental time axis without cell type labels, e.g. neural stem cell differentiation.

## When NOT to use

- The developmental trajectory is expected to be branched rather than linear — use the transition-matrix PCA trajectory-detection approach instead of a single cHMT ordering
- Cell type labels are available and a supervised ordering would suffice, or the input is not a raw gene-by-cell scRNA-seq matrix
- The dataset requires removing cycling genes via traditional cell-cycle-effect removal — redPATH instead relies on GO subsetting and dispersion filtering

## Inputs

- gene-by-cell scRNA-seq expression matrix (m genes rows by n cells columns), e.g. llorens_exprs (145 neural stem cells)
- GO-subset expression matrix from get_GO_exp
- dispersion-filtered matrix from filter_exp (dispersion_threshold = 10)
- optional reCAT mean_scores (e.g. llorens_reCAT_scores) for G0-like cell removal via kmeans_clustering and statistical_test

## Outputs

- redpath_pseudotime: N-by-one pseudotime vector covering all input cells
- norm_res_new: multiple normalized Hamiltonian path solutions saved to the global environment, reusable for transition-matrix PCA trajectory visualization
- optional CSV export of per-cell pseudotime ordering (redpath_pseudotime.csv)

## How to apply

First preprocess the m-genes-by-n-cells expression matrix by subsetting to Gene Ontology genes related to Cell Development, Cell Morphology, Cell Differentiation, Cell Fate and Cell Maturation with get_GO_exp (takelog = F), then filter with filter_exp using dispersion_threshold = 10. Compute asymmetric cell-to-cell edge weights with the redPATH symmetric Kullback-Leibler divergence function (backed by Rcpp/RcppArmadillo), then run the modified arbitrary-insertion Hamiltonian path heuristic (hamiltonian_path_as / hamiltonian_path_force) on spectral-style clustered solutions with threadnum = 4. Call redpath(filtered_GO_data, threadnum = 4, base_path_range = c(4:8)), where base_path_range spans the estimated number of cell types k0 to k0 + 4, letting the algorithm infer a reference path from k = 3 to 7, orient each path by pairwise correlation of the path versus its reverse, and merge all subsequent paths onto the reference. The output redpath_pseudotime is an N-by-one vector (e.g. 145 for the Llorens-Bobadilla neural stem cell dataset) with cell-type labels progressing monotonically along inferred development time; the multiple solutions are also stored in norm_res_new for downstream transition-matrix trajectory visualization.

## Related tools

- **redPATH** (R package implementing the cHMT algorithm: KL divergence edge weights (Rcpp/RcppArmadillo), hamiltonian_path_as / hamiltonian_path_force heuristics, and the redpath() consensus merge) — https://github.com/tinglabs/redPATH
- **doParallel** (parallel backend for threadnum = 4 execution of the Hamiltonian path heuristics and filtering)
- **Rcpp** (C++ backend for the symmetric Kullback-Leibler divergence computation)
- **RcppArmadillo** (linear algebra backend for the C++ KL-divergence distance calculations)
- **mclust** (spectral-style clustering of the distance matrix over the base path range)

## Examples

```
redpath_pseudotime <- redpath(filtered_GO_data, threadnum = 4, base_path_range = c(4:8))
```

## Evaluation signals

- Returned pseudotime is an N-by-one vector whose length equals the number of input cells (e.g. 145 for llorens_exprs)
- Cell-type labels progress monotonically along the inferred pseudotime ordering
- base_path_range was set to the estimated number of cell types k0 through k0 + 4 (e.g. c(4:8))
- norm_res_new exists in the global environment after the redpath() call, confirming multiple Hamiltonian path solutions were generated
- G0-like cell removal, if applied, reported a possible group via statistical_test with threshold = 0.001 (e.g. 'Possible group of G0-like cells is: 2')

## Limitations

- The reference-path merging approach may fail when the pseudotime results perform poorly
- base_path_range must be informed by a reasonable estimate of the number of cell types k0
- Redundant compilation caveats on Apple Silicon Macs: the TBB library path must be set via PKG_LIBS/PKG_CPPFLAGS before installation

## Evidence

- [abstract] we developed a computational method, called redPATH, to reconstruct the pseudo developmental time of cell lineages using a consensus asymmetric Hamiltonian path algorithm: "we developed a computational method, called redPATH, to reconstruct the pseudo developmental time of cell lineages using a consensus asymmetric Hamiltonian path algorithm"
- [discussion] The cHMT merges path solutions by inferring a reference path from k=3 to 7, determining path direction via pairwise correlation of each path and its reverse, then merging all subsequent paths onto the reference to yield the final pseudotime.: [evidence span withheld — anchor status: not_found]
- [readme] redpath_pseudotime <- redpath(filtered_GO_data, threadnum = 4, base_path_range = c(4:8)) # base_path_range is set to the estimated number of cell types, k0, and k0 +4.: "redpath_pseudotime <- redpath(filtered_GO_data, threadnum = 4, base_path_range = c(4:8)) # base_path_range is set to the estimated number of cell types, k0, and k0 +4."
- [methods] filtered_GO_data <- filter_exp(subset_GO_data, dispersion_threshold = 10, threads = 2): "filtered_GO_data <- filter_exp(subset_GO_data, dispersion_threshold = 10, threads = 2)"
- [discussion] Although the advantage is that cell type labeling is not required here, this approach may fail when the pseudotime results perform poorly.: "Although the advantage is that cell type labeling is not required here, this approach may fail when the pseudotime results perform poorly."
