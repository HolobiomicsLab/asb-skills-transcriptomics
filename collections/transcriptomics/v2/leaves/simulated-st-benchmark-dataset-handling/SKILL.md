---
name: simulated-st-benchmark-dataset-handling
description: Use when benchmarking cell type deconvolution methods on simulated ST
  data where ground-truth per-spot cell type proportions are known, or when reproducing
  SPACEL/Spoint benchmark metrics (PCC/SSIM/RMSE/JSD/AS) on the 32 pseudo-spot datasets.
when_to_use_negative:
- Real ST datasets without ground-truth spot-level cell type compositions — metrics
  like PCC/SSIM/RMSE/JSD against truth cannot be computed
- Spatial domain identification or 3D alignment tasks (Splane/Scube) rather than cell
  type deconvolution benchmarking
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SPACEL
  role: Toolkit providing the Spoint deconvolution module under benchmark
  repo: https://github.com/QuKunLab/SPACEL
- name: Spoint
  role: 'Method under test: MLP with probabilistic model predicting per-spot cell
    type proportions via SpointModel.train and SpointModel.deconv_spatial'
  repo: https://github.com/QuKunLab/SPACEL
- name: Tangram
  role: Baseline deconvolution method run on the same 32 simulated datasets with recommended
    settings
  repo: https://github.com/broadinstitute/Tangram
- name: Cell2location
  role: Baseline deconvolution method included in the benchmark comparison
- name: Python
  role: Implementation language using scanpy/AnnData for data loading and metric computation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:d9e4f64c44d86016305e1aabbceef0f00d65907ca2b0b378663e4566cb8213c6
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# simulated-st-benchmark-dataset-handling

## Summary

Load and evaluate cell type deconvolution on simulated spatial transcriptomics pseudo-spot datasets (32 simulated benchmark datasets with ground-truth spot-level cell type compositions), scoring methods by PCC, SSIM, RMSE, JSD, and accuracy score as in the SPACEL benchmark.

## When to use

Use when benchmarking cell type deconvolution methods on simulated ST data where ground-truth per-spot cell type proportions are known, or when reproducing SPACEL/Spoint benchmark metrics (PCC/SSIM/RMSE/JSD/AS) on the 32 pseudo-spot datasets.

## When NOT to use

- Real ST datasets without ground-truth spot-level cell type compositions — metrics like PCC/SSIM/RMSE/JSD against truth cannot be computed
- Spatial domain identification or 3D alignment tasks (Splane/Scube) rather than cell type deconvolution benchmarking

## Inputs

- 32 simulated pseudo-spot spatial transcriptomics datasets with ground-truth spot-level cell type compositions (scanpy/AnnData .h5ad)
- Single-cell reference data with cell type annotations for training deconvolution models

## Outputs

- Per-spot predicted cell type proportions for each method and dataset
- Per-dataset PCC, SSIM, RMSE, JSD, and accuracy score (AS) metrics per method
- Aggregated summary table deconvolution_benchmark_metrics.csv with mean metrics across the 32 datasets

## How to apply

Load each of the 32 simulated ST benchmark datasets with its ground-truth spot-level cell type compositions using scanpy/AnnData. Run the method under test — e.g., train Spoint via SPACEL.Spoint.base_model.SpointModel.train then predict proportions with SpointModel.deconv_spatial — alongside baseline tools (Cell2location, SpatialDWLS, RCTD, STRIDE, Stereoscope, Tangram, DestVI, Seurat, SPOTlight, DSTG) with default recommended settings on identical inputs. For each dataset, compute PCC, SSIM, RMSE, and JSD between predicted and ground-truth cell type proportions, and derive the accuracy score (AS) by comparing the dominant predicted cell type to the ground-truth dominant cell type at each spot. Aggregate per-method mean metrics across all 32 datasets and write the summary table to deconvolution_benchmark_metrics.csv. Validation uses published reference values: Spoint should reach approximately PCC/SSIM = 0.73/0.69, RMSE/JSD = 0.05/0.41, AS = 0.93, with baselines falling in the 0.24–0.82 AS range.

## Related tools

- **SPACEL** (Toolkit providing the Spoint deconvolution module under benchmark) — https://github.com/QuKunLab/SPACEL
- **Spoint** (Method under test: MLP with probabilistic model predicting per-spot cell type proportions via SpointModel.train and SpointModel.deconv_spatial) — https://github.com/QuKunLab/SPACEL
- **Tangram** (Baseline deconvolution method run on the same 32 simulated datasets with recommended settings) — https://github.com/broadinstitute/Tangram
- **Cell2location** (Baseline deconvolution method included in the benchmark comparison)
- **Python** (Implementation language using scanpy/AnnData for data loading and metric computation)

## Examples

```
from SPACEL.Spoint.base_model import SpointModel; m = SpointModel(adata); m.train(); m.deconv_spatial()
```

## Evaluation signals

- Spoint's aggregated metrics across the 32 datasets approximate PCC/SSIM = 0.73/0.69 and RMSE/JSD = 0.05/0.41
- Spoint's accuracy score (AS) is approximately 0.93, with all ten baselines in the 0.24–0.82 AS range
- deconvolution_benchmark_metrics.csv contains mean PCC, SSIM, RMSE, JSD, and AS per method over all 32 datasets
- Per-dataset JSD and RMSE are computed between predicted and ground-truth per-spot cell type proportions (non-negative, proportions sum to 1)
- Each baseline tool is run with its default recommended settings on the identical 32 datasets for a fair comparison

## Limitations

- Evaluation depends on simulated ground truth; simulated pseudo-spot composition may not fully reflect real-tissue deconvolution difficulty
- Baselines run with default settings may under-represent methods' tuned performance; SPACEL paper claims comparisons against 19 state-of-the-art methods on both simulated and real datasets
- The benchmark uses five distinct ST technologies and 32 simulated plus 11 real datasets in the full study; results may vary by technology

## Evidence

- [methods] Spoint benchmark metrics: [evidence span withheld — anchor status: not_found]
- [methods] Spoint model methods: "Spoint.base_model.SpointModel.deconv_spatial"
- [discussion] Simulated and real datasets: "the SPACEL modules outperform state-of-the-art methods for each of these tasks through analyses of 32 simulated and 11 real ST datasets acquired using five distinct ST technologies"
- [readme] SPACEL package description: "Spoint embedded a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition for each spot on single ST slice."
- [abstract] Comparison scope: "Comparisons against 19 state-of-the-art methods using both simulated and real ST datasets from various tissues and ST technologies demonstrate that SPACEL outperforms the others for cell type"
