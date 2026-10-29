---
name: runtime-performance-curve-plotting
description: 'Decision support for choosing scRNA-seq sample size and candidate-TF
  budget before running scGATE_logic(): when you must verify inference quality saturates
  (log10 BF plateau) and how runtime scales with cell count and TFs per target.'
when_to_use_negative:
- The goal is genome-wide de novo network inference rather than a sensitivity check
  on a small (<200-gene) pre-selected target set with refined candidate TFs — scGATE
  is explicitly limited to that scope
- 'Decoding high-order Boolean logic with more than 5 TFs per target (k > 5), where
  overfitting is a major concern and the log10(n) ≈ #TFs sample-size rule is no longer
  adequate'
- Pseudotime-sorted single-cell data along differentiation trajectories, which scGATE
  does not yet handle
edam_operation: http://edamontology.org/operation_0337
tools:
- name: scGATE
  role: Infers Boolean logic gates and log10 Bayes Factor outputs per TF-subset configuration
    via scGATE_logic()
  repo: https://github.com/CompBioIPM/scGATE
- name: R
  role: Environment (version 4.1.3) hosting the scGATE package and sensitivity-analysis
    scripts
  repo: https://github.com/CompBioIPM/scGATE
- name: doParallel
  role: Parallel workers (with foreach/doSNOW) to time scGATE_logic() wall-clock runtime
    across configurations
- name: arrow
  role: Reads base GRN and expression .parquet inputs (read_parquet) for subsampled
    configurations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:36b620cd4677a869a1e37d15decac5893768b6b06aa529298504cdd78e3c9aff
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.85
n_claims: 20
---

# Runtime performance curve plotting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sensitivity-analysis skill for scGATE network inference that grades cell subsamples and candidate-TF-set sizes, logs per-gene -log10 L0, -log10 L1, and log10 Bayes Factor outputs plus wall-clock time, and plots AUROC/EPR saturation and runtime curves (mirroring Figure 5C) to establish minimum sample size via the log10(n) ≈ #TFs heuristic.

## When to use

Decision support for choosing scRNA-seq sample size and candidate-TF budget before running scGATE_logic(): when you must verify inference quality saturates (log10 BF plateau) and how runtime scales with cell count and TFs per target.

## When NOT to use

- The goal is genome-wide de novo network inference rather than a sensitivity check on a small (<200-gene) pre-selected target set with refined candidate TFs — scGATE is explicitly limited to that scope
- Decoding high-order Boolean logic with more than 5 TFs per target (k > 5), where overfitting is a major concern and the log10(n) ≈ #TFs sample-size rule is no longer adequate
- Pseudotime-sorted single-cell data along differentiation trajectories, which scGATE does not yet handle

## Inputs

- scRNA-seq count matrix (cells x genes, e.g. subset_counts_cluster_11.csv) or .parquet expression file
- Base GRN of candidate TF-target edges (e.g. base_grn_mouse_blood_cell_differentiation_toggle_switch.csv or *_base_GRN_dataframe.parquet)
- Graded cell-count subsample scheme and per-target candidate TF-set sizes

## Outputs

- Per-configuration table of -log10 L0, -log10 L1, log10 Bayes Factor and elapsed wall-clock time (sensitivity_analysis.csv)
- Saturation curve of inference quality (AUROC > 0.9, EPR > 0.6 thresholds) versus number of cells, mirroring Figure 5C
- Runtime curves versus cell count and number of candidate TFs per target

## How to apply

Download the mouse haematopoiesis scRNA-seq counts (e.g. subset_counts_cluster_11.csv) and its base GRN from Zenodo 10.5281/zenodo.8353409 and preprocess with scRNA_seq_preprocessing() (library_size_normalization off for pre-normalized deposits, rescaling into (0,1)). Generate graded cell subsamples (down to ~1000 cells) and vary the number of candidate TFs per target while keeping subsets at k<=5 to prevent overfitting, as scGATE recommends. For each configuration run scGATE_logic() with number_of_em_iterations = 10, top_gates = 50, switching run_mode between 'simple' for large configurations and 'complex' for a precise hill-parameter search, timing each run under doParallel/foreach/doSNOW workers. Aggregate log10 BF and runtime per cell count and TF-set size into a single table, then plot inference quality vs. number of cells (saturation) and runtime vs. cell count / candidate TF count as in Figure 5C.

## Related tools

- **scGATE** (Infers Boolean logic gates and log10 Bayes Factor outputs per TF-subset configuration via scGATE_logic()) — https://github.com/CompBioIPM/scGATE
- **R** (Environment (version 4.1.3) hosting the scGATE package and sensitivity-analysis scripts) — https://github.com/CompBioIPM/scGATE
- **doParallel** (Parallel workers (with foreach/doSNOW) to time scGATE_logic() wall-clock runtime across configurations)
- **arrow** (Reads base GRN and expression .parquet inputs (read_parquet) for subsampled configurations)

## Examples

```
gates <- scGATE_logic(data = data, base_GRN = NA, number_of_em_iterations = 10, max_num_regulators = 3, top_gates = 50, run_mode = "simple")
```

## Evaluation signals

- log10 BF quality metric plateaus near the reported ~3000-cell subsample
- Downsampled runs of ~1000 cells (15 regulatory TFs) retain AUROC > 0.9 and EPR > 0.6
- Runtime curve shape matches Figure 5C, with 'simple' mode faster than 'complex' mode on large configurations
- Sampling adequacy satisfies the heuristic log10(n) ≈ average number of TFs per target
- Candidate TF subsets per target are capped at k = 5 in every configuration tested

## Limitations

- Decoding high-order Boolean logic with >5 transcriptional regulators remains challenging due to overfitting
- scGATE is applicable only to a small (<200) subset of genes with a refined candidate TF list based on external hints, not large-scale inference from scratch
- Runtime of 'complex' run_mode scales poorly on large configurations, requiring the faster 'simple' mode
- Results rest on the mouse haematopoiesis benchmark; minimum-cell-number conclusions may not transfer to other datasets

## Evidence

- [results] With 15 regulatory TFs, downsampling scRNA-seq data to as few as 1000 cells — a size common in many scRNA-seq datasets — still guarantees an AUROC > 0.9 and an EPR > 0.6: [evidence span withheld — anchor status: not_found]
- [methods] we recommend fitting logic gates among subsets with up to $k=5$ factors of the candidate TF lists. This approach helps prevent overfitting: [evidence span withheld — anchor status: not_found]
