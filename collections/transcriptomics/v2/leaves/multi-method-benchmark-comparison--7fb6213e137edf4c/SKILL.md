---
name: multi-method-benchmark-comparison
description: Use when a new method's paper claims ity over named baselines (e.g.,
  11 deconvolution tools or STAligner/PASTE) and you must reproduce or extend the
  head-to-head comparison on the paper's benchmark datasets.
when_to_use_negative:
- When no ground-truth cell type compositions or anatomically validated slice order
  exist — simulated benchmarks with known compositions are required to compute RMSE/JSD/AS
  meaningfully
- When only one method is of interest and no baselines need comparison — the orchestration
  overhead of running 10+ external tools is unnecessary
- When datasets have substantial slice defects (e.g., crop ratios ≥ 0.25) and hyperparameter
  sensitivity of baselines like PASTE would confound a fair comparison
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0203
tools:
- name: SPACEL
  role: 'Tool under test: Spoint (deconvolution), Splane (spatial domains), Scube
    (3D alignment) modules evaluated against baselines'
  repo: github.com/QuKunLab/SPACEL
- name: Cell2location
  role: Baseline deconvolution method run with default recommended settings on the
    32 simulated datasets
- name: Tangram
  role: Baseline deconvolution/mapping method (map_cells_to_space) compared in the
    benchmark
  repo: github.com/broadinstitute/Tangram
- name: Stereoscope
  role: Baseline deconvolution method run with default settings
- name: DestVI
  role: Baseline deconvolution method run with default settings
- name: SPOTlight
  role: Baseline deconvolution method run with default settings
- name: RCTD
  role: Baseline deconvolution method run with default settings
- name: STRIDE
  role: Baseline deconvolution method run with default settings
- name: SpatialDWLS
  role: Baseline deconvolution method run with default settings
- name: DSTG
  role: Baseline deconvolution method run with default settings
- name: Seurat
  role: Baseline deconvolution method run with default settings
- name: STAligner
  role: Baseline 3D slice-alignment method compared on the 33 MERFISH MOp slices
- name: PASTE
  role: Baseline 3D slice-alignment method compared on the 33 MERFISH MOp slices
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:d89afc95b2016f837b86d3aa57de440bc962479076d7e2cf3a03baa4debcde81
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell2location
  - DSTG
  - DestVI
  - PASTE
  - Python
  - RCTD
  - SPACEL
  - SPOTlight
  - STAligner
  - STRIDE
  - Scube
  - Seurat
  - SpatialDWLS
  - Splane
  - Spoint
  - Stereoscope
  - Tangram
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 21
---

# multi-method-benchmark-comparison

## Summary

