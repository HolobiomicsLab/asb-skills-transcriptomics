---
name: gene-set-enrichment-analysis
description: Use when a scRNA-seq/scATAC-seq cluster (e.g., a Leiden cluster from
  Cellar) needs a cell-type label and differential expression has yielded ranked genes
  that must be interpreted against GO, KEGG, or MSigDB functional gene sets.
when_to_use_negative:
- When differential expression has not yet been performed — the input must already
  be a ranked list of DE genes, not a raw count matrix or cell-by-gene AnnData object
- When assigning cell types based on known canonical markers alone (e.g., SLC5A12
  for Proximal Tubule), where marker intersection suffices and no functional gene-set
  testing is needed
- When cluster labels already exist from reference-based label transfer (e.g., scanpy
  Ingest) and no functional interpretation of the clusters is required
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GSEApy
  role: Performs the enrichment analysis on the top 500 DE genes, implementing the
    GSEA method against GO, KEGG, and MSigDB gene sets
  repo: https://github.com/zqfang/GSEApy
- name: diffxpy
  role: Produces the ranked DE genes (Welch's t-test, cluster vs. rest) that feed
    the enrichment step
  repo: https://github.com/theislab/diffxpy
- name: Cellar
  role: Interactive pipeline that orchestrates clustering (PCA + UMAP + Leiden) and
    the subsequent DE + enrichment workflow for cluster annotation
  repo: https://github.com/euxhenh/cellar
- name: AnnData (Annotated Data object)
  role: In-memory object format holding the clustered cells from which cluster-specific
    gene lists are subset
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:ee92e35a8c27bc337456adece385b6b1a1c7ee31f279f416a5e257287fa2abf7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zqfang/GSEApy
  tools:
  - AnnData
  - AnnData (Annotated Data object)
  - Cellar
  - GSEApy
  - Python
  - cisTopic
  - diffxpy
  - faiss
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# gene-set-enrichment-analysis

## Summary

Functional enrichment of cluster marker genes to assign cell types: after differential expression with diffxpy, the top 500 DE genes by fold-change are tested against GO, KEGG, and MSigDB gene sets with GSEApy, producing a q-value-sorted enrichment table that supports cell-type annotation of Cellar clusters.

## When to use

Use when a scRNA-seq/scATAC-seq cluster (e.g., a Leiden cluster from Cellar) needs a cell-type label and differential expression has yielded ranked genes that must be interpreted against GO, KEGG, or MSigDB functional gene sets.

## When NOT to use

- When differential expression has not yet been performed — the input must already be a ranked list of DE genes, not a raw count matrix or cell-by-gene AnnData object
- When assigning cell types based on known canonical markers alone (e.g., SLC5A12 for Proximal Tubule), where marker intersection suffices and no functional gene-set testing is needed
- When cluster labels already exist from reference-based label transfer (e.g., scanpy Ingest) and no functional interpretation of the clusters is required

## Inputs

- Ranked/filtered list of top 500 DE genes by log2 fold-change from a diffxpy Welch's t-test (cluster vs. rest)
- Clustered AnnData object with Leiden cluster assignments (Cellar pipeline output)
- Gene set collections: GO, KEGG, MSigDB

## Outputs

- Enrichment table of GO/KEGG/MSigDB categories with p-values and corrected q-values (enrichment_results.csv)
- Cell-type assignment supported by top enrichment terms (e.g., cluster 0 = B-cells, cluster 1 = Proximal Tubule Cells)

## How to apply

First run differential expression with diffxpy using a Welch's t-test of one cluster versus the rest (e.g., spleen cluster 0 in the HBMP3-spleen-CC2 HuBMAP data, or SNARE-seq kidney clusters after cisTopic topic clustering and cell-ID label transfer). Sort DE genes by log2 fold-change and select the 500 DE genes with the greatest fold-change values for enrichment analysis. Pass this gene list to GSEApy, which implements the GSEA method, testing against GO, KEGG, and MSigDB functional gene sets. Write the resulting enrichment table sorted by corrected p-value (q-value) to a file such as enrichment_results.csv. Judge success by whether the top categories recapitulate known biology: e.g., 'B-Cell Activation' and 'B-Cell Receptor Signaling Pathway' with q value = 0 confirm spleen cluster 0 as B-cells, while GO 'Apical Plasma Membrane' (p value = 1e-4) plus marker SLC5A12 (p value = 0) support SNARE-seq kidney cluster 1 as Proximal Tubule Cells.

## Related tools

- **GSEApy** (Performs the enrichment analysis on the top 500 DE genes, implementing the GSEA method against GO, KEGG, and MSigDB gene sets) — https://github.com/zqfang/GSEApy
- **diffxpy** (Produces the ranked DE genes (Welch's t-test, cluster vs. rest) that feed the enrichment step) — https://github.com/theislab/diffxpy
- **Cellar** (Interactive pipeline that orchestrates clustering (PCA + UMAP + Leiden) and the subsequent DE + enrichment workflow for cluster annotation) — https://github.com/euxhenh/cellar
- **AnnData (Annotated Data object)** (In-memory object format holding the clustered cells from which cluster-specific gene lists are subset)

## Examples

```
import gseapy; enr = gseapy.enrichr(gene_list=top500_de_genes, gene_sets=['GO_Biological_Process_2021','KEGG_2021_Human','MSigDB_Hallmark_2020'], outdir='enrichment_results', cutoff=0.05)
```

## Evaluation signals

- Top GO category for spleen cluster 0 is 'B-Cell Activation' with q value = 0, matching the reported result
- Top KEGG category for spleen cluster 0 is 'B-Cell Receptor Signaling Pathway' with q value = 0
- SNARE-seq kidney joint-modality cluster 1 shows GO 'Apical Plasma Membrane' with p value = 1e-4 alongside marker SLC5A12 with p value = 0
- Enrichment table is written to enrichment_results.csv sorted by corrected p-value (q-value)
- Exactly the 500 DE genes with the greatest fold-change values were submitted to GSEApy

## Limitations

- diffxpy is unmaintained — the README warns 'This repo is unmaintained' and suggests PyDESeq2 for pure-python differential expression, so the upstream DE step may need substitution
- The cisTopic R package is deprecated (use pycisTopic), affecting the SNARE-seq clustering step upstream of enrichment
- No software versions are stated for GSEApy or diffxpy in the article, so exact gene-set library versions used are not recorded, which can affect q-value reproduction

## Evidence

- [methods] The 500 DE genes with the greatest fold-change values were selected for enrichment analysis: "The 500 DE genes with the greatest fold-change values were selected for enrichment analysis"
- [methods] enrichment via gseapy: "enrichment analysis via the package gseapy (https://github.com/zqfang/GSEApy)"
- [methods] Differential gene expression analysis was performed with diffxpy: "Differential gene expression analysis was performed with diffxpy (https://github.com/theislab/diffxpy)"
- [readme] GSEApy README description: [evidence span withheld — anchor status: external_doc]
