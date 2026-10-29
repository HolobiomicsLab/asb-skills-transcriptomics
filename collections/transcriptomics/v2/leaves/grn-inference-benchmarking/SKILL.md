---
name: grn-inference-benchmarking
description: Use when you need to decide how many cells and how many candidate TFs
  per target are sufficient for reliable scGATE logic-gate inference, or when reproducing
  runtime/saturation curves like Figure 5C before committing to a full-network run.
when_to_use_negative:
- 'When the goal is large-scale de novo GRN inference: scGATE is applicable only to
  a small (<200) subset of genes with a refined candidate TF list based on external
  hints'
- When candidate TF sets exceed k=5 regulators per target, since overfitting makes
  high-order Boolean logic (>5 TFs) unreliable
- 'When the scRNA-seq data have far fewer cells than the log10(n) ≈ #TFs heuristic
  requires for the chosen candidate TF count (inference quality will not have saturated)'
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scGATE
  role: 'Core inference engine: scGATE_logic() computes per-gene logic gates, -log10
    L0/-log10 L1, and log10 Bayes Factor across subsampled data configurations'
  repo: https://github.com/CompBioIPM/scGATE
- name: R
  role: Implementation environment (R 4.1.3) hosting the sensitivity loop, subsampling,
    and aggregation scripts
  repo: https://github.com/CompBioIPM/scGATE
- name: VGAM
  role: Dependency supporting the zero-inflated distribution fitting used in scGATE's
    likelihood computation
- name: truncnorm
  role: Dependency for truncated-normal fits in scGATE's Hill-function parameter space
- name: doParallel
  role: Parallel backend (with foreach/doSNOW) used to time wall-clock runtime across
    benchmark configurations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:7409a09e082ebf2d4d6832f27ad0d8996dec3fd8a156a51c7624a0aa79b20b80
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
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# grn-inference-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A sensitivity-analysis skill for scGATE-based gene regulatory network inference that quantifies how inference quality (log10 Bayes Factor, AUROC, EPR) and runtime scale with the number of sampled cells and the number of candidate regulatory TFs per target, establishing sample-size adequacy via the log10(n) ≈ #TFs heuristic.

## When to use

Use when you need to decide how many cells and how many candidate TFs per target are sufficient for reliable scGATE logic-gate inference, or when reproducing runtime/saturation curves like Figure 5C before committing to a full-network run.

## When NOT to use

- When the goal is large-scale de novo GRN inference: scGATE is applicable only to a small (<200) subset of genes with a refined candidate TF list based on external hints
- When candidate TF sets exceed k=5 regulators per target, since overfitting makes high-order Boolean logic (>5 TFs) unreliable
- When the scRNA-seq data have far fewer cells than the log10(n) ≈ #TFs heuristic requires for the chosen candidate TF count (inference quality will not have saturated)

## Inputs

- Cell-type-specific scRNA-seq count matrix (cells × genes CSV, e.g. subset_counts_cluster_11.csv from Zenodo 10.5281/zenodo.8353409)
- Base GRN TF-target dataframe derived from scATAC-seq and TF binding motifs (e.g. base_grn_mouse_blood_cell_differentiation_toggle_switch.csv / .parquet)
- Graded cell subsample sizes (e.g. 1000 cells up to the full cluster) and candidate TF counts per target (k ≤ 5)

## Outputs

- sensitivity_analysis.csv aggregating log10 BF and runtime per cell count and TF-set size
- Saturation curve of inference quality (AUROC, EPR, log10 BF) versus number of cells
- Runtime scaling plot versus cell count and candidate TF count (mirroring Figure 5C)

## How to apply

Download a cell-type-specific scRNA-seq count matrix (e.g. subset_counts_cluster_11.csv) and its base GRN from Zenodo (10.5281/zenodo.8353409) and preprocess with scRNA_seq_preprocessing(), setting library_size_normalization='False' for pre-normalized deposits and rescaling expression into the (0,1) interval. Generate graded cell subsamples (down to ~1000 cells) and, for each, vary the number of candidate TFs per target while keeping gate subsets at or below k=5, since scGATE recommends 'fitting logic gates among subsets with up to k=5 factors' to prevent overfitting. Run scGATE_logic() with number_of_em_iterations=10, top_gates=50, and run_mode toggled between 'simple' (large configurations) and 'complex' (precise hill-parameter search), logging per-gene -log10 L0, -log10 L1, log10 BF, and elapsed wall-clock time under doParallel/foreach/doSNOW workers. Aggregate log10 BF and runtime per cell count and TF-set size into a sensitivity table (sensitivity_analysis.csv), plot quality-versus-cells saturation and runtime scaling curves, and judge success when log10 BF plateaus near the reported ~3000-cell subsample and sampling adequacy satisfies log10(n) ≈ average TFs per target (e.g. 15 TFs are supported by ~1000 cells with AUROC > 0.9 and EPR > 0.6).

## Related tools

- **scGATE** (Core inference engine: scGATE_logic() computes per-gene logic gates, -log10 L0/-log10 L1, and log10 Bayes Factor across subsampled data configurations) — https://github.com/CompBioIPM/scGATE
- **R** (Implementation environment (R 4.1.3) hosting the sensitivity loop, subsampling, and aggregation scripts) — https://github.com/CompBioIPM/scGATE
- **VGAM** (Dependency supporting the zero-inflated distribution fitting used in scGATE's likelihood computation)
- **truncnorm** (Dependency for truncated-normal fits in scGATE's Hill-function parameter space)
- **doParallel** (Parallel backend (with foreach/doSNOW) used to time wall-clock runtime across benchmark configurations)

## Examples

```
gates <- scGATE_logic(data = subsample_counts, base_GRN = candidate_tf_target, number_of_em_iterations = 10, max_num_regulators = 5, top_gates = 50, run_mode = "simple")
```

## Evaluation signals

- log10 BF quality metric plateaus near the reported ~3000-cell subsample, indicating sample-size saturation
- Downsampling to ~1000 cells with 15 regulatory TFs still yields AUROC > 0.9 and EPR > 0.6
- Sampling adequacy satisfies the heuristic log10(n) ≈ average number of TFs per target
- Runtime curve shape matches Figure 5C, with 'simple' run_mode completing faster than 'complex' on large configurations
- sensitivity_analysis.csv contains non-missing log10 BF, -log10 L0, -log10 L1, and elapsed-time entries for every cell-count × TF-set-size configuration

## Limitations

- scGATE must be applied to a relatively small set of pre-selected TFs based on prior biological knowledge; it is not a from-scratch network discovery method
- Decoding high-order Boolean logic involving more than 5 transcriptional regulators remains challenging due to overfitting
- Runtime grows with configuration size, requiring the faster 'simple' run_mode for large configurations at the cost of hill-parameter search precision

## Evidence

- [results] With 15 regulatory TFs, downsampling scRNA-seq data to as few as 1000 cells still guarantees an AUROC > 0.9 and an EPR > 0.6: [evidence span withheld — anchor status: not_found]
- [methods] k=5 recommendation to prevent overfitting: [evidence span withheld — anchor status: not_found]
