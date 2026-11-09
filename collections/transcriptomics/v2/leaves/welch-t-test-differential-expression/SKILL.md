---
name: welch-t-test-differential-expression
description: Use when each cell in a single-cell dataset (scRNA-seq or a scATAC-seq
  cell-by-gene activity matrix) already has cluster labels and you must identify marker
  genes per cluster via one-vs-rest testing, ranked by fold-change for marker or enrichment
  analysis.
when_to_use_negative:
- The input is still a cell-by-peak chromatin-accessibility matrix with no gene-level
  representation — derive topics (cisTopic) or a gene activity score first.
- The goal is reference-based label transfer/annotation rather than de novo marker
  discovery — use scanpy Ingest or SingleR instead.
- Differential expression results already exist and the task is downstream GO-term
  enrichment — feed the top 500 fold-change genes directly to GSEApy.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: diffxpy
  role: Runs the one-vs-rest Welch's t-test per Leiden cluster and ranks genes by
    log fold-change
  repo: https://github.com/theislab/diffxpy
- name: Cellar
  role: 'Orchestrates the workflow: preprocessing, clustering, DE testing, and marker/enrichment
    intersection'
  repo: github.com/euxhenh/cellar
- name: GSEApy
  role: Consumes the top 500 DE genes by fold-change to produce GO-term enrichment
    p-values per cluster
  repo: https://github.com/zqfang/GSEApy
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:10b1195463ef304b73abaff5a81a7fe05903f765e31dc2f720155886835d3999
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/diffxpy
  tools:
  - AnnData
  - Cellar
  - GSEApy
  - cisTopic
  - diffxpy
  - faiss
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# welch-t-test-differential-expression

## Summary

Performs one-vs-rest differential gene expression testing for each Leiden cluster using diffxpy's Welch's t-test, ranking genes by log fold-change to surface cluster markers (e.g., KLRD1 for NK cells, SLC5A12 for proximal tubule cells) in Cellar-style single-cell workflows.

## When to use

Use when each cell in a single-cell dataset (scRNA-seq or a scATAC-seq cell-by-gene activity matrix) already has cluster labels and you must identify marker genes per cluster via one-vs-rest testing, ranked by fold-change for marker or enrichment analysis.

## When NOT to use

- The input is still a cell-by-peak chromatin-accessibility matrix with no gene-level representation — derive topics (cisTopic) or a gene activity score first.
- The goal is reference-based label transfer/annotation rather than de novo marker discovery — use scanpy Ingest or SingleR instead.
- Differential expression results already exist and the task is downstream GO-term enrichment — feed the top 500 fold-change genes directly to GSEApy.

## Inputs

- AnnData (Annotated Data object) cell-by-gene matrix, e.g., PBMC 10k scATAC-seq gene activity score matrix, CPM-normalized (total count 1e5), log1p-transformed, and scaled
- Per-cell cluster labels (Leiden clusters at resolution 1 from a faiss 15-neighbor graph, or clusters copied across SNARE-seq modalities by cell-ID matching)
- Gene annotation (e.g., GENCODE v35) mapping peaks/regions to genes when starting from a cell-by-peak matrix

## Outputs

- de_genes_per_cluster.csv: per-cluster Welch's t-test DE results ranked by log fold-change (one vs rest)
- Top 500 DE genes by fold-change per cluster, used as input to GSEApy enrichment analysis
- Marker-gene evidence table per cluster (e.g., KLRD1 for NK cells; SLC5A12 with p-value = 0 for Proximal Tubule Cells)

## How to apply

First prepare the expression matrix as in Cellar: CPM-normalize to a total count of 1e5, log1p-transform, and scale to unit variance and zero mean. With cluster assignments in hand (Leiden at resolution 1 on a faiss 15-neighbor graph, or cluster labels transferred between paired modalities by cell-ID barcode matching), run diffxpy's Welch's t-test separately for each cluster in a one-vs-rest design. Rank the resulting genes by log fold-change and compile them per cluster (e.g., into de_genes_per_cluster.csv). For downstream functional interpretation, select the 500 DE genes with the greatest fold-change values and pass them to GSEApy for GO-term enrichment. A correct application is confirmed when known markers surface as top hits: KLRD1 for clusters 0 and 4 in the PBMC 10k cell-by-gene scATAC-seq run, and SLC5A12 (p-value = 0) for cluster 1 (Proximal Tubule Cells) in the kidney SNARE-seq joint analysis.

## Related tools

- **diffxpy** (Runs the one-vs-rest Welch's t-test per Leiden cluster and ranks genes by log fold-change) — https://github.com/theislab/diffxpy
- **Cellar** (Orchestrates the workflow: preprocessing, clustering, DE testing, and marker/enrichment intersection) — github.com/euxhenh/cellar
- **GSEApy** (Consumes the top 500 DE genes by fold-change to produce GO-term enrichment p-values per cluster) — https://github.com/zqfang/GSEApy

## Examples

```
import diffxpy.api as de; res = de.test.t_test(adata.X, grouping=adata.obs['leiden'], is_logged=True); res.summary().sort_values('log2fc', ascending=False)
```

## Evaluation signals

- de_genes_per_cluster.csv exists and contains per-cluster Welch's t-test results ranked by fold-change (one vs rest).
- KLRD1 appears among the top marker genes for clusters 0 and 4 in the PBMC 10k cell-by-gene scATAC-seq run, consistent with the NK-cell annotation in Supplementary Fig. 3.
- For the kidney SNARE-seq joint analysis, the cluster-1 evidence table reproduces SLC5A12 with p-value = 0.
- Exactly the 500 DE genes with the greatest fold-change values are selected as enrichment input, and GSEApy returns GO-term p-values (e.g., 'Apical Plasma Membrane' with p-value = 1e-4 for cluster 1).
- DE statistics (p-values, fold-change/logFC) are present and finite for every cluster tested.

## Limitations

- The diffxpy repository is unmaintained; its README directs users needing pure-Python differential expression analysis to PyDESeq2 instead.
- Reported marker calls (KLRD1, SLC5A12) are tied to the specific Cellar preprocessing chain (CPM 1e5, log1p, scaling, 40-dim PCA with randomized SVD, Leiden resolution 1); changing these steps can change cluster membership and markers.
- No software versions are stated for Cellar, Python, or diffxpy, so exact reproduction may vary.

## Evidence

- [methods] Differential gene expression analysis was performed with diffxpy (https://github.com/theislab/diffxpy): "Differential gene expression analysis was performed with diffxpy (https://github.com/theislab/diffxpy)"
- [methods] The 500 DE genes with the greatest fold-change values were selected for enrichment analysis: "The 500 DE genes with the greatest fold-change values were selected for enrichment analysis"
- [intro] identification of differentially expressed genes, intersection with functional and marker sets: "identification of differentially expressed genes, intersection with functional and marker sets"
- [readme] Fast and scalable differential expression analysis on single-cell RNA-seq data: [evidence span withheld — anchor status: external_doc]
- [readme] WARNING: This repo is unmaintained.  If you are interested in pure-python differential expression analysis, please look at PyDESeq2_: [evidence span withheld — anchor status: external_doc]
