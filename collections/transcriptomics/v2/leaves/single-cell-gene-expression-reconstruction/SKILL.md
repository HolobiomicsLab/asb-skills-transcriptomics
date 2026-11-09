---
name: single-cell-gene-expression-reconstruction
description: Use when benchmarking how well 50 markers selected by unsupervised methods
  (unsupervised MarkerMap, RandomBaseline, Scanpy HVGs, unsupervised PERSIST) reconstruct
  full transcriptome expression from CITE-seq, Mouse Brain, Paul, or Zeisel scRNA-seq
  data.
when_to_use_negative:
- The goal is cell-type classification from markers (misclassification rate, f1-score
  via new_model_metrics with a RandomForestClassifier) rather than full-transcriptome
  reconstruction loss
- Markers are already fixed and only supervised evaluation is needed — this skill
  specifically compares unsupervised marker selection methods
- Datasets outside the four benchmark sets (CITE-seq, Mouse Brain, Paul, Zeisel) are
  used without adapting the loaders and 70/10/20 split protocol
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: MarkerMap
  role: Unsupervised generative model (loss_tradeoff=1.0) selecting k=50 markers that
    enable whole transcriptome reconstruction; provides split_data and dataset loaders
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- name: numpy
  role: Dense array handling of adata.X and concatenation of train/val indices
- name: scanpy
  role: Reads .h5ad files into AnnData objects; highly-variable genes used as an unsupervised
    marker selection baseline
- name: anndata
  role: AnnData object format holding gene data (adata.X) and cell labels (adata.obs['annotation'])
- name: persist
  role: Unsupervised PERSIST marker selection baseline, installed from https://github.com/iancovert/persist/
  repo: https://github.com/iancovert/persist/
- name: RandomBaseline
  role: Control that selects k markers at random for benchmarking reconstruction loss
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:2f13260748aa025828e5817be527f29325e10f63e02f49ca467c901045e9fae3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
  tools:
  - MarkerMap
  - RandomBaseline
  - anndata
  - numpy
  - persist
  - scanpy
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# single-cell-gene-expression-reconstruction

## Summary

Reconstruct full single-cell RNA-seq transcriptomes from a small marker set (k=50) using linear regression, quantifying reconstruction quality with ℓ2 and ℓ1 losses averaged over 10 runs, as done in the MarkerMap paper's Table 2 for unsupervised MarkerMap, Random Markers, Scanpy HVGs, and unsupervised PERSIST.

## When to use

Use when benchmarking how well 50 markers selected by unsupervised methods (unsupervised MarkerMap, RandomBaseline, Scanpy HVGs, unsupervised PERSIST) reconstruct full transcriptome expression from CITE-seq, Mouse Brain, Paul, or Zeisel scRNA-seq data.

## When NOT to use

- The goal is cell-type classification from markers (misclassification rate, f1-score via new_model_metrics with a RandomForestClassifier) rather than full-transcriptome reconstruction loss
- Markers are already fixed and only supervised evaluation is needed — this skill specifically compares unsupervised marker selection methods
- Datasets outside the four benchmark sets (CITE-seq, Mouse Brain, Paul, Zeisel) are used without adapting the loaders and 70/10/20 split protocol

## Inputs

- Single-cell RNA-seq AnnData objects (.h5ad) for CITE-seq, Mouse Brain, Paul, and Zeisel datasets loaded via get_citeseq, get_mouse_brain, get_paul, get_zeisel
- Dense gene expression matrices (adata.X, converted from sparse via adata.X.toarray())
- k=50 marker sets from unsupervised MarkerMap (loss_tradeoff=1.0), RandomBaseline, Scanpy HVGs, and unsupervised PERSIST

## Outputs

- Per-dataset, per-method averaged ℓ2 and ℓ1 reconstruction losses over 10 runs (reconstruction_losses.csv)
- Comparison against Results Table 2 values, e.g. (0.711, 0.656) on CITE-seq and (0.854, 0.721) on SSv4 for unsupervised MarkerMap

## How to apply

