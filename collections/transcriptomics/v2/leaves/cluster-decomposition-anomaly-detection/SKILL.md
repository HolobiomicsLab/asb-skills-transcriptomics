---
name: cluster-decomposition-anomaly-detection
description: Use when a cells × genes scRNA-seq count matrix (e.g., the bundled 1%Jurkat.h5
  with 1540 293T + 16 Jurkat cells) is suspected to contain rare cell populations
  that one-time clustering may overlook, and you need rare sub-cluster labels, independence
  scores, and candidate DE genes.
when_to_use_negative:
- The goal is comprehensive annotation of all major cell types — the paper positions
  scCAD as a specialized rare-cell method complementing comprehensive tools like Seurat,
  not a replacement.
- The input is a genes × cells matrix or a precomputed feature/clustering table rather
  than a cells × genes expression matrix — the README specifies 'genes in columns
  and cells in rows'.
- The matrix is already normalized — leaving the default normalization=True would
  re-normalize already-processed values; the README exposes the normalization flag
  specifically for this decision.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scCAD
  role: 'Core method: iteratively decomposes clusters on the most differential signals,
    scores candidates via isolation forest, and returns rare sub-clusters, independence
    scores, per-cell labels, and DE gene lists through the scCAD.scCAD entry point'
  repo: https://github.com/xuyp-csu/scCAD.git
- name: h5py
  role: Loads the 1%Jurkat.h5 example, extracting the X (cells × genes) matrix, Y
    labels, gn gene names, and cn cell names
- name: numpy
  role: Array conversion of the loaded data (np.array, np.vectorize(float)) and member
    lookup of sub-cluster cells via np.where/np.isin
- name: pandas
  role: Tabular handling of returned sub-cluster membership, scores, labels, and DE
    gene lists (e.g., exported to a results CSV in the source workflow)
- name: Scanpy
  role: Version-pinned prerequisite (1.9.1) supporting the pipeline's preprocessing
    (log-normalization, PCA to top 40 PCs, Louvain clustering via python-louvain 0.16)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:71b63f35b2a60facbfcc4a48eb6a6d971879d058d89ba0b1b9d1000c68578fba
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xuyp-csu/scCAD
  repo_url: https://github.com/xuyp-csu/scCAD.git
  tools:
  - Scanpy
  - h5py
  - numpy
  - pandas
  - scCAD
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# cluster-decomposition-anomaly-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

scCAD is a cluster decomposition-based anomaly detection method that iteratively decomposes clusters on their most differential signals to identify rare cell types in scRNA-seq data (e.g., 16 Jurkat cells hidden among 1540 293T cells), returning rare sub-clusters, independence scores, per-cell labels, and candidate DE genes.

## When to use

Use when a cells × genes scRNA-seq count matrix (e.g., the bundled 1%Jurkat.h5 with 1540 293T + 16 Jurkat cells) is suspected to contain rare cell populations that one-time clustering may overlook, and you need rare sub-cluster labels, independence scores, and candidate DE genes.

## When NOT to use

- The goal is comprehensive annotation of all major cell types — the paper positions scCAD as a specialized rare-cell method complementing comprehensive tools like Seurat, not a replacement.
- The input is a genes × cells matrix or a precomputed feature/clustering table rather than a cells × genes expression matrix — the README specifies 'genes in columns and cells in rows'.
- The matrix is already normalized — leaving the default normalization=True would re-normalize already-processed values; the README exposes the normalization flag specifically for this decision.

## Inputs

- Cells × genes scRNA-seq expression matrix (rows = cells, columns = genes), e.g., the 'X' array from 1%Jurkat.h5 read via h5py and converted to float
- Optional geneNames and cellNames arrays (the 'gn'/'cn' fields, UTF-8 decoded); scCAD generates names automatically if omitted
- Optional ground-truth labels array (the 'Y' field) for validating identified rare sub-clusters when available

## Outputs

- result: list of rare sub-clusters identified by scCAD
- score: np.array of independence scores, one per identified sub-cluster
- sub_clusters: np.array of per-cell sub-cluster assignments (n cells)
- degs_list: list of candidate differentially expressed genes per rare sub-cluster
- Saved full result files written to the save_path directory (save_full=True by default)

## How to apply

