---
name: pseudotime-marker-gene-correlation-analysis
description: Use when comparing trajectory inference methods on the HSMM scRNA-seq
  dataset, or when quantifying pseudotime accuracy via Pearson/Spearman/Kendall correlation
  against known marker genes such as ENO3, MEF2C, MYH2.
when_to_use_negative:
- The dataset is single-cell chromatin accessibility (scATAC-seq) data where marker-gene
  expression correlations are not directly defined
- The trajectory graph is cyclic, disconnected, or a complex multifurcation where
  restricting to a single correctly inferred bifurcation is not possible — the article
  notes further work is needed to adapt STREAM to such general structures
- You have no prior marker genes to serve as ground truth for pseudotime accuracy
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: STREAM
  role: Primary trajectory inference method under benchmark; run via the bifurcation
    tutorial notebook to obtain pseudotime on the HSMM dataset
  repo: https://github.com/pinellolab/STREAM
- name: ElPiGraph
  role: Core elastic principal graph algorithm behind STREAM, used to learn the trajectory
    graph from which pseudotime is extracted
  repo: sysbio-curie/ElPiGraph.R
- name: Monocle2
  role: Comparison trajectory inference method whose pseudotime is also correlated
    with the marker genes
- name: anndata
  role: Class used to store each method's inferred trajectory and pseudotime outputs
    for extraction
- name: Python 3
  role: Runtime for the correlation computation (scipy.stats pearsonr/spearmanr/kendalltau)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:a6a6c4c712ccbdc632c7cb4d8f33b291fe1adeefe8afada077923234b5517832
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pseudotime-marker-gene-correlation-analysis

## Summary

A benchmarking skill that scores trajectory inference methods (STREAM/ElPiGraph, Monocle2, etc.) on the HSMM myoblast dataset by correlating each method's rank- and distance-based pseudotime with differentiation marker genes ENO3, MEF2C, and MYH2, reproducing Fig. 5c-d of the STREAM paper.

## When to use

Use when comparing trajectory inference methods on the HSMM scRNA-seq dataset, or when quantifying pseudotime accuracy via Pearson/Spearman/Kendall correlation against known marker genes such as ENO3, MEF2C, MYH2.

## When NOT to use

- The dataset is single-cell chromatin accessibility (scATAC-seq) data where marker-gene expression correlations are not directly defined
- The trajectory graph is cyclic, disconnected, or a complex multifurcation where restricting to a single correctly inferred bifurcation is not possible — the article notes further work is needed to adapt STREAM to such general structures
- You have no prior marker genes to serve as ground truth for pseudotime accuracy

## Inputs

- HSMM human skeletal muscle myoblast scRNA-seq dataset (from STREAM bifurcation tutorial)
- anndata objects containing each method's inferred trajectory with rank-based and distance-based pseudotime
- Marker gene expression matrix for ENO3, MEF2C, MYH2

## Outputs

- correlations_hsmm_markers.csv — correlation table per method, per pseudotime type (rank/distance), per marker gene (ENO3, MEF2C, MYH2) and combined three-gene average rank
- Pearson, Spearman, and Kendall correlation coefficients per method/marker/pseudotime variant

## How to apply

Load the HSMM human skeletal muscle myoblast scRNA-seq dataset from the STREAM bifurcation tutorial and restrict cells to the correctly inferred bifurcation. Run each bifurcation-capable method (STREAM/ElPiGraph, Monocle2, etc.) and extract both rank-based and distance-based pseudotime from the resulting anndata objects. Using scipy.stats (pearsonr, spearmanr, kendalltau) in Python 3, compute the three correlation coefficients between each pseudotime variant and each of the three marker genes, plus the correlation with the combined average rank of all three markers. Assemble the per-method, per-pseudotime-type, per-marker correlations into a table written to correlations_hsmm_markers.csv. The finding to validate against: STREAM achieves the best overall pseudotime accuracy, with the highest average ENO3 correlation across the four metrics and the best average rank across all three markers (Fig. 5c-d).

## Related tools

- **STREAM** (Primary trajectory inference method under benchmark; run via the bifurcation tutorial notebook to obtain pseudotime on the HSMM dataset) — https://github.com/pinellolab/STREAM
- **ElPiGraph** (Core elastic principal graph algorithm behind STREAM, used to learn the trajectory graph from which pseudotime is extracted) — sysbio-curie/ElPiGraph.R
- **Monocle2** (Comparison trajectory inference method whose pseudotime is also correlated with the marker genes)
- **anndata** (Class used to store each method's inferred trajectory and pseudotime outputs for extraction)
- **Python 3** (Runtime for the correlation computation (scipy.stats pearsonr/spearmanr/kendalltau))

## Examples

```
from scipy.stats import pearsonr, spearmanr, kendalltau; r, rho, tau = pearsonr(pseudotime, adata[:, 'ENO3'].X), spearmanr(pseudotime, adata[:, 'ENO3'].X), kendalltau(pseudotime, adata[:, 'ENO3'].X)
```

## Evaluation signals

- correlations_hsmm_markers.csv contains rank- and distance-based pseudotime correlations for every marker gene (ENO3, MEF2C, MYH2) for every bifurcation-capable method
- Correlation table values qualitatively match the bar/point values reported in Fig. 5c-d of the STREAM paper
- STREAM shows the highest average correlation coefficient for ENO3 across the four metrics
- STREAM has the best overall performance measured by the average rank of the four correlation metrics when combining all three marker genes
- All three correlation coefficients (Pearson, Spearman, Kendall) are computed in Python 3 with scipy.stats for each pseudotime variant

## Limitations

- STREAM does not yet support cyclic or disconnected graph topologies; further improvement is necessary
- Scalability to millions of cells is not established for STREAM and many other trajectory inference methods
- Comparison is restricted to methods capable of bifurcation inference; conclusions apply to the HSMM bifurcation and markers used

## Evidence

- [results] STREAM achieves the best overall pseudotime accuracy on the HSMM dataset: [evidence span withheld — anchor status: not_found]
- [methods] STREAM written using anndata class: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [abstract] STREAM applied to myoblast differentiation: "We further demonstrate its utility for understanding myoblast differentiation and disentangling known heterogeneity in hematopoiesis for different organisms."
- [discussion] ElPiGraph is the core algorithm behind STREAM: "Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations"
- [readme] Bifurcation tutorial notebook for scRNA-seq: "Example for scRNA-seq: [1.1-STREAM_scRNA-seq (Bifurcation).ipynb]"
