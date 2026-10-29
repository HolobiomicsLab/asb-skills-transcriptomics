---
name: pseudotime-gene-expression-correlation-analysis
description: Use when comparing trajectory-inference algorithms that each output a
  pseudotime ordering of the same scRNA-seq cells and you need a quantitative, gene-centric
  smoothness metric to rank them.
when_to_use_negative:
- You need a trajectory reconstructed between two chosen cell states (e.g. from a
  rare or transitory subpopulation) rather than to score an already-computed ordering
  — Monocle 2 and other linear/branch-limited methods are unsuited to that task per
  the paper.
- The data lack lineage branch markers or a biologically interpretable pseudotime
  ordering, so neither the correlation counts nor marker autocorrelation are meaningful.
- The goal is dimensionality-reduction comparison (e.g. the theislab/scAnalysisTutorial
  diffusion-pseudotime tutorial) rather than algorithm-vs-algorithm trajectory smoothness
  benchmarking.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellRouter
  role: Produces the subpopulation-aware flow-network trajectories being benchmarked,
    using R on the Paul et al. dataset
  repo: https://github.com/edroaldo/cellrouter
- name: Monocle 2
  role: Competing trajectory algorithm whose pseudotime ordering is scored for smoothness
- name: Diffusion Pseudotime (DPT)
  role: Competing diffusion-map pseudotime algorithm whose ordering is scored
- name: Wishbone
  role: Competing branched-trajectory algorithm whose ordering is scored
- name: Waterfall
  role: Competing linear-trajectory algorithm whose ordering is scored
- name: R
  role: Environment in which all trajectory algorithms are run and correlation/autocorrelation
    metrics are computed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:b11df540fca72a5d4d5b2109acc36719343f39b156465e323d106cff92d758c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - Diffusion Pseudotime (DPT)
  - Monocle 2
  - R
  - Waterfall
  - Wishbone
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# pseudotime-gene-expression-correlation-analysis

## Summary

Scores the smoothness of single-cell trajectories by counting genes whose expression is significantly correlated with a trajectory's pseudotime ordering and by computing lag-1 autocorrelation of branch markers, used in the CellRouter paper to benchmark CellRouter against Monocle 2, DPT, Wishbone, and Waterfall on the Paul et al. myeloid progenitor data (GSE72857).

## When to use

Use when comparing trajectory-inference algorithms that each output a pseudotime ordering of the same scRNA-seq cells and you need a quantitative, gene-centric smoothness metric to rank them.

## When NOT to use

- You need a trajectory reconstructed between two chosen cell states (e.g. from a rare or transitory subpopulation) rather than to score an already-computed ordering — Monocle 2 and other linear/branch-limited methods are unsuited to that task per the paper.
- The data lack lineage branch markers or a biologically interpretable pseudotime ordering, so neither the correlation counts nor marker autocorrelation are meaningful.
- The goal is dimensionality-reduction comparison (e.g. the theislab/scAnalysisTutorial diffusion-pseudotime tutorial) rather than algorithm-vs-algorithm trajectory smoothness benchmarking.

## Inputs

- Pseudotime orderings of the same cells from each trajectory algorithm (CellRouter, Monocle 2, DPT, Wishbone, Waterfall)
- Preprocessed Paul et al. (Cell 2015) GSE72857 mouse myeloid progenitor scRNA-seq expression matrix
- Branch marker gene list: Klf1, Car2, Cited4 (erythroid); Mpo, Prtn3, Ctsg (myeloid)

## Outputs

- Per-algorithm counts of genes with expression significantly correlated with pseudotime
- Per-algorithm lag-1 autocorrelation values for erythroid and myeloid branch markers
- algorithm_comparison.csv with one row per algorithm variant and both metric columns populated

## How to apply

