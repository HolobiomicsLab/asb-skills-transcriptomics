---
name: spearman-rank-correlation-of-enrichment-profiles
description: Use when two alternative lineage groupings (e.g., GEMLI-predicted vs.
  ground-truth barcode lineages) must be compared at the level of gene-category enrichment
  rather than per-gene lists, to quantify recovery of memory gene GO-terms.
when_to_use_negative:
- Cell populations contain only distantly related cells, where GEMLI lineage prediction
  (and hence the predicted-lineage enrichment profile) is expected to be less robust
- The goal is long-term lineage tracing over weeks or months; GEMLI's scope is limited
  to lineages over about 1-5 cell divisions
- You need per-lineage identity agreement rather than category-level (GO-term) agreement;
  a rank correlation of enrichment profiles does not verify cell-to-cell lineage assignments
edam_operation: http://edamontology.org/operation_3782
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0085
tools:
- name: GEMLI
  role: Predicts lineages from scRNA-seq via predict_lineages to supply the alternative
    (predicted) grouping for memory gene identification
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Implementation environment for GEMLI, memory gene calling, topGO enrichment,
    and correlation computation
  repo: https://github.com/UPSUTER/GEMLI
- name: devtools
  role: Installs GEMLI via install_github('UPSUTER/GEMLI', subdir='GEMLI_package_v0')
  repo: https://github.com/UPSUTER/GEMLI
- name: topGO
  role: Performs GO biological process over-representation analysis on each memory
    gene set before correlation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:bd8dc7f799b03bb051481421d0a0ccf6ff10a0c6f50a5b79b4c98e0f5394e5e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  - topGO
  edam_operation: http://edamontology.org/operation_3782
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spearman-rank-correlation-of-enrichment-profiles

## Summary

Validates that GEMLI-predicted lineages recover the same heritable (gene expression memory) gene categories as ground-truth barcode lineages by comparing GO biological process over-representation profiles with a Spearman rank correlation (target > 0.7, per Fig. 3a).

## When to use

Use when two alternative lineage groupings (e.g., GEMLI-predicted vs. ground-truth barcode lineages) must be compared at the level of gene-category enrichment rather than per-gene lists, to quantify recovery of memory gene GO-terms.

## When NOT to use

- Cell populations contain only distantly related cells, where GEMLI lineage prediction (and hence the predicted-lineage enrichment profile) is expected to be less robust
- The goal is long-term lineage tracing over weeks or months; GEMLI's scope is limited to lineages over about 1-5 cell divisions
- You need per-lineage identity agreement rather than category-level (GO-term) agreement; a rank correlation of enrichment profiles does not verify cell-to-cell lineage assignments

## Inputs

- Normalized scRNA-seq gene expression matrix (rows = genes, columns = cells) from GEO accession GSE99915
- Ground-truth barcode lineage annotations (barcode ID values, cell ID names)
- GEMLI-predicted cell-by-cell lineage confidence matrix from GEMLI::predict_lineages
- GO biological process ontology annotation (for topGO)

## Outputs

- GO enrichment score table for ground-truth-lineage memory genes (topGO)
- GO enrichment score table for GEMLI-predicted-lineage memory genes (topGO)
- Spearman rank correlation value between the two enrichment profiles (expected > 0.7)
- Combined results file, e.g., correlation_results.csv

## How to apply

From the normalized MEF scRNA-seq matrix (GEO GSE99915), identify memory genes twice: once grouping cells by ground-truth barcode lineage annotations and once by the cell-by-cell lineage confidence matrix from GEMLI::predict_lineages. Run GO biological process over-representation analysis on each memory gene set separately using the Bioconductor topGO package. Then compute the Spearman rank correlation between the two vectors of GO enrichment scores, pairing terms by GO ID. Validate that the correlation exceeds 0.7, consistent with Fig. 3a of Eisele et al. 2024, and export the enrichment score table plus correlation value (e.g., to correlation_results.csv).

## Related tools

- **GEMLI** (Predicts lineages from scRNA-seq via predict_lineages to supply the alternative (predicted) grouping for memory gene identification) — https://github.com/UPSUTER/GEMLI
- **R** (Implementation environment for GEMLI, memory gene calling, topGO enrichment, and correlation computation) — https://github.com/UPSUTER/GEMLI
- **devtools** (Installs GEMLI via install_github('UPSUTER/GEMLI', subdir='GEMLI_package_v0')) — https://github.com/UPSUTER/GEMLI
- **topGO** (Performs GO biological process over-representation analysis on each memory gene set before correlation)

## Examples

```
GEMLI_items <- list(gene_expression = expr_matrix, barcodes = barcode_annotations); predicted <- GEMLI::predict_lineages(GEMLI_items)
```

## Evaluation signals

- Spearman rank correlation between ground-truth and GEMLI-predicted GO enrichment scores exceeds 0.7 (Fig. 3a benchmark)
- Memory gene sets are derived from identical expression matrix and identical memory-gene definition, differing only in grouping (barcode vs. predicted lineages)
- Enrichment tables pair GO terms by ID with no unmatched terms before ranking
- High correlation holds across all datasets tested in the paper, not only the MEF dataset
- correlation_results.csv contains both the enrichment score table and the final correlation value

## Limitations

- GEMLI performs less robustly in cell populations that contain only distantly related cells, weakening the predicted-lineage enrichment profile
- Large variability in growth rates between lineages may decrease GEMLI's performance (cannot be excluded)
- GEMLI's scope is limited to lineages over about 1-5 cell divisions; long-term lineage comparisons need other approaches

## Evidence

- [results] Memory genes and associated GO-terms recovered from GEMLI-predicted lineages are highly correlated with those from ground-truth lineage annotation across all datasets: [evidence span withheld — anchor status: not_found]
- [methods] predict_lineages function: "We can then identify cell lineages through repeated iterative clustering (this may take 2-3min). The `predict_lineages` function takes our GEMLI_items as input"
- [methods] topGO enrichment: [evidence span withheld — anchor status: not_found]
- [readme] GEMLI R installation: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`."
- [discussion] GEMLI limitation on distant cells: "However, GEMLI will generally perform less robustly in cell populations that contain only distantly related cells."