Load the expression data as a cells × genes matrix — e.g., read the X array (plus Y labels, gn gene names, cn cell names) from 1%Jurkat.h5 with h5py, convert to float and decode names to UTF-8 — then call scCAD.scCAD with defaults normalization=True, seed=2023, rare_h=0.01, merge_h=0.3, overlap_h=0.7 and a save_path (the 1556-cell demo runs in ~40 seconds). Internally, genes expressed in fewer than three cells are dropped, data are log-normalized with cell-specific size factors (log2 after a pseudo-count of 1), projected onto the top 40 principal components, and clustered via a k=15 KNN graph with Louvain. Ensemble feature selection combines the top 2000 most-variable genes with the top 2,000 random-forest-importance genes, and each cluster containing more than R=1% of total cells is iteratively decomposed on its most differential signals. Decomposed clusters are merged at THM = median nearest-neighbor center distance (merge_h=0.3); the top 20 differential genes per cluster feed an isolation-forest anomaly model, and clusters whose independence score exceeds I=0.7 (with overlap_h=0.7 defining rare sub-clusters) are called rare. Unpack the four returns — result (rare sub-cluster list), score (independence scores), sub_clusters (per-cell assignments), degs_list (candidate DE genes) — and validate by checking Counter(labels[indices]) shows each sub-cluster enriched for the 16 Jurkat cells with scores above 0.7.

## Related tools

- **scCAD** (Core method: iteratively decomposes clusters on the most differential signals, scores candidates via isolation forest, and returns rare sub-clusters, independence scores, per-cell labels, and DE gene lists through the scCAD.scCAD entry point) — https://github.com/xuyp-csu/scCAD.git
- **h5py** (Loads the 1%Jurkat.h5 example, extracting the X (cells × genes) matrix, Y labels, gn gene names, and cn cell names)
- **numpy** (Array conversion of the loaded data (np.array, np.vectorize(float)) and member lookup of sub-cluster cells via np.where/np.isin)
- **pandas** (Tabular handling of returned sub-cluster membership, scores, labels, and DE gene lists (e.g., exported to a results CSV in the source workflow))
- **Scanpy** (Version-pinned prerequisite (1.9.1) supporting the pipeline's preprocessing (log-normalization, PCA to top 40 PCs, Louvain clustering via python-louvain 0.16))

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, save_path='./')
```

## Evaluation signals

- The call returns exactly the four documented objects: result (rare sub-cluster list), score (np.array with one independence score per sub-cluster), sub_clusters (np.array of per-cell labels), and degs_list (candidate DE genes per sub-cluster).
- On the 1%Jurkat demo, Counter(labels[indices]) for each returned sub-cluster is dominated by the 16 Jurkat cells rather than the 1540 293T cells.
- Every called rare sub-cluster has an independence score above the default threshold I=0.7.
- With the pinned dependencies (Python 3.7.13, numpy 1.21.5, Scanpy 1.9.1, python-louvain 0.16), the 1556-cell example completes in approximately 40 seconds; large deviations indicate environment problems.
- Inspect degs_list for rare-cell-type-specific genes — in benchmarks, rare cell type-specific genes accounted for over 86% of selected genes on average.

## Limitations

- Accuracy is not universal: scCAD successfully identified rare cell types in 20 of 25 benchmarked real-world datasets, so rare populations can be missed on some inputs.
- The package requires enough RAM for in-memory operations (README hardware note); very large expression matrices may not fit in memory.
- Tested only on Linux (Ubuntu 16.04.7) and macOS (Monterey 12.6.3) with exact version pins in requirements.txt (e.g., Python 3.7.13, scikit-learn 1.0.2, scipy 1.7.3); other versions/environments are untested.
- If gene and cell names are not provided, scCAD generates them automatically, making result sub-clusters harder to map back to annotated cells.

## Evidence

- [abstract] we propose a Cluster decomposition-based Anomaly Detection method (scCAD), which iteratively decomposes clusters based on the most differential signals in each cluster: "we propose a Cluster decomposition-based Anomaly Detection method (scCAD), which iteratively decomposes clusters based on the most differential signals in each cluster"
- [discussion] Genes expressed in at least three cells are retained: "genes that are expressed in at least three cells are retained for downstream analysis"
- [discussion] PCA to top 40 principal components: "applies PCA to obtain the top 40 principal components"
- [discussion] Top 2000 genes by random-forest importance: "selects the top 2,000 genes with the highest importance"
- [discussion] Independence score threshold I default 0.7: [evidence span withheld — anchor status: not_found]
- [discussion] Over 86% of selected genes were rare cell type-specific: "rare cell type-specific genes identified by scCAD accounted for over 86% on average among the selected genes"
- [readme] Input data orientation expected by scCAD: "data: The expression data matrix with genes in columns and cells in rows."
- [readme] Example dataset contents: "1%Jurkat.h5 -- An example scRNA-seq dataset comprising 1540 293T cells and 16 Jurkat cells."
- [readme] Expected demo runtime: "The execution of this code approximately takes 40 seconds."
