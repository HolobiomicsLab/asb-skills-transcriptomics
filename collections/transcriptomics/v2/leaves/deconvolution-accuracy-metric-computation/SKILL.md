---
name: deconvolution-accuracy-metric-computation
description: Use when evaluating a cell type deconvolution method on spatial transcriptomics
  (ST) data (e.g., 10X Visium spots) where ground-truth spot-level cell type proportions
  are available, and a quantitative PCC/SSIM/RMSE/JSD/AS comparison against baselines
  is needed.
when_to_use_negative:
- No ground-truth spot-level cell type composition exists (real ST datasets without
  simulation), so PCC/SSIM/RMSE/JSD against truth cannot be computed
- The input is single-cell-resolution in-situ ST data rather than spot-based seq ST
  data — the AS spot-dominance comparison assumes spot-level mixtures
- The task is spatial domain identification or 3D alignment (Splane/Scube territory),
  not cell type proportion deconvolution evaluation
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3170
tools:
- name: Spoint
  role: MLP with probabilistic model used to deconvolute per-spot cell type composition;
    the method benchmarked via SpointModel.train and SpointModel.deconv_spatial
  repo: https://github.com/QuKunLab/SPACEL
- name: SPACEL
  role: Python package hosting the Spoint deconvolution module used as the test method
    in the benchmark
  repo: https://github.com/QuKunLab/SPACEL
- name: Tangram
  role: Baseline deconvolution/mapping method run on the same 32 simulated datasets
    for PCC/SSIM/RMSE/JSD/AS comparison
  repo: https://github.com/broadinstitute/Tangram
- name: Cell2location
  role: Baseline deconvolution tool benchmarked with default recommended settings
- name: SpatialDWLS
  role: Baseline deconvolution tool benchmarked with default recommended settings
- name: RCTD
  role: Baseline deconvolution tool benchmarked with default recommended settings
- name: STRIDE
  role: Baseline deconvolution tool benchmarked with default recommended settings
- name: Stereoscope
  role: Baseline deconvolution tool benchmarked with default recommended settings
- name: DestVI
  role: Baseline deconvolution tool benchmarked with default recommended settings
- name: Seurat
  role: Baseline deconvolution tool benchmarked with default recommended settings
- name: SPOTlight
  role: Baseline deconvolution tool benchmarked with default recommended settings
- name: DSTG
  role: Baseline deconvolution tool benchmarked with default recommended settings
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:e7aa94bbd98db8bbdad0dfb8f8bea8a653458653e8777831eede1dbbb199ff2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QuKunLab/SPACEL
  tools:
  - Cell2location
  - DSTG
  - DestVI
  - Python
  - RCTD
  - SPACEL
  - SPOTlight
  - STRIDE
  - Seurat
  - SpatialDWLS
  - Spoint
  - Stereoscope
  - Tangram
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# deconvolution-accuracy-metric-computation

## Summary

Benchmark a spatial transcriptomics cell type deconvolution method by comparing predicted per-spot cell type proportions against ground truth using PCC, SSIM, RMSE, JSD, and a dominant-cell-type accuracy score (AS), as done for Spoint versus 10 competing deconvolution tools on 32 simulated pseudo-spot datasets.

## When to use

Use when evaluating a cell type deconvolution method on spatial transcriptomics (ST) data (e.g., 10X Visium spots) where ground-truth spot-level cell type proportions are available, and a quantitative PCC/SSIM/RMSE/JSD/AS comparison against baselines is needed.

## When NOT to use

- No ground-truth spot-level cell type composition exists (real ST datasets without simulation), so PCC/SSIM/RMSE/JSD against truth cannot be computed
- The input is single-cell-resolution in-situ ST data rather than spot-based seq ST data — the AS spot-dominance comparison assumes spot-level mixtures
- The task is spatial domain identification or 3D alignment (Splane/Scube territory), not cell type proportion deconvolution evaluation

## Inputs

- 32 simulated pseudo-spot ST datasets in AnnData/scanpy format with ground-truth spot-level cell type compositions
- Predicted per-spot cell type proportion matrices from the method under test (e.g., Spoint output of deconv_spatial)
- Predicted per-spot cell type proportion matrices from baseline tools (Cell2location, SpatialDWLS, RCTD, STRIDE, Stereoscope, Tangram, DestVI, Seurat, SPOTlight, DSTG)

