---
name: scrnaseq-cell-subsampling-sensitivity-analysis
description: Use when you must determine the minimum cell count needed for reliable
  logic-gate/GRN inference from scRNA-seq, or characterize how inference quality and
  runtime saturate with sample size and candidate TF-set size.
when_to_use_negative:
- You need inference with more than 5 candidate regulators per target — scGATE warns
  that decoding high-order Boolean logic (>5 TFs) remains challenging due to overfitting
- Your goal is large-scale GRN inference from scratch across all genes — scGATE is
  applicable only to a small (<200) subset of genes with a refined candidate TF list
- The dataset is too small to reach the sample-adequacy heuristic (log10(n) ≈ average
  number of TFs per target), e.g. tens of cells with multiple regulators
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scGATE
  role: R package whose scGATE_logic() function performs the logic-gate inference
    whose sensitivity to cell count and TF-set size is being measured
  repo: https://github.com/CompBioIPM/scGATE
- name: R
  role: Runtime environment (R 4.1.3) in which the downsampling loop and scGATE calls
    are executed
- name: doParallel
  role: Parallel backend (with foreach/doSNOW) used to time wall-clock runtime of
    scGATE runs across subsample configurations
- name: arrow
  role: Reads .parquet base GRN files (e.g. Buenrostro2018_base_GRN_dataframe.parquet)
    used as candidate TF-target inputs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:783568bfec82156ccb6b3d5c53cd355782680108a27f0e98997d61e4df9a2bc3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CompBioIPM/scGATE
  repo_url: https://github.com/CompBioIPM/scGATE
  tools:
  - R
  - VGAM
  - arrow
  - doParallel
  - doSNOW
  - foreach
  - scGATE
  - truncnorm
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# scrnaseq-cell-subsampling-sensitivity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A downsampling sensitivity protocol for assessing how scGATE network inference quality (log10 Bayes Factor, AUROC, EPR) and runtime scale with the number of scRNA-seq cells and the number of candidate regulatory TFs per target, grounded in the log10(n) ≈ #TFs sample-adequacy heuristic.

## When to use

Use when you must determine the minimum cell count needed for reliable logic-gate/GRN inference from scRNA-seq, or characterize how inference quality and runtime saturate with sample size and candidate TF-set size.

## When NOT to use

- You need inference with more than 5 candidate regulators per target — scGATE warns that decoding high-order Boolean logic (>5 TFs) remains challenging due to overfitting
- Your goal is large-scale GRN inference from scratch across all genes — scGATE is applicable only to a small (<200) subset of genes with a refined candidate TF list
- The dataset is too small to reach the sample-adequacy heuristic (log10(n) ≈ average number of TFs per target), e.g. tens of cells with multiple regulators

## Inputs

- Cell-type-specific scRNA-seq count matrix (e.g. subset_counts_cluster_11.csv) from Zenodo 10.5281/zenodo.8353409
- Base GRN derived from external hints, e.g. base_grn_mouse_blood_cell_differentiation_toggle_switch.csv
- Graded cell subsample sizes (e.g. 1000 cells up to full cluster ~3000 cells)
- Candidate TF lists per target gene (subsets of size ≤ k=5)

## Outputs

- sensitivity_analysis.csv — aggregated log10 BF and runtime per cell count and TF-set size
- Saturation curve of inference quality (log10 BF / AUROC / EPR) versus number of cells (cf. Figure 5C)
- Runtime scaling plots versus cell count and candidate TF count

## How to apply

Download the mouse haematopoiesis cell-type-specific scRNA-seq count matrix (e.g. subset_counts_cluster_11.csv) and its base GRN (base_grn_mouse_blood_cell_differentiation_toggle_switch.csv) from Zenodo (10.5281/zenodo.8353409) and preprocess with scRNA_seq_preprocessing(), disabling library size normalization for pre-normalized deposits and rescaling into the (0,1) interval. Generate graded cell subsamples from small subsets up to the full cluster (~3000 cells), and for each subsample vary the number of candidate TFs per target while keeping subsets at or below k=5, since scGATE recommends fitting logic gates among subsets with up to k=5 factors to prevent overfitting. Run scGATE_logic() per configuration with number_of_em_iterations = 10, top_gates = 50, and run_mode toggled between 'simple' (large configurations, faster) and 'complex' (precise hill-parameter search), logging per-gene -log10 L0, -log10 L1, log10 BF, and elapsed wall-clock time using doParallel/foreach/doSNOW workers. Aggregate log10 BF and runtime per cell count and TF-set size into a single sensitivity table (sensitivity_analysis.csv) and plot quality-versus-cells saturation curves and runtime curves mirroring Figure 5C.

## Related tools

- **scGATE** (R package whose scGATE_logic() function performs the logic-gate inference whose sensitivity to cell count and TF-set size is being measured) — https://github.com/CompBioIPM/scGATE
- **R** (Runtime environment (R 4.1.3) in which the downsampling loop and scGATE calls are executed)
- **doParallel** (Parallel backend (with foreach/doSNOW) used to time wall-clock runtime of scGATE runs across subsample configurations)
- **arrow** (Reads .parquet base GRN files (e.g. Buenrostro2018_base_GRN_dataframe.parquet) used as candidate TF-target inputs)

## Examples

```
normalized_counts <- scRNA_seq_preprocessing(data = data_scRNA_seq, library_size_normalization = "False", tf_list = NA); gates <- scGATE_logic(data = normalized_counts, number_of_em_iterations = 10, max_num_regulators = 5, top_gates = 50, run_mode = "simple")
```

## Evaluation signals

- AUROC > 0.9 and EPR > 0.6 are still achieved when downsampling to as few as 1000 cells with 15 regulatory TFs
- The log10 BF quality metric plateaus near the reported ~3000-cell subsample
- Runtime-versus-cells curve matches the shape of Figure 5C
- Sampling adequacy satisfies the heuristic log10(n) ≈ average number of TFs per target
- All TF subsets kept at or below k=5 to prevent overfitting

## Limitations

- Fitting logic gates among subsets larger than k=5 candidate TFs risks overfitting and is not recommended
- scGATE is applicable only to a rather small (<200) subset of genes with a refined list of candidate regulatory TFs based on external hints
- scGATE must be applied to a small pre-selected set of TFs based on prior biological knowledge
- Decoding high-order Boolean logic involving more than 5 transcriptional regulators remains challenging due to overfitting

## Evidence

- [other] With 15 regulatory TFs, downsampling scRNA-seq data to as few as 1000 cells — a size common in many scRNA-seq datasets — still guarantees an AUROC > 0.9 and an EPR > 0.6: [evidence span withheld — anchor status: not_found]
- [methods] k=5 recommendation from methods: [evidence span withheld — anchor status: not_found]
