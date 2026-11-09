---
name: bayes-factor-based-edge-ranking
description: Use when you have preprocessed scRNA-seq expression (counts rescaled
  into (0,1)) with a small refined candidate TF set and need a ranked, directed TF-gene
  edge list or top logic gates per target gene.
when_to_use_negative:
- When the target set is large-scale from scratch — scGATE is only applicable to a
  small (<200) subset of genes with a refined list of candidate TFs from external
  hints
- When the regulatory logic involves more than ~5 transcriptional regulators per target,
  where overfitting is a major concern
- On pseudotime-sorted single-cell data along differentiation trajectories or with
  temporal delays, which scGATE does not yet handle
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3391
tools:
- name: scGATE
  role: Provides scGATE_logic() and scGATE_edge() functions that output log10 Bayes
    Factor-ranked logic gates and TF-target edge lists
  repo: https://github.com/CompBioIPM/scGATE
- name: VGAM
  role: R dependency used for zero-inflated distribution fitting underlying the Bayes
    Factor computation
- name: truncnorm
  role: R dependency for the truncated-normal (Hill activation) expression model used
    in edge scoring
- name: BoolODE
  role: Generates synthetic toggle-switch scRNA-seq data with known ground-truth networks
    to validate BF-ranked edges via AUROC/EPR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:3e895b2875cb0ba911fe20adf91d97a840d4ac7d8d9d0c5f5fb9dbcde598d049
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
  - BoolODE
  - CellOracle
  - R 4.1.3
  - VGAM
  - arrow
  - scGATE
  - truncnorm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# bayes-factor-based-edge-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ranks inferred TF–target regulatory edges (and Boolean logic gates) by their Bayes Factor (BF), as implemented in scGATE's scGATE_logic()/scGATE_edge() functions, to prioritize the most probable regulatory rules from single-cell expression data.

## When to use

Use when you have preprocessed scRNA-seq expression (counts rescaled into (0,1)) with a small refined candidate TF set and need a ranked, directed TF-gene edge list or top logic gates per target gene.

## When NOT to use

- When the target set is large-scale from scratch — scGATE is only applicable to a small (<200) subset of genes with a refined list of candidate TFs from external hints
- When the regulatory logic involves more than ~5 transcriptional regulators per target, where overfitting is a major concern
- On pseudotime-sorted single-cell data along differentiation trajectories or with temporal delays, which scGATE does not yet handle

## Inputs

- Preprocessed scRNA-seq expression matrix (normalized counts within (0,1), cells as rows, genes as columns) from scRNA_seq_preprocessing()
- Optional base GRN dataframe from .parquet (e.g. Buenrostro2018_base_GRN_dataframe.parquet) processed by read_base_GRN()
- Ground-truth network (e.g. BoolODE toggle-switch ground truth) when used for benchmarking

## Outputs

- Dataframe of TF-target edges ranked by log10 Bayes Factor (from scGATE_edge())
- Dataframe of top Boolean logic gates per target gene ranked by Bayes Factor (from scGATE_logic())
- Benchmark scores (AUROC, AUPRC/EPR) per tool and condition when evaluated against ground truth

## How to apply

Preprocess the expression matrix with scRNA_seq_preprocessing() (library_size_normalization flag set appropriately; for BoolODE-simulated data use "False" for quantile rescaling into (0,1)) and drop genes with standard deviation <= 0.20. Optionally filter candidate edges with a base GRN from scATAC-seq/motif data via read_base_GRN() and/or an abs_cor Pearson-correlation cutoff. Run scGATE_logic() with run_mode="simple", top_gates=1, and max_num_regulators up to 3 (default) or 5 to avoid overfitting, so each target gene's logic combinations are scored by zero-inflated-distribution fits and emitted as log10 Bayes Factors; alternatively run scGATE_edge() with h_act=7 for BF-ranked edges without logic gates. Higher log10 BF edges are the more probable regulators, and the ranking can be scored against ground-truth networks (e.g. BoolODE toggle-switch) using AUROC and EPR.

## Related tools

- **scGATE** (Provides scGATE_logic() and scGATE_edge() functions that output log10 Bayes Factor-ranked logic gates and TF-target edge lists) — https://github.com/CompBioIPM/scGATE
- **VGAM** (R dependency used for zero-inflated distribution fitting underlying the Bayes Factor computation)
- **truncnorm** (R dependency for the truncated-normal (Hill activation) expression model used in edge scoring)
- **BoolODE** (Generates synthetic toggle-switch scRNA-seq data with known ground-truth networks to validate BF-ranked edges via AUROC/EPR)

## Examples

```
gates <- scGATE_logic(data = data, top_gates = 1, run_mode = "simple"); edges <- scGATE_edge(data = data, base_GRN = candidate_tf_target, h_act = 7)
```

## Evaluation signals

- Against BoolODE toggle-switch ground truth, BF-ranked scGATE edges should reach AUROC > 0.98 across cell types and dropout levels, exceeding baselines (GRNBOOST2, LEAP, CellOracle)
- scGATE_logic() output with top_gates=1 should contain exactly one Bayes-Factor-ranked logic combination per target gene
- log10 BF values should be ordered so that higher BF corresponds to the inferred gate/partition, reproducible across network-complexity and decoy-TF variants
- Edge list should be reduced by the abs_cor Pearson-correlation filter — edges with low |Pearson r| between TF and target must be absent
- Output of head(gates) must be a well-formed dataframe with columns for TFs, target gene, logic rule, and Bayes Factor

## Limitations

- Decoding high-order Boolean logic involving more than 5 transcriptional regulators remains challenging due to overfitting
- scGATE must be applied to a relatively small set of pre-selected TFs based on prior biological knowledge; it does not infer networks from scratch
- Large protein complex interactions are missed both in the model and in the ground-truth networks
- Directed BF-ranked edges do not by themselves convey combinatorial relationships unless logic gates are also inferred

## Evidence

- [methods] Zero-inflated fit for BF: [evidence span withheld — anchor status: not_found]
- [methods] Overfitting limit on k: [evidence span withheld — anchor status: not_found]