Benchmark a new computational method (e.g., SPACEL's Spoint/Splane/Scube) against multiple state-of-the-art baselines on the same simulated and real ST datasets, computing standardized metrics (PCC, SSIM, RMSE, JSD, accuracy score) per method and aggregating them into a summary table.

## When to use

Use when a new method's paper claims superiority over named baselines (e.g., 11 deconvolution tools or STAligner/PASTE) and you must reproduce or extend the head-to-head comparison on the paper's benchmark datasets.

## When NOT to use

- When no ground-truth cell type compositions or anatomically validated slice order exist — simulated benchmarks with known compositions are required to compute RMSE/JSD/AS meaningfully
- When only one method is of interest and no baselines need comparison — the orchestration overhead of running 10+ external tools is unnecessary
- When datasets have substantial slice defects (e.g., crop ratios ≥ 0.25) and hyperparameter sensitivity of baselines like PASTE would confound a fair comparison

## Inputs

- 32 simulated pseudo-spot ST benchmark datasets with ground-truth spot-level cell type compositions (AnnData/scanpy format)
- 33 consecutive MERFISH mouse primary motor cortex (MOp) slices with per-cell cell type annotations
- Reference single-cell dataset for supervised deconvolution baselines
- Baseline tool configurations (default recommended settings)

## Outputs

- Per-dataset metric tables (PCC, SSIM, RMSE, JSD, accuracy score) per method
- Aggregated mean metrics across all benchmark datasets, e.g., deconvolution_benchmark_metrics.csv
- Adjacent-slice SSIM/PCC summary per alignment method, e.g., ssim_pcc_metrics.csv
- Stacked-slice / 3D visualizations via Scube.plot.plot_stacked_slices and Scube.plot.plot_3d

## How to apply

Run the proposed method and every baseline (e.g., Cell2location, SpatialDWLS, RCTD, STRIDE, Stereoscope, Tangram, DestVI, Seurat, SPOTlight, DSTG for deconvolution; STAligner and PASTE for 3D alignment) with their default recommended settings on identical inputs — for SPACEL this means the 32 simulated pseudo-spot datasets with ground-truth spot-level cell type compositions, or the 33 consecutive MERFISH mouse primary motor cortex slices. For deconvolution, load each dataset with scanpy/AnnData, train Spoint via SPACEL.Spoint.base_model.SpointModel.train and SpointModel.deconv_spatial, then compute PCC, SSIM, RMSE, and JSD between predicted and ground-truth per-spot cell type proportions, deriving the accuracy score (AS) by comparing the dominant predicted cell type to the ground-truth dominant type per spot. For 3D alignment, first identify spatial domains with SplaneModel.identify_spatial_domain (after Splane.utils.add_cell_type_composition), transform coordinates with Scube.alignment.align, then compute pairwise SSIM and PCC between adjacent transformed slices for each method. Aggregate per-method mean metrics across datasets (e.g., into deconvolution_benchmark_metrics.csv or ssim_pcc_metrics.csv) and validate against paper-reported values: Spoint PCC/SSIM ≈ 0.73/0.69, RMSE/JSD ≈ 0.05/0.41, AS ≈ 0.93; Scube adjacent-slice SSIM/PCC ≈ 0.71/0.76 versus STAligner 0.61/0.65 and PASTE 0.57/0.61.

## Related tools

- **SPACEL** (Tool under test: Spoint (deconvolution), Splane (spatial domains), Scube (3D alignment) modules evaluated against baselines) — github.com/QuKunLab/SPACEL
- **Cell2location** (Baseline deconvolution method run with default recommended settings on the 32 simulated datasets)
- **Tangram** (Baseline deconvolution/mapping method (map_cells_to_space) compared in the benchmark) — github.com/broadinstitute/Tangram
- **Stereoscope** (Baseline deconvolution method run with default settings)
- **DestVI** (Baseline deconvolution method run with default settings)
- **SPOTlight** (Baseline deconvolution method run with default settings)
- **RCTD** (Baseline deconvolution method run with default settings)
- **STRIDE** (Baseline deconvolution method run with default settings)
- **SpatialDWLS** (Baseline deconvolution method run with default settings)
- **DSTG** (Baseline deconvolution method run with default settings)
- **Seurat** (Baseline deconvolution method run with default settings)
- **STAligner** (Baseline 3D slice-alignment method compared on the 33 MERFISH MOp slices)
- **PASTE** (Baseline 3D slice-alignment method compared on the 33 MERFISH MOp slices)

## Examples

```
from SPACEL.Spoint.base_model import SpointModel; model = SpointModel(adata); model.train(); model.deconv_spatial(); # then compute PCC/SSIM/RMSE/JSD vs ground truth and repeat for each baseline tool
```

## Evaluation signals

- Spoint reaches mean PCC/SSIM ≈ 0.73/0.69 and RMSE/JSD ≈ 0.05/0.41 on the 32 simulated datasets, with AS ≈ 0.93; baselines fall in the 0.24–0.82 AS range
- Scube achieves adjacent-slice SSIM/PCC ≈ 0.71/0.76, exceeding STAligner (0.61/0.65) and PASTE (0.57/0.61)
- All methods are run on identical inputs with default recommended settings; per-dataset metrics must be computed for every method before aggregation
- Aggregated output CSV contains one row per method with mean metrics across all 32 datasets (or 33 MERFISH slices)
- PASTE-stacked results should show a twisted 3D architecture as a sanity check of the comparison's discriminating power

## Limitations

- Current Scube version does not support nonlinear alignment, unlike STAligner and PASTE, limiting comparison scope
- Adding a new slice to SPACEL requires retraining the entire deep learning model from scratch, affecting computational efficiency for large-scale benchmark reruns
- Splane depends on cell type composition input, requiring a deconvolution step (e.g., Spoint) for seq-based ST datasets like 10X Visium before domain identification and Scube alignment
- PASTE's alignment results are sensitive to hyperparameters in datasets with significant defects (e.g., slices with crop ratios ≥ 0.25), which can bias baseline comparisons

## Evidence

- [results] Spoint yielded the highest average PCC/SSIM values (0.73/0.69) and the lowest average RMSE/JSD values (0.05/0.41) among the 11 deconvolution methods: [evidence span withheld — anchor status: not_found]
- [results] Scube SSIM/PCC vs baselines: [evidence span withheld — anchor status: not_found]
- [methods] Baselines run with defaults: [evidence span withheld — anchor status: not_found]
- [discussion] PASTE hyperparameter sensitivity: "This design makes PASTE’s alignment results sensitive to hyperparameters in datasets with significant defects (e.g., slices with crop ratios ≥ 0.25)"
- [abstract] SPACEL outperforms 19 methods: "Comparisons against 19 state-of-the-art methods using both simulated and real ST datasets from various tissues and ST technologies demonstrate that SPACEL outperforms the others for cell type"
- [discussion] Scube lacks nonlinear alignment: "the current version of Scube does not support nonlinear alignment as well as STAligner and PASTE"
