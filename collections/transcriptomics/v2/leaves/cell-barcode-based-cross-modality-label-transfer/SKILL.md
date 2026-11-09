---
name: cell-barcode-based-cross-modality-label-transfer
description: Use when paired multi-omics single-cell data (e.g. SNARE-seq chromatin
  + expression) share cell barcodes across modalities, and clusters computed on one
  modality (here the cell-by-peak chromatin side) must be copied to the other for
  DE/enrichment.
when_to_use_negative:
- The modalities are not paired — barcodes/cell-IDs differ between datasets (e.g.
  independent scATAC-seq and scRNA-seq experiments); use embedding/Ingest-style label
  transfer instead of barcode copying.
- The expression data is already clustered or annotated — re-transferring chromatin-side
  clusters would overwrite existing labels for no gain.
- The input is a cell-by-topic or other reduced matrix rather than raw paired modalities
  — cisTopic topic discovery and peak-level clustering do not apply.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: Cellar
  role: Interactive pipeline that combines the SNARE-seq datasets, runs cisTopic/faiss/Leiden
    clustering, copies cluster assignments to the paired expression modality via cell-ID-based
    label transfer, and drives diffxpy/GSEApy follow-up
  repo: github.com/euxhenh/cellar
- name: cisTopic
  role: Discovers 40 cis-regulatory topics from the cell-by-peak matrix; topic number
    chosen via cisTopic's log-likelihood model selection method
  repo: https://github.com/aertslab/cisTopic
- name: faiss
  role: Approximate nearest-neighbors graph construction (15 neighbors) on the cell-by-topic
    matrix to speed up Leiden clustering
  repo: https://github.com/facebookresearch/faiss
- name: scanpy
  role: Preprocessing and label transfer between datasets via scanpy's Ingest; AnnData
    memory-mapping supports the large paired data
