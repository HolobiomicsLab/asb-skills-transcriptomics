---
name: lineage-relatedness-statistical-testing
description: Use when scRNA-seq expression matrices are paired with cellular barcode
  lineage annotations (e.g., GEO GSE74923 L1210 or CD8+ T-lymphocyte data) and you
  must statistically test whether lineage-related cells are transcriptionally more
  similar than random pairs.
when_to_use_negative:
- When no independent lineage barcode ground truth is available and the goal is to
  predict lineages de novo — use GEMLI's predict_lineages workflow instead of validating
  known barcodes
- When the cell population contains only distantly related cells, since GEMLI-based
  lineage analyses perform less robustly in such populations
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GEMLI
  role: R package whose GEMLI_items structure ('gene_expression' and 'barcodes' slots)
    holds the expression matrix and barcode annotations used to enumerate related
    cell pairs for the correlation distance test
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Runtime environment in which the Pearson correlation distances, random pair
    sampling, and Mann-Whitney U rank-sum comparison are computed
  repo: https://github.com/UPSUTER/GEMLI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:e226db02b4563d54e4b2c2696da748c5935ee9a642191cbf852e87e13e4c7ce4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# lineage-relatedness-statistical-testing

## Summary

A validation strategy from the GEMLI gene-expression-memory framework that quantifies whether cells sharing a lineage barcode exhibit lower whole-transcriptome Pearson correlation distances than randomly sampled unrelated cell pairs, using a non-parametric rank-sum test as the acceptance criterion.

## When to use

Use when scRNA-seq expression matrices are paired with cellular barcode lineage annotations (e.g., GEO GSE74923 L1210 or CD8+ T-lymphocyte data) and you must statistically test whether lineage-related cells are transcriptionally more similar than random pairs.

## When NOT to use

- When no independent lineage barcode ground truth is available and the goal is to predict lineages de novo — use GEMLI's predict_lineages workflow instead of validating known barcodes
- When the cell population contains only distantly related cells, since GEMLI-based lineage analyses perform less robustly in such populations

## Inputs

- Normalized, quality-controlled scRNA-seq gene expression matrix (e.g., from GEO GSE74923 L1210 or CD8+ T-lymphocyte datasets)
- Cellular barcode lineage annotations (cell ID -> barcode ID mapping, e.g., a named vector or table)
- A GEMLI_items list with 'gene_expression' and 'barcodes' slots (R)

## Outputs

- correlation_distance_comparison.csv with mean/median correlation distances for related vs. random pairs and the Mann-Whitney U p-value
- correlation_distance_comparison.png with overlaid correlation distance distributions for related vs. random pairs

## How to apply

Assemble the normalized, quality-controlled gene expression matrix and barcode annotations into a GEMLI_items list with 'gene_expression' and 'barcodes' slots, then group cells by shared barcode ID via a named vector (cell IDs as names, barcode IDs as values) to enumerate all within-lineage (related) cell pairs. Compute Pearson correlation distance (1 - Pearson r) across the whole transcriptome for every related pair from the normalized matrix. Sample an equal number of unrelated cell pairs (different barcodes) and compute the same metric as the negative control. Compare the two distance distributions with a Mann-Whitney U (rank-sum) test; the analysis is accepted if related cells show significantly lower correlation distance (p < 0.05), which is the gene-expression-memory basis underlying GEMLI's lineage predictions. Record mean/median distances and the p-value in a correlation_distance_comparison.csv and plot the distributions as correlation_distance_comparison.png.

## Related tools

- **GEMLI** (R package whose GEMLI_items structure ('gene_expression' and 'barcodes' slots) holds the expression matrix and barcode annotations used to enumerate related cell pairs for the correlation distance test) — https://github.com/UPSUTER/GEMLI
- **R** (Runtime environment in which the Pearson correlation distances, random pair sampling, and Mann-Whitney U rank-sum comparison are computed) — https://github.com/UPSUTER/GEMLI

## Examples

```
library(devtools); install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")
```

## Evaluation signals

- Lineage-related cells must exhibit significantly lower correlation distance than random pairs (Mann-Whitney U p < 0.05)
- The number of randomly sampled unrelated pairs must equal the number of within-lineage related pairs for a fair comparison
- correlation_distance_comparison.csv must contain mean and median distances per group plus the recorded p-value
- All distance values must lie in [0, 2], the valid range of Pearson correlation distance (1 - r)
- Every cell appearing in a related pair must map to a non-NA barcode ID in the named-vector cell-to-barcode mapping

## Limitations

- GEMLI performs less robustly in cell populations that contain only distantly related cells, which would weaken the related-vs-random distance separation
- Large variability in growth rates between lineages may decrease GEMLI's performance (cannot be excluded)
- The gene-expression-memory signal spans lineages over about 1-5 cell divisions; this test is not suited to long-term lineage relationships

## Evidence

- [abstract] a computational tool allowing to robustly identify small to medium-sized cell lineages solely from scRNA-seq datasets: "Here we introduce Gene Expression Memory-based Lineage Inference (GEMLI), a computational tool allowing to robustly identify small to medium-sized cell lineages solely from scRNA-seq datasets."
- [methods] GEMLI installation via devtools in R: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`"
- [intro] quantitative dissection of gene expression maintenance across cell types: "Here, we quantitatively dissect the maintenance of gene expression in cell lineages for a broad range of cell types."
- [readme] GEMLI R package description: "GEMLI is an R package to predict cell lineages (cells with a common ancestor) from single cell RNA sequencing datasets"
