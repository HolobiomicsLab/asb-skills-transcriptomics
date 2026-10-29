---
name: fold-change-gene-ranking
description: 'Use when a Leiden/louvain cluster must be annotated by function and
  you need a compact, high-confidence gene list: rank Welch t-test DE results by log2
  fold-change and keep the top 500 genes for enrichment analysis.'
when_to_use_negative:
- Input is already a curated marker gene list or the clusters are already annotated
  — ranking by fold-change adds nothing before enrichment.
- Only a handful of genes are differentially expressed (small clusters or weak contrasts),
  where an arbitrary top-500 cutoff would dilute enrichment with noise.
- You need statistical significance ranking rather than effect-size ranking — the
  article's method sorts by fold-change, not q-value, for gene selection.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: diffxpy
  role: Performs the Welch's t-test differential expression of the target cluster
    versus the rest, yielding log2 fold-changes used for ranking
  repo: https://github.com/theislab/diffxpy
- name: GSEApy
  role: Consumes the top-500 fold-change-ranked gene list for GO/KEGG/MSigDB enrichment
    analysis via the GSEA method
  repo: https://github.com/zqfang/GSEApy
- name: Cellar
  role: Hosts the upstream pipeline (PCA + UMAP + Leiden clustering) and produces
    the clustered AnnData whose clusters are annotated by this skill
  repo: https://github.com/euxhenh/cellar
- name: AnnData
  role: In-memory annotated data object holding the filtered, CPM-normalized, log1p-transformed
    expression matrix and cluster labels
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:3c90f362f20c5c91b4aafa85f5f80a06ae3bd30911753d50390ed5bc93c2860c
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# fold-change-gene-ranking

## Summary

Rank differentially expressed genes by log2 fold-change and select the top 500 by greatest fold-change to feed downstream functional enrichment (GO/KEGG/MSigDB via GSEApy), as done in Cellar's cell-type annotation of the HuBMAP HBMP3 spleen cluster 0.

## When to use

Use when a Leiden/louvain cluster must be annotated by function and you need a compact, high-confidence gene list: rank Welch t-test DE results by log2 fold-change and keep the top 500 genes for enrichment analysis.

## When NOT to use

- Input is already a curated marker gene list or the clusters are already annotated — ranking by fold-change adds nothing before enrichment.
- Only a handful of genes are differentially expressed (small clusters or weak contrasts), where an arbitrary top-500 cutoff would dilute enrichment with noise.
- You need statistical significance ranking rather than effect-size ranking — the article's method sorts by fold-change, not q-value, for gene selection.

## Inputs

- Clustered AnnData object (.h5ad) with a Leiden clustering (resolution = 1) column from Cellar's PCA + UMAP + Leiden pipeline (16 clusters)
- Subset of cells assigned to the target cluster (e.g., cluster 0) versus remaining cells
- Preprocessed expression matrix: CPM total count normalized (total count = 1e5), log1p-transformed, scaled to unit variance/zero mean

## Outputs

- Ranked DE gene table with log2 fold-change values from diffxpy's Welch's t-test
- List of the top 500 DE genes by fold-change
- Enrichment results table (GO/KEGG/MSigDB via GSEApy) sorted by q-value, e.g., enrichment_results.csv

## How to apply

After subsetting the AnnData object to the target cluster (e.g., spleen cluster 0 from Cellar's 16-cluster Leiden solution at resolution = 1), run differential expression of the cluster versus the rest with diffxpy using a Welch's t-test. Sort genes by log2 fold-change and select the 500 DE genes with the greatest fold-change values — this hard cutoff is the article's stated filter step and focuses enrichment on the strongest, most interpretable markers. Pass the ranked/selected gene list to GSEApy, which implements the GSEA method, testing against GO, KEGG, and MSigDB functional gene sets, and write the enrichment table sorted by corrected p-value (q-value). Validate that the top categories are biologically coherent for the cluster (e.g., 'B-Cell Activation' and 'B-Cell Receptor Signaling Pathway' with q value = 0 for the B-cell spleen cluster). DE is meaningful here only because the data were preprocessed per Cellar's recipe: cell/gene filtering (50–3000 expressed genes/cells), CPM normalization to total count 1e5, log1p transformation, and scaling to unit variance and zero mean.

## Related tools

- **diffxpy** (Performs the Welch's t-test differential expression of the target cluster versus the rest, yielding log2 fold-changes used for ranking) — https://github.com/theislab/diffxpy
- **GSEApy** (Consumes the top-500 fold-change-ranked gene list for GO/KEGG/MSigDB enrichment analysis via the GSEA method) — https://github.com/zqfang/GSEApy
- **Cellar** (Hosts the upstream pipeline (PCA + UMAP + Leiden clustering) and produces the clustered AnnData whose clusters are annotated by this skill) — https://github.com/euxhenh/cellar
- **AnnData** (In-memory annotated data object holding the filtered, CPM-normalized, log1p-transformed expression matrix and cluster labels)

## Examples

```
import diffxpy.api as de; res = de.test.t_test(data=adata_cluster0, sample_desc=groups); top500 = res.summary().sort_values('log2fc', ascending=False).head(500); import gseapy; gseapy.enrichr(gene_list=list(top500.gene), gene_sets=['GO_Biological_Process_2021','KEGG_2021_Human','MSigDB_Hallmark_2020'], outdir='enrichment_results')
```

## Evaluation signals

- The selected gene set contains exactly 500 genes and matches the top 500 rows of the DE table sorted by log2 fold-change (greatest fold-change values).
- The DE test is a Welch's t-test comparing cluster 0 against all other cells, not a one-sided or pairwise cluster comparison.
- Enrichment table is sorted by q-value and the top entries include 'B-Cell Activation' and 'B-Cell Receptor Signaling Pathway' with q value = 0 for the HuBMAP spleen cluster 0.
- Downstream enrichment was run with GSEApy against GO, KEGG, and MSigDB gene sets, and results were written to enrichment_results.csv.
- Preprocessing invariants hold before the DE test: cells with <50 or >3000 expressed genes removed, genes expressed in <50 or >3000 cells removed, CPM (total count = 1e5), log1p, unit-variance scaling.

## Limitations

- diffxpy is officially unmaintained — the README warns about this and points to PyDESeq2 as an alternative for pure-python differential expression.
- The fixed top-500 cutoff is arbitrary and was not justified statistically in the article; results may vary with gene-set size.
- No software versions are stated for diffxpy, GSEApy, or Cellar in the article, so exact reproducibility of fold-change values is not guaranteed.
- Fold-change ranking depends on upstream normalization choices (CPM 1e5, log1p, unit-variance scaling); different preprocessing can change which genes make the top 500.

## Evidence

- [methods] The 500 DE genes with the greatest fold-change values were selected for enrichment analysis: "The 500 DE genes with the greatest fold-change values were selected for enrichment analysis"
- [methods] diffxpy used for differential expression: "Differential gene expression analysis was performed with diffxpy (https://github.com/theislab/diffxpy)"
- [methods] GSEApy used for enrichment: "enrichment analysis via the package gseapy (https://github.com/zqfang/GSEApy)"
- [readme] diffxpy README purpose: [evidence span withheld — anchor status: external_doc]
- [readme] GSEApy README purpose: [evidence span withheld — anchor status: external_doc]
