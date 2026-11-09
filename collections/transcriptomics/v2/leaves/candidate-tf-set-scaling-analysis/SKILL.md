---
name: candidate-tf-set-scaling-analysis
description: 'Use when planning or auditing scGATE runs on scRNA-seq data: you need
  to know how many cells and how many candidate regulatory TFs per target are required
  for reliable (AUROC/EPR-validated) logic-gate inference before committing to a full-scale
  run.'
when_to_use_negative:
- The target gene set is large (>200 genes) or lacks a refined candidate TF list —
  scGATE is stated to apply only to a small (<200) subset of genes with refined candidate
  regulatory TFs from external hints
- You need to decode logic gates involving more than 5 TFs per target — overfitting
  makes k > 5 unreliable and outside the paper's recommended scope
- The data are pseudotime-sorted cells along a differentiation trajectory with temporal
  delays — scGATE does not yet handle such temporal ordering
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scGATE
  role: R package whose scGATE_logic() function is downsampling-benchmarked for quality
    (log10 BF) and runtime scaling
  repo: github.com/CompBioIPM/scGATE
- name: VGAM
  role: R dependency used by scGATE for fitting zero-inflated distributions during
    model fitting
- name: doParallel
  role: Parallel backend for timing wall-clock runtime of scGATE_logic() across subsample
    configurations
- name: arrow
  role: Reads .parquet base-GRN dataframes (e.g. Buenrostro2018_base_GRN_dataframe.parquet)
    as input
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:a983fd3dac9626b3a7d5fa02b5e588a970631f661ac55da69e0cf60776903d05
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CompBioIPM/scGATE
  tools:
  - R
  - VGAM
  - arrow
  - doParallel
  - doSNOW
  - foreach
  - scGATE
  - truncnorm
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Candidate-TF-set scaling analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A sensitivity-analysis skill for scGATE-based Boolean logic-gate inference that grades cell-count subsamples and candidate-TF set sizes to locate the sample-size saturation point and runtime scaling of network inference quality (log10 Bayes Factor). It operationalizes the paper's sampling-adequacy heuristic log10(n) ≈ average number of TFs per target.

## When to use

Use when planning or auditing scGATE runs on scRNA-seq data: you need to know how many cells and how many candidate regulatory TFs per target are required for reliable (AUROC/EPR-validated) logic-gate inference before committing to a full-scale run.

## When NOT to use

- The target gene set is large (>200 genes) or lacks a refined candidate TF list — scGATE is stated to apply only to a small (<200) subset of genes with refined candidate regulatory TFs from external hints
- You need to decode logic gates involving more than 5 TFs per target — overfitting makes k > 5 unreliable and outside the paper's recommended scope
- The data are pseudotime-sorted cells along a differentiation trajectory with temporal delays — scGATE does not yet handle such temporal ordering

## Inputs

- scRNA-seq cell-type-specific count matrix (e.g. subset_counts_cluster_11.csv from Zenodo 10.5281/zenodo.8353409)
- Base GRN candidate TF-target dataframe (base_grn_mouse_blood_cell_differentiation_toggle_switch.csv, or .parquet read via read_parquet)
- Graded cell subsample sizes and candidate-TF-per-target set sizes (k <= 5)

## Outputs

- sensitivity_analysis.csv — table of log10 BF, -log10 L0, -log10 L1, and elapsed runtime per cell count and TF-set size
- Saturation curve of inference quality (log10 BF / AUROC / EPR) versus number of cells (Figure 5C analogue)
- Runtime scaling plot versus cell count and candidate TF count
- Sampling-adequacy verdict via the log10(n) ≈ average TFs-per-target heuristic

## How to apply

Download the mouse haematopoiesis count matrix (e.g. subset_counts_cluster_11.csv) and its base GRN (base_grn_mouse_blood_cell_differentiation_toggle_switch.csv) from Zenodo (10.5281/zenodo.8353409), then preprocess with scRNA_seq_preprocessing(), keeping library size normalization off for pre-normalized deposits and rescaling expression into the (0,1) interval. Generate graded cell subsamples (small subsets up to the full cluster) and, per subsample, vary the number of candidate TFs per target while keeping subsets at or below k=5, the paper's recommended cap to prevent overfitting. For each configuration run scGATE_logic() with number_of_em_iterations = 10, top_gates = 50, and run_mode toggled between 'simple' (large configurations) and 'complex' (precise hill-parameter search), logging per-gene -log10 L0, -log10 L1, and log10 BF plus elapsed wall-clock time using doParallel/foreach/doSNOW workers. Aggregate log10 BF and runtime per cell count and TF-set size into a sensitivity table, plot the inference-quality saturation curve and runtime versus cell count/candidate TF count (mirroring Figure 5C), and save the table as sensitivity_analysis.csv. The paper reports that with 15 regulatory TFs even ~1000 cells guarantees AUROC > 0.9 and EPR > 0.6, with quality plateauing near the ~3000-cell subsample.

## Related tools

- **scGATE** (R package whose scGATE_logic() function is downsampling-benchmarked for quality (log10 BF) and runtime scaling) — github.com/CompBioIPM/scGATE
- **VGAM** (R dependency used by scGATE for fitting zero-inflated distributions during model fitting)
- **doParallel** (Parallel backend for timing wall-clock runtime of scGATE_logic() across subsample configurations)
- **arrow** (Reads .parquet base-GRN dataframes (e.g. Buenrostro2018_base_GRN_dataframe.parquet) as input)

## Examples

```
gates <- scGATE_logic(data = normalized_counts, base_GRN = candidate_tf_target, number_of_em_iterations = 10, top_gates = 50, run_mode = 'simple')
```

## Evaluation signals

- log10 BF quality metric plateaus near the reported ~3000-cell subsample
- With 15 regulatory TFs, ~1000-cell subsamples still yield AUROC > 0.9 and EPR > 0.6
- Runtime curve matches Figure 5C (simple vs complex run_mode behaviour)
- Sampling adequacy satisfies the heuristic log10(n) ≈ average number of TFs per target
- All configurations keep candidate TF subsets at k <= 5 to avoid overfitting

## Limitations

- Decoding high-order Boolean logic involving more than 5 TFs per target remains challenging due to overfitting
- Runtime scales with configuration size, requiring run_mode = 'simple' for large configurations; 'complex' gives more precise hill-parameter search but takes longer
- Findings were established on the mouse haematopoiesis dataset (Figure 5C); transfer of the exact saturation point (~3000 cells) to other tissues/datasets is not guaranteed

## Evidence

- [results] Downsampling guarantees AUROC/EPR at 1000 cells: [evidence span withheld — anchor status: not_found]
- [methods] k=5 cap to prevent overfitting: [evidence span withheld — anchor status: not_found]
