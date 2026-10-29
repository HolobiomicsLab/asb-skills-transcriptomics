---
name: welch-t-test-de-testing
description: Use when a clustered single-cell RNA-seq AnnData object needs a per-cluster
  DE profile to assign a cell type, e.g. testing one Leiden cluster against the rest
  before GO/KEGG/MSigDB enrichment.
when_to_use_negative:
- The data are not yet clustered into cell groups — Welch's t-test of cluster vs.
  rest requires a prior Leiden/clustering step
- The goal is bulk RNA-seq differential expression across conditions rather than single-cell
  cluster-vs-rest testing (the unmaintained diffxpy README suggests PyDESeq2 for pure-python
  bulk DE analysis)
- Only marker-based annotation without enrichment is needed — marker set intersection
  may be more appropriate than a ranked fold-change gene list
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: diffxpy
  role: Performs the Welch's t-test differential expression of the target cluster
    versus the rest
  repo: https://github.com/theislab/diffxpy
- name: GSEApy
  role: Runs GSEA enrichment on the top 500 fold-change-selected DE genes against
    GO, KEGG, and MSigDB gene sets
  repo: https://github.com/zqfang/GSEApy
- name: Cellar
  role: Produces the clustered AnnData input and orchestrates the DE and enrichment
    steps; written in Python using Dash
  repo: https://github.com/euxhenh/cellar
- name: AnnData
  role: In-memory annotated data object holding the clustered spleen expression matrix,
    used in memory-mapping mode for large datasets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:2baf20bbc512a5cc419565c3df5506bd8737d98ed105544988beda97cc88f4ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/diffxpy
  tools:
  - AnnData
  - Cellar
  - GSEApy
  - Python
  - diffxpy
  - scanpy
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# welch-t-test-de-testing

## Summary

Differential expression analysis of a single Leiden cluster versus all remaining cells using diffxpy's Welch's t-test, followed by selection of the top fold-change genes for downstream enrichment. In the Cellar paper this workflow assigned spleen cluster 0 (HBMP3-spleen data) to B-cells.

## When to use

Use when a clustered single-cell RNA-seq AnnData object needs a per-cluster DE profile to assign a cell type, e.g. testing one Leiden cluster against the rest before GO/KEGG/MSigDB enrichment.

## When NOT to use

- The data are not yet clustered into cell groups — Welch's t-test of cluster vs. rest requires a prior Leiden/clustering step
- The goal is bulk RNA-seq differential expression across conditions rather than single-cell cluster-vs-rest testing (the unmaintained diffxpy README suggests PyDESeq2 for pure-python bulk DE analysis)
- Only marker-based annotation without enrichment is needed — marker set intersection may be more appropriate than a ranked fold-change gene list

## Inputs

- Clustered AnnData (.h5ad) object from Cellar's PCA + UMAP + Leiden clustering (16 clusters, resolution = 1) on the HuBMAP spleen dataset
- Cluster label of the target cluster (e.g. cluster 0) and the set of all remaining cells as the contrast group

## Outputs

- Differential expression table with per-gene statistics (log2 fold-change, Welch's t-test p-values)
- Ranked list of the top 500 DE genes by fold-change
- enrichment_results.csv: GO/KEGG/MSigDB enrichment table sorted by corrected p-value (q-value)

## How to apply

Load the clustered AnnData object (produced by Cellar's PCA + UMAP + Leiden pipeline, e.g. resolution = 1 on the HuBMAP HBMP3-spleen data with 16 clusters) and subset cells of the target cluster. Run differential expression with diffxpy (https://github.com/theislab/diffxpy) using a Welch's t-test of the cluster of interest versus all other cells. Sort the resulting DE genes by log2 fold-change and retain the 500 genes with the greatest fold-change values for enrichment analysis, following the Cellar methods. Pass this ranked gene list to GSEApy for GO, KEGG, and MSigDB enrichment and sort the resulting table by q-value (corrected p-value).

## Related tools

- **diffxpy** (Performs the Welch's t-test differential expression of the target cluster versus the rest) — https://github.com/theislab/diffxpy
- **GSEApy** (Runs GSEA enrichment on the top 500 fold-change-selected DE genes against GO, KEGG, and MSigDB gene sets) — https://github.com/zqfang/GSEApy
- **Cellar** (Produces the clustered AnnData input and orchestrates the DE and enrichment steps; written in Python using Dash) — https://github.com/euxhenh/cellar
- **AnnData** (In-memory annotated data object holding the clustered spleen expression matrix, used in memory-mapping mode for large datasets)

## Examples

```
import diffxpy.api as de; test = de.test.t_test(data=adata[adata.obs['leiden']=='0'], grouping='rest', is_logged=True)
```

## Evaluation signals

- The top enrichment categories for spleen cluster 0 include 'B-Cell Activation' (GO) and 'B-Cell Receptor Signaling Pathway' (KEGG), both with q-value = 0
- The DE gene list is capped at exactly the 500 genes with the greatest fold-change values before enrichment
- The enrichment table written to enrichment_results.csv is sorted by corrected p-value (q-value)
- The Welch's t-test contrast is one Leiden cluster (cluster 0 of 16) versus all remaining cells, not pairwise cluster comparisons

## Limitations

- diffxpy is explicitly unmaintained; its README recommends PyDESeq2 for pure-python differential expression analysis instead
- Reproducibility depends on the Cellar web server (https://cellar.cmu.hubmapconsortium.org/app/cellar) rather than a locally installable archived artifact
- No software versions are stated for Cellar, Python, or diffxpy, and no accession IDs are given for the HuBMAP reference datasets

## Evidence

- [methods] Differential gene expression analysis was performed with diffxpy: "Differential gene expression analysis was performed with diffxpy (https://github.com/theislab/diffxpy)"
- [methods] enrichment via GSEApy: "enrichment analysis via the package gseapy (https://github.com/zqfang/GSEApy)"
- [methods] Top 500 DE genes by fold-change: "The 500 DE genes with the greatest fold-change values were selected for enrichment analysis"
- [methods] Preprocessing via scanpy: "Preprocessing of the data was done via scanpy"
- [readme] diffxpy unmaintained warning: [evidence span withheld — anchor status: external_doc]
