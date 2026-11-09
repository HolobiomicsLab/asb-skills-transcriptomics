---
name: lineage-tracing-clonal-fate-bias-prediction
description: Use when a longitudinal scRNA-seq dataset with lineage-tracing barcodes
  and time-point labels (e.g. days 2/4/6) is available and the goal is to predict
  the probability each progenitor cell enters a particular fate, quantifying whether
  proliferation estimates improve prediction.
when_to_use_negative:
- The dataset's final time point is not at steady-state — PRESCIENT's model explicitly
  requires this assumption
- The data is single-timepoint or lacks lineage-tracing information and integer time-point
  labels, so no clonal fate ground truth or longitudinal training signal exists
- The goal is gene-level causal inference — association of genes with fates does not
  imply causality, and PCA transformation discards per-gene information
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: PRESCIENT
  role: Core generative framework; `prescient process_data` builds the data_pt object
    and the potential-energy model is trained on days 2/4/6 to simulate progenitor
    trajectories and predict fate bias
  repo: https://github.com/gifford-lab/prescient
- name: scanpy
  role: Recommended pre-processing of the lineage-tracing scRNA-seq matrix into normalized
    expression with time-point and cell type annotations
- name: PyTorch
  role: Backend for the serialized data_pt torch object and for training the potential-energy
    model
- name: scikit-learn
  role: Supplies the PCA decomposition (data_pt['pca']) fit to normalized expression
    and used to produce PCs
- name: UMAP
  role: Supplies the UMAP embedding (data_pt['um']) fit to PCs for visualization
- name: wot
  role: Baseline method (WOT) whose fate-bias predictions are compared against PRESCIENT's
    proliferation-aware runs
  repo: https://github.com/broadinstitute/wot
- name: prescient-analysis
  role: Provides weinreb-fate.sh (5-seed training script for clonal fate bias prediction)
    and the 03x-weinreb2020-fate preprocessing/analysis notebooks
  repo: https://github.com/gifford-lab/prescient-analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:c86064f5df5c071101f4fe0e054f52af2a505684016a1c40b619801b88fc2930
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gifford-lab/prescient
  tools:
  - PRESCIENT
  - PyTorch
  - UMAP
  - prescient-analysis
  - scanpy
  - scikit-learn
  - wot
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# lineage-tracing-clonal-fate-bias-prediction

## Summary

Predict clonal fate bias of progenitor cells on a lineage-tracing scRNA-seq dataset (Weinreb et al. mouse hematopoiesis) by training PRESCIENT potential-energy models with and without cell proliferation rates, and benchmarking against WOT, FateID, and the clonal upper bound.

## When to use

Use when a longitudinal scRNA-seq dataset with lineage-tracing barcodes and time-point labels (e.g. days 2/4/6) is available and the goal is to predict the probability each progenitor cell enters a particular fate, quantifying whether proliferation estimates improve prediction.

## When NOT to use

- The dataset's final time point is not at steady-state — PRESCIENT's model explicitly requires this assumption
- The data is single-timepoint or lacks lineage-tracing information and integer time-point labels, so no clonal fate ground truth or longitudinal training signal exists
- The goal is gene-level causal inference — association of genes with fates does not imply causality, and PCA transformation discards per-gene information

## Inputs

- Lineage-tracing scRNA-seq matrix from Weinreb et al. mouse hematopoiesis (normalized expression, integer time-point labels for days 2/4/6, cell type annotations) in .csv, .tsv, .txt, or .h5ad format
- Empirically measured lineage-tracing cell proliferation rates
- Growth rates derived from gene expression
- Held-out day-2 clonal fate labels for the 335-cell test set

## Outputs

- data_pt torch object (serialized dictionary with data, celltype, genes, tps, x/xp/xu tensors, sklearn PCA object, umap.UMAP object, growth weights w)
- Trained PRESCIENT potential-energy models (5 seeds per proliferation regime)
- Per-cell clonal fate bias predictions for the day-2 progenitor test set
- Method-by-method metric table fate_bias_metrics.csv (Pearson r, AUROC, covered fraction per proliferation regime, including WOT, FateID, and clonal upper bound rows)

## How to apply

