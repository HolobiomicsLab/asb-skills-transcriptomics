---
name: multi-omic-resampling-stability-analysis
description: Use when evaluating multi-omic single-cell integration output (e.g.,
  MaCroDNA assignments on scRNA-seq/scDNA-seq pairs) in the absence of ground truth,
  to quantify statistical significance and assignment stability via resampling.
when_to_use_negative:
- A partially known ground-truth correspondence between scRNA-seq and scDNA-seq cells
  exists (e.g., scTrio-seq CRC data) — direct assignment-accuracy evaluation against
  ground truth is the appropriate skill instead.
- You need probabilistic confidence measures per assignment — MaCroDNA does not provide
  them, unlike clonealign.
- The sample is highly homogeneous (e.g., some healthy biopsies) and you expect stable
  one-to-one cell matching — the paper notes matching is distinctively challenging
  in homogeneous samples.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3173
tools:
- name: MaCroDNA
  role: Core method whose cell2cell_assignment() output is re-run on resampled cells
    and compared against random-assignment nulls to compute stability and significance
  repo: github.com/NakhlehLab/MaCroDNA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:5189adb8d838f8ae312aea4abc6b9b28c6e9148c36b84029437cb7ae99480a47
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MaCroDNA
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# multi-omic-resampling-stability-analysis

## Summary

Assesses the reliability of MaCroDNA cell-to-cell assignments between scRNA-seq and scDNA-seq data when no ground-truth cell correspondence exists, by combining random-assignment null tests with cell subsampling to measure per-assignment stability and definiteness across resampled runs.

## When to use

Use when evaluating multi-omic single-cell integration output (e.g., MaCroDNA assignments on scRNA-seq/scDNA-seq pairs) in the absence of ground truth, to quantify statistical significance and assignment stability via resampling.

## When NOT to use

- A partially known ground-truth correspondence between scRNA-seq and scDNA-seq cells exists (e.g., scTrio-seq CRC data) — direct assignment-accuracy evaluation against ground truth is the appropriate skill instead.
- You need probabilistic confidence measures per assignment — MaCroDNA does not provide them, unlike clonealign.
- The sample is highly homogeneous (e.g., some healthy biopsies) and you expect stable one-to-one cell matching — the paper notes matching is distinctively challenging in homogeneous samples.

## Inputs

- scDNA-seq copy-number matrix (dna_df: pandas DataFrame, genes as rows, cells as columns) per BE biopsy
- scRNA-seq gene-expression matrix (rna_df: pandas DataFrame, genes as rows, cells as columns) per BE biopsy
- Biopsy group labels (homogeneous healthy/NDBE vs heterogeneous HGD/EAC)

## Outputs

- cell2cell and cell2cell_step dataframes per biopsy (MaCroDNA assignments with 'step' confidence values)
- Random-assignment null distribution and per-biopsy significance p-values
- Per-assignment stability and definiteness metrics across resampled runs
- Combined per-biopsy results table saved as be_stability_results.csv

## How to apply

Load each biopsy's scDNA-seq copy-number matrix and scRNA-seq expression matrix as dna_df and rna_df dataframes with genes as rows and cells as columns (pandas). Run MaCroDNA cell2cell_assignment per biopsy via maximum weighted bipartite matching of per-gene read counts, recording the cell2cell_step output whose 'step' column encodes confidence (smaller step = more confident). Build a null distribution by randomly permuting RNA-to-DNA assignments and compare the observed matching objective against this null to obtain per-biopsy significance. Then subsample cells from each biopsy and repeat the assignment across resampled runs to quantify per-assignment stability and definiteness. Aggregate stability and significance results per biopsy, grouping biopsies into homogeneous healthy/NDBE versus heterogeneous HGD/EAC, and save as be_stability_results.csv; validate that homogeneous biopsies show lower stability than heterogeneous ones, matching the paper's reported pattern.

## Related tools

- **MaCroDNA** (Core method whose cell2cell_assignment() output is re-run on resampled cells and compared against random-assignment nulls to compute stability and significance) — github.com/NakhlehLab/MaCroDNA

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df); cell2cell, cell2cell_step = macrodna.cell2cell_assignment()
```

## Evaluation signals

- Observed matching objective for each biopsy exceeds the random-assignment null distribution at statistical significance.
- cell2cell_step 'step' values are recorded for every RNA cell, with smaller steps indicating more confident assignments.
- Stability/definiteness metrics are proportional to biopsy heterogeneity: homogeneous healthy/NDBE biopsies show lower stability than heterogeneous HGD/EAC biopsies, matching the paper's reported finding.
- All per-biopsy results aggregate into be_stability_results.csv with both significance and stability columns populated.
- Repeated subsampling runs converge on consistent stability rankings across biopsies.

## Limitations

- No ground truth information exists about cell correspondence for the BE dataset, so results cannot be validated as accuracy — only significance and stability.
- MaCroDNA does not provide probabilistic confidence measures for the assignments, unlike statistical methods such as clonealign.
- Matching cells is distinctively challenging in homogeneous samples — a biologically driven issue when applying MaCroDNA to such cases.
- Realistic simulation-based evaluation is impractical because knowledge of the DNA-to-RNA process is too limited to produce realistic simulations.

## Evidence

- [discussion] Resampling and stability analyses in absence of ground truth: "To assess the performance of MaCroDNA on the BE data, in the absence of ground truth information about the correspondence between cells, we performed comprehensive resampling and stability analyses"
- [discussion] MaCroDNA does not provide probabilistic confidence: "unlike statistical methods such as clonealign, MaCroDNA does not provide probabilistic confidence measures for the assignments"
- [readme] cell2cell_assignment returns confidence via step column: ""step" indicates at which step this result is inferred. And the smaller the step number is, the more confident the result is."
- [readme] Input format: genes as rows, cells as columns: "In these two dataframes, the row ids should be the genes and the column ids should be the cells."
- [abstract] Maximum weighted bipartite matching of per-gene read counts: "we present MaCroDNA, a method that uses maximum weighted bipartite matching of per-gene read counts from single-cell DNA and RNA-seq data."
