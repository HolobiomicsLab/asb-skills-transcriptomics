---
name: barcoded-lineage-cell-pairing
description: Use when scRNA-seq data comes with barcode lineage annotations (e.g.
  GEO GSE74923 L1210 or CD8+ T-cell datasets) and you need to verify that related
  cells are more transcriptomically similar than random pairs.
when_to_use_negative:
- The scRNA-seq dataset lacks barcode or other ground-truth lineage annotations, so
  related cell pairs cannot be enumerated
- The cell population contains only distantly related cells, where gene-expression-memory
  signals are weak
- The goal is long-term lineage tracing over weeks or months (e.g. via genetic marks),
  beyond the ~1-5 cell division scope of expression-memory approaches
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GEMLI
  role: R package whose GEMLI_items structure ('gene_expression' and 'barcodes' slots)
    holds the paired expression and lineage data used for the related-vs-random correlation
    distance comparison
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Analysis language for loading the normalized matrix, enumerating barcode-sharing
    cell pairs, computing Pearson correlation distances, and running the Mann-Whitney
    U test
  repo: https://github.com/UPSUTER/GEMLI
- name: devtools
  role: Used to install the GEMLI R package from GitHub before running the pairing
    analysis
  repo: https://github.com/UPSUTER/GEMLI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:b3cd24879a47972d979d09655e1b866488eeaf0ac30433856655d9010050771c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# barcoded-lineage-cell-pairing

## Summary

Groups single cells by shared barcode lineage identifiers and quantifies whole-transcriptome similarity of lineage-related versus random cell pairs, testing the gene-expression-memory premise underlying GEMLI.

## When to use

Use when scRNA-seq data comes with barcode lineage annotations (e.g. GEO GSE74923 L1210 or CD8+ T-cell datasets) and you need to verify that related cells are more transcriptomically similar than random pairs.

## When NOT to use

- The scRNA-seq dataset lacks barcode or other ground-truth lineage annotations, so related cell pairs cannot be enumerated
- The cell population contains only distantly related cells, where gene-expression-memory signals are weak
- The goal is long-term lineage tracing over weeks or months (e.g. via genetic marks), beyond the ~1-5 cell division scope of expression-memory approaches

## Inputs

- Normalized, quality-controlled gene expression matrix (cells x genes)
- Barcode lineage annotations for L1210/CD8 dataset (GEO GSE74923)
- Named vector with cell IDs as names and barcode IDs as values defining related-cell groups

## Outputs

- correlation_distance_comparison.csv with mean/median correlation distances and Mann-Whitney U p-value
- correlation_distance_comparison.png with overlaid related-pair vs random-pair distance distributions

## How to apply

Load the normalized, quality-controlled expression matrix and barcode lineage annotations in R and assemble them into a GEMLI_items list with 'gene_expression' and 'barcodes' slots. Build a named vector mapping cell IDs to barcode IDs to enumerate all within-lineage (related) cell pairs, then compute Pearson correlation distances (1 - Pearson r) across the whole transcriptome for each related pair. Randomly sample an equal number of unrelated pairs (cells with different barcodes) as the negative control and compute the same distances. Compare the two distributions with a non-parametric Mann-Whitney U (rank-sum) test; the rationale is that gene-expression memory should make lineage-related cells correlate more strongly, which is the basis GEMLI exploits for lineage prediction.

## Related tools

- **GEMLI** (R package whose GEMLI_items structure ('gene_expression' and 'barcodes' slots) holds the paired expression and lineage data used for the related-vs-random correlation distance comparison) — https://github.com/UPSUTER/GEMLI
- **R** (Analysis language for loading the normalized matrix, enumerating barcode-sharing cell pairs, computing Pearson correlation distances, and running the Mann-Whitney U test) — https://github.com/UPSUTER/GEMLI
- **devtools** (Used to install the GEMLI R package from GitHub before running the pairing analysis) — https://github.com/UPSUTER/GEMLI

## Examples

```
library(devtools); install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"); library(GEMLI)
```

## Evaluation signals

- Lineage-related cells show significantly lower correlation distance than random pairs (Mann-Whitney U p < 0.05)
- Mean/median Pearson correlation distance of related pairs is lower than that of the random-pair control in correlation_distance_comparison.csv
- The number of randomly sampled unrelated pairs equals the number of within-lineage related pairs
- Every cell in a pair shares the same barcode ID for the related set, and different barcode IDs for the control set
- GEMLI_items list contains populated 'gene_expression' and 'barcodes' slots before pair enumeration

## Limitations

- GEMLI will generally perform less robustly in cell populations that contain only distantly related cells
- Large variability in growth rates between lineages may decrease GEMLI's performance (cannot be excluded)
- Expression-memory-based lineage inference is limited to lineages over about 1-5 cell divisions; long-term tracing requires other approaches

## Evidence

- [abstract] Here we introduce Gene Expression Memory-based Lineage Inference (GEMLI), a computational tool allowing to robustly identify small to medium-sized cell lineages solely from scRNA-seq datasets.: "Here we introduce Gene Expression Memory-based Lineage Inference (GEMLI), a computational tool allowing to robustly identify small to medium-sized cell lineages solely from scRNA-seq datasets."
- [methods] install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"): "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`"
- [readme] GEMLI is an R package to predict cell lineages (cells with a common ancestor) from single cell RNA sequencing datasets: "GEMLI is an R package to predict cell lineages (cells with a common ancestor) from single cell RNA sequencing datasets and to call genes with a high gene expression memory on the predicted cell"
- [discussion] However, GEMLI will generally perform less robustly in cell populations that contain only distantly related cells.: "However, GEMLI will generally perform less robustly in cell populations that contain only distantly related cells."
