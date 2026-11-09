---
name: go-biological-process-enrichment-testing
description: Use when scRNA-seq lineage predictions must be functionally validated
  by checking whether GO BP terms recovered from memory genes of predicted lineages
  match those from ground-truth lineage annotations (e.g., Spearman correlation of
  enrichment scores, Fig. 3a).
when_to_use_negative:
- The cell population contains only distantly related cells, so GEMLI-predicted lineages
  (and thus the memory gene sets being tested) are unreliable as groupings
- The input is already a GO term list or enrichment table — there is no gene-level
  memory set to run over-representation on
- The lineage question spans long time scales (weeks/months) beyond GEMLI's ~1–5 cell
  division scope, so predicted lineages are not a valid basis for defining memory
  genes
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0203
tools:
- name: topGO
  role: Bioconductor package used to perform GO biological-process over-representation
    analysis on each memory gene set
- name: GEMLI
  role: R package that supplies the predicted lineages (predict_lineages on a GEMLI_items
    list) from which memory genes are called for enrichment testing
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Runtime environment in which GEMLI, topGO, and the Spearman correlation of
    enrichment scores are executed
- name: devtools
  role: Used to install the GEMLI package from GitHub (install_github('UPSUTER/GEMLI',
    subdir='GEMLI_package_v0'))
  repo: https://github.com/UPSUTER/GEMLI
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:fc4518013d0a12ed3f49aa83ffeae8c22a693424c8333aef46bdb7d590c40afd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  - topGO
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# go-biological-process-enrichment-testing

## Summary

Runs Gene Ontology biological-process over-representation analysis (Bioconductor topGO) on heritably expressed 'memory gene' sets derived from GEMLI-predicted and barcode ground-truth lineages, then compares the enrichment score profiles to validate lineage-inference quality.

## When to use

Use when scRNA-seq lineage predictions must be functionally validated by checking whether GO BP terms recovered from memory genes of predicted lineages match those from ground-truth lineage annotations (e.g., Spearman correlation of enrichment scores, Fig. 3a).

## When NOT to use

- The cell population contains only distantly related cells, so GEMLI-predicted lineages (and thus the memory gene sets being tested) are unreliable as groupings
- The input is already a GO term list or enrichment table — there is no gene-level memory set to run over-representation on
- The lineage question spans long time scales (weeks/months) beyond GEMLI's ~1–5 cell division scope, so predicted lineages are not a valid basis for defining memory genes

## Inputs

- Normalized MEF scRNA-seq gene expression matrix (genes × cells, GEO accession GSE99915)
- Cellular barcode lineage annotations from GSE99915 (barcode ID values, cell ID names)
- GEMLI-predicted lineage confidence matrix (cell-by-cell) from predict_lineages
- Memory (heritably expressed) gene sets derived from each lineage grouping

## Outputs

- GO biological-process over-representation score table per memory gene set (topGO)
- Spearman rank correlation between ground-truth and predicted-lineage GO enrichment scores
- correlation_results.csv containing enrichment scores and the correlation value

## How to apply

Assemble the normalized scRNA-seq matrix (rows = genes, columns = cells) and barcode annotations into a GEMLI_items list with 'gene_expression' and 'barcodes' slots, run GEMLI::predict_lineages to obtain the cell-by-cell lineage confidence matrix, and identify heritably expressed memory genes separately under (a) barcode ground-truth groupings and (b) GEMLI-predicted groupings. For each memory gene set, perform GO biological-process over-representation analysis with the Bioconductor topGO package, keeping annotation settings identical between the two sets so scores are comparable. Compute the Spearman rank correlation between the GO enrichment scores of the ground-truth-derived and predicted-derived memory gene sets; in the GEMLI paper this correlation exceeds 0.7 across datasets (Fig. 3a), confirming that GEMLI reliably identifies categories of memory genes. Save the enrichment score table and correlation value (e.g., correlation_results.csv) as the validation record.

## Related tools

- **topGO** (Bioconductor package used to perform GO biological-process over-representation analysis on each memory gene set)
- **GEMLI** (R package that supplies the predicted lineages (predict_lineages on a GEMLI_items list) from which memory genes are called for enrichment testing) — https://github.com/UPSUTER/GEMLI
- **R** (Runtime environment in which GEMLI, topGO, and the Spearman correlation of enrichment scores are executed)
- **devtools** (Used to install the GEMLI package from GitHub (install_github('UPSUTER/GEMLI', subdir='GEMLI_package_v0'))) — https://github.com/UPSUTER/GEMLI

## Examples

```
library(devtools); install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"); GEMLI_items <- list(gene_expression = expr, barcodes = barcodes); pred <- predict_lineages(GEMLI_items)
```

## Evaluation signals

- Spearman rank correlation between ground-truth and predicted-lineage GO enrichment scores exceeds 0.7, consistent with Fig. 3a
- Memory genes and associated GO-terms from GEMLI-predicted lineages are highly correlated with those from ground-truth barcode lineages across all datasets
- topGO over-representation was run separately on both memory gene sets (barcode-derived and GEMLI-derived) with identical settings, yielding comparable score tables
- correlation_results.csv exists and contains both the enrichment score table and the correlation value
- Memory gene identification was performed on both the barcode ground-truth groupings and the GEMLI-predicted groupings from the same expression matrix

## Limitations

- GEMLI generally performs less robustly in cell populations that contain only distantly related cells, weakening the predicted-lineage memory gene set
- Large variability in growth rates between lineages may decrease GEMLI's performance, which cannot be excluded
- GEMLI's scope is limited to lineages over about 1–5 cell divisions; long-term lineage relationships require other approaches

## Evidence

- [results] Memory genes and associated GO-terms recovered from GEMLI-predicted lineages are highly correlated with those from ground-truth lineage annotation across all datasets: [evidence span withheld — anchor status: not_found]
- [readme] to call genes with a high gene expression memory on the predicted cell lineages: "GEMLI is an R package to predict cell lineages (cells with a common ancestor) from single cell RNA sequencing datasets and to call genes with a high gene expression memory on the predicted cell"
- [methods] predict_lineages takes GEMLI_items as input: "The `predict_lineages` function takes our GEMLI_items as input"
- [abstract] GEMLI allows studying heritable gene expression and cell fate decisions: "GEMLI allows to study heritable gene expression, to discriminate symmetric and asymmetric cell fate decisions and to reconstruct individual multicellular structures from pooled scRNA-seq datasets."
