---
name: scrnaseq-bifurcation-trajectory-benchmarking
description: Use when comparing accuracy of pseudotime inferred by bifurcation-capable
  trajectory methods on a single-cell RNA-seq differentiation dataset, using marker
  genes as an external pseudotime reference (e.g. Pearson/Spearman/Kendall correlation
  with ENO3, MEF2C, MYH2).
when_to_use_negative:
- The trajectory structure is cyclic or disconnected — STREAM/ElPiGraph does not properly
  describe such general structures and the benchmark's bifurcation assumption does
  not apply
- Cells span a non-differentiation process with no known marker genes along pseudotime
  — correlation-to-marker validation has no reference
- Data are scATAC-seq or other epigenomic data without gene-level marker expression
  — the marker-gene correlation protocol is defined for transcriptomic features
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STREAM
  role: Primary bifurcation-capable trajectory inference method whose rank- and distance-based
    pseudotime is evaluated; also supplies the HSMM bifurcation tutorial dataset
  repo: https://github.com/pinellolab/STREAM
- name: Monocle2
  role: Comparison trajectory inference method run on the HSMM data for pseudotime
    accuracy ranking
- name: ElPiGraph
  role: Core elastic principal graph algorithm behind STREAM used to learn the bifurcation
    graph
  repo: https://github.com/pinellolab/STREAM
- name: anndata
  role: Object class in which each method's trajectory output and pseudotime variants
    are stored and extracted
  repo: http://anndata.rtfd.io
- name: Python 3
  role: Environment (via Anaconda/conda) for computing scipy.stats correlations and
    assembling the CSV table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:e7559342a0a6843c7d8503344730e4e772607edc9a7fbe07999038566d1a082c
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
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# scrnaseq-bifurcation-trajectory-benchmarking

## Summary

Benchmarks trajectory inference methods (STREAM/ElPiGraph, Monocle2) on a bifurcating scRNA-seq dataset such as the Trapnell HSMM myoblast differentiation data by correlating each method's rank- and distance-based pseudotime with marker-gene expression (ENO3, MEF2C, MYH2), as used to validate STREAM in the Nature Communications 2019 paper.

## When to use

Use when comparing accuracy of pseudotime inferred by bifurcation-capable trajectory methods on a single-cell RNA-seq differentiation dataset, using marker genes as an external pseudotime reference (e.g. Pearson/Spearman/Kendall correlation with ENO3, MEF2C, MYH2).

## When NOT to use

- The trajectory structure is cyclic or disconnected — STREAM/ElPiGraph does not properly describe such general structures and the benchmark's bifurcation assumption does not apply
- Cells span a non-differentiation process with no known marker genes along pseudotime — correlation-to-marker validation has no reference
- Data are scATAC-seq or other epigenomic data without gene-level marker expression — the marker-gene correlation protocol is defined for transcriptomic features

## Inputs

- HSMM human skeletal muscle myoblast scRNA-seq count dataset (Trapnell data as used in the STREAM bifurcation tutorial)
- Per-method inferred trajectories stored in anndata objects with rank-based and distance-based pseudotime
- Expression vectors of marker genes ENO3, MEF2C, MYH2 across the bifurcation cells

## Outputs

- correlations_hsmm_markers.csv — correlation table per method, per pseudotime type (rank/distance), per marker (ENO3, MEF2C, MYH2) across Pearson, Spearman, Kendall metrics
- Per-method average rank of the four correlation metrics (reproducing Fig. 5c-d)

## How to apply

Load the HSMM human skeletal muscle myoblast scRNA-seq dataset used in the STREAM bifurcation tutorial and restrict cells to the correctly inferred bifurcation. Run each eligible trajectory inference method (STREAM, which is built on ElPiGraph, plus comparison methods such as Monocle2) and extract both rank-based and distance-based pseudotime from the resulting anndata objects. Compute Pearson, Spearman and Kendall correlations (scipy.stats pearsonr/spearmanr/kendalltau in Python 3) between each pseudotime variant and the expression of ENO3, MEF2C and MYH2, plus the correlation with the average of the three marker genes' ranks. Rationale: the paper validated STREAM this way and found it achieved the highest average ENO3 correlation and the best overall average rank of the four correlation metrics across all three markers (Fig. 5c-d), so method ranking is assessed by average rank of the four correlation metrics per method rather than by a single coefficient.

## Related tools

- **STREAM** (Primary bifurcation-capable trajectory inference method whose rank- and distance-based pseudotime is evaluated; also supplies the HSMM bifurcation tutorial dataset) — https://github.com/pinellolab/STREAM
- **Monocle2** (Comparison trajectory inference method run on the HSMM data for pseudotime accuracy ranking)
- **ElPiGraph** (Core elastic principal graph algorithm behind STREAM used to learn the bifurcation graph) — https://github.com/pinellolab/STREAM
- **anndata** (Object class in which each method's trajectory output and pseudotime variants are stored and extracted) — http://anndata.rtfd.io
- **Python 3** (Environment (via Anaconda/conda) for computing scipy.stats correlations and assembling the CSV table)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter && conda activate env_stream && jupyter notebook
```

## Evaluation signals

- correlations_hsmm_markers.csv contains rank- and distance-based pseudotime correlations for every marker gene (ENO3, MEF2C, MYH2) for every bifurcation-capable method
- Correlation values qualitatively match the bar/point values of Fig. 5c-d of the STREAM paper
- STREAM shows the highest average correlation for ENO3 across the four metrics and the best overall average rank when combining all three markers
- Correlations were computed with scipy.stats (pearsonr, spearmanr, kendalltau) in Python 3, giving three metric families per pseudotime variant
- All cells analysed belong to the correctly inferred bifurcation of the HSMM dataset

## Limitations

- Benchmark conclusions are tied to the bifurcation topology of the HSMM dataset; STREAM was noted to need further improvement for cyclic or disconnected graph topologies
- Comparison covers only bifurcation-capable methods; only a few methods (Monocle2 and STREAM) have been extended to other data modalities, limiting generality
- Many current trajectory inference methods do not scale to the millions of cells possible with recent single-cell assays

## Evidence

- [results] STREAM achieves best pseudotime accuracy on HSMM: [evidence span withheld — anchor status: not_found]
- [discussion] STREAM reconstructs trajectories without prior knowledge: "Our method does not require prior knowledge such as time points, start cell, or the number of branching events to reconstruct trajectories"
- [readme] STREAM is an anndata-based pipeline installable via conda: [evidence span withheld — anchor status: not_found]
- [abstract] STREAM validated on myoblast differentiation across organisms and technologies: "We further demonstrate its utility for understanding myoblast differentiation and disentangling known heterogeneity in hematopoiesis for different organisms."
- [discussion] ElPiGraph is the core algorithm of STREAM: "Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations"
