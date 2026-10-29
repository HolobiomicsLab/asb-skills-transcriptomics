---
name: rank-based-vs-distance-based-pseudotime-comparison
description: Use when multiple bifurcation-capable trajectory inference methods (e.g.,
  STREAM/ElPiGraph, Monocle2) have been run on a branching single-cell dataset and
  you need an objective, marker-gene-anchored measure of pseudotime accuracy to rank
  the methods.
when_to_use_negative:
- Data with cyclic or disconnected graph topologies — the paper notes further improvement
  is necessary for STREAM to properly describe these more general structures, so bifurcation-anchored
  pseudotime accuracy is not meaningful
- When no prior-knowledge marker genes are available to anchor pseudotime evaluation,
  since the whole comparison depends on correlating with ENO3/MEF2C/MYH2-like markers
- When only one trajectory method was run — the skill is a cross-method accuracy comparison,
  not a single-method pseudotime computation
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: STREAM
  role: Bifurcation-capable method under evaluation; supplies rank-based and distance-based
    pseudotime on the HSMM data and achieved the best overall performance in the paper
  repo: https://github.com/pinellolab/STREAM
- name: Monocle2
  role: Comparison trajectory inference method whose pseudotime output is benchmarked
    against STREAM's on the HSMM bifurcation
- name: ElPiGraph
  role: Core elastic principal-graph algorithm underlying STREAM's trajectory learning
    on the HSMM data
  repo: sysbio-curie/ElPiGraph.R
- name: anndata
  role: Data class in which STREAM stores trajectory results; pseudotime variants
    are extracted from anndata objects for correlation computation
  repo: http://anndata.rtfd.io
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:8bb6434b1a28c1d82ef71c964a0f952cfe925444ed1d448753d833a660987a9c
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# rank-based-vs-distance-based-pseudotime-comparison

## Summary

Benchmarks trajectory-inference methods on the Trapnell HSMM myoblast dataset by correlating each method's rank-based and distance-based pseudotime with myogenic marker genes (ENO3, MEF2C, MYH2), using Pearson/Spearman/Kendall coefficients to reproduce the STREAM paper's Fig. 5c-d accuracy comparison.

## When to use

Use when multiple bifurcation-capable trajectory inference methods (e.g., STREAM/ElPiGraph, Monocle2) have been run on a branching single-cell dataset and you need an objective, marker-gene-anchored measure of pseudotime accuracy to rank the methods.

## When NOT to use

- Data with cyclic or disconnected graph topologies — the paper notes further improvement is necessary for STREAM to properly describe these more general structures, so bifurcation-anchored pseudotime accuracy is not meaningful
- When no prior-knowledge marker genes are available to anchor pseudotime evaluation, since the whole comparison depends on correlating with ENO3/MEF2C/MYH2-like markers
- When only one trajectory method was run — the skill is a cross-method accuracy comparison, not a single-method pseudotime computation

## Inputs

- HSMM human skeletal muscle myoblast scRNA-seq dataset (STREAM bifurcation tutorial data)
- anndata objects holding each method's inferred trajectory with rank-based and distance-based pseudotime per cell
- Expression matrix for marker genes ENO3, MEF2C, MYH2
- Cell labels restricting analysis to the correctly inferred bifurcation

## Outputs

- correlations_hsmm_markers.csv — table of Pearson/Spearman/Kendall correlations per method, pseudotime type, and marker gene
- Combined three-gene average-rank correlation per method
- Average rank of the four correlation metrics per method (Fig. 5c-d analogue)

## How to apply

Load the HSMM scRNA-seq dataset from the STREAM bifurcation tutorial and restrict cells to the correctly inferred bifurcation. Run each candidate method (STREAM, which uses ElPiGraph as its core algorithm, plus comparison methods such as Monocle2) and extract both pseudotime variants stored in the anndata output objects: the rank-based pseudotime and the distance-based pseudotime. For each method and each pseudotime variant, compute Pearson, Spearman, and Kendall correlation coefficients against the expression of the three myogenic markers ENO3, MEF2C, and MYH2 with scipy.stats in Python 3. Also correlate each pseudotime with the average rank of the three markers combined. Assemble method × pseudotime-type × marker correlations into a table (correlations_hsmm_markers.csv) reproducing Fig. 5c-d; average rank of the four correlation metrics across all three genes identifies the best method, which the paper reports as STREAM.

## Related tools

- **STREAM** (Bifurcation-capable method under evaluation; supplies rank-based and distance-based pseudotime on the HSMM data and achieved the best overall performance in the paper) — https://github.com/pinellolab/STREAM
- **Monocle2** (Comparison trajectory inference method whose pseudotime output is benchmarked against STREAM's on the HSMM bifurcation)
- **ElPiGraph** (Core elastic principal-graph algorithm underlying STREAM's trajectory learning on the HSMM data) — sysbio-curie/ElPiGraph.R
- **anndata** (Data class in which STREAM stores trajectory results; pseudotime variants are extracted from anndata objects for correlation computation) — http://anndata.rtfd.io

## Examples

```
from scipy.stats import pearsonr, spearmanr, kendalltau; r_p, _ = pearsonr(pseudotime_rank, expr_ENO3); r_s, _ = spearmanr(pseudotime_dist, expr_MYH2); r_k, _ = kendalltau(pseudotime_rank, expr_MEF2C)
```

## Evaluation signals

- The correlation table contains rank-based and distance-based pseudotime correlations for every marker gene (ENO3, MEF2C, MYH2) for every bifurcation-capable method
- Table values qualitatively match the bar/point values reported in Fig. 5c-d of the STREAM paper
- STREAM shows the highest average correlation coefficient for ENO3 across the four metrics, as reported in the paper
- All three coefficients (Pearson, Spearman, Kendall) are computed per pseudotime variant, not just one
- The combined three-gene average-rank correlation is present so methods can be ranked by average rank of the four correlation metrics

## Limitations

- STREAM's ElPiGraph core supports complex graphs but intuitive visualizations for sophisticated topologies are still lacking
- STREAM is not yet adapted to cyclic or disconnected graph topologies
- The evaluation depends on prior marker-gene knowledge (ENO3, MEF2C, MYH2) and on restricting to the correctly inferred bifurcation
- Many trajectory inference methods, including STREAM, may not scale to the millions of cells recent assays can profile

## Evidence

- [results] STREAM achieves best overall pseudotime accuracy on HSMM bifurcation: "We further demonstrate its utility for understanding myoblast differentiation and disentangling known heterogeneity in hematopoiesis"
- [methods] anndata storage of STREAM results: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [discussion] Monocle2 and STREAM as the few bifurcation-capable methods compared: "so far very few trajectory inference methods (Monocle2 and STREAM) have been applied to analyze single-cell chromatin accessibility"
- [discussion] no prior knowledge required for trajectory reconstruction: "Our method does not require prior knowledge such as time points, start cell, or the number of branching events to reconstruct trajectories"
- [readme] environment setup for running the analysis in Python 3: "$ conda create -n env_stream python=3.7 stream=1.0 jupyter"
