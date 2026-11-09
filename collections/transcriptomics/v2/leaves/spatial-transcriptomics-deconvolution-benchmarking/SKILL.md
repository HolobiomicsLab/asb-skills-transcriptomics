---
name: spatial-transcriptomics-deconvolution-benchmarking
description: Use when you need to quantitatively compare ST deconvolution methods
  on datasets with known ground-truth cell type proportions, or measure runtime and
  peak RAM of SpatialDDLS vs RCTD vs cell2location on identical inputs.
when_to_use_negative:
- Your ST data lacks a scRNA-seq reference or ground-truth composition, so accuracy
  metrics like PCC/CCC/JSD cannot be computed against known proportions
- You only need to run one deconvolution method on new data rather than compare methods
  head-to-head on identical inputs
- Spatial continuity does not hold for your dataset, making spatially regularized
  predictions (k.spots neighborhood smoothing) inappropriate
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: SpatialDDLS
  role: Neural-network deconvolution method under benchmark; trained via genMixedCellProp,
    simMixedProfiles, trainDeconvModel and applied via deconvSpatialDDLS with spatial
    regularization
  repo: https://github.com/diegommcc/SpatialDDLS
- name: RCTD
  role: Comparator deconvolution method run on the same pseudo-spot inputs with runtime/RAM
    profiling
- name: cell2location
  role: Comparator deconvolution method; slowest of the three but lowest peak RAM
- name: SpatialDDLSdata
  role: Provides MouseDLN.ST (SpatialExperiment) and MouseDLN.SCE (SingleCellExperiment)
    benchmark datasets from GSE173778
  repo: https://github.com/diegommcc/SpatialDDLSdata
- name: ggplot2
  role: Reproduces benchmark figures (Supplementary S5b/c, S6b/c, S7b, S8a)
- name: ggpubr
  role: Reproduces benchmark figure panels with significance comparisons
merged_aliases:
- spatial-transcriptomics-deconvolution
merged_alias_records:
- alias: spatial-transcriptomics-deconvolution
  slug: spatial-transcriptomics-deconvolution
  jaccard_score: 0.75
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae072
    title: 'SpatialDDLS: an R package to deconvolute spatial transcriptomics data
      using neural networks'
  source_content_hash: sha256:23c74341516e9d7526f93ee73772af70b9b4de7d90db696d3ad5efd1244ed464
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/diegommcc/SpatialDDLS
  tools:
  - BiocManager
  - R
  - RCTD
  - SPOTlight
  - STdeconvolve
  - SingleCellExperiment
  - SpatialDDLS
  - SpatialDDLSdata
  - SpatialExperiment
  - cell2location
  - devtools
  - ggplot2
  - ggpubr
  - remotes
  - spatialDWLS
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# spatial-transcriptomics-deconvolution-benchmarking

## Summary

Benchmark cell type deconvolution of spatial transcriptomics spots — comparing SpatialDDLS against RCTD and cell2location on prediction accuracy (PCC, RMSE, CCC, JSD) and on runtime/RAM consumption — using binned single-cell-resolution ST datasets with known ground-truth compositions.

## When to use

Use when you need to quantitatively compare ST deconvolution methods on datasets with known ground-truth cell type proportions, or measure runtime and peak RAM of SpatialDDLS vs RCTD vs cell2location on identical inputs.

## When NOT to use

- Your ST data lacks a scRNA-seq reference or ground-truth composition, so accuracy metrics like PCC/CCC/JSD cannot be computed against known proportions
- You only need to run one deconvolution method on new data rather than compare methods head-to-head on identical inputs
- Spatial continuity does not hold for your dataset, making spatially regularized predictions (k.spots neighborhood smoothing) inappropriate

## Inputs

- Single-cell-resolution ST datasets (seqFISH, STARmap, MERFISH) with paired scRNA-seq references
- 10x Visium murine lymph node dataset (GSE173778, MouseDLN.ST) with paired 10x Chromium scRNA-seq (MouseDLN.SCE)
- SingleCellExperiment and SpatialExperiment objects
- Trained SpatialDDLS model object

## Outputs

