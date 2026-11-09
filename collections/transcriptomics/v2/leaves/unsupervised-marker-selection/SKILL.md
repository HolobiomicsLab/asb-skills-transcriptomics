---
name: unsupervised-marker-selection
description: Use when you must pick a minimal gene panel (e.g. k=50 markers) from
  scRNA-seq data without cell-type labels, and want to benchmark unsupervised selectors
  on full-transcriptome reconstruction loss.
when_to_use_negative:
- Cell-type labels are available and the goal is cell-type classification — use supervised
  (loss_tradeoff=0) or mixed (loss_tradeoff=0.5) MarkerMap instead, as in the README's
  three tradeoff levels
- The task is cluster annotation or differential expression ranking (e.g. COSG-style
  marker identification) rather than choosing a minimal panel predictive of the whole
  transcriptome
- Input is not single-cell expression data in an AnnData-compatible format, or the
  sparse-to-dense conversion (adata.X.toarray()) is infeasible because the dataset
  is too large for memory
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: MarkerMap
  role: Unsupervised marker selection (loss_tradeoff=1.0) via generative VAE; provides
    getBenchmarker, split_data, prepareData, and dataset loaders
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- name: persist
  role: Unsupervised PERSIST baseline for predictive gene selection compared against
    unsupervised MarkerMap
  repo: https://github.com/iancovert/persist/
- name: scanpy
  role: Reads h5ad data into AnnData objects; supplies the highly-variable genes (HVGs)
    baseline marker set
- name: anndata
  role: AnnData object holding adata.X expression and adata.obs['annotation'] labels
    for splitting and marker extraction
- name: numpy
  role: Handles dense gene-count arrays, e.g. concatenating train/val indices after
    split_data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:17ab432a50e36ae256d8dcb0fdecb26a2bec41a399518b781dee759f015a75c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
  tools:
  - MarkerMap
  - anndata
  - numpy
  - persist
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# unsupervised-marker-selection

## Summary

Select a small set of k gene markers without using cell labels by benchmarking unsupervised MarkerMap (loss_tradeoff=1.0) against RandomBaseline, Scanpy highly-variable genes, and unsupervised PERSIST, then judge marker quality by how well a linear regression reconstructs the full transcriptome from the k markers (average ℓ2/ℓ1 loss over 10 runs).

## When to use

Use when you must pick a minimal gene panel (e.g. k=50 markers) from scRNA-seq data without cell-type labels, and want to benchmark unsupervised selectors on full-transcriptome reconstruction loss.

## When NOT to use

- Cell-type labels are available and the goal is cell-type classification — use supervised (loss_tradeoff=0) or mixed (loss_tradeoff=0.5) MarkerMap instead, as in the README's three tradeoff levels
- The task is cluster annotation or differential expression ranking (e.g. COSG-style marker identification) rather than choosing a minimal panel predictive of the whole transcriptome
- Input is not single-cell expression data in an AnnData-compatible format, or the sparse-to-dense conversion (adata.X.toarray()) is infeasible because the dataset is too large for memory

## Inputs

- Single-cell RNA-seq AnnData objects (CITE-seq CITEseq.h5ad, Mouse Brain, Paul, Zeisel) read via scanpy, with adata.X gene expression and labels in adata.obs['annotation']
- Number of markers k=50
- Train/val/test split proportions [0.7, 0.1, 0.2]

## Outputs

- Selected marker gene sets (k=50) per method per dataset
- Averaged ℓ2 and ℓ1 full-transcriptome reconstruction losses per dataset-method combination (over 10 runs), written to reconstruction_losses.csv
- Comparison against Results Table 2 (e.g. unsupervised MarkerMap ℓ2/ℓ1 = (0.711, 0.656) on CITE-seq and (0.854, 0.721) on SSv4)

## How to apply

