---
name: scanpy-hvg-marker-selection
description: Use when benchmarking unsupervised marker selection for single-cell RNA-seq
  data — e.g., reconstructing the full transcriptome from k=50 markers on CITE-seq,
  Mouse Brain, Paul, or Zeisel datasets — and a Scanpy HVG baseline is needed to compare
  against MarkerMap, RandomBaseline, or PERSIST.
when_to_use_negative:
- The goal is supervised marker selection for cell-type classification — Scanpy HVGs
  are unsupervised variance-based picks and ignore labels in adata.obs['annotation']
- The task requires cell-type-specific markers informative of cell origin rather than
  genome-wide reconstruction baselines
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scanpy
  role: Provides the highly variable gene (HVG) selection baseline and handles data
    loading (sc.read_h5ad returning an AnnData object)
- name: markermap
  role: Package supplying dataset loaders (get_citeseq, get_mouse_brain, get_paul,
    get_zeisel), split_data, and the unsupervised MarkerMap comparator benchmarked
    against Scanpy HVGs
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- name: numpy
  role: Array handling for dense conversion (adata.X.toarray()) and concatenating
    train/val indices
- name: persist
  role: Unsupervised PERSIST comparator method in the same reconstruction-loss benchmark
  repo: https://github.com/iancovert/persist/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:9c05f7dd4d09dfa89936c54aab10fb49302d3a0abb0b7eb0fe581dd29e23277b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
  tools:
  - MarkerMap
  - markermap
  - numpy
  - persist
  - scanpy
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# scanpy-hvg-marker-selection

## Summary

Selects the top k highly variable genes with Scanpy as a baseline marker set, used in the MarkerMap paper to benchmark full-transcriptome reconstruction (average ℓ2 and ℓ1 loss from a linear regression on 50 markers) against unsupervised MarkerMap, Random Markers, and unsupervised PERSIST.

## When to use

Use when benchmarking unsupervised marker selection for single-cell RNA-seq data — e.g., reconstructing the full transcriptome from k=50 markers on CITE-seq, Mouse Brain, Paul, or Zeisel datasets — and a Scanpy HVG baseline is needed to compare against MarkerMap, RandomBaseline, or PERSIST.

## When NOT to use

- The goal is supervised marker selection for cell-type classification — Scanpy HVGs are unsupervised variance-based picks and ignore labels in adata.obs['annotation']
- The task requires cell-type-specific markers informative of cell origin rather than genome-wide reconstruction baselines

## Inputs

- Single-cell RNA-seq AnnData objects (CITE-seq, Mouse Brain, Paul, Zeisel .h5ad files) with gene expression in adata.X
- Marker count k=50
- Train/validation/test split proportions [0.7, 0.1, 0.2]

## Outputs

- A set of 50 highly variable gene markers per dataset
- Average ℓ2 and ℓ1 reconstruction losses per dataset per method (written to reconstruction_losses.csv)

## How to apply

Load the scRNA-seq dataset (CITE-seq, Mouse Brain, Paul, or Zeisel) via the MarkerMap package loader functions (get_citeseq, get_mouse_brain, get_paul, get_zeisel), converting sparse matrices to dense with adata.X = adata.X.toarray(). Split into 70%/10%/20% train/validation/test with split_data. Select k=50 markers using Scanpy's highly-variable-genes functionality, then fit a linear regression on the training cells to reconstruct full gene expression from the 50 markers and record ℓ2 and ℓ1 reconstruction losses on held-out test cells. Average losses over 10 random train/test splits and write all dataset-method combinations to reconstruction_losses.csv for comparison against Results Table 2, where Scanpy HVGs should perform on par with or worse than unsupervised MarkerMap (e.g., MarkerMap achieves (0.711, 0.656) on CITE-seq and (0.854, 0.721) on SSv4).

## Related tools

- **scanpy** (Provides the highly variable gene (HVG) selection baseline and handles data loading (sc.read_h5ad returning an AnnData object))
- **markermap** (Package supplying dataset loaders (get_citeseq, get_mouse_brain, get_paul, get_zeisel), split_data, and the unsupervised MarkerMap comparator benchmarked against Scanpy HVGs) — https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- **numpy** (Array handling for dense conversion (adata.X.toarray()) and concatenating train/val indices)
- **persist** (Unsupervised PERSIST comparator method in the same reconstruction-loss benchmark) — https://github.com/iancovert/persist/

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('data/cite_seq/CITEseq.h5ad'); adata.X = adata.X.toarray(); sc.pp.highly_variable_genes(adata, n_top_genes=50)
```

## Evaluation signals

- Markers set size equals k=50 highly variable genes per dataset
- Reconstruction losses averaged over 10 random train/test splits for each dataset-method combination
- Scanpy HVG ℓ2/ℓ1 losses should be comparable to or higher than unsupervised MarkerMap values (e.g., MarkerMap (0.711, 0.656) on CITE-seq, (0.854, 0.721) on SSv4) in Results Table 2
- All dataset-method loss rows present in reconstruction_losses.csv before comparison against Table 2

## Limitations

- HVGs are variance-based and not optimized for whole-transcriptome reconstruction, so they may underperform unsupervised MarkerMap on most datasets
- Sparse expression matrices must be converted to dense arrays first; on Google Colab a runtime restart may be required after installing scanpy because it installs a newer matplotlib

## Evidence

- [methods] Scanpy highly variable genes as baseline marker selection: "Data is handled with numpy and with scanpy which is used for many computational biology datasets."
- [methods] Sparse to dense conversion before marker selection: "To convert, merely do `adata.X = adata.X.toarray()`"
- [methods] 70/10/20 split used in the reconstruction benchmark: "We then split the data into training, validation, and test sets with a 70%, 10%, 20% split."
- [results] Table 2 reconstruction benchmark with 50 markers and 10 runs: [evidence span withheld — anchor status: not_found]
- [readme] Package installation including persist comparator: "To use the persist package, you will need to install it following the instructions: https://github.com/iancovert/persist/"
