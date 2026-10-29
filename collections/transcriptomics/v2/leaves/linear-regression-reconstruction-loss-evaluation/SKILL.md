---
name: linear-regression-reconstruction-loss-evaluation
description: Use when you have selected a minimal gene marker set from single-cell
  RNA-seq data (e.g., .h5ad loaded via get_citeseq/get_mouse_brain/get_paul/get_zeisel)
  and need to quantify whole-transcriptome reconstruction quality as ℓ2/ℓ1 losses.
when_to_use_negative:
- The goal is cell-type classification accuracy from markers (use new_model_metrics
  with a RandomForestClassifier instead of transcriptome reconstruction loss)
- The expression matrix must remain sparse or is too large to convert to dense via
  adata.X.toarray()
- Markers must be evaluated for their informativeness about cell type origin rather
  than genome-wide expression recovery
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: markermap
  role: Provides dataset loaders (get_citeseq, get_mouse_brain, get_paul, get_zeisel),
    split_data, and the unsupervised MarkerMap model (loss_tradeoff=1.0) whose k=50
    markers are evaluated by linear-regression reconstruction
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap
- name: numpy
  role: Dense array handling of gene count matrices after converting sparse adata.X,
    and numeric computation of ℓ2/ℓ1 losses
- name: scanpy
  role: Reads .h5ad into AnnData and supplies the highly-variable-genes marker-selection
    baseline
- name: anndata
  role: AnnData container holding adata.X gene counts and adata.obs annotations used
    by the loaders and splits
- name: persist
  role: Unsupervised PERSIST marker selection, one of the baselines whose 50 markers
    are scored by the regression reconstruction loss
  repo: https://github.com/iancovert/persist/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:15bbc0e3bd1dc4800cd03284b79fad3aacbb12a0208cbe5335528ed29ffed068
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap
  tools:
  - MarkerMap
  - anndata
  - markermap
  - numpy
  - persist
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# linear-regression-reconstruction-loss-evaluation

## Summary

Evaluates how well a small set of k markers (e.g., k=50 selected by unsupervised MarkerMap, RandomBaseline, Scanpy HVGs, or PERSIST) can reconstruct the full transcriptome by fitting a linear regression on the markers and reporting average ℓ2 and ℓ1 loss on held-out test cells across scRNA-seq datasets.

## When to use

Use when you have selected a minimal gene marker set from single-cell RNA-seq data (e.g., .h5ad loaded via get_citeseq/get_mouse_brain/get_paul/get_zeisel) and need to quantify whole-transcriptome reconstruction quality as ℓ2/ℓ1 losses.

## When NOT to use

- The goal is cell-type classification accuracy from markers (use new_model_metrics with a RandomForestClassifier instead of transcriptome reconstruction loss)
- The expression matrix must remain sparse or is too large to convert to dense via adata.X.toarray()
- Markers must be evaluated for their informativeness about cell type origin rather than genome-wide expression recovery

## Inputs

- Single-cell RNA-seq AnnData objects (.h5ad), e.g., CITE-seq (CITEseq.h5ad), Mouse Brain, Paul, and Zeisel datasets
- Selected marker indices (k=50) from MarkerMap, RandomBaseline, Scanpy HVGs, or PERSIST
- 70/10/20 train/validation/test index splits from split_data

## Outputs

- Per-dataset, per-method average ℓ2 and ℓ1 reconstruction losses on test cells (averaged over 10 runs)
- reconstruction_losses.csv with all dataset-method loss combinations

## How to apply

Load the CITE-seq, Mouse Brain, Paul, and Zeisel datasets with the markermap package loader functions, converting sparse matrices to dense with adata.X = adata.X.toarray(). Split each dataset 70%/10%/20% into train/validation/test using split_data, then select k=50 markers with each candidate method (unsupervised MarkerMap with loss_tradeoff=1.0, RandomBaseline, Scanpy highly-variable genes, unsupervised PERSIST). For each marker set, fit a linear regression on training cells mapping the 50 markers to the full gene expression vector and compute ℓ2 and ℓ1 reconstruction losses on the 20% test cells. Repeat over 10 random train/test splits and average losses per dataset-method pair, writing results to reconstruction_losses.csv for comparison against Table 2.

## Related tools

- **markermap** (Provides dataset loaders (get_citeseq, get_mouse_brain, get_paul, get_zeisel), split_data, and the unsupervised MarkerMap model (loss_tradeoff=1.0) whose k=50 markers are evaluated by linear-regression reconstruction) — https://github.com/Computational-Morphogenomics-Group/MarkerMap
- **numpy** (Dense array handling of gene count matrices after converting sparse adata.X, and numeric computation of ℓ2/ℓ1 losses)
- **scanpy** (Reads .h5ad into AnnData and supplies the highly-variable-genes marker-selection baseline)
- **anndata** (AnnData container holding adata.X gene counts and adata.obs annotations used by the loaders and splits)
- **persist** (Unsupervised PERSIST marker selection, one of the baselines whose 50 markers are scored by the regression reconstruction loss) — https://github.com/iancovert/persist/

## Examples

```
adata.X = adata.X.toarray(); train_indices, val_indices, test_indices = split_data(adata.X, adata.obs[group_by], [0.7, 0.1, 0.2])
```

## Evaluation signals

- reconstruction_losses.csv contains all dataset-method combinations (CITE-seq, Mouse Brain, Paul, Zeisel × 4 methods) with both ℓ2 and ℓ1 losses
- Averaged losses are stable across the 10 repeated random splits (low variance per method-dataset pair)
- Unsupervised MarkerMap losses land near the paper's reported values, e.g., (0.711, 0.652) on CITE-seq and (0.854, 0.721) on SSv4, on par with or better than Random Markers, Scanpy HVGs, and unsupervised PERSIST on most datasets
- Each split respects the 70%/10%/20% proportions and the regression is fit only on training cells with losses computed only on held-out test cells
- Sparse inputs were converted to dense (adata.X.toarray()) before loading, avoiding silent errors in the regression

## Limitations

- Results are averaged over 10 runs with random splits, so single-split results may differ from Table 2 values
- Only linear regression is used for reconstruction; nonlinear reconstructive power of the full generative model is not captured by this metric
- Google Colab users may need to restart the runtime after installing because scanpy installs a newer matplotlib; PERSIST requires a separate installation and GPU-compatible PyTorch

## Evidence

- [methods] To convert, merely do `adata.X = adata.X.toarray()`: "To convert, merely do `adata.X = adata.X.toarray()`"
- [readme] We then split the data into training, validation, and test sets with a 70%, 10%, 20% split.: "We then split the data into training, validation, and test sets with a 70%, 10%, 20% split."
- [readme] This model selects k markers at random, and it is always a good idea to include this one: "This model selects k markers at random, and it is always a good idea to include this one"
- [abstract] Here we introduce MarkerMap, a generative model for selecting minimal gene sets which are maximally informative of cell type origin and enable whole transcriptome reconstruction.: "Here we introduce MarkerMap, a generative model for selecting minimal gene sets which are maximally informative of cell type origin and enable whole transcriptome reconstruction."
- [readme] we run the benchmark by passing in all the models as a dictionary, the number of times to run the model: "we run the benchmark by passing in all the models as a dictionary, the number of times to run the model"
