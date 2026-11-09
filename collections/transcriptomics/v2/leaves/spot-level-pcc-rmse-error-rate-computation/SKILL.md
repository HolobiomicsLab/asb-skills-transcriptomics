---
name: spot-level-pcc-rmse-error-rate-computation
description: Use when comparing predicted vs ground-truth spatial assignments of cell
  types or gene expressions on simulated spatial transcriptomics benchmarks with per-cell/spot-level
  labels.
when_to_use_negative:
- Input spatial data lacks ground-truth per-cell labels or simulated truth (e.g.,
  real tissue sections like the 3813-spot Visium human heart dataset), since error
  rates cannot be computed
- Evaluating gene imputation methods (Tangram, gimVI, SpaGE) for unmeasured genes
  rather than cell type identification or expression decomposition
- The goal is deconvolution alone where only spot-level cell-type proportions (not
  single-cell assignments) are produced and only proportion-based metrics apply
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SpatialScope
  role: Primary method under evaluation; its single-cell assignments and decomposed
    expressions are scored with error rate, PCC/RMSE, and cosine similarity
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: Tangram
  role: Alignment-based baseline; its mapped cells are compared against SpatialScope
    in error rate and cosine similarity
- name: CytoSPACE
  role: Alignment-based baseline; mapped single-cell ids are used directly as decomposed
    expressions and evaluated for error rate and cosine similarity
- name: StarDist+RCTD
  role: Combined segmentation + deconvolution baseline (StarDist for H&E nucleus segmentation,
    RCTD with doublet_mode='full') scored against SpatialScope
  repo: https://github.com/dmcable/spacexr
- name: Cell2location
  role: Deconvolution baseline evaluated by spot-level PCC/RMSE of cell-type proportions
    (regression max_epochs=250, batch_size=2500, lr=0.002; model max_epochs=30000)
  repo: https://github.com/BayraktarLab/cell2location
- name: CARD
  role: Spatially informed deconvolution baseline evaluated by spot-level PCC/RMSE
    of cell-type proportions
  repo: https://github.com/YingMa0107/CARD
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:6a693fba2b6f263425bd3a408f1120d063c211eab4c49c7e792e29d65179c5a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - CARD
  - Cell2location
  - CytoSPACE
  - DSTG
  - DestVI
  - RCTD
  - SPOTlight
  - STRIDE
  - SpaOTsc
  - SpatialDWLS
  - SpatialScope
  - StarDist
  - StarDist+RCTD
  - Tangram
  - novoSpaRc
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# Spot-level PCC/RMSE and single-cell error rate benchmark evaluation

## Summary

Evaluation skill for benchmarking spatial transcriptomics cell type identification and gene expression decomposition methods, using single-cell misclassification error rate, spot-level Pearson correlation (PCC) and RMSE, and per-cell cosine similarity, as applied to SpatialScope versus Tangram, CytoSPACE, and StarDist+RCTD on simulated single-slice datasets.

## When to use

Use when comparing predicted vs ground-truth spatial assignments of cell types or gene expressions on simulated spatial transcriptomics benchmarks with per-cell/spot-level labels.

## When NOT to use

- Input spatial data lacks ground-truth per-cell labels or simulated truth (e.g., real tissue sections like the 3813-spot Visium human heart dataset), since error rates cannot be computed
- Evaluating gene imputation methods (Tangram, gimVI, SpaGE) for unmeasured genes rather than cell type identification or expression decomposition
- The goal is deconvolution alone where only spot-level cell-type proportions (not single-cell assignments) are produced and only proportion-based metrics apply

## Inputs

- Simulated spatial transcriptomics benchmark datasets with ground-truth per-cell labels and expression (Datasets 1-6 from the SpatialScope study)
- Predicted single-cell assignments or decomposed gene expression matrices from SpatialScope, Tangram, CytoSPACE, or StarDist+RCTD
- Paired scRNA-seq reference data (e.g., human heart snRNA-seq atlas with 10 major cell types)
- Ground-truth spot-level cell-type composition matrices

## Outputs

- Single-cell misclassification error rate per method
- Spot-level PCC and RMSE of cell-type proportions per method
- Per-cell cosine similarity distributions for gene expression decomposition
- Comparative tables of PCC gain (%) and RMSE gain (%) across methods and UMI-subsample/cell-number scenarios

## How to apply

