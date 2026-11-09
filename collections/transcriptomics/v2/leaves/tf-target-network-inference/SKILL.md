---
name: tf-target-network-inference
description: Use when you have a preprocessed scRNA-seq expression matrix plus a candidate
  list of TFs (optionally a base GRN .parquet from scATAC-seq/motif data) and need
  directed, cell-type-specific TF-target edges with Bayes-Factor-ranked Boolean logic
  gates.
when_to_use_negative:
- 'You need large-scale network inference from scratch: scGATE is applicable only
  to a small (<200) subset of genes with a refined list of candidate TFs based on
  external hints.'
- 'The regulatory logic involves more than 5 TFs per target: decoding high-order Boolean
  logic beyond k=5 regulators is prone to overfitting.'
- You need temporal/pseudotime-aware inference along differentiation trajectories
  — scGATE does not yet handle pseudotime-sorted single-cell data or temporal delays.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scGATE
  role: 'Primary inference tool: scRNA_seq_preprocessing(), read_base_GRN(), scGATE_edge(),
    and scGATE_logic() perform preprocessing, base-GRN summarization, and TF-target
    edge/logic-gate inference'
  repo: github.com/CompBioIPM/scGATE
- name: R 4.1.3
  role: Language environment in which scGATE is written and executed (tested on Windows
    and Linux)
- name: VGAM
  role: R dependency used by scGATE for zero-inflated distribution fitting of Hill-transformed
    expression profiles
- name: truncnorm
  role: R dependency for truncated-normal distributions in the scGATE statistical
    model
- name: arrow
  role: R dependency used to read base GRN .parquet files via read_parquet()
- name: CellOracle
  role: Baseline directed TF-gene network inference tool used for comparison benchmarking
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:0fb1b66a227b5daa20dea666186b8d67a7f4828a2db6aae0f50020b44450bc56
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CompBioIPM/scGATE
  tools:
  - CellOracle
  - R 4.1.3
  - VGAM
  - arrow
  - scGATE
  - truncnorm
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# tf-target-network-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infer directed TF-to-target-gene regulatory networks from single-cell RNA-seq data using scGATE's Hill-function and Bayesian logic-gate framework, optionally refined by a base GRN derived from scATAC-seq peaks and TF binding motifs.

## When to use

Use when you have a preprocessed scRNA-seq expression matrix plus a candidate list of TFs (optionally a base GRN .parquet from scATAC-seq/motif data) and need directed, cell-type-specific TF-target edges with Bayes-Factor-ranked Boolean logic gates.

## When NOT to use

- You need large-scale network inference from scratch: scGATE is applicable only to a small (<200) subset of genes with a refined list of candidate TFs based on external hints.
- The regulatory logic involves more than 5 TFs per target: decoding high-order Boolean logic beyond k=5 regulators is prone to overfitting.
- You need temporal/pseudotime-aware inference along differentiation trajectories — scGATE does not yet handle pseudotime-sorted single-cell data or temporal delays.

## Inputs

- Raw scRNA-seq count matrix (cells in rows, genes in columns; e.g. ExpressionData.csv from BoolODE simulation or Tabula Muris Spleen-10X_P4_7)
- Candidate TF-target base GRN dataframe in .parquet format derived from scATAC-seq peaks and TF DNA binding motifs (e.g. Buenrostro2018_base_GRN_dataframe.parquet)
- List of transcription factors to consider (optional; defaults to all columns)

## Outputs

- Directed TF-target edge list ranked by Bayes Factor (from scGATE_edge(), e.g. h_act=7)
- Boolean logic gates (AND/OR/XOR combinations) per target gene with log10 Bayes Factor scores (from scGATE_logic(), top_gates ranked)

## How to apply

Install the scGATE R package (scGATE_0.1.0.tar.gz, R 4.1.3) with VGAM, truncnorm, and arrow dependencies. Preprocess raw scRNA-seq counts with scRNA_seq_preprocessing(), rescaling expression into the (0,1) interval and dropping genes with low variability (e.g. SD <= 0.20 on the synthetic toggle-switch data); supply a base GRN read via read_base_GRN() from a .parquet file of scATAC-seq/motif-derived candidate TF-target pairs to filter non-relevant TFs. Run scGATE_edge() (e.g. h_act=7) to obtain a directed TF-target edge list without logic gates, or scGATE_logic() (run_mode='simple' or 'complex', top_gates=1, max_num_regulators<=5, abs_cor for Pearson-correlation edge filtration) to additionally assign each target the top Boolean logic gates ranked by log10 Bayes Factor. The Hill activation function models continuous (non-binarized) expression, and the Bayesian EM framework (default 3 iterations) updates priors over TF combinations so the Boolean rule is inferred a posteriori rather than per-gate likelihood fitting. Score inferred edges against ground truth (AUROC/AUPRC/EPR) when truth networks exist, as in the BoolODE-simulated toggle-switch benchmarks where scGATE exceeded GRNBOOST2, LEAP, and CellOracle.

## Related tools

- **scGATE** (Primary inference tool: scRNA_seq_preprocessing(), read_base_GRN(), scGATE_edge(), and scGATE_logic() perform preprocessing, base-GRN summarization, and TF-target edge/logic-gate inference) — github.com/CompBioIPM/scGATE
- **R 4.1.3** (Language environment in which scGATE is written and executed (tested on Windows and Linux))
- **VGAM** (R dependency used by scGATE for zero-inflated distribution fitting of Hill-transformed expression profiles)
- **truncnorm** (R dependency for truncated-normal distributions in the scGATE statistical model)
- **arrow** (R dependency used to read base GRN .parquet files via read_parquet())
- **CellOracle** (Baseline directed TF-gene network inference tool used for comparison benchmarking)

## Examples

```
gates <- scGATE_logic(data = data, base_GRN = NA, number_of_em_iterations = 3, max_num_regulators = 3, top_gates = 1, run_mode = "simple")
```

## Evaluation signals

- Edge and gate outputs scored against a known ground-truth network (e.g. BoolODE toggle-switch truth) should yield AUROC > 0.98 and exceed baselines (GRNBOOST2, LEAP, CellOracle) in AUROC and EPR across cell types and dropout levels.
- Input expression matrix to scGATE_edge()/scGATE_logic() must be normalized within the (0,1) interval after scRNA_seq_preprocessing() preprocessing.
- Genes with low variability (e.g. standard deviation <= 0.20 on simulated data) should be removed before inference.
- Logic-gate fits should be restricted to subsets of up to k=5 candidate TFs per target to prevent overfitting; max_num_regulators default is 3.
- For base-GRN-guided runs, the .parquet candidate TF-target dataframe must be successfully read via read_parquet() and summarized with read_base_GRN().

## Limitations

- scGATE requires a relatively small pre-selected set of TFs based on prior biological knowledge; it cannot infer networks from scratch.
- Applicable only to a small (<200 gene) subset of genes with refined candidate regulatory TFs from external hints.
- Decoding high-order Boolean logic involving >5 regulators remains challenging due to overfitting.
- Does not yet handle pseudotime-sorted single-cell data along differentiation trajectories or temporal delays.
- Large protein complex interactions are missed both in the current model and in the ground-truth networks.

## Evidence
