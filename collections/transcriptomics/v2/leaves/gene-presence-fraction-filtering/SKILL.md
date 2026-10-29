---
name: gene-presence-fraction-filtering
description: Use when preparing a raw scRNA-seq/CITE-seq count matrix (e.g., an AnnData
  .h5ad loaded with scanpy) for marker selection, and you need to discard genes expressed
  in too few or too many cells before normalization and z-scaling.
when_to_use_negative:
- When the input is already a preprocessed/feature-reduced matrix or a marker list
  rather than raw per-gene counts — re-filtering would discard meaningful genes.
- When the data is not single-cell gene expression (e.g., bulk RNA-seq or spatial
  FISH target panels), since the presence/absence fraction logic is calibrated to
  per-cell expression sparsity.
- When filtering is the only needed step and no downstream MarkerMap training is planned
  — a standard scanpy filter may suffice without the dense conversion overhead.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scanpy
  role: Reads the .h5ad CITE-seq data into an AnnData object whose adata.X gene-by-cell
    matrix the presence/absence gene filters are applied to
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap
- name: numpy
  role: Handles the dense gene-count array during filtering and the subsequent log2(1+x)
    and z-scaling transformations
- name: anndata
  role: Container format for the filtered expression matrix; the filtered object is
    saved as preprocessed_adata.h5ad
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap
- name: MarkerMap
  role: Downstream marker selection model whose requirement for dense input motivates
    the toarray() conversion before filtering
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:9fe8d994f9c6d6ad19f945f6103e8856477c860ffa785c8124dcd0dcdb28a4be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap
  tools:
  - MarkerMap
  - anndata
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# gene-presence-fraction-filtering

## Summary

A preprocessing filter for single-cell expression matrices that removes housekeeping/general genes and drops genes expressed in too few or too many cells, so that only informative genes survive into normalization, log2(1+x) transformation, and MarkerMap marker selection.

## When to use

Use when preparing a raw scRNA-seq/CITE-seq count matrix (e.g., an AnnData .h5ad loaded with scanpy) for marker selection, and you need to discard genes expressed in too few or too many cells before normalization and z-scaling.

## When NOT to use

- When the input is already a preprocessed/feature-reduced matrix or a marker list rather than raw per-gene counts — re-filtering would discard meaningful genes.
- When the data is not single-cell gene expression (e.g., bulk RNA-seq or spatial FISH target panels), since the presence/absence fraction logic is calibrated to per-cell expression sparsity.
- When filtering is the only needed step and no downstream MarkerMap training is planned — a standard scanpy filter may suffice without the dense conversion overhead.

## Inputs

- Raw scRNA-seq/CITE-seq count matrix stored in AnnData format (.h5ad, e.g., data/cite_seq/CITEseq.h5ad)
- Sparse gene-by-cell count matrix in adata.X (converted to dense via toarray())
- Cell labels (e.g., adata.obs['annotation']) used downstream, not for the filter itself

## Outputs

- Gene-filtered AnnData object retaining only informative genes (saved as preprocessed_adata.h5ad)
- Dense expression matrix ready for library normalization, log2(1+x), and z-scaling prior to marker selection

## How to apply

Load the dataset (e.g., CITE-seq data at data/cite_seq/CITEseq.h5ad) with scanpy into an AnnData object and convert adata.X from sparse to dense via toarray(), since MarkerMap models expect dense input. First remove general/housekeeping genes, then apply presence/absence gene-fraction filters that drop genes expressed in too few cells (rare, uninformative) and genes expressed in too many cells (ubiquitous/housekeeping). After filtering, proceed with the rest of the MarkerMap preprocessing chain: per-cell library-size normalization, log2(1+x) transformation with numpy, and per-gene z-scaling (zero mean, unit variance), then save the result as an .h5ad file (preprocessed_adata.h5ad). The rationale is that genes failing presence/absence thresholds carry little cell-type-discriminative signal and would dilute downstream marker selection.

## Related tools

- **scanpy** (Reads the .h5ad CITE-seq data into an AnnData object whose adata.X gene-by-cell matrix the presence/absence gene filters are applied to) — https://github.com/Computational-Morphogenomics-Group/MarkerMap
- **numpy** (Handles the dense gene-count array during filtering and the subsequent log2(1+x) and z-scaling transformations)
- **anndata** (Container format for the filtered expression matrix; the filtered object is saved as preprocessed_adata.h5ad) — https://github.com/Computational-Morphogenomics-Group/MarkerMap
- **MarkerMap** (Downstream marker selection model whose requirement for dense input motivates the toarray() conversion before filtering) — https://github.com/Computational-Morphogenomics-Group/MarkerMap

## Examples

```
adata = sc.read_h5ad('data/cite_seq/CITEseq.h5ad'); adata.X = adata.X.toarray()
```

## Evaluation signals

- After filtering, every retained gene is expressed in a plausible fraction of cells — neither near-zero (too few cells) nor near-1.0 (too many cells / housekeeping).
- adata.X is a dense numpy array (not sparse) once the toarray() conversion and filters have run, matching MarkerMap's dense-input expectation.
- The gene dimension of adata.X is strictly smaller after filtering than the raw CITE-seq input, while cell labels in adata.obs['annotation'] are unchanged.
- The filtered object persists as preprocessed_adata.h5ad and reloads via sc.read_h5ad without errors before the normalization/log2/z-scaling chain runs.
- Downstream: MarkerMap trained on the filtered matrix converges and produces k markers (e.g., k=50 for the 500-gene CITE-seq setup) with a measurable misclassification rate.

## Limitations

- No concrete numeric gene-fraction thresholds are specified in the article or README — the cutoffs for 'too few or too many cells' must be chosen by the practitioner per dataset.
- Data must be converted from sparse to dense before filtering for the MarkerMap pipeline, which increases memory usage for large atlases.
- Windows installation of the MarkerMap package was not yet supported at the time of the README ('Coming soon!'), limiting where the downstream pipeline can run.

## Evidence

- [methods] Presence/absence filter rationale: [evidence span withheld — anchor status: not_found]
- [full_text] Ordering of filter before normalization and transform: [evidence span withheld — anchor status: not_found]
- [readme] Data loading format: "We then read in the data using scanpy which returns an AnnData object."
- [methods] Dense conversion requirement: "To convert, merely do `adata.X = adata.X.toarray()`"
