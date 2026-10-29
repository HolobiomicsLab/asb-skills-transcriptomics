---
name: cell-type-functional-annotation
description: Use when an unsupervised clustering (e.g., Leiden) of scRNA-seq data
  has produced clusters with no labels and you must assign a cell type from the functional
  enrichment of cluster-specific genes.
when_to_use_negative:
- Cells are already annotated (e.g., via SingleR reference-based label transfer or
  scanpy Ingest from HBMP2-spleen-2 to HBMP3-spleen-CC3) — enrichment would be redundant
- The input is raw, unclustered, or unfiltered count data — preprocessing (filtering
  50–3000 genes/cells, CPM 1e5 normalization, log1p, scaling) and clustering must
  precede this skill
- The organism or tissue lacks adequate GO/KEGG/MSigDB gene-set coverage, making enrichment
  categories uninterpretable
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: diffxpy
  role: Performs the Welch's t-test differential expression of the target cluster
    versus the rest
  repo: https://github.com/theislab/diffxpy
- name: GSEApy
  role: Runs GSEA-based enrichment analysis of the top 500 DE genes against GO, KEGG,
    and MSigDB gene sets
  repo: https://github.com/zqfang/GSEApy
- name: Cellar
  role: Interactive pipeline in which clustering, differential expression, and functional-set
    intersection steps are executed
  repo: https://github.com/euxhenh/cellar
- name: scanpy
  role: Used for preprocessing of the data and handling of the AnnData object that
    feeds the DE and enrichment steps
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:e864a614b755a11b1ab536d902f2a8d6bfd7935046a284ca900ebe6e2b5b7bef
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# cell-type-functional-annotation

## Summary

Annotate single-cell clusters by cell type by running differential expression on one cluster versus the rest (diffxpy), selecting the top 500 DE genes by fold-change, and performing GO/KEGG/MSigDB enrichment with GSEApy (GSEA) to infer function from enriched gene sets.

## When to use

Use when an unsupervised clustering (e.g., Leiden) of scRNA-seq data has produced clusters with no labels and you must assign a cell type from the functional enrichment of cluster-specific genes.

## When NOT to use

- Cells are already annotated (e.g., via SingleR reference-based label transfer or scanpy Ingest from HBMP2-spleen-2 to HBMP3-spleen-CC3) — enrichment would be redundant
- The input is raw, unclustered, or unfiltered count data — preprocessing (filtering 50–3000 genes/cells, CPM 1e5 normalization, log1p, scaling) and clustering must precede this skill
- The organism or tissue lacks adequate GO/KEGG/MSigDB gene-set coverage, making enrichment categories uninterpretable

## Inputs

- Clustered AnnData object from PCA + UMAP + Leiden clustering (e.g., resolution = 1, 16 clusters) of the HuBMAP spleen dataset
- Cluster label of interest (e.g., cluster 0) to test versus all other cells
- Functional gene set databases: GO, KEGG, MSigDB

## Outputs

- enrichment_results.csv — enrichment table with GO/KEGG/MSigDB categories, p-values, and corrected q-values sorted by q-value
- Assigned cell-type label for the target cluster (e.g., cluster 0 = B-cells)

## How to apply

Take the clustered AnnData object and subset the cells of the target cluster (e.g., cluster 0 of a 16-cluster Leiden solution at resolution = 1). Run differential expression with diffxpy using a Welch's t-test of the target cluster versus all remaining cells, then sort DE genes by log2 fold-change and select the 500 with the greatest fold-change for enrichment analysis. Run enrichment with GSEApy, which implements the GSEA method, testing the gene list against functional gene sets from GO, KEGG, and MSigDB. Write the enrichment table sorted by corrected p-value (q-value), e.g., to enrichment_results.csv. In the Cellar spleen analysis this identified HuBMAP HBMP3-spleen-CC2 cluster 0 as B-cells, with 'B-Cell Activation' (GO, q = 0) and 'B-Cell Receptor Signaling Pathway' (KEGG, q = 0) as top categories. Judge success by confirming the top q-value = 0 categories are biologically coherent (e.g., B-cell terms for a B-cell cluster) and that the table is properly q-value-sorted.

## Related tools

- **diffxpy** (Performs the Welch's t-test differential expression of the target cluster versus the rest) — https://github.com/theislab/diffxpy
- **GSEApy** (Runs GSEA-based enrichment analysis of the top 500 DE genes against GO, KEGG, and MSigDB gene sets) — https://github.com/zqfang/GSEApy
- **Cellar** (Interactive pipeline in which clustering, differential expression, and functional-set intersection steps are executed) — https://github.com/euxhenh/cellar
- **scanpy** (Used for preprocessing of the data and handling of the AnnData object that feeds the DE and enrichment steps)

## Examples

```
import gseapy; enr = gseapy.enrichr(gene_list=top500_de_genes, gene_sets=['GO_Biological_Process_2018','KEGG_2016_Human','MSigDB_Hallmark_2020'], out_dir='enrichment_results')
```

## Evaluation signals

- Top GO and KEGG enrichment categories have corrected q-value = 0 (e.g., 'B-Cell Activation' and 'B-Cell Receptor Signaling Pathway' for spleen cluster 0)
- Exactly 500 DE genes with the greatest log2 fold-change values were passed to GSEApy
- enrichment_results.csv exists, is sorted by q-value, and contains columns for GO, KEGG, and MSigDB categories
- The assigned cell type is consistent with the tissue of origin (e.g., B-cells in spleen) and with known marker biology
- The DE test used is Welch's t-test (diffxpy) comparing the single target cluster against all remaining clusters

## Limitations

- Cellar relies on annotation of reference datasets; HuBMAP annotated references are given without accession IDs, complicating exact reproduction
- No software versions stated for Cellar, Python, Dash, or faiss; diffxpy is unmaintained (upstream suggests PyDESeq2 for pure-python differential expression)
- Enrichment-based annotation is indirect: it assigns function from gene sets rather than a supervised classifier, so interpretation depends on the quality of GO/KEGG/MSigDB coverage for the tissue

## Evidence

- [methods] Differential gene expression analysis was performed with diffxpy: "Differential gene expression analysis was performed with diffxpy (https://github.com/theislab/diffxpy)"
- [methods] enrichment via gseapy: "enrichment analysis via the package gseapy (https://github.com/zqfang/GSEApy)"
- [methods] 500 DE genes by fold-change: "The 500 DE genes with the greatest fold-change values were selected for enrichment analysis"
- [intro] functional and marker set intersection step: "identification of differentially expressed genes, intersection with functional and marker sets"
- [readme] diffxpy unmaintained warning: [evidence span withheld — anchor status: external_doc]