Load the simulated benchmark dataset (e.g., Datasets 1-4 of the SpatialScope study) with paired scRNA-seq references, then run the methods under comparison — SpatialScope's three-step pipeline (nucleus segmentation, cell type identification with Potts smoothness at ν=10, gene expression decomposition via a score-based generative model trained at 7500 epochs with L=232, T=5), StarDist+RCTD (doublet_mode='full'), Tangram, and CytoSPACE. For cell type identification, compute the single-cell misclassification error rate against the simulated ground-truth per-cell labels, plus spot-level PCC and RMSE of aggregated cell-type proportions. For gene expression decomposition, compute per-cell cosine similarity between predicted and ground-truth expression profiles. Judge success by pre-registered quantitative targets: error-rate reductions of 50.3%/20.6%/6.3% vs Tangram/CytoSPACE/StarDist+RCTD, per-cell cosine similarity ≈0.90 vs 0.44 (Tangram) and 0.57 (CytoSPACE) on Dataset 1, and highest cosine similarity across all UMI-subsample and cells-per-spot scenarios.

## Related tools

- **SpatialScope** (Primary method under evaluation; its single-cell assignments and decomposed expressions are scored with error rate, PCC/RMSE, and cosine similarity) — https://github.com/YangLabHKUST/SpatialScope
- **Tangram** (Alignment-based baseline; its mapped cells are compared against SpatialScope in error rate and cosine similarity)
- **CytoSPACE** (Alignment-based baseline; mapped single-cell ids are used directly as decomposed expressions and evaluated for error rate and cosine similarity)
- **StarDist+RCTD** (Combined segmentation + deconvolution baseline (StarDist for H&E nucleus segmentation, RCTD with doublet_mode='full') scored against SpatialScope) — https://github.com/dmcable/spacexr
- **Cell2location** (Deconvolution baseline evaluated by spot-level PCC/RMSE of cell-type proportions (regression max_epochs=250, batch_size=2500, lr=0.002; model max_epochs=30000)) — https://github.com/BayraktarLab/cell2location
- **CARD** (Spatially informed deconvolution baseline evaluated by spot-level PCC/RMSE of cell-type proportions) — https://github.com/YingMa0107/CARD

## Examples

```
from scipy.stats import pearsonr; from scipy.spatial.distance import cosine; err_rate = sum(p != g) / len(g); pcc, _ = pearsonr(pred_props, true_props); sim = 1 - cosine(pred_expr, true_expr)
```

## Evaluation signals

- Single-cell misclassification error rate reproduces the reported reductions: 50.3% vs Tangram, 20.6% vs CytoSPACE, 6.3% vs StarDist+RCTD
- Per-cell cosine similarity on Dataset 1 reaches ≈0.90 for SpatialScope vs 0.44 (Tangram) and 0.57 (CytoSPACE)
- Spot-level PCC gains over baselines fall in the reported 9.4%-157.5% range (up to 51.4%) and RMSE gains in 25.3%-89.7%
- Cosine-similarity ranking holds consistently across all UMI subsample rates and numbers of cells per simulated spot
- Predicted per-cell labels have one-to-one correspondence with simulated ground-truth cell ids, allowing exact error-rate computation without alignment

## Limitations

- Metrics are only computable on simulation benchmarks with ground truth; on real data the accuracy of cell type identification and gene expression decomposition is unverifiable
- Gene expression decomposition metrics for the ten deconvolution baselines require mapping their spot-level proportions back to single cells, which may distort comparisons versus alignment-based methods
- The evaluation inherits dependence on upstream nucleus segmentation quality, which serves as the building block to quantify the cell count at each spot

## Evidence

- [results] Benchmarking against Tangram/CytoSPACE/StarDist+RCTD with error rate and cosine similarity: [evidence span withheld — anchor status: not_found]
- [other] Evaluation by single-cell error rate and spot-level PCC/RMSE, and decomposition by per-cell cosine similarity: [evidence span withheld — anchor status: not_found]
- [methods] Three-step pipeline defines what outputs are scored (cell type identification and gene expression decomposition): "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [readme] RCTD baseline produces the cell type assignments and proportions that the error-rate and PCC/RMSE metrics score: [evidence span withheld — anchor status: external_doc]
- [methods] Simulated benchmark datasets on which the metrics are computed: "cell type identification on six benchmarking datasets (Supplementary Fig. 4, Dataset 1-6)"