## Outputs

- Per-dataset PCC, SSIM, RMSE, and JSD values comparing predicted vs ground-truth cell type proportions
- Accuracy score (AS) per method: fraction of spots where the dominant predicted cell type matches the ground truth
- Aggregated summary table deconvolution_benchmark_metrics.csv with mean metrics per method across the 32 datasets

## How to apply

Load each simulated ST benchmark dataset (with ground-truth spot-level cell type compositions) as scanpy/AnnData objects. Run the method under test — here Spoint via SPACEL.Spoint.base_model.SpointModel.train followed by SpointModel.deconv_spatial — and each baseline tool (Cell2location, SpatialDWLS, RCTD, STRIDE, Stereoscope, Tangram, DestVI, Seurat, SPOTlight, DSTG) with default recommended settings on the same datasets. For each dataset, compute PCC, SSIM, RMSE, and JSD between predicted and ground-truth cell type proportions per spot, and derive the accuracy score (AS) by checking whether the dominant predicted cell type matches the ground-truth dominant cell type at each spot. Aggregate per-method mean metrics across all 32 datasets and write the summary to deconvolution_benchmark_metrics.csv.

## Related tools

- **Spoint** (MLP with probabilistic model used to deconvolute per-spot cell type composition; the method benchmarked via SpointModel.train and SpointModel.deconv_spatial) — https://github.com/QuKunLab/SPACEL
- **SPACEL** (Python package hosting the Spoint deconvolution module used as the test method in the benchmark) — https://github.com/QuKunLab/SPACEL
- **Tangram** (Baseline deconvolution/mapping method run on the same 32 simulated datasets for PCC/SSIM/RMSE/JSD/AS comparison) — https://github.com/broadinstitute/Tangram
- **Cell2location** (Baseline deconvolution tool benchmarked with default recommended settings)
- **SpatialDWLS** (Baseline deconvolution tool benchmarked with default recommended settings)
- **RCTD** (Baseline deconvolution tool benchmarked with default recommended settings)
- **STRIDE** (Baseline deconvolution tool benchmarked with default recommended settings)
- **Stereoscope** (Baseline deconvolution tool benchmarked with default recommended settings)
- **DestVI** (Baseline deconvolution tool benchmarked with default recommended settings)
- **Seurat** (Baseline deconvolution tool benchmarked with default recommended settings)
- **SPOTlight** (Baseline deconvolution tool benchmarked with default recommended settings)
- **DSTG** (Baseline deconvolution tool benchmarked with default recommended settings)

## Examples

```
from SPACEL.Spoint.base_model import SpointModel; m = SpointModel(adata_st); m.train(); m.deconv_spatial()  # then compute PCC/SSIM/RMSE/JSD vs ground-truth proportions
```

## Evaluation signals

- Spoint should reach approximately PCC/SSIM = 0.73/0.69 and RMSE/JSD = 0.05/0.41 averaged over the 32 simulated datasets, with AS ≈ 0.93
- All ten baseline methods should fall within the paper-reported 0.24–0.82 AS range
- Spoint should rank first on average PCC/SSIM and lowest on average RMSE/JSD among the 11 deconvolution methods compared
- Per-method metric rows aggregate correctly across all 32 datasets into deconvolution_benchmark_metrics.csv (no missing datasets)
- Predicted proportions should be non-negative and sum to ~1 per spot before metrics are computed

## Limitations

- The benchmark relies on 32 simulated pseudo-spot datasets; performance on real ST datasets without ground truth cannot be validated by these metrics directly
- Baseline tools were run with default recommended settings, so relative rankings may not reflect fully tuned configurations
- The AS metric only checks the dominant cell type per spot and is insensitive to errors in sub-dominant proportions

## Evidence

- [results] Spoint benchmark metrics: [evidence span withheld — anchor status: not_found]
- [methods] methods Spoint model: "Spoint embedded a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition for each spot on single ST slice."
- [methods] methods deconv API: "Spoint.base_model.SpointModel.deconv_spatial"
- [abstract] abstract module description: "Spoint embeds a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition for each spot in a single ST slice"
- [discussion] discussion benchmark scope: "the SPACEL modules outperform state-of-the-art methods for each of these tasks through analyses of 32 simulated and 11 real ST datasets acquired using five distinct ST technologies"