Load the CITE-seq, Mouse Brain, Paul, and Zeisel datasets via the markermap package loaders (get_citeseq, get_mouse_brain, get_paul, get_zeisel), converting any sparse matrices to dense arrays with adata.X = adata.X.toarray(). Split each dataset into training, validation, and test sets with the 70%, 10%, 20% split using split_data. On each dataset select k=50 markers with each method: unsupervised MarkerMap (loss_tradeoff=1.0, z_size=16, hidden_layer_size≈10% of input dimension), RandomBaseline (selects k markers at random), Scanpy highly-variable genes, and unsupervised PERSIST. For each marker set, fit a linear regression on training cells to predict full gene expression from the 50 markers, then evaluate ℓ2 and ℓ1 losses on the 20% held-out test cells. Repeat over 10 random splits, average the losses per dataset and method, and write reconstruction_losses.csv for comparison against Results Table 2 values (e.g. unsupervised MarkerMap achieves (0.711, 0.656) on CITE-seq and (0.854, 0.721) on SSv4).

## Related tools

- **MarkerMap** (Unsupervised generative model (loss_tradeoff=1.0) selecting k=50 markers that enable whole transcriptome reconstruction; provides split_data and dataset loaders) — https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- **numpy** (Dense array handling of adata.X and concatenation of train/val indices)
- **scanpy** (Reads .h5ad files into AnnData objects; highly-variable genes used as an unsupervised marker selection baseline)
- **anndata** (AnnData object format holding gene data (adata.X) and cell labels (adata.obs['annotation']))
- **persist** (Unsupervised PERSIST marker selection baseline, installed from https://github.com/iancovert/persist/) — https://github.com/iancovert/persist/
- **RandomBaseline** (Control that selects k markers at random for benchmarking reconstruction loss) — https://github.com/Computational-Morphogenomics-Group/MarkerMap.git

## Examples

```
from markermap.utils import split_data; train_indices, val_indices, test_indices = split_data(adata.X, adata.obs[group_by], [0.7, 0.1, 0.2])
```

## Evaluation signals

- reconstruction_losses.csv contains an ℓ2 and ℓ1 loss for every dataset-method combination (4 datasets × 4 methods)
- Averaged losses reproduce Table 2 magnitudes: unsupervised MarkerMap ≈ (0.711, 0.656) on CITE-seq and (0.854, 0.721) on SSv4
- Each reported loss is a mean over 10 random train/test splits, not a single run
- Split proportions match 70% training, 10% validation, 20% test via split_data
- Sparse matrices were converted to dense with adata.X = adata.X.toarray() before fitting the linear regression

## Limitations

- Results depend on random train/test splits; losses must be averaged over 10 runs for stability
- Unsupervised MarkerMap performs on par with or better than Random Markers, Scanpy HVGs, and unsupervised PERSIST only on most, not all, datasets
- Scanpy may install a newer matplotlib requiring runtime restart (e.g. on Google Colab); PERSIST requires a separate install and a GPU-compatible PyTorch

## Evidence

- [results] Table 2 quantifies full-transcriptome reconstruction by reporting the average ℓ2 and ℓ1 loss of single-cell RNA-seq datasets reconstructed from 50 selected markers using a linear regression model: [evidence span withheld — anchor status: not_found]
- [readme] We then split the data into training, validation, and test sets with a 70%, 10%, 20% split.: "We then split the data into training, validation, and test sets with a 70%, 10%, 20% split."
- [methods] To convert, merely do `adata.X = adata.X.toarray()`: "To convert, merely do `adata.X = adata.X.toarray()`"
- [abstract] Here we introduce MarkerMap, a generative model for selecting minimal gene sets which are maximally informative of cell type origin and enable whole transcriptome reconstruction.: "Here we introduce MarkerMap, a generative model for selecting minimal gene sets which are maximally informative of cell type origin and enable whole transcriptome reconstruction."
- [readme] This model selects k markers at random, and it is always a good idea to include this one: "This model selects k markers at random, and it is always a good idea to include this one"
