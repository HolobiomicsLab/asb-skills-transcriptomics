---
name: single-cell-dna-rna-cell-assignment-stability-analysis
description: Use when scRNA-seq and scDNA-seq cells from the same samples have been
  assigned with MaCroDNA but no paired ground truth exists, and you need statistical
  significance and stability of those assignments.
when_to_use_negative:
- Ground-truth cell correspondence is available (e.g., scTrio-seq CRC data) — direct
  assignment-accuracy evaluation against truth is the appropriate approach instead
- The goal is cell-to-clone mapping rather than cell-to-cell assignment confidence
  — use MaCroDNA's cell2clone_assignment with a dna_label clone/cell table
- A probabilistic confidence measure per assignment is required — the article notes
  MaCroDNA does not provide probabilistic confidence measures; step numbers and resampling
  stability are the available proxies
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3676
tools:
- name: MaCroDNA
  role: Runs the cell-to-cell assignment by maximum weighted bipartite matching of
    per-gene read counts; its cell2cell_step output supplies assignment-confidence
    steps used for stability and definiteness analysis
  repo: github.com/NakhlehLab/MaCroDNA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:ebfaa959f36f0f11ef97b160a0590b3e56426c67e9d328db07424d5bf530bf9b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MaCroDNA
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3676
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# single-cell-dna-rna-cell-assignment-stability-analysis

## Summary

Assess reliability of MaCroDNA cell-to-cell assignments between scRNA-seq and scDNA-seq data when no ground-truth correspondence exists, via random-assignment null tests and resampling-based stability/definiteness analysis (e.g., per-biopsy on the Barrett's esophagus dataset).

## When to use

Use when scRNA-seq and scDNA-seq cells from the same samples have been assigned with MaCroDNA but no paired ground truth exists, and you need statistical significance and stability of those assignments.

## When NOT to use

- Ground-truth cell correspondence is available (e.g., scTrio-seq CRC data) — direct assignment-accuracy evaluation against truth is the appropriate approach instead
- The goal is cell-to-clone mapping rather than cell-to-cell assignment confidence — use MaCroDNA's cell2clone_assignment with a dna_label clone/cell table
- A probabilistic confidence measure per assignment is required — the article notes MaCroDNA does not provide probabilistic confidence measures; step numbers and resampling stability are the available proxies

## Inputs

- BE scDNA-seq copy-number matrix (dna_df: genes x cells, pandas DataFrame)
- BE scRNA-seq expression matrix (rna_df: genes x cells, pandas DataFrame)
- Per-biopsy grouping of samples by heterogeneity class (healthy/NDBE vs HGD/EAC)

## Outputs

- Per-biopsy significance results from random-assignment null tests
- Per-assignment stability and definiteness metrics across resampled runs
- Combined per-biopsy results table (be_stability_results.csv)

## How to apply

Load each sample's scDNA-seq copy-number matrix and scRNA-seq expression matrix as `dna_df` and `rna_df` (genes as rows, cells as columns) and run MaCroDNA cell-to-cell assignment via maximum weighted bipartite matching of per-gene read counts, retaining `cell2cell_step` where a smaller 'step' value means a more confident assignment. Build a null distribution by randomly permuting RNA-to-DNA assignments and compare the observed matching objective against this null to obtain per-sample statistical significance. Then resample (subsample) cells per sample and repeat the assignment across runs, aggregating stability and definiteness metrics. Group samples by biological heterogeneity (e.g., homogeneous healthy/NDBE vs heterogeneous HGD/EAC Barrett's esophagus biopsies) and save the combined table (e.g., be_stability_results.csv). Validation: homogeneous samples should show lower stability than heterogeneous ones, and observed assignment scores must exceed the random-assignment null.

## Related tools

- **MaCroDNA** (Runs the cell-to-cell assignment by maximum weighted bipartite matching of per-gene read counts; its cell2cell_step output supplies assignment-confidence steps used for stability and definiteness analysis) — github.com/NakhlehLab/MaCroDNA

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df); cell2cell, cell2cell_step = macrodna.cell2cell_assignment()
```

## Evaluation signals

- Observed matching objective for each biopsy exceeds the random-assignment (permuted) null at statistical significance
- Assignments in homogeneous healthy/NDBE biopsies show lower stability than those in heterogeneous HGD/EAC biopsies, matching the paper's reported finding
- cell2cell_step output has 'predict_cell' and 'step' columns, with smaller step values indicating more confident assignments
- Combined per-biopsy table saved as be_stability_results.csv contains both significance and stability/definiteness metrics for every biopsy

## Limitations

- No ground truth information about cell correspondence exists for the BE dataset, so reliability must be assessed indirectly via resampling and stability analyses
- MaCroDNA does not provide probabilistic confidence measures for the assignments, unlike statistical methods such as clonealign
- Matching cells is distinctively challenging in homogeneous samples — a biologically driven issue when using MaCroDNA in such cases

## Evidence

- [discussion] To assess the performance of MaCroDNA on the BE data, in the absence of ground truth information about the correspondence between cells, we performed comprehensive resampling and stability analyses: "To assess the performance of MaCroDNA on the BE data, in the absence of ground truth information about the correspondence between cells, we performed comprehensive resampling and stability analyses"
- [discussion] MaCroDNA does not provide probabilistic confidence measures for the assignments: "unlike statistical methods such as clonealign, MaCroDNA does not provide probabilistic confidence measures for the assignments"
- [readme] step indicates at which step this result is inferred. And the smaller the step number is, the more confident the result is: ""step" indicates at which step this result is inferred. And the smaller the step number is, the more confident the result is."
- [readme] For mapping RNA cells to DNA cells, the input should be two dataframes `dna_df` and `rna_df`: "For mapping RNA cells to DNA cells, the input should be two dataframes `dna_df` and `rna_df`."
- [discussion] matching cells is distinctively challenging in homogeneous samples and highlights a biologically driven issue: "This suggests that matching cells is distinctively challenging in homogeneous samples and highlights a biologically driven issue that may arise when utilizing MaCroDNA in such cases"