Take each algorithm's pseudotime ordering of the same cells (e.g. CellRouter trajectories from CMP subpopulation 20 to erythroid subpopulation 13 and GMP subpopulation 9, plus Monocle 2, DPT, Wishbone, and Waterfall orderings) and, per trajectory, test every gene's expression for significant correlation with pseudotime, recording the count of significant genes. Then compute the lag-1 autocorrelation of the lineage branch markers along each ordering — Klf1, Car2, Ciced4 (erythroid) and Mpo, Prtn3, Ctsg (myeloid) on the Paul et al. data — since a smooth, biologically coherent trajectory should show smoothly rising marker expression rather than abrupt jumps. Assemble one row per algorithm (including variants such as CellRouter run with Paul et al. cluster labels as subpopulation annotations) into a comparison table with columns for significantly correlated gene counts and lag-1 autocorrelations, written to algorithm_comparison.csv. The rationale, from the CellRouter paper, is that smoother trajectories identify more significantly correlated genes and higher marker autocorrelation, revealing which algorithms are most robust at reconstructing transitions from early states towards intermediate or transitory cell states.

## Related tools

- **CellRouter** (Produces the subpopulation-aware flow-network trajectories being benchmarked, using R on the Paul et al. dataset) — https://github.com/edroaldo/cellrouter
- **Monocle 2** (Competing trajectory algorithm whose pseudotime ordering is scored for smoothness)
- **Diffusion Pseudotime (DPT)** (Competing diffusion-map pseudotime algorithm whose ordering is scored)
- **Wishbone** (Competing branched-trajectory algorithm whose ordering is scored)
- **Waterfall** (Competing linear-trajectory algorithm whose ordering is scored)
- **R** (Environment in which all trajectory algorithms are run and correlation/autocorrelation metrics are computed)

## Examples

```
source('path/to/CellRouter_Class.R'); libdir <- 'path/to/CellRouter/'
```

## Evaluation signals

- algorithm_comparison.csv exists and contains a row for each of the six algorithm variants (CellRouter, CellRouter-with-Paul-cluster-labels, Monocle 2, DPT, Wishbone, Waterfall) with both metric columns populated.
- CellRouter trajectories show more significantly correlated genes than all compared algorithms.
- CellRouter trajectories show higher lag-1 autocorrelation for Klf1, Car2, Cited4 and Mpo, Prtn3, Ctsg than the compared algorithms.
- The overall pattern reproduces the paper's finding that CellRouter is more robust in reconstructing trajectories from early states towards intermediate or transitory cell states.
- All algorithms are scored on the identical set of preprocessed GSE72857 cells, so metric differences reflect trajectory quality rather than input differences.

## Limitations

- The benchmark relies on correlation and autocorrelation as proxies for trajectory smoothness; the paper defers thorough functional characterization of identified genes to future work.
- CellRouter was tested in Unix-based systems or Mac only and was not tested in Windows, which constrains where the pipeline can be run.
- Monocle 2 and similar methods are not designed to reconstruct trajectories between any two chosen cell states, so their inclusion may under-represent transitions to or from rare cell types.

## Evidence

- [intro] Current algorithms are less robust: "they are less robust in reconstructing trajectories from early states towards intermediate or transitory cell states"
- [intro] Metric rationale: "implements a flow network algorithm to flexibly reconstruct multi-state transition trajectories"
- [discussion] Subpopulation-aware trajectories: "CellRouter introduces the concept of subpopulation-awareness to identify cellular trajectories and gene expression dynamics between any subpopulations"
- [methods] Sourcing CellRouter with libdir: "source the CellRouter class and make sure you provide the correct directory for the Java libraries inside the folder "CellRouter", as specified by the variable 'libdir'"
- [methods] Dataset: "Analysis of mouse myeloid progenitor single-cell RNA-seq data generated by Paul et al.(Cell 2015)"
- [readme] Tested platforms: "CellRouter was tested in Unix-based systems. Therefore, we strongly recommend to use CellRouter in Unix-based systems or Mac. CellRouter was not tested in Windows."
