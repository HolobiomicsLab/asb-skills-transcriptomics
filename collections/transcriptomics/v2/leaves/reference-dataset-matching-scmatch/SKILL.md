---
name: reference-dataset-matching-scmatch
description: Use when a clustered Scanpy AnnData object (scanpyobj.h5ad) needs cell
  type labels per Louvain cluster, or when single-cell clusters must be matched against
  a curated reference GEP dataset like FANTOM5.
when_to_use_negative:
- Cells have not yet been clustered — scMatch within scDrug annotates cluster-wise
  average GEPs, not individual raw cells
- No suitable reference dataset is available for the tissue/cell types of interest
  (FANTOM5 and SingleR-merged references cover specific curated cell types)
- The goal is drug response prediction (use Drug Response Prediction with CaDRReS-Sc
  / PRISM models), not cell type identification
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMatch
  role: Performs cell-type annotation by matching cluster GEPs against reference datasets
    (FANTOM5 truncated reference)
  repo: https://github.com/asrhou/scMatch
- name: scDrug
  role: Wraps scMatch in single_cell_analysis.py via the --annotation flag and writes
    annotations into the Scanpy AnnData object
  repo: https://github.com/ailabstw/scDrug
- name: Scanpy
  role: Provides the clustered AnnData object (scanpyobj.h5ad) and cluster assignments
    from which the cluster GEPs are computed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:f8fa47c88142adccf6ee7cef6dee52b70dfd890a666bce4d1e80a2913c204ef0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/asrhou/scMatch
  tools:
  - Python
  - Scanpy
  - scDrug
  - scMatch
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# reference-dataset-matching-scmatch

## Summary

Cluster-wise cell type annotation for scRNA-seq data using scMatch, which scores each cluster's average gene expression profile against reference datasets such as the truncated FANTOM5 database (916 cell types) to assign a known cell type label.

## When to use

Use when a clustered Scanpy AnnData object (scanpyobj.h5ad) needs cell type labels per Louvain cluster, or when single-cell clusters must be matched against a curated reference GEP dataset like FANTOM5.

## When NOT to use

- Cells have not yet been clustered — scMatch within scDrug annotates cluster-wise average GEPs, not individual raw cells
- No suitable reference dataset is available for the tissue/cell types of interest (FANTOM5 and SingleR-merged references cover specific curated cell types)
- The goal is drug response prediction (use Drug Response Prediction with CaDRReS-Sc / PRISM models), not cell type identification

## Inputs

- Clustered Scanpy AnnData object (scanpyobj.h5ad) with Louvain cluster assignments
- Cluster-wise gene expression profile file (GEP.txt), computed as mean expression per cluster
- Truncated FANTOM5 reference dataset (decompressed files from scMatch refDB/FANTOM5, 916 cell types)
- Number of CPUs (--cpus) for parallel annotation

## Outputs

- Cluster-to-cell-type annotations recorded in the AnnData object
- umap_cell_type.png — UMAP colored by assigned cell type
- GEP.txt — cluster gene expression profiles used for annotation and downstream correlation

## How to apply

After Scanpy clustering, compute each cluster's gene expression profile (GEP) as the mean expression of its cells across all genes and save it as GEP.txt. Run scMatch (installed from https://github.com/asrhou/scMatch) on these cluster-wise GEPs against the truncated FANTOM5 reference database (decompress the compressed reference files first; they contain 916 cell types) to assign a known cell type to each cluster, then write the annotation back into the AnnData object. Generate the cell-type UMAP (umap_cell_type.png) to visually verify the labels. The same GEP file feeds the subsequent reference-cell-line selection step, where the Pearson correlation coefficient between the single-cell GEP and the bulk GEP of eight supported cell lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC) is computed and the highest-correlation line is chosen for CIBERSORTx deconvolution. The cluster-wise design reflects that scMatch annotates by comparing averaged expression levels rather than per-cell low-coverage data.

## Related tools

- **scMatch** (Performs cell-type annotation by matching cluster GEPs against reference datasets (FANTOM5 truncated reference)) — https://github.com/asrhou/scMatch
- **scDrug** (Wraps scMatch in single_cell_analysis.py via the --annotation flag and writes annotations into the Scanpy AnnData object) — https://github.com/ailabstw/scDrug
- **Scanpy** (Provides the clustered AnnData object (scanpyobj.h5ad) and cluster assignments from which the cluster GEPs are computed)

## Examples

```
python3 single_cell_analysis.py --input INPUT --resolution 1.0 --annotation --GEP
```

## Evaluation signals

- umap_cell_type.png exists and is a readable image showing cell-type labels
- Every identified cluster in scanpyobj.h5ad has a non-empty cell type annotation from the FANTOM5 reference
- GEP.txt contains one mean-expression column (or row) per cluster across all genes
- Annotation step completes within the allocated --cpus budget without scMatch reference-loading errors (reference files were decompressed before use)

## Limitations

- Annotations are only as good as the reference dataset: scDrug uses the truncated FANTOM5 reference (916 cell types), so novel or tissue-specific cell states may be mislabeled or unmatched
- scMatch assigns one identity to all cells within each cluster based on averaged expression; low-coverage or heterogeneous clusters can absorb multiple cell types
- scDrug's subsequent treatment-selection step (CIBERSORTx) only supports eight cell lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC), constraining which annotations are useful downstream

## Evidence

- [methods] functional enrichment by GSEAPY and cell-type annotation by scMatch: "functional enrichment by GSEAPY [22], [23] and cell-type annotation by scMatch [24]"
- [readme] readme annotation flag: "For cell type annotation, we used [scMatch: a single-cell gene expression profile annotation tool using reference datasets](https://github.com/asrhou/scMatch)."
- [readme] readme FANTOM5 reference: [evidence span withheld — anchor status: external_doc]
- [readme] readme cluster-averaged expression rationale: [evidence span withheld — anchor status: external_doc]
