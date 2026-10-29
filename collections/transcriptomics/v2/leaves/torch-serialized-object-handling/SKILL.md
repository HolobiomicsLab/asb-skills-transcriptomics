---
name: torch-serialized-object-handling
description: Use when resuming PRESCIENT workflows from the `prescient process_data`
  output or when a downstream task (fate simulation, TF perturbation screen, UMAP
  plotting) needs the bundled data_pt contents.
when_to_use_negative:
- Raw, unnormalized scRNA-seq files that have not been pre-processed with Seurat or
  scanpy — process_data expects normalized inputs
- Analyses needing full per-gene resolution outside PCA space — the article notes
  information is lost about individual genes when transforming data into PCA space
- Datasets whose final time point is not at steady-state — the PRESCIENT model constraint
  would make downstream training from data_pt invalid
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: PRESCIENT
  role: Provides the `prescient process_data` command that generates the data_pt torch
    object consumed by this skill
  repo: https://github.com/gifford-lab/prescient
- name: PyTorch
  role: Serialization format of data_pt; torch.load is used to read the .pt file and
    the object feeds PRESCIENT torch-based training
- name: scikit-learn
  role: data_pt['pca'] holds the sklearn.decomposition.PCA object fit to normalized
    expression and used to produce PCs
- name: UMAP
  role: data_pt['um'] holds the umap.UMAP object fit to PCs used to produce UMAP dimensions
- name: scanpy
  role: Recommended pre-processing tool for generating normalized expression inputs
    accepted by process_data (.h5ad)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:02ef2eb7caf4be9843d359b11882ead1aca7414a851f6ca9e2a448a3783aaed8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gifford-lab/prescient
  tools:
  - PRESCIENT
  - PyTorch
  - UMAP
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# torch-serialized-object-handling

## Summary

Load and work with the PRESCIENT serialized PyTorch object (data_pt), a torch .pt dictionary containing normalized gene expression, gene features, cell type annotations, time-point labels, and fitted PCA/UMAP objects, to enable downstream training, simulation, and perturbation screens.

## When to use

Use when resuming PRESCIENT workflows from the `prescient process_data` output or when a downstream task (fate simulation, TF perturbation screen, UMAP plotting) needs the bundled data_pt contents.

## When NOT to use

- Raw, unnormalized scRNA-seq files that have not been pre-processed with Seurat or scanpy — process_data expects normalized inputs
- Analyses needing full per-gene resolution outside PCA space — the article notes information is lost about individual genes when transforming data into PCA space
- Datasets whose final time point is not at steady-state — the PRESCIENT model constraint would make downstream training from data_pt invalid

## Inputs

- Pre-processed scRNA-seq data in .csv, .tsv, .txt, or .h5ad format (normalized gene expression, time-point labels, cell type annotations)
- Serialized PRESCIENT torch .pt file (data_pt) produced by `prescient process_data`
- Optional candidate gene list (e.g., transcription factors) to intersect with data_pt gene features

## Outputs

- In-memory data_pt dictionary with normalized expression matrix, gene features, cell type labels, timepoint assignments, PCA object, and UMAP object
- Gene feature intersection (e.g., TF list overlapping data_pt features) used for perturbation simulations

## How to apply

Generate the object with the `prescient process_data` command, which writes a serialized torch .pt file named data_pt; load it with torch.load rather than parsing raw files. Treat data_pt as a dictionary: access normalized gene expression and the gene feature list, cell type labels, time-point assignments, and the sklearn.decomposition.PCA object (data_pt['pca']) and umap.UMAP object (data_pt['um']). Inputs must first be pre-processed with Seurat or scanpy into accepted formats (.csv, .tsv, .txt, .h5ad), since training requires only normalized gene expression, time-point labels, and cell type annotations. For perturbation tasks, intersect data_pt gene features with a candidate list (e.g., transcription factors from the Veres et al. in vitro beta-cell differentiation highly variable gene set) before simulating z=5 overexpression trajectories over multiple random initializations. Verify integrity by confirming the PCA and UMAP objects are fitted and that feature names align between the expression matrix and downstream gene queries.

## Related tools

- **PRESCIENT** (Provides the `prescient process_data` command that generates the data_pt torch object consumed by this skill) — https://github.com/gifford-lab/prescient
- **PyTorch** (Serialization format of data_pt; torch.load is used to read the .pt file and the object feeds PRESCIENT torch-based training)
- **scikit-learn** (data_pt['pca'] holds the sklearn.decomposition.PCA object fit to normalized expression and used to produce PCs)
- **UMAP** (data_pt['um'] holds the umap.UMAP object fit to PCs used to produce UMAP dimensions)
- **scanpy** (Recommended pre-processing tool for generating normalized expression inputs accepted by process_data (.h5ad))

## Examples

```
import torch; data_pt = torch.load('data_pt'); genes = data_pt['features']; pca, umap_obj = data_pt['pca'], data_pt['um']
```

## Evaluation signals

- data_pt loads successfully via torch.load as a Python dictionary with keys for expression, gene features, cell type labels, timepoints, 'pca', and 'um'
- data_pt['pca'] is a fitted sklearn.decomposition.PCA object and data_pt['um'] is a fitted umap.UMAP object
- Gene feature list length matches the expression matrix dimensionality, and candidate TF queries intersect non-trivially with data_pt features
- Time-point labels and cell type annotations cover all cells in the expression matrix with expected counts per Veres et al. or Weinreb et al. dataset
- Downstream PRESCIENT simulations train and run without key errors, confirming the serialized object is complete

## Limitations

- Information is lost about individual genes when transforming data into PCA space, so gene-level queries may miss signal
- Lowly-expressed genes important to cell fate decisions may be dropped altogether in the scRNA-seq data
- The model requires the final time point of the dataset to be at steady-state, constraining which datasets can be serialized and trained

## Evidence

- [methods] process_data command generates data_pt dictionary: "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary) that contains all the necessary information for downstream training, simulations, and"
- [methods] PRESCIENT training inputs: "PRESCIENT takes as input longitudinal scRNA-seq data. For training, all that is needed is normalized gene expression, time-point labels, and cell type annotations."
- [methods] PCA object stored in data_pt: "data_pt["pca"]: sklearn.decomposition.PCA object fit to normalized expression and used to produce PCs."
- [methods] UMAP object stored in data_pt: "data_pt["um"]: umap.UMAP object fit to PCs used to produce UMAP dims."
- [methods] accepted input formats: "For pre-processing, we recommend using Seurat or scanpy. PRESCIENT accepts the following formats: .csv, .tsv, .txt, .h5ad"
- [discussion] PCA information loss caveat: "information is lost about individual genes when transforming data into PCA space"
