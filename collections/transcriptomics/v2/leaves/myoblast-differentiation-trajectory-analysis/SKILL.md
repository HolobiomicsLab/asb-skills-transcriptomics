---
name: myoblast-differentiation-trajectory-analysis
description: Use when benchmarking trajectory inference methods on the Trapnell HSMM
  myoblast dataset, or when assessing pseudotime accuracy via correlations with differentiation
  marker genes (ENO3, MEF2C, MYH2) as in STREAM Fig. 5c-d.
when_to_use_negative:
- The dataset is scATAC-seq or another epigenomic modality where marker-gene expression
  correlations are not defined; use STREAM's ATAC workflow (peaks/k-mers/motifs features)
  instead.
- The trajectory topology is not a bifurcation (e.g., multifurcation, cyclic, or disconnected
  graphs), since restricting to 'the correctly inferred bifurcation' is part of this
  protocol.
- No marker genes with known differentiation ordering are available for the system,
  so pseudotime accuracy cannot be assessed by correlation.
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: STREAM
  role: Primary trajectory inference method; reconstructs the HSMM myoblast bifurcation
    and provides pseudotime stored in anndata objects
  repo: https://github.com/pinellolab/STREAM
- name: ElPiGraph
  role: Core elastic principal graph algorithm underlying STREAM's trajectory learning
  repo: sysbio-curie/ElPiGraph.R
- name: Monocle2
  role: Comparison trajectory inference method benchmarked against STREAM on the HSMM
    data
- name: anndata
  role: Data class in which STREAM is written and in which each method's inferred
    trajectory and pseudotime are stored
  repo: http://anndata.rtfd.io
- name: Python 3
  role: Runtime for computing Pearson/Spearman/Kendall correlations with scipy.stats
    and assembling the correlation table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:c05c7e491e47d4953691873e4349d9927a09d592673215ed178049db2e237c6d
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
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# myoblast-differentiation-trajectory-analysis

## Summary

Reproduces the STREAM paper's myoblast differentiation (HSMM) trajectory benchmark: infer trajectories on the human skeletal muscle myoblast scRNA-seq dataset with STREAM/ElPiGraph and comparison methods such as Monocle2, extract rank- and distance-based pseudotime, and quantify pseudotime accuracy by correlating with marker genes ENO3, MEF2C, and MYH2.

## When to use

Use when benchmarking trajectory inference methods on the Trapnell HSMM myoblast dataset, or when assessing pseudotime accuracy via correlations with differentiation marker genes (ENO3, MEF2C, MYH2) as in STREAM Fig. 5c-d.

## When NOT to use

- The dataset is scATAC-seq or another epigenomic modality where marker-gene expression correlations are not defined; use STREAM's ATAC workflow (peaks/k-mers/motifs features) instead.
- The trajectory topology is not a bifurcation (e.g., multifurcation, cyclic, or disconnected graphs), since restricting to 'the correctly inferred bifurcation' is part of this protocol.
- No marker genes with known differentiation ordering are available for the system, so pseudotime accuracy cannot be assessed by correlation.

## Inputs

- HSMM human skeletal muscle myoblast scRNA-seq expression matrix (the dataset from the STREAM bifurcation tutorial)
- Inferred trajectories from bifurcation-capable methods (STREAM/ElPiGraph, Monocle2) stored as anndata objects
- Marker gene expression values for ENO3, MEF2C, and MYH2 per cell

## Outputs

- correlations_hsmm_markers.csv: correlation values per method, per pseudotime type (rank-based and distance-based), and per marker (ENO3, MEF2C, MYH2, plus combined three-gene average rank)
- Table reproducing Fig. 5c-d bar/point values of the STREAM paper

## How to apply

Load the HSMM human skeletal muscle myoblast scRNA-seq dataset used in the STREAM bifurcation tutorial (1.1-STREAM_scRNA-seq (Bifurcation).ipynb) and restrict cells to the correctly inferred bifurcation. Run STREAM (ElPiGraph-based) and eligible comparison methods such as Monocle2, storing each method's output in anndata objects, and extract both the rank-based pseudotime and the distance-based pseudotime. Compute Pearson, Spearman, and Kendall correlation coefficients (scipy.stats pearsonr/spearmanr/kendalltau) between each pseudotime variant and expression of ENO3, MEF2C, and MYH2 in Python 3. Additionally correlate each pseudotime with the average of the three marker genes' ranks (combined three-gene average rank). Assemble correlations per method, per pseudotime type, and per marker into a table reproducing Fig. 5c-d and write to correlations_hsmm_markers.csv; STREAM should show the best overall performance by average rank of the four correlation metrics.

## Related tools

- **STREAM** (Primary trajectory inference method; reconstructs the HSMM myoblast bifurcation and provides pseudotime stored in anndata objects) — https://github.com/pinellolab/STREAM
- **ElPiGraph** (Core elastic principal graph algorithm underlying STREAM's trajectory learning) — sysbio-curie/ElPiGraph.R
- **Monocle2** (Comparison trajectory inference method benchmarked against STREAM on the HSMM data)
- **anndata** (Data class in which STREAM is written and in which each method's inferred trajectory and pseudotime are stored) — http://anndata.rtfd.io
- **Python 3** (Runtime for computing Pearson/Spearman/Kendall correlations with scipy.stats and assembling the correlation table)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter && conda activate env_stream; python -c "from scipy.stats import pearsonr, spearmanr, kendalltau; r=spearmanr(pseudotime, marker_expr)"
```

## Evaluation signals

- Correlation table values qualitatively match the bar/point values reported in Fig. 5c-d of the STREAM paper.
- Rank- and distance-based pseudotime correlations for each marker gene (ENO3, MEF2C, MYH2) are present for every bifurcation-capable method.
- Both rank-based and distance-based pseudotime variants are extracted from each method's anndata output and evaluated separately.
- STREAM attains the highest average correlation coefficient for ENO3 across the four metrics and the best overall average rank when combining all three markers.
- Output file correlations_hsmm_markers.csv exists with rows keyed by method, pseudotime type, and marker, using scipy.stats Pearson/Spearman/Kendall values.

## Limitations

- The workflow assumes the trajectory is a bifurcation and restricts to cells belonging to the correctly inferred branch, so results do not generalize to more complex topologies.
- STREAM's ElPiGraph core supports complex graphs, but 'more work is necessary to provide intuitive visualizations' for sophisticated topologies.
- STREAM is not adapted to cyclic or disconnected graph topologies, limiting applicability beyond tree-like differentiation paths.
- Benchmark conclusions are qualitative (correlation/rank agreement with Fig. 5c-d), not an independent statistical validation.

## Evidence

- [results] STREAM achieves the best overall pseudotime accuracy on the HSMM dataset: it has the highest average correlation coefficient for ENO3 across the four metrics: [evidence span withheld — anchor status: not_found]
- [methods] STREAM is written using the class anndata: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [abstract] STREAM is an interactive pipeline capable of disentangling and visualizing complex branching trajectories: "Here we present STREAM, an interactive pipeline capable of disentangling and visualizing complex branching trajectories from both single-cell transcriptomic and epigenomic data."
- [readme] README tutorial reference for the bifurcation scRNA-seq example: "Example for scRNA-seq: [1.1-STREAM_scRNA-seq (Bifurcation).ipynb]"
- [discussion] Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary: "Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations"