- benchmark_metrics.csv with per-method, per-dataset PCC, RMSE, CCC and JSD values
- performance_benchmark_table.csv with per-method wall-clock runtime and peak RAM
- Supplementary figure reproductions (S5b/c, S6b/c, S7b, S8a) via ggplot2/ggpubr
- spot-by-cell-type matrices of deconvoluted proportions

## How to apply

Obtain single-cell-resolution ST datasets (seqFISH Eng 2019, STARmap Wang 2018, MERFISH Moffitt 2018) with paired scRNA-seq references, and bin neighboring single cells into mixed pseudo-spots with known ground-truth composition (e.g. n.cells = 50). Train SpatialDDLS on simulated mixed profiles via genMixedCellProp (num.sim.spots = 10000, min.zero.prop = 5) and simMixedProfiles, then trainDeconvModel (two hidden layers of 200 neurons, 60 epochs); run all methods, including deconvSpatialDDLS with spatial regularization (k.spots = 6), on the identical pseudo-spot inputs. Compute PCC, RMSE, Lin's CCC and JSD per method and dataset, with separate comparisons for the excitatory neuron subtypes L2/L3, L4, L5 and L6 in the mouse neocortex data. For the computational benchmark, run all three methods on the murine lymph node 10x Visium data (GSE173778, MouseDLN.ST with MouseDLN.SCE reference), recording wall-clock time and peak RAM. Aggregate metrics into CSV tables (benchmark_metrics.csv, performance_benchmark_table.csv) and reproduce figures with ggplot2/ggpubr.

## Related tools

- **SpatialDDLS** (Neural-network deconvolution method under benchmark; trained via genMixedCellProp, simMixedProfiles, trainDeconvModel and applied via deconvSpatialDDLS with spatial regularization) — https://github.com/diegommcc/SpatialDDLS
- **RCTD** (Comparator deconvolution method run on the same pseudo-spot inputs with runtime/RAM profiling)
- **cell2location** (Comparator deconvolution method; slowest of the three but lowest peak RAM)
- **SpatialDDLSdata** (Provides MouseDLN.ST (SpatialExperiment) and MouseDLN.SCE (SingleCellExperiment) benchmark datasets from GSE173778) — https://github.com/diegommcc/SpatialDDLSdata
- **ggplot2** (Reproduces benchmark figures (Supplementary S5b/c, S6b/c, S7b, S8a))
- **ggpubr** (Reproduces benchmark figure panels with significance comparisons)

## Examples

```
mouseDLN.SDDLS <- deconvSpatialDDLS(mouseDLN.SDDLS, index.st = 1, k.spots = 6, fast.pca = TRUE, normalize = TRUE)
```

## Evaluation signals

- Per-method PCC, RMSE, CCC and JSD values agree with the benchmark results reported in Supplementary Figures S5b/c, S6b/c, S7b and S8a
- Runtime ordering reproduced: SpatialDDLS comparable to RCTD, cell2location the slowest of the three
- RAM ordering reproduced: cell2location has the lowest peak RAM, superior to SpatialDDLS and RCTD
- SpatialDDLS outperforms cell2location and RCTD at predicting the most important cell types (excitatory L2/L3, L4, L5, L6 neurons) in STARmap and seqFISH datasets
- Deconvoluted output contains proportions for all spots and all 12 cell types

## Limitations

- Spatial regularization toward neighboring spots assumes spatial continuity, which may not hold for all datasets; the regularization step is optional
- Benchmarking on binned pseudo-spots mimics spot-level resolution but depends on the binning parameters (e.g. n.cells = 50) matching real spot characteristics
- cell2location is the slowest method, which may make full runtime benchmarking impractical on large datasets

## Evidence

- [results] SpatialDDLS outperformed cell2location and RCTD at predicting the most important cell types in the mouse neo-cortex (STARmap and seqFISH datasets): the excitatory L2/L3, L4, L5, and L6 neurons.: [evidence span withheld — anchor status: not_found]
- [results] Runtime finding: [evidence span withheld — anchor status: not_found]
- [methods] Spatial regularization: [evidence span withheld — anchor status: not_found]
- [methods] Extrinsic smoothing: "the extrinsic predictions are a smoothed version of the final ones"
- [readme] Skill purpose: "The package takes advantage of single-cell RNA sequencing (scRNA-seq) data to simulate mixed transcriptional profiles with known cell composition and train fully-connected neural networks"
