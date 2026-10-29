---
name: posterior-assignment-of-regulatory-combinations
description: Use when you have a preprocessed (0,1)-scaled scRNA-seq matrix with a
  shortlist of candidate TFs per target gene and need to infer which Boolean logic
  combination (e.g., TF1 AND NOT TF2) regulates each target without per-gate likelihood
  enumeration.
when_to_use_negative:
- When candidate TF lists are large and unrefined — scGATE is applicable only to a
  small (<200) subset of genes with refined candidate regulatory TFs based on external
  hints, and >5 regulators risks overfitting.
- When the goal is a linear directed TF-target network without logic gates — use scGATE_edge()
  instead of the gate-inference mixture fit.
- When data are pseudotime-sorted single-cell profiles along differentiation trajectories
  — scGATE does not yet handle temporal ordering or delays.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scGATE
  role: R package implementing the Bayesian mixture-model logic-gate inference (scGATE_logic
    / scGATE_gate); installed as scGATE_0.1.0.tar.gz
  repo: https://github.com/CompBioIPM/scGATE
- name: VGAM
  role: Fits the zero-inflated distribution (point mass at zero for dropouts plus
    normal densities per logic combination) used in the mixture likelihood
- name: truncnorm
  role: Provides truncated normal density evaluation for the continuous component
    of the zero-inflated mixture
- name: R
  role: Runtime environment; scGATE code written in R version 4.1.3, tested on Windows
    and Linux
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:79e94639a179b047416da280e081388886c9c18ba06f3066f80db4b0945480a8
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
  - scGATE
  - truncnorm
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# posterior-assignment-of-regulatory-combinations

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A Bayesian mixture-model step in scGATE that fits a zero-inflated density over enumerated Boolean TF combinations (AND/OR/XOR products of Hill-transformed expressions), estimates prior weights w_v with EM, and assigns each cell to its maximum-posterior combination — inferring the most probable logic gate a posteriori instead of computing a separate likelihood per rule.

## When to use

Use when you have a preprocessed (0,1)-scaled scRNA-seq matrix with a shortlist of candidate TFs per target gene and need to infer which Boolean logic combination (e.g., TF1 AND NOT TF2) regulates each target without per-gate likelihood enumeration.

## When NOT to use

- When candidate TF lists are large and unrefined — scGATE is applicable only to a small (<200) subset of genes with refined candidate regulatory TFs based on external hints, and >5 regulators risks overfitting.
- When the goal is a linear directed TF-target network without logic gates — use scGATE_edge() instead of the gate-inference mixture fit.
- When data are pseudotime-sorted single-cell profiles along differentiation trajectories — scGATE does not yet handle temporal ordering or delays.

## Inputs

- scRNA-seq expression matrix with normalized counts in the (0,1) interval, cells in rows and genes in columns, produced by scRNA_seq_preprocessing()
- Optional base GRN dataframe (e.g., Buenrostro2018_base_GRN_dataframe.parquet) preprocessed with read_base_GRN() to restrict candidate TFs per target
- Run parameters: h_set for the Hill exponent search space, number_of_em_iterations (default 3), max_num_regulators (default 3), abs_cor, top_gates (default 1), run_mode ('simple'/'complex'), weight_threshold

## Outputs

- Per-target-gene top Boolean logic gates ranked by Bayes factor (columns: gene_name, -log10 L0, -log10 L1, log10 BF, logic_gate), e.g., gates_results.csv
- Posterior assignment of each cell to its maximum-posterior logic combination Pr(v*|H(t_s), theta_hat)
- Posterior weight per logic combination used to flag ON combinations (>5% of target-gene observations)

## How to apply

For each target gene, enumerate the 2^k logic combinations over its candidate TFs (k kept <= 3 by default via max_num_regulators; up to 5 to limit overfitting) and compute each combination's output as products of the Hill activation H(tf) = (k_sat^h + 1) * tf^h / (k_sat^h + tf^h) and its complement. Fit a zero-inflated distribution — a point mass at zero for dropouts plus a normal density centered on each combination's output with variance sigma^2 — using the VGAM and truncnorm R packages, and estimate prior weights w_v with the EM algorithm (number_of_em_iterations = 3 by default). Assign each observation to its maximum-posterior logic combination Pr(v*|H(t_s), theta_hat), marking a combination active (ON) only when it generates more than 5% of the target gene's observations (weight_threshold). Compute the Bayes factor BF = L1/L0, where L0 is the base likelihood under uniform priors w_v = 1/2^k, and report the top gates ranked by BF (top_gates default 1) with max_num_regulators <= 3 to avoid overfitting.

## Related tools

- **scGATE** (R package implementing the Bayesian mixture-model logic-gate inference (scGATE_logic / scGATE_gate); installed as scGATE_0.1.0.tar.gz) — https://github.com/CompBioIPM/scGATE
- **VGAM** (Fits the zero-inflated distribution (point mass at zero for dropouts plus normal densities per logic combination) used in the mixture likelihood)
- **truncnorm** (Provides truncated normal density evaluation for the continuous component of the zero-inflated mixture)
- **R** (Runtime environment; scGATE code written in R version 4.1.3, tested on Windows and Linux)

## Examples

```
gates <- scGATE_logic(data = data, base_GRN = NA, h_set = NA, number_of_em_iterations = 3, max_num_regulators = 3, abs_cor = 0, top_gates = 1, run_mode = "simple")
```

## Evaluation signals

- gates_results.csv contains a positive log10 BF and a non-empty logic_gate string for each reported target gene
- Output reproduces the qualitative pattern of the synthetic toggle-switch example (e.g., ~gF for gE, gE.~gE2 for gE1)
- Each ON logic combination accounts for more than 5% of the target gene's observations (weight_threshold)
- Fitted -log10 L1 (mixture likelihood with EM-estimated priors) improves on -log10 L0 (uniform priors w_v = 1/2^k), yielding BF = L1/L0 > 1
- max_num_regulators stays <= 3 (at most 5) per gate so overfitting is not a concern

## Limitations

- Decoding high-order Boolean logic involving more than 5 transcriptional regulators remains challenging due to overfitting.
- scGATE must currently be applied to a small (<200) subset of genes with a refined list of candidate TFs pre-selected from prior biological knowledge or external hints (scATAC-seq, motifs).
- Results depend on the Hill exponent search space (h_set) and run_mode; 'simple' mode is faster but less precise over the Hill parameter space.

## Evidence

- [methods] Zero-inflated mixture fitting per logic combination: [evidence span withheld — anchor status: not_found]
- [methods] Recommend <=5 TFs to prevent overfitting: [evidence span withheld — anchor status: not_found]