Pre-process the Weinreb et al. lineage-tracing matrix (normalized expression, integer time-point labels, cell type annotations) with scanpy and write it to .h5ad/.csv. Run `prescient process_data` to build the data_pt torch object containing data, celltype, genes, tps, the train/holdout splits x/xp/xu, sklearn PCA and umap objects, and pre-computed growth weights w. Train the 2-layer, 400-unit potential-energy model three times: once with the growth-weight tensor w zeroed (no proliferation), once with empirically measured lineage-tracing growth rates, and once with growth rates derived from gene expression — the README's weinreb-fate.sh trains 5 seeds of this configuration. Simulate trajectories from held-out day-2 progenitor states, compute clonal fate bias (the probability a cell enters a fate given its initial state) for the 335-cell test set, and score each variant by Pearson r with observed fate bias, AUROC for fate classification, and covered fraction. The proliferation-aware variant is expected to reach roughly r = 0.347 and AUROC ≈ 0.692, outperforming no-proliferation runs and the WOT/FateID baselines.

## Related tools

- **PRESCIENT** (Core generative framework; `prescient process_data` builds the data_pt object and the potential-energy model is trained on days 2/4/6 to simulate progenitor trajectories and predict fate bias) — https://github.com/gifford-lab/prescient
- **scanpy** (Recommended pre-processing of the lineage-tracing scRNA-seq matrix into normalized expression with time-point and cell type annotations)
- **PyTorch** (Backend for the serialized data_pt torch object and for training the potential-energy model)
- **scikit-learn** (Supplies the PCA decomposition (data_pt['pca']) fit to normalized expression and used to produce PCs)
- **UMAP** (Supplies the UMAP embedding (data_pt['um']) fit to PCs for visualization)
- **wot** (Baseline method (WOT) whose fate-bias predictions are compared against PRESCIENT's proliferation-aware runs) — https://github.com/broadinstitute/wot
- **prescient-analysis** (Provides weinreb-fate.sh (5-seed training script for clonal fate bias prediction) and the 03x-weinreb2020-fate preprocessing/analysis notebooks) — https://github.com/gifford-lab/prescient-analysis

## Examples

```
bash scripts/weinreb-fate.sh
```

## Evaluation signals

- fate_bias_metrics.csv exists and contains rows for all PRESCIENT variants plus the WOT, FateID, and clonal upper-bound baselines
- The proliferation-aware PRESCIENT variant achieves Pearson r and AUROC matching or approaching the Fig. 2f-h reported values (mean r ≈ 0.347 ± 0.029, AUROC ≈ 0.692 ± 0.012)
- Proliferation-aware runs (empirical and expression-derived growth rates) improve over the no-proliferation (w zeroed) run on Pearson r and AUROC
- PRESCIENT with proliferation outperforms the best-performing existing method (WOT) as reported in the paper
- The data_pt object contains the expected keys: data, celltype, genes, tps, x/xp/xu tensors, pca (sklearn PCA), um (umap.UMAP), and w growth weights

## Limitations

- The model requires the final time point of the dataset to be at steady-state
- Model performance depends on growth rates and would benefit from better estimates of proliferation
- Information is lost about individual genes when transforming data into PCA space, and lowly-expressed genes important to cell fate decisions may be dropped altogether
- Association of certain genes with specific cell fates does not necessarily imply causality, and challenges remain to confidently suggest gene sets for experimental perturbation

## Evidence

- [abstract] PRESCIENT is able to predict the fate biases of progenitor cells in hematopoiesis when accounting for cell proliferation, improving upon the best-performing existing method.: "PRESCIENT is able to predict the fate biases of progenitor cells in hematopoiesis when accounting for cell proliferation, improving upon the best-performing existing method."
- [methods] preprocessing formats: "For pre-processing, we recommend using Seurat or scanpy. PRESCIENT accepts the following formats: .csv, .tsv, .txt, .h5ad"
- [methods] process_data command: "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary) that contains all the necessary information for downstream training"
- [discussion] growth-rate benefit: "The model also improves when incorporating growth rates, and would likely benefit from better estimates of proliferation."
- [discussion] steady-state constraint: "the model is subject to constraints and assumptions, for example requiring that the final time point of the dataset be at steady-state."
- [readme] weinreb-fate training script: [evidence span withheld — anchor status: external_doc]
