---
name: cross-model-reconstruction-evaluation
description: Use when comparing marker-selection or latent-variable models on their
  ability to reconstruct full gene expression from a small marker panel (k markers),
  with per-cell-type and overall reconstruction metrics such as Jaccard Index, Spearman
  rho, and l2 distance.
when_to_use_negative:
- The goal is cell-type classification accuracy from markers (use new_model_metrics
  with a RandomForestClassifier instead of reconstruction metrics)
- You need to benchmark marker counts over a range of k values rather than compare
  two models at fixed k
- The data cannot be converted to a dense numpy array and the models require dense
  input
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: MarkerMap
  role: Unsupervised generative model (loss_tradeoff=1, z_size=16, hidden_layer_size=64,
    k=50) whose selected markers are used to reconstruct the full transcriptome
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- name: scanpy
  role: Loads the Mouse Brain dataset into an AnnData object via get_mouse_brain;
    adata.X is densified with .toarray() if sparse
- name: numpy
  role: Handles dense gene expression arrays used for splitting and reconstruction
    metric computation
- name: anndata
  role: AnnData container holding adata.X gene counts and adata.obs cell labels for
    the evaluation splits
- name: pip
  role: Installation mechanism for the markermap package (pip install markermap)
  repo: https://pypi.org/project/markermap/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:bfee3fb8721871d7bc73def9fa8f0cf24a4f1e659090a4c0fcbc5a797c7f5b3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
  tools:
  - MarkerMap
  - anndata
  - numpy
  - pip
  - scanpy
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cross-model-reconstruction-evaluation

## Summary

A benchmarking procedure that compares how well different marker-selection models (e.g., unsupervised MarkerMap vs scVI) reconstruct the full transcriptome from only k=50 selected markers, using Jaccard Index, Spearman rho, and l2 distance metrics averaged over 5 random 80/20 splits, as in Table 4 of the MarkerMap paper for the Mouse Brain dataset.

## When to use

Use when comparing marker-selection or latent-variable models on their ability to reconstruct full gene expression from a small marker panel (k markers), with per-cell-type and overall reconstruction metrics such as Jaccard Index, Spearman rho, and l2 distance.

## When NOT to use

- The goal is cell-type classification accuracy from markers (use new_model_metrics with a RandomForestClassifier instead of reconstruction metrics)
- You need to benchmark marker counts over a range of k values rather than compare two models at fixed k
- The data cannot be converted to a dense numpy array and the models require dense input

## Inputs

- Single-cell RNA-seq dataset as scanpy AnnData (e.g., Mouse Brain via get_mouse_brain), with adata.X as dense numpy array
- Cell type labels (adata.obs['annotation']) for per-cell-type metric computation
- Number of markers k (k=50) and number of random splits (5)

## Outputs

- Per-split and averaged Jaccard Index, Spearman rho, and l2 distance for each model (unsupervised MarkerMap, scVI), per cell type and overall
- Table 4-style comparison CSV: reconstruction_metrics_mouse_brain.csv

## How to apply

Load the dataset (e.g., Mouse Brain) with the markermap package's get_mouse_brain loader into a scanpy AnnData object, converting adata.X to a dense numpy array with .toarray() if sparse. Split cells into 80% train / 20% test over 5 random splits and build train_dataloader/val_dataloader with MarkerMap.prepareData. Train unsupervised MarkerMap with loss_tradeoff=1, z_size=16, hidden_layer_size=64, and k=50 markers, and train scVI on the same splits, extracting its top k=50 marker genes as the comparator. For each model and split, reconstruct the full transcriptome from the k=50 markers and compute Jaccard Index, Spearman rho, and l2 distance between original and reconstructed data per cell type and overall. Average metrics across the 5 splits and write the Table 4-style comparison to reconstruction_metrics_mouse_brain.csv. Rationale: variance-based metrics (Jaccard, Spearman rho) capture agreement in expression variation while l2 distance captures direct reconstruction error, so models can trade off across them.

## Related tools

- **MarkerMap** (Unsupervised generative model (loss_tradeoff=1, z_size=16, hidden_layer_size=64, k=50) whose selected markers are used to reconstruct the full transcriptome) — https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- **scanpy** (Loads the Mouse Brain dataset into an AnnData object via get_mouse_brain; adata.X is densified with .toarray() if sparse)
- **numpy** (Handles dense gene expression arrays used for splitting and reconstruction metric computation)
- **anndata** (AnnData container holding adata.X gene counts and adata.obs cell labels for the evaluation splits)
- **pip** (Installation mechanism for the markermap package (pip install markermap)) — https://pypi.org/project/markermap/

## Examples

```
from markermap.utils import split_data; train_indices, val_indices, test_indices = split_data(adata.X, adata.obs[group_by], [0.8, 0, 0.2]); train_dataloader, val_dataloader = MarkerMap.prepareData(adata, train_indices, val_indices, group_by, None, batch_size=64)
```

## Evaluation signals

- Reproduced table should match the Table 4 trend: unsupervised MarkerMap attains the better (lower) l2 reconstruction distance while scVI scores better on the variance-based metrics (Jaccard Index, Spearman rho)
- All metrics are averaged over 5 random 80/20 train/test splits before comparison
- Reconstruction metrics are computed between the original and reconstructed full transcriptome, both per cell type and overall
- Both MarkerMap and scVI must be evaluated on the same splits with the same k=50 markers
- Output CSV (reconstruction_metrics_mouse_brain.csv) contains columns for all three metrics per model

## Limitations

- With only 50 markers, unsupervised MarkerMap generally scores worse than scVI on variance-based metrics (Jaccard Index, Spearman rho) even though it wins on l2 distance — neither model dominates on all metrics
- Results depend on the random train/test splits; the protocol averages over 5 splits to mitigate split-specific variance
- Installation issues can arise on Google Colab because scanpy installs a newer version of matplotlib than the default one (runtime restart may be needed)

## Evidence

- [results] Compared to scVI, unsupervised MarkerMap with 50 markers on the Mouse Brain data generally scores worse on the variance-based metrics (Jaccard Index and Spearman ρ) but better on the ℓ2 distance (Table 4).: [evidence span withheld — anchor status: not_found]
- [methods] Data is handled with numpy and with scanpy which is used for many computational biology datasets.: "Data is handled with numpy and with scanpy which is used for many computational biology datasets."
- [methods] To convert, merely do `adata.X = adata.X.toarray()`: "To convert, merely do `adata.X = adata.X.toarray()`"
- [readme] The easiest way to install is with pip from https://pypi.org/project/markermap/: "The easiest way to install is with pip from https://pypi.org/project/markermap/"
