---
name: percent-spliced-read-quality-assessment
description: Use when evaluating snRNA-seq droplet filtering on fresh/frozen nucleus
  data — e.g., to check what fraction of droplets removed by a hard count or quantile
  cutoff are truly background, using percent spliced reads as a nuclear-content proxy.
when_to_use_negative:
- Input is single-cell RNA-seq rather than single-nucleus data — the percent spliced
  read signal is weaker because differences between background and cell type profiles
  are smaller (as seen for PBMC scRNA-seq in the paper)
- You need a filtering decision itself — this skill evaluates/labels droplets via
  percent spliced reads; use DIEM's debris score (estimate_dbr_score + call_targets)
  or EmptyDrops to actually filter
- Count matrix is already filtered to cell barcodes only — background droplet classification
  requires the raw droplet matrix
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: diem
  role: Provides read_10x to load the 10X matrix, create_SCE to build the SCE object,
    and get_gene_pct to compute percent spliced reads per droplet for nuclear vs.
    background labeling
  repo: https://github.com/marcalva/diem
- name: R
  role: Analysis environment in which the diem package is run and the summary table
    is written
- name: Matrix
  role: Stores raw counts as a column-wise sparse matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:b0048e589d1152350e0e45a689987db9ba0f4a1e4d9de66a400a66485f1344ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - Matrix
  - R
  - diem
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# percent-spliced-read-quality-assessment

## Summary

Computes the percent of spliced (nuclear-localized) reads per droplet with the diem package's get_gene_pct and uses this metric to classify droplets as nuclear vs. background, quantifying how well a count-threshold filter discriminates debris from nuclei in snRNA-seq data.

## When to use

Use when evaluating snRNA-seq droplet filtering on fresh/frozen nucleus data — e.g., to check what fraction of droplets removed by a hard count or quantile cutoff are truly background, using percent spliced reads as a nuclear-content proxy.

## When NOT to use

- Input is single-cell RNA-seq rather than single-nucleus data — the percent spliced read signal is weaker because differences between background and cell type profiles are smaller (as seen for PBMC scRNA-seq in the paper)
- You need a filtering decision itself — this skill evaluates/labels droplets via percent spliced reads; use DIEM's debris score (estimate_dbr_score + call_targets) or EmptyDrops to actually filter
- Count matrix is already filtered to cell barcodes only — background droplet classification requires the raw droplet matrix

## Inputs

- 10X raw_feature_bc_matrix (MTX/HDF5 droplet count matrix) from fresh mouse brain snRNA-seq (nuclei_2k)
- SCE object created by diem's create_SCE from a column-wise sparse Matrix
- Per-droplet total counts and barcode rank ordering
- Hard count or quantile threshold (e.g., 99th percentile of top 3,000 barcodes divided by 10)

## Outputs

- Per-droplet percent spliced (nuclear-localized) reads computed via get_gene_pct
- Passing vs. removed droplet sets under the threshold
- quantile_filter_summary.csv with number and percentage of passing nuclear droplets and removed background droplets per condition

## How to apply

Load the raw 10X droplet count matrix (e.g., raw_feature_bc_matrix from the 10X mouse brain nuclei_2k fresh tissue sample) into R as a column-wise sparse Matrix via diem's read_10x, then build the SCE object with create_SCE. Use get_gene_pct to compute the percent of spliced reads per droplet, identifying nuclear-localized transcripts (e.g., genes matching the intron-retaining/spliced set used in the paper). Split droplets into a passing set (counts at or above the threshold, e.g., the 99th percentile of top-3,000 barcodes divided by 10) and a removed set (below it), then tabulate per condition the number and percentage of passing nuclear droplets and removed background droplets. In the mouse brain reference run, the quantile cutoff removed 26 of 98 background droplets (26.5%) versus 68.4% for DIEM and 64.4% for EmptyDrops, showing the hard-count cutoff removed mostly nuclear droplets — this contrast is the metric of interest. Write the per-condition summary (counts and percentages) to quantile_filter_summary.csv for validation against reference values such as 1,832 of 2,083 passing nuclear droplets.

## Related tools

- **diem** (Provides read_10x to load the 10X matrix, create_SCE to build the SCE object, and get_gene_pct to compute percent spliced reads per droplet for nuclear vs. background labeling) — https://github.com/marcalva/diem
- **R** (Analysis environment in which the diem package is run and the summary table is written)
- **Matrix** (Stores raw counts as a column-wise sparse matrix)

## Examples

```
counts <- diem::read_10x('nuclei_2k/raw_feature_bc_matrix'); sce <- diem::create_SCE(counts); sce <- diem::get_gene_pct(x = sce, genes = spliced_genes, name = 'pct.spliced')
```

## Evaluation signals

- Quantile-filter rows in quantile_filter_summary.csv match reported reference values: 1,832 of 2,083 passing nuclear droplets and 26 of 98 (26.5%) removed background droplets in the mouse brain nuclei_2k run
- Removed background fractions fall within the reported ranges across methods: 117/171 (68.4%) for DIEM, 65/101 (64.4%) for EmptyDrops, 26/98 (26.5%) for quantile filtering
- Every droplet receives a defined percent-spliced value and a nuclear/background label with no missing barcodes after read_10x/create_SCE (check the v2.4.1 barcode-ID bug fix)
- Passing set is exactly the droplets with counts >= threshold; removed set is exactly counts < threshold, and set sizes sum to total droplets
- Percentages computed as removed-background / total-removed per condition reproduce the tabulated ratios

## Limitations

- The quantile-based hard-count cutoff removed mostly nuclear droplets (only 26.5% background), so it is a poor discriminator of debris versus nuclei in snRNA-seq
- Correlation of debris scoring with percent spliced reads was less successful in PBMC single-cell RNA-seq data, likely due to smaller differences between background and cell type profiles
- Percent spliced reads is a proxy for nuclear content; it supports evaluation but does not itself model ambient RNA like DIEM or EmptyDrops do

## Evidence

- [results] Quantile cutoff removed mostly nuclear droplets in mouse brain: [evidence span withheld — anchor status: not_found]
- [methods] get_gene_pct computes the metric: "We can generate this using the `get_gene_pct` function"
- [methods] read_10x loads 10X data: "We provide a convenience function `read_10x` that can read the 10X output into a sparse Matrix."
- [readme] README workflow showing get_gene_pct usage on SCE object: [evidence span withheld — anchor status: external_doc]
- [discussion] Hard cutoff underperforms vs expression-based methods: "using a hard count threshold underperformed in comparison to EmptyDrops and DIEM."
