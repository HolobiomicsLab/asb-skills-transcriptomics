---
name: em-estimation-of-logic-combination-priors
description: Use when inferring which Boolean logic gate (e.g. TF1 AND TF2, TF1 AND
  NOT TF2) regulates a target gene from scRNA-seq profiles, and you need to replace
  per-rule likelihood computations with a single mixture model whose combination priors
  are updated by EM.
when_to_use_negative:
- The candidate TF set exceeds ~5 regulators — decoding high-order Boolean logic (>5
  regulators) is intractable due to overfitting per the article's discussion
- The task is plain directed TF-gene edge inference without logic gates — use scGATE_edge()
  instead of the EM logic-combination fitting
- Data is pseudotime-sorted along a differentiation trajectory — scGATE does not yet
  handle temporal delays in regulatory logic
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scGATE
  role: R package implementing the zero-inflated mixture fit and EM update of logic-combination
    priors via scGATE_logic()
  repo: https://github.com/CompBioIPM/scGATE
- name: VGAM
  role: R package used to fit the zero-inflated distribution over logic combination
    outputs
  repo: https://github.com/CompBioIPM/scGATE
- name: truncnorm
  role: R package used for the truncated normal density component of the mixture model
  repo: https://github.com/CompBioIPM/scGATE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:befadaa99031f020f70bcc819b47ee38ec71c1b58af68757f17577c388579e73
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# em-estimation-of-logic-combination-priors

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits a zero-inflated mixture distribution over all 2^k Boolean logic combinations of candidate TFs to a target gene's expression profile and uses an expectation-maximization (EM) algorithm to estimate prior weights w_v for each combination, so the Boolean rule can be inferred a posteriori via Bayes factor rather than by fitting a separate likelihood per rule (AND/OR/XOR).

## When to use

Use when inferring which Boolean logic gate (e.g. TF1 AND TF2, TF1 AND NOT TF2) regulates a target gene from scRNA-seq profiles, and you need to replace per-rule likelihood computations with a single mixture model whose combination priors are updated by EM.

## When NOT to use

- The candidate TF set exceeds ~5 regulators — decoding high-order Boolean logic (>5 regulators) is intractable due to overfitting per the article's discussion
- The task is plain directed TF-gene edge inference without logic gates — use scGATE_edge() instead of the EM logic-combination fitting
- Data is pseudotime-sorted along a differentiation trajectory — scGATE does not yet handle temporal delays in regulatory logic

## Inputs

- Preprocessed scRNA-seq expression matrix with normalized counts in the (0,1) interval (cells in rows, genes in columns), produced by scRNA_seq_preprocessing()
- Hill-transformed TF and target gene expression profiles H(tf) per candidate TF
- Enumerated set of 2^k logic combinations for candidate TFs (k <= 5, default max_num_regulators = 3)
- Optional base GRN (.parquet, e.g. from scATAC-seq + TF binding motifs) refined via read_base_GRN()

## Outputs

- Estimated prior weights w_v for each logic combination per target gene
- Assignment of each observation to its maximum-posterior logic combination (Pr(v*|H(t_s), theta_hat) per equation 5)
- Bayes factor BF = L1/L0 per target gene
- gates_results.csv with columns gene_name, -log10 L0, -log10 L1, log10 BF, logic_gate

## How to apply

For each target gene, enumerate the 2^k logic combinations (e.g. TF1 AND TF2, TF1 AND NOT TF2, ...) of its candidate TFs and compute each combination's output as products of Hill-transformed TF activities H(tf) = (k_sat^h + 1) * tf^h / (k_sat^h + tf^h) and [1 - H(tf)] terms. Fit a zero-inflated distribution — a point mass at zero for dropouts plus a normal density centered on each combination's output with variance sigma^2 — using the VGAM and truncnorm packages in R. Run the EM algorithm (number_of_em_iterations defaults to 3 in scGATE_logic()) to update prior weights w_v over combinations from the data, then assign each observation to its maximum-posterior combination and mark a combination active (ON) only if it accounts for more than 5% of the target gene's observations. Compare the fitted likelihood L1 against the base likelihood L0 under uniform priors w_v = 1/2^k to obtain the Bayes factor BF = L1/L0, and report the top gate per target gene by log10 BF. Keep max_num_regulators <= 3 (article recommends up to k = 5 TFs maximum) to avoid overfitting.

## Related tools

- **scGATE** (R package implementing the zero-inflated mixture fit and EM update of logic-combination priors via scGATE_logic()) — https://github.com/CompBioIPM/scGATE
- **VGAM** (R package used to fit the zero-inflated distribution over logic combination outputs) — https://github.com/CompBioIPM/scGATE
- **truncnorm** (R package used for the truncated normal density component of the mixture model) — https://github.com/CompBioIPM/scGATE

## Examples

```
gates <- scGATE_logic(data = data, base_GRN = NA, number_of_em_iterations = 3, max_num_regulators = 3, run_mode = "simple")
```

## Evaluation signals

- gates_results.csv contains a positive log10 BF for each reported target gene
- Each reported target gene has a non-empty logic_gate string
- Synthetic BoolODE toggle-switch run reproduces the qualitative expected pattern (e.g. ~gF for gE, gE.~gE2 for gE1)
- Each active logic combination contributes more than 5% of the target gene's observations (weight_threshold)
- max_num_regulators kept <= 3 so fitted gates are within the <5-TF regime where overfitting is not a concern

## Limitations

- Decoding high-order Boolean logic involving more than 5 transcriptional regulators remains challenging due to overfitting
- scGATE must currently be applied to a small (<200 gene) subset with refined candidate TFs from prior biological knowledge or external hints, not large-scale inference from scratch
- Number of EM iterations defaults to 3; run_mode "simple" trades precision of the Hill parameter space search for speed versus "complex"

## Evidence

- [methods] we fit a zero-inflated distribution to calculate the probability of H(t_s) being generated from the vth logic combination: [evidence span withheld — anchor status: not_found]
