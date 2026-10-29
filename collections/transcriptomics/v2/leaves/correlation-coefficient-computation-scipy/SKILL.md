---
name: correlation-coefficient-computation-scipy
description: Use when benchmarking pseudotime accuracy of trajectory methods (e.g.,
  STREAM/ElPiGraph vs Monocle2) by correlating inferred pseudotime with known marker
  gene expression from scRNA-seq data in anndata objects.
when_to_use_negative:
- The dataset has no biologically meaningful marker genes with a known monotonic relationship
  to the trajectory, since marker-gene correlation cannot validate pseudotime ordering.
- The trajectory is cyclic or disconnected, where pseudotime ordering against a single
  lineage's markers is ill-defined (STREAM itself notes further work is needed for
  such topologies).
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: STREAM
  role: Produces the inferred bifurcation trajectory and rank-/distance-based pseudotime
    on the HSMM dataset that the correlations are computed against
  repo: https://github.com/pinellolab/STREAM
- name: ElPiGraph
  role: Core elastic principal graph algorithm behind STREAM used for trajectory inference
  repo: sysbio-curie/ElPiGraph.R
- name: Monocle2
  role: Comparison trajectory inference method whose pseudotime is correlated with
    the same marker genes
- name: anndata
  role: Container class in which STREAM stores trajectory outputs (pseudotime) from
    which vectors are extracted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:fc5b3c7ba673349cb1c9fe563bbefde11decbf57b109eb8d2654aac630ef75e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pinellolab/STREAM
  tools:
  - ElPiGraph
  - Monocle2
  - Python 3
  - STREAM
  - anndata
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# correlation-coefficient-computation-scipy

## Summary

Compute Pearson, Spearman and Kendall correlation coefficients between inferred pseudotime variants (rank-based and distance-based) and myogenic marker gene expression (ENO3, MEF2C, MYH2) to quantitatively benchmark trajectory inference methods on the HSMM myoblast dataset.

## When to use

Use when benchmarking pseudotime accuracy of trajectory methods (e.g., STREAM/ElPiGraph vs Monocle2) by correlating inferred pseudotime with known marker gene expression from scRNA-seq data in anndata objects.

## When NOT to use

- The dataset has no biologically meaningful marker genes with a known monotonic relationship to the trajectory, since marker-gene correlation cannot validate pseudotime ordering.
- The trajectory is cyclic or disconnected, where pseudotime ordering against a single lineage's markers is ill-defined (STREAM itself notes further work is needed for such topologies).

## Inputs

- anndata objects with per-method inferred trajectories (rank-based and distance-based pseudotime per cell)
- HSMM human skeletal muscle myoblast scRNA-seq expression matrix restricted to cells in the correctly inferred bifurcation
- Marker gene expression vectors for ENO3, MEF2C and MYH2

## Outputs

- Table of Pearson/Spearman/Kendall correlations per method, pseudotime type and marker gene (correlations_hsmm_markers.csv)
- Average rank of the four correlation metrics per method across ENO3, MYH2 and MEF2C (combined three-gene average rank)

## How to apply

Extract both rank-based and distance-based pseudotime vectors from each method's output stored in anndata objects, restricted to cells in the correctly inferred bifurcation of the HSMM human skeletal muscle myoblast dataset. For each pseudotime variant, compute Pearson, Spearman (scipy.stats.spearmanr) and Kendall (scipy.stats.kendalltau) correlations against the expression of ENO3, MEF2C and MYH2, plus the correlation with the average of the three genes' ranks. The rationale is that myogenic marker genes provide an independent ordering of differentiation state, so higher correlation indicates more accurate pseudotime, as reported in Fig. 5c-d of the STREAM paper. Assemble per-method, per-pseudotime-type, per-marker correlations into a table (correlations_hsmm_markers.csv) and compute average ranks across the four correlation metrics to compare methods.

## Related tools

- **STREAM** (Produces the inferred bifurcation trajectory and rank-/distance-based pseudotime on the HSMM dataset that the correlations are computed against) — https://github.com/pinellolab/STREAM
- **ElPiGraph** (Core elastic principal graph algorithm behind STREAM used for trajectory inference) — sysbio-curie/ElPiGraph.R
- **Monocle2** (Comparison trajectory inference method whose pseudotime is correlated with the same marker genes)
- **anndata** (Container class in which STREAM stores trajectory outputs (pseudotime) from which vectors are extracted)

## Examples

```
from scipy.stats import pearsonr, spearmanr, kendalltau; r_s, p_s = spearmanr(pseudotime, marker_expr); r_k, p_k = kendalltau(pseudotime, marker_expr)
```

## Evaluation signals

- Correlation values in correlations_hsmm_markers.csv qualitatively match the bar/point values reported in Fig. 5c-d of the STREAM paper.
- Rank- and distance-based pseudotime correlations for each marker gene (ENO3, MEF2C, MYH2) are present for every bifurcation-capable method.
- Each correlation coefficient lies within the valid range [-1, 1], with three coefficients (Pearson, Spearman, Kendall) per method/pseudotime/marker combination.
- STREAM shows the highest average correlation for ENO3 across the four metrics, consistent with the paper's finding of best overall performance by average rank.

## Limitations

- Correlation is only qualitative validation: results must match the paper's reported values qualitatively, not exactly, due to stochasticity and version differences in trajectory methods.
- STREAM does not adapt to cyclic or disconnected graph topologies, so this correlation-based validation is not meaningful outside tree/bifurcation structures.
- Correlations are only computed for cells within the correctly inferred bifurcation; misassignment of cells to branches invalidates the comparison.

## Evidence

- [methods] STREAM stores trajectory output in anndata: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [readme] STREAM disentangles branching trajectories from scRNA-seq data: "STREAM (**S**ingle-cell **T**rajectories **R**econstruction, **E**xploration **A**nd **M**apping) is an interactive pipeline capable of disentangling and visualizing complex branching trajectories"
