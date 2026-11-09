---
name: leiden-graph-community-clustering-on-topic-embeddings
description: Use when clustering cells from single-cell epigenomics data such as SNARE-seq
  or scATAC-seq that has been reduced to a topic embedding (e.g., cisTopic topics),
  and cluster labels must be propagated to paired single-cell transcriptomic modality
  via shared cell barcodes.
when_to_use_negative:
- Input is raw cell-by-peak data without a topic (or other low-dimensional) embedding
  — build the cisTopic representation first rather than clustering peak space directly
- ATAC and RNA measurements are not paired by shared cell barcodes, so cell-ID-based
  label transfer between modalities is impossible (e.g., unpaired scATAC-seq/scRNA-seq
  references)
- Analysis needs reference-based supervised annotation rather than de novo unsupervised
  community detection (Cellar also offers supervised label transfer methods for that
  case)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: faiss
  role: Builds the approximate nearest-neighbors graph (15 neighbors) on the cell-by-topic
    matrix to speed up neighbors graph construction for Leiden clustering
  repo: https://github.com/facebookresearch/faiss
- name: Leiden
  role: Community detection algorithm run at resolution 1 on the faiss neighbors graph
    to produce cell clusters
- name: cisTopic
  role: Discovers the 40 cis-regulatory topics forming the cell-by-topic embedding
    on which clustering is performed; topic number chosen by log-likelihood model
    selection
  repo: https://github.com/aertslab/cisTopic
- name: Cellar
  role: 'Interactive tool orchestrating the full pipeline: topic modeling, faiss neighbors
    graph, Leiden clustering, and label transfer to the expression modality'
  repo: https://github.com/euxhenh/cellar
- name: AnnData (Annotated Data object)
  role: In-memory container storing cell-by-topic and cell-by-gene matrices and cluster
    labels; supports memory-mapping for large datasets
  repo: https://github.com/theislab/anndata2ri
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:0814d58826fe82f7c9562b4fe45653f06774ccca5766bdd63a0c6f1f6185241e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/facebookresearch/faiss
  tools:
  - AnnData (Annotated Data object)
  - Cellar
  - Leiden
  - cisTopic
  - diffxpy
  - faiss
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# Leiden graph community clustering on topic embeddings

## Summary

Cluster cells in a topic-reduced epigenomic space by building an approximate nearest-neighbor graph and running Leiden community detection, then transfer the resulting cluster labels to a paired expression modality. In the Cellar kidney SNARE-seq analysis this turned the cisTopic cell-by-topic matrix (40 topics) into discrete cell groups, including cluster 1 assigned as Proximal Tubule Cells.

## When to use

Use when clustering cells from single-cell epigenomics data such as SNARE-seq or scATAC-seq that has been reduced to a topic embedding (e.g., cisTopic topics), and cluster labels must be propagated to paired single-cell transcriptomic modality via shared cell barcodes.

## When NOT to use

- Input is raw cell-by-peak data without a topic (or other low-dimensional) embedding — build the cisTopic representation first rather than clustering peak space directly
- ATAC and RNA measurements are not paired by shared cell barcodes, so cell-ID-based label transfer between modalities is impossible (e.g., unpaired scATAC-seq/scRNA-seq references)
- Analysis needs reference-based supervised annotation rather than de novo unsupervised community detection (Cellar also offers supervised label transfer methods for that case)

## Inputs

- Cell-by-peak chromatin accessibility matrix from SNARE-seq (e.g., 31,758 cells x 575,265 peaks, kidney SNARE ATAC/RNA 20201005)
- Cell-by-topic matrix from cisTopic (40 topics)
- Cell barcodes / cell IDs shared between paired ATAC and RNA modalities
- AnnData (Annotated Data) object holding the expression modality

## Outputs

- Leiden cluster assignments (resolution 1) for each cell in the chromatin modality
- Cluster label column transferred to paired expression modality cells via cell-ID matching
- Cluster labels usable for downstream diffxpy differential expression and GSEApy enrichment (e.g., cluster 1 marker table)

## How to apply

First reduce the cell-by-peak matrix to a cell-by-topic matrix by running cisTopic, choosing the number of topics via cisTopic's log-likelihood model selection method (40 topics for the kidney SNARE ATAC/RNA 20201005 dataset of 31,758 cells and 575,265 peaks). Build an approximate nearest-neighbor graph on the cell-by-topic matrix with faiss using 15 neighbors, which speeds up graph construction compared to exact kNN. Run Leiden community detection on this graph at resolution 1 to obtain cluster assignments. Copy cluster labels to the paired RNA expression modality by matching cell barcodes (cell-ID-based label transfer), since both modalities profile the same cells. Downstream, run differential expression (diffxpy Welch's t-test, top 500 genes by fold-change) on transferred clusters to assign cell types via markers such as SLC5A12.

## Related tools

- **faiss** (Builds the approximate nearest-neighbors graph (15 neighbors) on the cell-by-topic matrix to speed up neighbors graph construction for Leiden clustering) — https://github.com/facebookresearch/faiss
- **Leiden** (Community detection algorithm run at resolution 1 on the faiss neighbors graph to produce cell clusters)
- **cisTopic** (Discovers the 40 cis-regulatory topics forming the cell-by-topic embedding on which clustering is performed; topic number chosen by log-likelihood model selection) — https://github.com/aertslab/cisTopic
- **Cellar** (Interactive tool orchestrating the full pipeline: topic modeling, faiss neighbors graph, Leiden clustering, and label transfer to the expression modality) — https://github.com/euxhenh/cellar
- **AnnData (Annotated Data object)** (In-memory container storing cell-by-topic and cell-by-gene matrices and cluster labels; supports memory-mapping for large datasets) — https://github.com/theislab/anndata2ri

## Examples

```
import faiss, scanpy as sc; D, I = faiss.knn(topic_matrix, topic_matrix, 15); sc.pp.neighbors(adata, n_neighbors=15, knn=True); sc.tl.leiden(adata, resolution=1)
```

## Evaluation signals

- The cell-by-topic matrix has 40 topic columns, matching the cisTopic log-likelihood model selection result reported in the paper
- Every cell in the chromatin modality receives exactly one Leiden cluster label, and cluster assignments exist for all cells in both modalities after cell-ID-based transfer
- Downstream diffxpy evidence table for cluster 1 reproduces marker SLC5A12 with p-value = 0 and GO term 'Apical Plasma Membrane' with p-value = 1e-4 (via GSEApy on the top 500 DE genes by fold-change)
- Neighbors graph construction used 15 faiss neighbors and Leiden ran at resolution 1, matching the stated parameters
- Unannotated cells were removed before clustering, so no cluster label exists for cells lacking annotations

## Limitations

- Leiden resolution (1) and neighbor count (15) were chosen empirically for the kidney SNARE-seq dataset; other datasets may require retuning
- Label transfer depends on exact cell barcode matching between paired modalities; unpaired or barcode-mismatched data cannot be handled this way
- cisTopic is deprecated in favor of pycisTopic, and no software versions for Cellar, Python, or faiss are stated, so reproducibility relies on the Cellar web server rather than a locally installable archived artifact

## Evidence

- [discussion] faiss neighbors graph for Leiden clustering: "approximate nearest neighbors based on faiss25 to speed up neighbors graph construction for Leiden clustering"
- [methods] 15 faiss neighbors on embeddings: [evidence span withheld — anchor status: not_found]
- [methods] 40 cisTopic topics via log-likelihood selection: [evidence span withheld — anchor status: not_found]
- [readme] cisTopic deprecated warning: [evidence span withheld — anchor status: external_doc]
