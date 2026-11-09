---
name: droplet-filtering-benchmark-tabulation
description: Use when benchmarking droplet-filtering methods (DIEM, EmptyDrops, hard-count
  cutoffs) on a raw 10X droplet count matrix and you need to quantify what fraction
  of removed droplets are background vs. nuclei via percent spliced reads.
when_to_use_negative:
- Input consists of a cell-by-gene matrix that has already been filtered and normalized
  (e.g., an SCE/Seurat object of retained cells) rather than a raw droplet-level count
  matrix
- The goal is DIEM's expression-based debris scoring itself (estimate_dbr_score, call_targets)
  rather than tabulating a hard-count/quantile baseline for benchmarking
- Single-cell RNA-seq data like PBMCs, where the paper found the spliced-read background
  correlation was less successful and hard thresholds underperform differently
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: diem
  role: Provides read_10x, create_SCE, and get_gene_pct used to load the 10X matrix,
    build the SCE object, and compute percent spliced reads for nuclear/background
    classification
  repo: https://github.com/marcalva/diem
- name: R
  role: Analysis language in which the quantile threshold computation and tabulation
    are performed
- name: Matrix
  role: R package used to store the raw counts as a column-wise sparse matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:5c0dce0c7897b1f94d0b5f9cebd02105c2e74a98cbf02439a5f82c0f0eb6e823
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - Matrix
  - R
  - diem
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# droplet-filtering-benchmark-tabulation

## Summary

Reproduces and tabulates a quantile-based hard-count threshold baseline (99th percentile of top-3000 barcode counts divided by 10) for droplet filtering in snRNA-seq, then evaluates how many removed droplets are true background using percent spliced reads, enabling comparison against DIEM and EmptyDrops.

## When to use

Use when benchmarking droplet-filtering methods (DIEM, EmptyDrops, hard-count cutoffs) on a raw 10X droplet count matrix and you need to quantify what fraction of removed droplets are background vs. nuclei via percent spliced reads.

## When NOT to use

- Input consists of a cell-by-gene matrix that has already been filtered and normalized (e.g., an SCE/Seurat object of retained cells) rather than a raw droplet-level count matrix
- The goal is DIEM's expression-based debris scoring itself (estimate_dbr_score, call_targets) rather than tabulating a hard-count/quantile baseline for benchmarking
- Single-cell RNA-seq data like PBMCs, where the paper found the spliced-read background correlation was less successful and hard thresholds underperform differently

## Inputs

- 10X raw_feature_bc_matrix droplet count matrix from 10X single-nucleus RNA-seq mouse brain data (nuclei_2k), CellRanger output readable by diem::read_10x
- List of spliceosomal/nuclear-localized genes for computing percent spliced reads via get_gene_pct

## Outputs

- quantile_filter_summary.csv with per-condition counts and percentages of passing nuclear droplets and removed background droplets
- Per-droplet table of total counts, genes detected, percent spliced reads, passing/removed label, and background/nuclear classification

## How to apply

Load the 10X raw_feature_bc_matrix (e.g., the nuclei_2k fresh mouse brain dataset) into R as a column-wise sparse Matrix using diem's read_10x, then build the SCE object with create_SCE and extract per-droplet total counts, genes detected, and barcode-rank ordering. Compute the hard count threshold as the 99th percentile of the counts of the top 3,000 barcodes divided by 10, in the spirit of a knee-point cutoff on the barcode rank plot. Split droplets into passing (counts >= threshold, labeled nuclear) and removed (below threshold, labeled background) sets, and compute percent spliced reads per droplet with get_gene_pct to classify them. Tabulate per condition the number and percentage of passing nuclear droplets and removed background droplets (e.g., 26 of 98 removed droplets, 26.5%, were background for the quantile filter versus 68.4% for DIEM), writing the summary to quantile_filter_summary.csv.

## Related tools

- **diem** (Provides read_10x, create_SCE, and get_gene_pct used to load the 10X matrix, build the SCE object, and compute percent spliced reads for nuclear/background classification) — https://github.com/marcalva/diem
- **R** (Analysis language in which the quantile threshold computation and tabulation are performed)
- **Matrix** (R package used to store the raw counts as a column-wise sparse matrix)

## Examples

```
counts <- read_10x("path/to/10x"); sce <- create_SCE(counts); sce <- get_gene_pct(x = sce, genes = malat, name = "MALAT1")
```

## Evaluation signals

- Quantile rows of quantile_filter_summary.csv must match the reference values: 1,832 of 2,083 passing nuclear droplets and 26 of 98 removed background droplets (26.5%)
- Per-method background fractions should reproduce the reported pattern: DIEM 117/171 (68.4%), EmptyDrops 65/101 (64.4%), quantile filtering 26/98 (26.5%) removed droplets classified as background
- Threshold invariant: passing set contains droplets with counts at or above the 99th-percentile-of-top-3000 / 10 cutoff; removed set strictly below it
- Percent spliced reads computed for every droplet; passing droplets labeled nuclear and removed droplets labeled background with no missing classifications

## Limitations

- A hard count threshold (including this quantile cutoff) underperformed in comparison to EmptyDrops and DIEM because it can remove many nuclear droplets while retaining debris
- DIEM's debris score is not on a comparable scale across independent experiments, complicating cross-sample integration of filtering results
- Debris scoring relies on accurate estimation of debris-enriched genes and is inaccurate if few or no genes are increased in the background distribution

## Evidence

- [methods] We can generate this using the `get_gene_pct` function: "We can generate this using the `get_gene_pct` function"
- [methods] diem package lightweight solution: "The `diem` package provides a lightweight solution to clean up single-nucleus or single-cell RNA-seq data during the pre-processing step."
- [discussion] hard count threshold underperformed: "using a hard count threshold underperformed in comparison to EmptyDrops and DIEM."
- [intro] hard cutoff loss of nuclear droplets: "in snRNA-seq, using a hard cutoff to remove droplets can result in a substantial loss of nuclear droplets and inclusion of debris droplets"
- [readme] read_10x reads 10X into sparse matrix: [evidence span withheld — anchor status: external_doc]
