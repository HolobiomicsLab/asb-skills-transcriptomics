---
name: fdr-thresholded-droplet-calling
description: Use when you need a model-based baseline droplet filter for raw 10X (CellRanger
  raw_feature_bc_matrix) snRNA-seq or scRNA-seq count matrices, or when comparing
  EmptyDrops calls to DIEM debris-score or quantile filtering on the same droplets.
when_to_use_negative:
- The goal is snRNA-seq-specific debris removal with an explicit contamination quantification
  — DIEM's debris-score filtering (call_targets with thresh_score = 0.5) was shown
  to better remove droplets with high levels of extranuclear RNA
- You would compare droplets across independent samples using a normalized score —
  EmptyDrops q-values are per-experiment, and the article warns debris-like scores
  cannot be assumed to share a scale across experiments
- The background distribution has little or no increased genes relative to cells,
  undermining profile-based filtering assumptions
edam_operation: http://edamontology.org/operation_3220
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: EmptyDrops
  role: Estimates the Dirichlet-Multinomial ambient RNA profile and tests each droplet
    against it; droplets with FDR q-value > 0.05 are removed
- name: R
  role: Implementation environment for the EmptyDrops baseline and droplet classification
    pipeline
- name: Matrix
  role: Stores the raw counts as a column-wise sparse matrix before testing
- name: CellRanger
  role: Source of the raw_feature_bc_matrix droplet count matrix used as input
- name: DIEM
  role: Comparison method whose debris-score filtering (call_targets, thresh_score
    = 0.5) is benchmarked against the EmptyDrops FDR calls
  repo: https://github.com/marcalva/diem
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:ffd6fc264145c7caeb78f9bdc18899313c606b9c55791aafb424b18584155083
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - CellRanger
  - DIEM
  - EmptyDrops
  - Matrix
  - R
  edam_operation: http://edamontology.org/operation_3220
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# fdr-thresholded-droplet-calling

## Summary

Benchmark baseline droplet-filtering skill that runs EmptyDrops on a raw 10X droplet count matrix (column-wise sparse Matrix) to estimate the ambient RNA Dirichlet-Multinomial profile, then removes droplets with EmptyDrops FDR q-value > 0.05. It serves as the statistical-filtering comparison point against DIEM debris-score filtering and hard-count cutoffs in snRNA-seq.

## When to use

Use when you need a model-based baseline droplet filter for raw 10X (CellRanger raw_feature_bc_matrix) snRNA-seq or scRNA-seq count matrices, or when comparing EmptyDrops calls to DIEM debris-score or quantile filtering on the same droplets.

## When NOT to use

- The goal is snRNA-seq-specific debris removal with an explicit contamination quantification — DIEM's debris-score filtering (call_targets with thresh_score = 0.5) was shown to better remove droplets with high levels of extranuclear RNA
- You would compare droplets across independent samples using a normalized score — EmptyDrops q-values are per-experiment, and the article warns debris-like scores cannot be assumed to share a scale across experiments
- The background distribution has little or no increased genes relative to cells, undermining profile-based filtering assumptions

## Inputs

- Raw 10X droplet count matrix from CellRanger raw_feature_bc_matrix output (e.g., mouse brain nuclei_2k), read into R as a column-wise sparse Matrix
- Per-droplet percent of spliced reads for downstream nuclear/background classification

## Outputs

- Per-droplet EmptyDrops test statistics and FDR q-values
- Binary kept/removed droplet calls at FDR q-value > 0.05 cutoff
- Droplet classification table (e.g., emptydrops_droplet_calls.csv) with calls and counts of nuclear vs. background droplets

## How to apply

Load the raw 10X droplet count matrix (CellRanger raw_feature_bc_matrix output) into R as a column-wise sparse Matrix using the Matrix package. Run EmptyDrops with default parameters on the full set of raw droplets so it estimates the ambient RNA Dirichlet-Multinomial profile from likely-empty droplets and tests every droplet against it. Retain droplets with FDR q-value <= 0.05 as candidate nuclei and remove the rest. For validation against published benchmarks (e.g., the 10X nuclei_2k mouse brain data), classify kept and removed droplets as nuclear or background using percent of spliced reads per droplet, then compare to the expected numbers (1,868/2,080 kept = 89.8% nuclear; 65/101 removed = 64.4% background). Note that EmptyDrops was originally developed and tested on single-cell data, so its model assumptions differ from DIEM's debris-score model.

## Related tools

- **EmptyDrops** (Estimates the Dirichlet-Multinomial ambient RNA profile and tests each droplet against it; droplets with FDR q-value > 0.05 are removed)
- **R** (Implementation environment for the EmptyDrops baseline and droplet classification pipeline)
- **Matrix** (Stores the raw counts as a column-wise sparse matrix before testing)
- **CellRanger** (Source of the raw_feature_bc_matrix droplet count matrix used as input)
- **DIEM** (Comparison method whose debris-score filtering (call_targets, thresh_score = 0.5) is benchmarked against the EmptyDrops FDR calls) — https://github.com/marcalva/diem

## Examples

```
library(Matrix); counts <- read_10x("path/to/10x/raw_feature_bc_matrix"); ed <- emptyDrops(counts); kept <- rownames(ed)[which(ed$FDR <= 0.05)]
```

## Evaluation signals

- On the 10X nuclei_2k mouse brain data, the calls CSV must report 1,868 of 2,080 kept droplets classified as nuclear (89.8%)
- The same CSV must report 65 of 101 removed droplets classified as background (64.4%), matching the published EmptyDrops benchmark
- All raw droplets from raw_feature_bc_matrix are tested — the count of tested droplets should match the raw matrix column count (2,080 + 101 = 2,181 for nuclei_2k)
- Removed set is exactly the droplets with EmptyDrops FDR q-value > 0.05; verify no kept droplet exceeds this threshold

## Limitations

- EmptyDrops was originally developed and tested on single-cell data, so its model assumptions differ from DIEM's; in the mouse brain snRNA-seq benchmark it left a larger background fraction among passing droplets than DIEM (89.8% vs. DIEM's higher purity)
- EmptyDrops underperformed relative to DIEM at removing droplets with high levels of extranuclear RNA in the article's comparisons
- A hard count threshold underperformed in comparison to EmptyDrops and DIEM, but EmptyDrops itself remains a baseline rather than the best snRNA-seq filter in this study

## Evidence

- [intro] EmptyDrops addresses this filtering issue for scRNA-seq by estimating a Dirichlet-Multinomial distribution of the ambient RNA.: "The recent method EmptyDrops12 addresses this filtering issue for scRNA-seq by estimating a Dirichlet-Multinomial distribution of the ambient RNA."
- [discussion] using a hard count threshold underperformed in comparison to EmptyDrops and DIEM: "using a hard count threshold underperformed in comparison to EmptyDrops and DIEM. Both DIEM and EmptyDrops12 remove droplets based on their expression distribution."
- [discussion] EmptyDrops, however, was originally developed and tested on single cell data and thus, the assumptions behind the model are different than that of DIEM.: "EmptyDrops, however, was originally developed and tested on single cell data and thus, the assumptions behind the model are different than that of DIEM."
- [methods] The raw counts are stored in a column-wise sparse matrix using the implementation from the R package [Matrix]: "The raw counts are stored in a column-wise sparse matrix using the implementation from the R package [Matrix]"
- [methods] This tutorial will use the publicly available 10X single-nucleus RNA-seq data that was generated from fresh mouse brain tissue.: "This tutorial will use the publicly available 10X single-nucleus RNA-seq data that was generated from fresh mouse brain tissue."