Load each single-cell dataset (CITE-seq, Mouse Brain, Paul, Zeisel) with the MarkerMap package loaders (get_citeseq, get_mouse_brain, get_paul, get_zeisel), converting sparse matrices to dense via adata.X = adata.X.toarray(). Split each dataset into training, validation, and test sets with a 70%, 10%, 20% split using split_data. Select k=50 markers with each method: unsupervised MarkerMap (loss_tradeoff=1.0, i.e. unsupervised only per the README's 0/0.5/1 tradeoff levels), RandomBaseline (selects k markers at random), Scanpy highly-variable genes, and unsupervised PERSIST (installed from github.com/iancovert/persist). For each marker set, fit a linear regression on training cells to reconstruct full gene expression from the 50 markers and evaluate ℓ2 and ℓ1 losses on the held-out 20% test cells. Repeat selection and reconstruction over 10 random train/test splits, average losses per dataset-method combination, and write results to reconstruction_losses.csv for comparison against Results Table 2.

## Related tools

- **MarkerMap** (Unsupervised marker selection (loss_tradeoff=1.0) via generative VAE; provides getBenchmarker, split_data, prepareData, and dataset loaders) — https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- **persist** (Unsupervised PERSIST baseline for predictive gene selection compared against unsupervised MarkerMap) — https://github.com/iancovert/persist/
- **scanpy** (Reads h5ad data into AnnData objects; supplies the highly-variable genes (HVGs) baseline marker set)
- **anndata** (AnnData object holding adata.X expression and adata.obs['annotation'] labels for splitting and marker extraction)
- **numpy** (Handles dense gene-count arrays, e.g. concatenating train/val indices after split_data)

## Examples

```
from markermap.vae_models import MarkerMap, train_model; unsup_mm = MarkerMap.getBenchmarker(create_kwargs={'input_size': adata.shape[1], 'hidden_layer_size': 64, 'z_size': 16, 'num_classes': None, 'k': 50, 'loss_tradeoff': 1.0})
```

## Evaluation signals

- Each selected marker set contains exactly k=50 genes per dataset per method
- Averaged ℓ2/ℓ1 losses per dataset-method combination fall in line with Table 2 values (e.g. unsupervised MarkerMap ≈ (0.711, 0.656) on CITE-seq, (0.854, 0.721) on SSv4)
- Losses are averaged over 10 runs of marker selection plus reconstruction, not a single split
- Evaluation uses only the 20% held-out test cells after a 70/10/20 split, never the train/val cells
- reconstruction_losses.csv contains entries for all dataset-method combinations (CITE-seq, Mouse Brain, Paul, Zeisel × unsup MarkerMap, Random Markers, Scanpy HVGs, unsup PERSIST)

## Limitations

- Unsupervised MarkerMap performs on par with or better than baselines only on most datasets — Random Markers, Scanpy HVGs, or unsupervised PERSIST may win on specific datasets
- Sparse expression matrices must first be converted to dense arrays, which may not scale to very large datasets
- On Google Colab, the runtime may need restarting after installation because scanpy installs a newer matplotlib than the default
- Reconstruction performance depends on k; k=50 is the evaluated setting and other panel sizes were not validated here

## Evidence

- [abstract] Here we introduce MarkerMap, a generative model for selecting minimal gene sets which are maximally informative of cell type origin and enable whole transcriptome reconstruction.: "Here we introduce MarkerMap, a generative model for selecting minimal gene sets which are maximally informative of cell type origin and enable whole transcriptome reconstruction."
- [results] Table 2 quantifies full-transcriptome reconstruction by reporting the average ℓ2 and ℓ1 loss of single-cell RNA-seq datasets reconstructed from 50 selected markers using a linear regression model: [evidence span withheld — anchor status: not_found]
- [readme] Loss_tradeoff is also important, see the paper for a further discussion. In general, we have 3 levels, 0 (supervised only), 0.5 (mixed supervised-unsupervised) and 1 (unsupervised only).: "In general, we have 3 levels, 0 (supervised only), 0.5 (mixed supervised-unsupervised) and 1 (unsupervised only)."
- [methods] We then split the data into training, validation, and test sets with a 70%, 10%, 20% split.: "We then split the data into training, validation, and test sets with a 70%, 10%, 20% split."
- [methods] To convert, merely do `adata.X = adata.X.toarray()`: "To convert, merely do `adata.X = adata.X.toarray()`"
