---
name: potential-landscape-model-training
description: Use when you have preprocessed longitudinal scRNA-seq (normalized expression,
  integer timepoint labels, cell type annotations) and need to fit a potential landscape
  for fate prediction or perturbation simulation.
when_to_use_negative:
- The dataset's final timepoint is not at steady-state — the model is explicitly constrained
  by this assumption
- The goal is gene-level interpretability of the trained model — information is lost
  about individual genes when data are transformed into PCA space
- Only cross-sectional (single-timepoint) scRNA-seq data are available — PRESCIENT
  requires longitudinal time-series input
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: PRESCIENT
  role: Generative framework whose `prescient process_data` command builds the torch
    data_pt object used for downstream training and simulations
  repo: https://github.com/gifford-lab/prescient
- name: scanpy
  role: Recommended preprocessing of normalized expression, timepoint labels, and
    cell annotations prior to process_data
- name: Seurat
  role: Alternative R preprocessing of the scRNA-seq time course before training
- name: PyTorch
  role: Fits the potential-energy model on the serialized data_pt dictionary across
    all 7 timepoints
- name: scikit-learn
  role: Provides the PCA object fit to normalized expression and stored in data_pt['pca']
- name: UMAP
  role: Provides the UMAP object fit to PCs and stored in data_pt['um']
- name: prescient-analysis
  role: Contains veres-fate.sh (trains 5 seeds of 2-layer 400-unit models with estimated
    proliferation rates) and preprocessing/visualization notebooks
  repo: https://github.com/gifford-lab/prescient-analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:c124ca1429a845a726f954624f743dd97a2a81da9d75d4ada9337db58bb35a1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gifford-lab/prescient
  tools:
  - PRESCIENT
  - PyTorch
  - Seurat
  - UMAP
  - prescient-analysis
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# potential-landscape-model-training

## Summary

Train the PRESCIENT potential-energy generative model on longitudinal scRNA-seq data (e.g. the 7-timepoint Veres et al. in vitro pancreatic beta-cell differentiation time course) so that fate-bias and in silico perturbation simulations can be run downstream.

## When to use

Use when you have preprocessed longitudinal scRNA-seq (normalized expression, integer timepoint labels, cell type annotations) and need to fit a potential landscape for fate prediction or perturbation simulation.

## When NOT to use

- The dataset's final timepoint is not at steady-state — the model is explicitly constrained by this assumption
- The goal is gene-level interpretability of the trained model — information is lost about individual genes when data are transformed into PCA space
- Only cross-sectional (single-timepoint) scRNA-seq data are available — PRESCIENT requires longitudinal time-series input

## Inputs

- Normalized scRNA-seq gene expression matrix (accepted formats: .csv, .tsv, .txt, .h5ad)
- Integer timepoint labels for each cell (e.g. 0-6 for the Veres et al. differentiation time course)
- Cell type annotations for each cell
- Growth/proliferation rate estimates per cell (e.g. from cell-cycle scoring)

## Outputs

- data_pt serialized torch dictionary containing the sklearn PCA object, UMAP object, celltype labels, gene features, timepoint assignments, and growth weights
- Trained PRESCIENT potential-energy model (2-layer 400-unit network, multiple seeds)

## How to apply

Preprocess the scRNA-seq time course with Seurat or scanpy, then run `prescient process_data` to generate the serialized torch data_pt dictionary, which stores a sklearn PCA fit to the normalized expression, a UMAP object fit to the PCs, celltype labels, gene features, timepoint assignments, and growth weights. Train the potential-energy model with PyTorch on all 7 timepoints using the 2-layer 400-unit architecture with estimated proliferation rates (the prescient-analysis veres-fate.sh workflow trains 5 seeds). The model requires the final timepoint to be at steady-state, which is a key precondition for the Veres et al. dataset. Validate training by checking that simulated trajectories reproduce expected cell-type fractions at the terminal timepoint before using the model for perturbation ensembles.

## Related tools

- **PRESCIENT** (Generative framework whose `prescient process_data` command builds the torch data_pt object used for downstream training and simulations) — https://github.com/gifford-lab/prescient
- **scanpy** (Recommended preprocessing of normalized expression, timepoint labels, and cell annotations prior to process_data)
- **Seurat** (Alternative R preprocessing of the scRNA-seq time course before training)
- **PyTorch** (Fits the potential-energy model on the serialized data_pt dictionary across all 7 timepoints)
- **scikit-learn** (Provides the PCA object fit to normalized expression and stored in data_pt['pca'])
- **UMAP** (Provides the UMAP object fit to PCs and stored in data_pt['um'])
- **prescient-analysis** (Contains veres-fate.sh (trains 5 seeds of 2-layer 400-unit models with estimated proliferation rates) and preprocessing/visualization notebooks) — https://github.com/gifford-lab/prescient-analysis

## Examples

```
prescient process_data --input veres_normalized.h5ad --timepoints time_labels --celltypes cell_annotations --growth growth_rates --out data_pt
```

## Evaluation signals

- data_pt output contains the expected keys: 'pca' (sklearn PCA object), 'um' (umap.UMAP object), celltype labels, gene features, timepoint assignments, and growth weights
- Training follows the veres-fate.sh pattern: 5 seeds of 2-layer 400-unit models trained with estimated proliferation rates on the Veres et al. dataset
- Simulated trajectories from the trained model reproduce expected endocrine/exocrine cell-type fractions at the final timepoint before perturbations are applied
- Downstream perturbation simulations recover expected fate shifts (e.g. NEUROG3/NKX6.1 increasing endocrine fractions at p < 0.05), indicating the landscape was fit correctly

## Limitations

- The model requires the final time point of the dataset to be at steady-state
- Model performance depends on growth rates and would benefit from better estimates of proliferation
- Information is lost about individual genes when transforming data into PCA space
- Lowly-expressed genes important to cell fate decisions may be dropped altogether in the scRNA-seq data
- Association of certain genes with specific cell fates does not necessarily imply causality

## Evidence

- [methods] Inputs from article methods: "For training, all that is needed is normalized gene expression, time-point labels, and cell type annotations."
- [methods] process_data command: "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary) that contains all the necessary information for downstream training, simulations"
- [methods] Preprocessing formats: "For pre-processing, we recommend using Seurat or scanpy. PRESCIENT accepts the following formats: .csv, .tsv, .txt, .h5ad"
- [methods] PCA and UMAP storage: "data_pt["pca"]: sklearn.decomposition.PCA object fit to normalized expression and used to produce PCs."
- [readme] veres-fate training script: [evidence span withheld — anchor status: external_doc]
- [discussion] Steady-state requirement: "the model is subject to constraints and assumptions, for example requiring that the final time point of the dataset be at steady-state."
