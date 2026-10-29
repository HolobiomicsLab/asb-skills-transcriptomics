---
name: resampling-based-statistical-significance-testing
description: Use when assignments from MaCroDNA (or a similar bipartite-matching integration
  method) must be validated on data lacking ground-truth DNA–RNA cell correspondence,
  especially single-cell data grouped into biopsies of varying heterogeneity.
when_to_use_negative:
- Ground-truth DNA–RNA cell correspondence is available (e.g., CRC scTrio-seq data)
  — evaluate assignment accuracy and clonal prevalence prediction directly instead
  of resampling-based reliability
- The goal is a per-assignment probabilistic confidence measure — MaCroDNA does not
  provide probabilistic confidence measures, so this skill cannot substitute for a
  probabilistic method such as clonealign
- Samples are highly homogeneous — the article warns matching is distinctively challenging
  in homogeneous samples, so stability results there reflect a biologically driven
  limitation rather than pipeline error
tools:
- name: MaCroDNA
  role: Performs the cell-to-cell assignment (maximum weighted bipartite matching
    of per-gene read counts, requiring gurobipy/Gurobi) whose output is re-run on
    random permutations and resampled cells to build null and stability distributions
  repo: github.com/NakhlehLab/MaCroDNA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:ed476c65a335b8180b85af459532ddda73788abb00519df915f022bb258ff7a0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MaCroDNA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# resampling-based-statistical-significance-testing

## Summary

A validation strategy for assessing reliability of MaCroDNA cell assignments on the Barrett's esophagus (BE) dataset, where no ground-truth correspondence between scRNA-seq and scDNA-seq cells exists. It combines random-assignment null tests with resampling-based stability/definiteness analysis per biopsy.

## When to use

Use when assignments from MaCroDNA (or a similar bipartite-matching integration method) must be validated on data lacking ground-truth DNA–RNA cell correspondence, especially single-cell data grouped into biopsies of varying heterogeneity.

## When NOT to use

- Ground-truth DNA–RNA cell correspondence is available (e.g., CRC scTrio-seq data) — evaluate assignment accuracy and clonal prevalence prediction directly instead of resampling-based reliability
- The goal is a per-assignment probabilistic confidence measure — MaCroDNA does not provide probabilistic confidence measures, so this skill cannot substitute for a probabilistic method such as clonealign
- Samples are highly homogeneous — the article warns matching is distinctively challenging in homogeneous samples, so stability results there reflect a biologically driven limitation rather than pipeline error

## Inputs

- BE scDNA-seq copy-number matrix (dna_df; genes as rows, cells as columns, pandas dataframe)
- BE scRNA-seq gene-expression matrix (rna_df; genes as rows, cells as columns, pandas dataframe)
- Per-biopsy sample grouping labels (homogeneous healthy/NDBE vs heterogeneous HGD/EAC)
- Resampling configuration (cell subsampling per biopsy, number of repeats)

## Outputs

- Per-biopsy statistical significance of MaCroDNA assignments against the random-assignment null distribution
- Per-assignment stability and definiteness metrics across resampled runs
- Combined results table saved as be_stability_results.csv

## How to apply

Load the BE scDNA-seq copy-number matrix and scRNA-seq expression matrix as dna_df and rna_df dataframes (genes as rows, cells as columns, processed with pandas). For each BE biopsy, run MaCroDNA cell-to-cell assignment via maximum weighted bipartite matching of per-gene read counts, recording the cell2cell_step output whose 'step' column indicates assignment confidence (smaller step = more confident). Build a null distribution by randomly permuting RNA-to-DNA assignments and compare the observed matching objective (sum of Pearson correlation coefficients) against this null to obtain statistical significance for each biopsy. Then subsample cells from each biopsy, repeat the MaCroDNA assignment across resampled runs, and aggregate stability and definiteness metrics per biopsy, grouping biopsies into homogeneous healthy/NDBE versus heterogeneous HGD/EAC. Save the combined table as be_stability_results.csv.

## Related tools

- **MaCroDNA** (Performs the cell-to-cell assignment (maximum weighted bipartite matching of per-gene read counts, requiring gurobipy/Gurobi) whose output is re-run on random permutations and resampled cells to build null and stability distributions) — github.com/NakhlehLab/MaCroDNA

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df); cell2cell, cell2cell_step = macrodna.cell2cell_assignment()
```

## Evaluation signals

- Observed assignment objective (sum of Pearson correlation coefficients) exceeds the random-permutation null at significance for each biopsy
- Stability/definiteness metrics in heterogeneous HGD/EAC biopsies are higher than in homogeneous healthy/NDBE biopsies, matching the paper's reported finding
- cell2cell_step output contains a 'step' column where smaller step numbers correspond to more confident assignments
- Aggregated results are organized per biopsy with healthy/NDBE vs HGD/EAC grouping and written to be_stability_results.csv

## Limitations

- There is no ground truth information about the correspondence between cells for the BE dataset, so reliability can only be assessed indirectly via resampling and randomization
- MaCroDNA does not provide probabilistic confidence measures for its assignments, unlike statistical methods such as clonealign; the 'step' column is the only built-in confidence proxy
- Matching cells is distinctively challenging in homogeneous samples — a biologically driven issue that can undermine stability/definiteness comparisons in healthy/NDBE biopsies
- Running MaCroDNA requires a Gurobi installation and academic (non-commercial) license for gurobipy

## Evidence

- [discussion] in the absence of ground truth information about the correspondence between cells, we performed comprehensive resampling and stability analyses: "To assess the performance of MaCroDNA on the BE data, in the absence of ground truth information about the correspondence between cells, we performed comprehensive resampling and stability analyses"
- [discussion] matching cells is distinctively challenging in homogeneous samples and highlights a biologically driven issue that may arise when utilizing MaCroDNA: "This suggests that matching cells is distinctively challenging in homogeneous samples and highlights a biologically driven issue that may arise when utilizing MaCroDNA in such cases"
- [readme] "step" indicates at which step this result is inferred. And the smaller the step number is, the more confident the result is.: ""step" indicates at which step this result is inferred. And the smaller the step number is, the more confident the result is."