- name: diffxpy
  role: Differential expression (Welch's t-test) on the transferred clusters in the
    expression modality
  repo: https://github.com/theislab/diffxpy
- name: GSEApy
  role: Enrichment analysis of the top 500 fold-change DE genes to obtain GO term
    p-values per cluster
  repo: https://github.com/zqfang/GSEApy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:9b4050651328889e989e032f368a955145a25890301f80b00752f0bc8d2a370e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cellar
  - GSEApy
  - cisTopic
  - diffxpy
  - faiss
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cell-barcode-based-cross-modality-label-transfer

## Summary

Transfers cluster labels between paired single-cell modalities (e.g. SNARE ATAC/RNA from the same cells) by matching shared cell barcodes/IDs, so clusters defined on the chromatin modality annotate the expression modality. In Cellar this underpins joint kidney SNARE-seq analysis, e.g. assigning cluster 1 as Proximal Tubule Cells via SLC5A12 (p-value = 0) and the GO 'Apical Plasma Membrane' term (p value = 1e-4).

## When to use

Use when paired multi-omics single-cell data (e.g. SNARE-seq chromatin + expression) share cell barcodes across modalities, and clusters computed on one modality (here the cell-by-peak chromatin side) must be copied to the other for DE/enrichment.

## When NOT to use

- The modalities are not paired — barcodes/cell-IDs differ between datasets (e.g. independent scATAC-seq and scRNA-seq experiments); use embedding/Ingest-style label transfer instead of barcode copying.
- The expression data is already clustered or annotated — re-transferring chromatin-side clusters would overwrite existing labels for no gain.
- The input is a cell-by-topic or other reduced matrix rather than raw paired modalities — cisTopic topic discovery and peak-level clustering do not apply.

## Inputs

- Cell-by-peak chromatin-accessibility matrix from SNARE-seq (e.g. 31,758 cells x 575,265 peaks), possibly combining multiple source datasets
- Paired cell-by-gene expression matrix from the same SNARE-seq run, sharing cell barcodes/cell-IDs with the chromatin modality
- AnnData (Annotated Data object) containers for both modalities
- Leiden cluster assignments computed on the chromatin modality (cisTopic topics -> faiss 15-neighbor graph -> Leiden resolution 1)

## Outputs

- Cluster-assignment column copied onto the expression modality's AnnData obs, keyed by cell-ID/barcode
- Per-cluster differential expression table from diffxpy (Welch's t-test), e.g. top 500 genes by fold-change
- GO enrichment p-values per cluster from GSEApy (e.g. cluster 1: 'Apical Plasma Membrane', p value = 1e-4)
- Cell-type assignment supported by known markers (e.g. SLC5A12, p-value = 0, for Proximal Tubule Cells)

## How to apply

First cluster the chromatin modality: combine the source datasets (four separate kidney SNARE-seq datasets for the kidney case), remove cells for which no annotations were found, run cisTopic on the cell-by-peak matrix (31,758 cells, 575,265 peaks in the kidney case) to discover 40 topics selected via cisTopic's log-likelihood model selection method, then build an approximate nearest-neighbors graph with faiss using 15 neighbors and cluster it with Leiden at resolution 1. Then perform the transfer itself: match cell barcodes/cell-IDs between the two modalities profiled from the same cells and copy the chromatic-side cluster assignments onto the paired expression modality (Cellar also supports scanpy's Ingest for label transfer between datasets). Downstream, run differential expression on the transferred clusters with diffxpy using a Welch's t-test, take the 500 DE genes with the greatest fold-change values, and run enrichment with GSEApy to obtain GO term p-values for the cluster of interest. Rationale: paired SNARE-seq profiles both modalities from the same nucleus, so barcode matching gives an exact, assumption-free cell-to-cell correspondence without embedding alignment.

## Related tools

- **Cellar** (Interactive pipeline that combines the SNARE-seq datasets, runs cisTopic/faiss/Leiden clustering, copies cluster assignments to the paired expression modality via cell-ID-based label transfer, and drives diffxpy/GSEApy follow-up) — github.com/euxhenh/cellar
- **cisTopic** (Discovers 40 cis-regulatory topics from the cell-by-peak matrix; topic number chosen via cisTopic's log-likelihood model selection method) — https://github.com/aertslab/cisTopic
- **faiss** (Approximate nearest-neighbors graph construction (15 neighbors) on the cell-by-topic matrix to speed up Leiden clustering) — https://github.com/facebookresearch/faiss
- **scanpy** (Preprocessing and label transfer between datasets via scanpy's Ingest; AnnData memory-mapping supports the large paired data)
- **diffxpy** (Differential expression (Welch's t-test) on the transferred clusters in the expression modality) — https://github.com/theislab/diffxpy
- **GSEApy** (Enrichment analysis of the top 500 fold-change DE genes to obtain GO term p-values per cluster) — https://github.com/zqfang/GSEApy

## Examples

```
rna.obs["cluster"] = atac.obs.loc[rna.obs_names, "leiden"]  # cell-ID-based transfer across paired SNARE-seq modalities
```

## Evaluation signals

- Cluster assignments exist for all cells in both modalities after barcode-based transfer (validation stated in the task workflow).
- The chromatin-side clustering reproduces 40 cisTopic topics (log-likelihood model selection), a faiss ANN graph with 15 neighbors, and Leiden resolution 1.
- diffxpy Welch's t-test yields the expected marker evidence for the target cluster — e.g. SLC5A12 with p-value = 0 for cluster 1.
- GSEApy enrichment of the top 500 fold-change DE genes reproduces the GO 'Apical Plasma Membrane' term with p value = 1e-4 for cluster 1.
- Label-invariant check: each expression-modality cell receives exactly one cluster label, matched 1:1 through shared cell-IDs (no unannotated cells remain).

## Limitations

- cisTopic (R package) is deprecated upstream in favor of pycisTopic, so the exact R implementation may be hard to maintain.
- diffxpy is unmaintained; its README recommends PyDESeq2 for pure-python differential expression analysis.
- Transfer only works when both modalities are profiled from the same cells (shared barcodes); cells with no annotations are removed beforehand.
- Reproducibility depends on the Cellar web server rather than a locally installable archived artifact, and no software versions are stated for Cellar, Python, Dash, or faiss.

## Evidence

- [methods] Label transfer between HBMP2-spleen-2 and HBMP3-spleen-CC3 was performed using scanpy's Ingest: "Label transfer between HBMP2-spleen-2 and HBMP3-spleen-CC3 was performed using scanpy’s Ingest"
- [methods] The SNARE-seq data was formed by combining four separate kidney SNARE-seq datasets.: "The SNARE-seq data was formed by combining four separate kidney SNARE-seq datasets."
- [methods] cisTopic to discover 40 topics. This number was selected via cisTopic's log-likelihood model selection method.: [evidence span withheld — anchor status: not_found]
- [methods] The embeddings were then used to construct an approximate neighbors graph using faiss with 15 neighbors: [evidence span withheld — anchor status: not_found]
- [methods] The 500 DE genes with the greatest fold-change values were selected for enrichment analysis: "The 500 DE genes with the greatest fold-change values were selected for enrichment analysis"
- [methods] We removed cells for which no annotations were found.: "We removed cells for which no annotations were found."
- [readme] cisTopic is deprecated, use pycisTopic instead.: [evidence span withheld — anchor status: not_found]
- [readme] WARNING: This repo is unmaintained. If you are interested in pure-python differential expression analysis, please look at PyDESeq2_ .: [evidence span withheld — anchor status: external_doc]
