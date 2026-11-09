---
name: scrna-scatac-preprocessing-and-normalization
description: Use when raw scRNA-seq and scATAC-seq count matrices must be QC-filtered
  and normalized with modality-specific recipes (log1p/PCA for RNA, TF-IDF for ATAC)
  before cross-modality integration.
when_to_use_negative:
- Data are already preprocessed MuData objects with normalized modalities and a stored
  cross-modality cost matrix (e.g. the preprocessed 10X Multiome demo dataset from
  figshare)
- Integrating modalities without cross-modality feature connections, such as electrophysiology
  measurements with gene expression
- CyTOF or other non-count modalities requiring a different normalization (e.g. Center
  Log Ratio for protein data only)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0769
tools:
- name: scConfluence
  role: Target integration method whose MuData input conventions (uns['cross_keys'],
    obsm latent) dictate the preprocessing output format
  repo: https://github.com/cantinilab/scconfluence
- name: Muon
  role: Provides filter_obs/filter_var for QC, tfidf for scATAC normalization, and
    Center Log Ratio for protein data
- name: Scanpy
  role: Provides normalize_total (to 10,000 counts per cell) and log1p for scRNA normalization
- name: SciPy
  role: scipy.spatial.distance.cdist computes the cross-modality cost matrix from
    connected features
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:2e8e1ffb2e0d7329da3799a67065d0891909f3ef7fff5cc42a2af29113d741fc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cantinilab/scconfluence
  repo_url: https://github.com/cantinilab/scconfluence
  tools:
  - Liger
  - MultiMAP
  - Muon
  - Scanpy
  - SciPy
  - Seurat v4.3.0
  - Uniport
  - scConfluence
  - scGLUE
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# scRNA-scATAC-preprocessing-and-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess paired-or-unpaired scRNA-seq and scATAC-seq counts into modality-appropriate normalized matrices (log-normalized scRNA with 100 PCs, TF-IDF scATAC) as required before scConfluence diagonal integration or similar multimodal embedding pipelines.

## When to use

Use when raw scRNA-seq and scATAC-seq count matrices must be QC-filtered and normalized with modality-specific recipes (log1p/PCA for RNA, TF-IDF for ATAC) before cross-modality integration.

## When NOT to use

- Data are already preprocessed MuData objects with normalized modalities and a stored cross-modality cost matrix (e.g. the preprocessed 10X Multiome demo dataset from figshare)
- Integrating modalities without cross-modality feature connections, such as electrophysiology measurements with gene expression
- CyTOF or other non-count modalities requiring a different normalization (e.g. Center Log Ratio for protein data only)

## Inputs

- Raw scRNA-seq count matrix (e.g. in a MuData .h5mu object)
- Raw scATAC-seq peak-by-cell count matrix
- Cell QC metadata: mitochondrial gene expression proportion, expressed gene counts, total counts
- Feature annotation linking scRNA genes to scATAC peaks for the cross-modality cost matrix

## Outputs

- Log-normalized scRNA matrix (normalized to 10,000 then log1p) plus first 100 principal components
- Gene set = union of 3000 most variable genes in normalized counts and 3000 most variable genes in raw counts
- TF-IDF normalized scATAC peak matrix
- Cross-modality cost matrix stored in mdata.uns with mdata.uns['cross_keys'] set

## How to apply

Load scRNA and scATAC counts into a MuData object and apply Muon's filter_obs/filter_var QC: scRNA cells are filtered on the proportion of mitochondrial gene expression, number of expressed genes and total counts, genes on the number of cells expressing them; scATAC cells are filtered on number of open peaks and total counts, peaks on the number of cells where the peak is open. Normalize scRNA counts with Scanpy's normalize_total to 10,000 counts per cell followed by log1p, then compute the first 100 principal components as autoencoder input; gene subselection uses the union of the 3000 most variable genes in normalized counts and the 3000 most variable genes in raw counts. Normalize scATAC cells with Muon's TF-IDF implementation. If a surface-protein modality is present, normalize it with Muon's Center Log Ratio technique, and pass the batch index as a decoder covariate for batch correction. Finally, build the cross-modality cost matrix from connected features (e.g. with scipy.spatial.distance.cdist) and store it under mdata.uns with mdata.uns['cross_keys'] as required by scConfluence.

## Related tools

- **scConfluence** (Target integration method whose MuData input conventions (uns['cross_keys'], obsm latent) dictate the preprocessing output format) — https://github.com/cantinilab/scconfluence
- **Muon** (Provides filter_obs/filter_var for QC, tfidf for scATAC normalization, and Center Log Ratio for protein data)
- **Scanpy** (Provides normalize_total (to 10,000 counts per cell) and log1p for scRNA normalization)
- **SciPy** (scipy.spatial.distance.cdist computes the cross-modality cost matrix from connected features)

## Examples

```
import mudata as md, scanpy as sc; from scipy.spatial.distance import cdist; mdata = md.read_h5mu('cell_lines.h5mu'); sc.pp.normalize_total(mdata['rna'], target_sum=1e4); sc.pp.log1p(mdata['rna']); sc.tl.pca(mdata['rna'], n_comps=100); mu.atac.pp.tfidf(mdata['atac']); D = cdist(rna_feats, atac_feats)
```

## Evaluation signals

- QC-filtered scRNA cells pass mitochondrial proportion, expressed-genes and total-counts filters applied with Muon's filter_obs/filter_var
- scRNA modality is normalized so counts sum to 10,000 per cell and log1p-transformed, with the first 100 PCs available as encoder input
- Gene panel equals the union of the 3000 most variable genes in normalized counts and the 3000 most variable genes in raw counts
- scATAC matrix is TF-IDF normalized via Muon's tfidf
- mdata.uns contains a finite cross-modality cost matrix and mdata.uns['cross_keys'] is set so scConfluence can consume the object

## Limitations

- Feature conversion to common features loses biological information because features across modalities are weakly connected; scConfluence instead keeps full features and uses connected-feature costs
- Rare cell populations are hard to handle and under-represented in mini-batch gradient descent parameter estimation
- Costs rely on cross-modality feature connections, which are not always available (e.g. electrophysiology vs gene expression)

## Evidence

- [discussion] Log-normalization of scRNA counts: [evidence span withheld — anchor status: not_found]
- [discussion] Gene subselection via union of 3000 most variable genes: [evidence span withheld — anchor status: not_found]
- [discussion] TF-IDF normalization of scATAC cells: [evidence span withheld — anchor status: not_found]
