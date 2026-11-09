---
name: proliferation-weight-incorporation-in-fate-modeling
description: Use when modeling cell fate bias from time-series scRNA-seq data (e.g.,
  day-2/4/6 lineage-tracing points) and empirical or gene-expression-derived growth
  rates are available to weight the learned differentiation landscape.
when_to_use_negative:
- The dataset's final time point is not at steady-state — PRESCIENT requires this
  constraint for training
- No proliferation estimates are obtainable (empirical or expression-derived) and
  only the no-growth model can be fit, since the improvement depends on growth-rate
  quality
- The analysis goal is per-gene interpretation rather than fate-bias prediction —
  information about individual genes is lost in the PCA transformation
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: PRESCIENT
  role: Generative model whose potential-energy training is weighted by growth rates
    `w` to predict clonal fate bias
  repo: https://github.com/gifford-lab/prescient
- name: prescient-analysis
  role: Provides `weinreb-fate.sh` training scripts (5 seeds, 2-layer 400-unit models)
    and fate-bias preprocessing/analysis notebooks
  repo: https://github.com/gifford-lab/prescient-analysis
- name: scanpy
  role: Recommended pre-processing of the Weinreb scRNA-seq matrix into .h5ad/.csv
    inputs with time-point and cell-type labels
- name: PyTorch
  role: Framework used to generate and train the data_pt torch object and simulate
    trajectories
- name: wot (Waddington-OT)
  role: Baseline method for comparison of fate-bias prediction performance
  repo: https://github.com/broadinstitute/wot
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:758f41ac9e380abdb9abc8b67bec20f1e2dc6cbf3d70793dc1fcadedcda6245f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gifford-lab/prescient
  tools:
  - PRESCIENT
  - PyTorch
  - prescient-analysis
  - scanpy
  - scikit-learn
  - wot (Waddington-OT)
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# proliferation-weight-incorporation-in-fate-modeling

## Summary

Incorporate empirically measured or expression-derived cell proliferation rates as growth weights into PRESCIENT's potential-energy model to improve clonal fate-bias prediction on longitudinal scRNA-seq data such as the Weinreb hematopoiesis lineage-tracing dataset.

## When to use

Use when modeling cell fate bias from time-series scRNA-seq data (e.g., day-2/4/6 lineage-tracing points) and empirical or gene-expression-derived growth rates are available to weight the learned differentiation landscape.

## When NOT to use

- The dataset's final time point is not at steady-state — PRESCIENT requires this constraint for training
- No proliferation estimates are obtainable (empirical or expression-derived) and only the no-growth model can be fit, since the improvement depends on growth-rate quality
- The analysis goal is per-gene interpretation rather than fate-bias prediction — information about individual genes is lost in the PCA transformation

## Inputs

- Normalized gene expression matrix with integer time-point labels (days 2/4/6) and cell type annotations (.h5ad/.csv from scanpy or Seurat pre-processing)
- Pre-computed growth weights tensor `w` from empirically measured lineage-tracing proliferation rates
- Expression-derived proliferation rate estimates as an alternative growth regime
- Held-out day-2 progenitor states with clonal fate labels for the 335-cell test set

## Outputs

- data_pt torch object containing x/xp/xu timepoint-split tensors, sklearn PCA and UMAP objects, and growth weights w
- Trained PRESCIENT potential-energy models under zeroed, empirical, and expression-derived growth regimes
- Simulated trajectories and clonal fate-bias predictions for the held-out day-2 progenitors
- Method-by-method metric table (Pearson r, AUROC, covered fraction per proliferation regime), e.g., fate_bias_metrics.csv

## How to apply

Build the PRESCIENT `data_pt` torch object with `prescient process_data`, which stores data, celltype, genes, tps, the timepoint-split tensors x/xp/xu, the sklearn PCA and umap.UMAP objects, and the pre-computed growth weights `w`. Train three 2-layer, 400-unit models for comparison (per `weinreb-fate.sh`): one with the growth-weight tensor `w` zeroed (no proliferation), one with empirically measured lineage-tracing growth rates, and one with growth rates derived from gene expression. Simulate trajectories from held-out day-2 progenitor states and compute clonal fate bias (the probability a cell enters a particular fate given its initial state) for the 335-cell test set. Score each variant by Pearson correlation with observed fate bias and AUROC for fate classification, benchmarking against WOT and FateID baselines and the held-out clonal upper bound; expect the proliferation-aware variant to improve (mean r ≈ 0.347 ± 0.029, AUROC ≈ 0.692 ± 0.012). The rationale is that the model improves when incorporating growth rates, as confirmed by the article's discussion section.

## Related tools

- **PRESCIENT** (Generative model whose potential-energy training is weighted by growth rates `w` to predict clonal fate bias) — https://github.com/gifford-lab/prescient
- **prescient-analysis** (Provides `weinreb-fate.sh` training scripts (5 seeds, 2-layer 400-unit models) and fate-bias preprocessing/analysis notebooks) — https://github.com/gifford-lab/prescient-analysis
- **scanpy** (Recommended pre-processing of the Weinreb scRNA-seq matrix into .h5ad/.csv inputs with time-point and cell-type labels)
- **PyTorch** (Framework used to generate and train the data_pt torch object and simulate trajectories)
- **wot (Waddington-OT)** (Baseline method for comparison of fate-bias prediction performance) — https://github.com/broadinstitute/wot

## Examples

```
bash scripts/weinreb-fate.sh  # trains 5 seeds of 2-layer 400-unit PRESCIENT models with estimated proliferation rates for clonal fate bias prediction
```

## Evaluation signals

- fate_bias_metrics.csv exists with rows for all PRESCIENT variants (zeroed, empirical, expression-derived growth) plus WOT, FateID, and held-out clonal upper-bound baselines
- Proliferation-aware PRESCIENT Pearson r and AUROC approach the reported values (mean r = 0.347 ± 0.029, AUROC = 0.692 ± 0.012 from Fig. 2f-h)
- Proliferation-aware runs outperform the no-proliferation (zeroed w) run on Pearson r and AUROC
- The data_pt object contains all required keys: data, celltype, genes, tps, x/xp/xu, PCA, UMAP, and w
- Fate bias is computed for the full 335-cell test set with covered fraction reported per method

## Limitations

- Model performance depends on growth rates and would benefit from better estimates of proliferation
- The model requires the final time point of the dataset to be at steady-state
- Information is lost about individual genes when transforming data into PCA space, and lowly-expressed genes important to cell fate decisions may be dropped altogether

## Evidence

- [abstract] PRESCIENT is able to predict the fate biases of progenitor cells in hematopoiesis when accounting for cell proliferation, improving upon the best-performing existing method.: "PRESCIENT is able to predict the fate biases of progenitor cells in hematopoiesis when accounting for cell proliferation"
- [methods] For training, all that is needed is normalized gene expression, time-point labels, and cell type annotations.: "For training, all that is needed is normalized gene expression, time-point labels, and cell type annotations."
- [methods] The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary): "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary)"
- [readme] weinreb-fate.sh trains 5 seeds of 2 layer 400 unit models on all data with estimated proliferation rates for clonal fate bias prediction: [evidence span withheld — anchor status: external_doc]
- [discussion] The model also improves when incorporating growth rates, and would likely benefit from better estimates of proliferation.: "The model also improves when incorporating growth rates, and would likely benefit from better estimates of proliferation."
