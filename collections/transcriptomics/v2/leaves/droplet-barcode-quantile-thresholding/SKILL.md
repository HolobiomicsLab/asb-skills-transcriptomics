---
name: droplet-barcode-quantile-thresholding
description: Use when you need to quantify what fraction of droplets removed by a
  hard count cutoff (knee/quantile-style) are background versus nuclear droplets —
  e.g., benchmarking a knee-point cutoff against EmptyDrops or DIEM using percent
  spliced reads in a raw 10X snRNA-seq matrix.
when_to_use_negative:
- When the goal is expression-profile-based debris scoring rather than a hard count
  cutoff — the paper's DIEM/EmptyDrops comparisons showed a hard count threshold underperformed,
  so this skill is the baseline comparator, not the preferred filter
- When the input is already a filtered cell-by-gene matrix (only passing barcodes
  remain) — the method requires the full raw droplet distribution to compute the barcode-rank
  quantile threshold
- When analyzing scRNA-seq where the distinction between background and cell profiles
  is smaller (e.g., the PBMC data), since the paper's validation of these cutoffs
  is on snRNA-seq datasets
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: diem
  role: Provides read_10x to load the 10X raw matrix, create_SCE to build the per-droplet
    SCE object, and get_gene_pct to compute percent spliced reads used to classify
    removed droplets as background
  repo: https://github.com/marcalva/diem
- name: R
  role: Language in which the thresholding, splitting, and tabulation of droplets
    are implemented
- name: Matrix
  role: Stores the raw counts as a column-wise sparse matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:2c2fe3fc04ad84f79ab396c0b0983e5b841c4acf424a3f7407c7e301467f1afd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - Matrix
  - R
  - diem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# droplet-barcode-quantile-thresholding

## Summary

Reproduces the quantile-based hard-count threshold baseline for droplet filtering in snRNA-seq: a count cutoff set from the barcode-rank distribution (99th percentile of the top 3,000 barcodes divided by 10) is applied to raw droplets, and the removed set is assessed against percent spliced (nuclear-localized) reads to measure how much debris versus nuclei the cutoff discards.

## When to use

Use when you need to quantify what fraction of droplets removed by a hard count cutoff (knee/quantile-style) are background versus nuclear droplets — e.g., benchmarking a knee-point cutoff against EmptyDrops or DIEM using percent spliced reads in a raw 10X snRNA-seq matrix.

## When NOT to use

- When the goal is expression-profile-based debris scoring rather than a hard count cutoff — the paper's DIEM/EmptyDrops comparisons showed a hard count threshold underperformed, so this skill is the baseline comparator, not the preferred filter
- When the input is already a filtered cell-by-gene matrix (only passing barcodes remain) — the method requires the full raw droplet distribution to compute the barcode-rank quantile threshold
- When analyzing scRNA-seq where the distinction between background and cell profiles is smaller (e.g., the PBMC data), since the paper's validation of these cutoffs is on snRNA-seq datasets

## Inputs

- Raw 10X droplet count matrix (raw_feature_bc_matrix) from a droplet-based snRNA-seq experiment, readable by diem::read_10x into a column-wise sparse Matrix
- Threshold parameters: top-N barcodes (3,000), quantile (0.99), and divisor (10) for the hard count cutoff
- Gene annotations enabling calculation of percent spliced (nuclear-localized) reads per droplet

## Outputs

- Hard count threshold value (99th percentile of top 3,000 barcodes divided by 10)
- Partition of droplets into passing (>= threshold) and removed (< threshold) sets
- Per-droplet percent spliced reads (nuclear-localized read fraction)
- quantile_filter_summary.csv tabulating counts and percentages of passing nuclear and removed background droplets per quantile condition

## How to apply

Load the raw 10X droplet count matrix (e.g., raw_feature_bc_matrix from the fresh mouse brain tissue nuclei_2k dataset) into R as a column-wise sparse Matrix with diem::read_10x, then build the SCE object with create_SCE and extract per-droplet total counts, genes detected, and the barcode rank ordering. Compute the hard threshold as the 99th percentile of the total counts of the top 3,000 barcodes divided by 10 — the knee-point-style cutoff discussed in the paper. Split droplets into a passing set (counts >= threshold) and a removed set (counts < threshold). Compute percent of spliced (nuclear-localized) reads per droplet with get_gene_pct, treating removed droplets as background candidates, then tabulate per-condition counts and percentages (e.g., 1,832 of 2,083 passing nuclear; 26 of 98 removed background = 26.5%) and write quantile_filter_summary.csv.

## Related tools

- **diem** (Provides read_10x to load the 10X raw matrix, create_SCE to build the per-droplet SCE object, and get_gene_pct to compute percent spliced reads used to classify removed droplets as background) — https://github.com/marcalva/diem
- **R** (Language in which the thresholding, splitting, and tabulation of droplets are implemented)
- **Matrix** (Stores the raw counts as a column-wise sparse matrix)

## Examples

```
counts <- read_10x("path/to/10x"); sce <- create_SCE(counts); thresh <- quantile(head(sort(sce@droplet_data$total_counts, decreasing=TRUE), 3000), 0.99)/10; removed <- sce@droplet_data$total_counts < thresh
```

## Evaluation signals

- The computed threshold equals the 99th percentile of total counts among the top 3,000 barcodes divided by 10
- Removed-set counts match the reference values on the mouse brain nuclei_2k dataset: 26 of 98 removed droplets classified as background (26.5%), versus 117/171 (68.4%) for DIEM and 65/101 (64.4%) for EmptyDrops
- Passing-set validation matches the reference: 1,832 of 2,083 passing droplets labeled nuclear
- Per-droplet percent spliced reads lie in [0, 1] and are defined for all droplets in both passing and removed sets
- quantile_filter_summary.csv contains, per quantile condition, both counts and percentages for passing nuclear and removed background droplets

## Limitations

- The quantile hard-count cutoff removed mostly nuclear droplets (only 26.5% of removed droplets were background), indicating poor discrimination of debris from nuclei
- Hard count thresholding underperformed in comparison to EmptyDrops and DIEM, which remove droplets based on their expression distribution
- A hard cutoff in snRNA-seq can result in a substantial loss of nuclear droplets and inclusion of debris droplets, leading to spurious clustering and false positive cell types

## Evidence

- [intro] Hard cutoff rationale: "A common practice to distinguish cell/nuclei- vs. background-containing barcodes relies on removing droplets below a hard cutoff of the number of reads, unique molecular indexes (UMI), or genes"
- [discussion] Threshold baseline underperformance: "using a hard count threshold underperformed in comparison to EmptyDrops and DIEM. Both DIEM and EmptyDrops12 remove droplets based on their expression distribution."
- [methods] Data loading function: "We provide a convenience function `read_10x` that can read the 10X output into a sparse Matrix."
- [methods] Percent spliced computation: "We can generate this using the `get_gene_pct` function"
- [readme] SCE construction from counts: [evidence span withheld — anchor status: external_doc]
