---
name: single-cell-differential-expression-testing
description: Use when a clustered single-cell AnnData object needs cluster-level marker/DE
  genes — e.g. to assign a cell type to a Leiden cluster via enrichment of its top
  differential genes.
when_to_use_negative:
- The data are bulk RNA-seq or already aggregated per condition rather than single-cell
  clusters — diffxpy is designed for fast and scalable differential expression analysis
  on single-cell RNA-seq data
- The goal is reference-based label transfer (e.g. scanpy Ingest or SingleR) rather
  than unsupervised cluster annotation via enrichment
- Cells are unfiltered or unnormalized (e.g. not CPM-normalized/log1p-transformed),
  making t-test assumptions on log-scale expression invalid
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: diffxpy
  role: Performs the Welch's t-test differential expression of the target cluster
    versus the rest
  repo: https://github.com/theislab/diffxpy
- name: scanpy
  role: Preprocessing (filtering, CPM normalization, log1p, scaling) of the AnnData
    prior to DE testing
- name: GSEApy
  role: Enrichment analysis of the top 500 DE genes against GO, KEGG, and MSigDB gene
    sets
  repo: https://github.com/zqfang/GSEApy
- name: Cellar
  role: Interactive pipeline producing the clustered AnnData (PCA + UMAP + Leiden)
    that the DE step annotates
  repo: https://github.com/euxhenh/cellar
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:3bd5112bd48434446f4a673490dc3f13334bff0aa7a44c3007b388f61eadd886
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/diffxpy
  tools:
  - AnnData
  - Cellar
  - GSEApy
  - Python
  - Python 3
  - diffxpy
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# single-cell-differential-expression-testing

## Summary

Identify differentially expressed genes between a single-cell cluster and the rest of the dataset using diffxpy's Welch's t-test on a log1p-transformed, CPM-normalized AnnData object, then rank by log2 fold-change to select a top-k gene list (e.g. 500 genes) for downstream functional enrichment.

## When to use

Use when a clustered single-cell AnnData object needs cluster-level marker/DE genes — e.g. to assign a cell type to a Leiden cluster via enrichment of its top differential genes.

## When NOT to use

- The data are bulk RNA-seq or already aggregated per condition rather than single-cell clusters — diffxpy is designed for fast and scalable differential expression analysis on single-cell RNA-seq data
- The goal is reference-based label transfer (e.g. scanpy Ingest or SingleR) rather than unsupervised cluster annotation via enrichment
- Cells are unfiltered or unnormalized (e.g. not CPM-normalized/log1p-transformed), making t-test assumptions on log-scale expression invalid

## Inputs

- Clustered AnnData object (.h5ad) from Cellar's PCA + UMAP + Leiden pipeline (resolution = 1)
- Cluster label assignment (e.g. Leiden cluster 0 from a 16-cluster spleen dataset)
- Preprocessed expression matrix (CPM total count = 1e5, log1p-transformed, unit-variance scaled)

## Outputs

- Differential expression table with test statistics and log2 fold-change per gene
- Ranked list of the top 500 DE genes by fold-change
- Enrichment results table (GO/KEGG/MSigDB) sorted by corrected p-value (q-value), e.g. enrichment_results.csv

## How to apply

Load the clustered AnnData (e.g. HuBMAP HBMP3-spleen-CC2 after PCA + UMAP + Leiden clustering at resolution 1) and subset cells belonging to the target cluster. Run differential expression with diffxpy using a Welch's t-test of the target cluster versus all remaining cells, operating on data preprocessed via scanpy (CPM normalization to total count 1e5, log1p-transform, scaled to unit variance and zero mean). Sort genes by log2 fold-change and retain the 500 DE genes with the greatest fold-change values. Feed this ranked gene list into GSEApy for GO/KEGG/MSigDB enrichment, writing the q-value-sorted enrichment table (e.g. enrichment_results.csv) to support cluster annotation — in the Cellar paper this pipeline identified spleen cluster 0 as B-cells.

## Related tools

- **diffxpy** (Performs the Welch's t-test differential expression of the target cluster versus the rest) — https://github.com/theislab/diffxpy
- **scanpy** (Preprocessing (filtering, CPM normalization, log1p, scaling) of the AnnData prior to DE testing)
- **GSEApy** (Enrichment analysis of the top 500 DE genes against GO, KEGG, and MSigDB gene sets) — https://github.com/zqfang/GSEApy
- **Cellar** (Interactive pipeline producing the clustered AnnData (PCA + UMAP + Leiden) that the DE step annotates) — https://github.com/euxhenh/cellar

## Examples

```
import diffxpy.api as de; test = de.test.t_test(data=adata[adata.obs['leiden']=='0'], grouping='rest', is_logged=True)
```

## Evaluation signals

- Top enrichment categories for spleen cluster 0 should include 'B-Cell Activation' and 'B-Cell Receptor Signaling Pathway', each with q-value = 0
- Enrichment table is sorted by corrected p-value (q-value) in enrichment_results.csv
- Exactly the 500 DE genes with the greatest fold-change values are passed to GSEApy
- DE genes are computed on the log1p-transformed, CPM-normalized (total count = 1e5) matrix, not raw counts
- The test is a two-group comparison of one Leiden cluster against all remaining cells

## Limitations

- diffxpy is unmaintained; the README recommends PyDESeq2 as an alternative for pure-python differential expression analysis
- Enrichment-based annotation depends on fold-change ranking; clusters without strong marker signatures may yield ambiguous cell-type assignments
- No software versions are stated for diffxpy/GSEApy in the Cellar paper, so exact reproducibility of the q-value = 0 results may vary across package releases

## Evidence

- [methods] diffxpy t-test on cluster vs rest: "Differential gene expression analysis was performed with diffxpy (https://github.com/theislab/diffxpy)"
- [methods] Top 500 by fold-change: "The 500 DE genes with the greatest fold-change values were selected for enrichment analysis"
- [methods] GSEApy enrichment: "enrichment analysis via the package gseapy (https://github.com/zqfang/GSEApy)"
- [methods] Preprocessing via scanpy: "Preprocessing of the data was done via scanpy"
- [readme] diffxpy unmaintained warning: [evidence span withheld — anchor status: external_doc]
