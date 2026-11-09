---
name: zero-inflated-mixture-likelihood-fitting
description: Use when modeling continuous scRNA-seq expression of a target gene as
  a mixture over candidate TF logic combinations, where scRNA-seq dropouts require
  a point mass at zero and per-rule likelihoods (as in Loregic/LogicTRN) would be
  computationally prohibitive.
when_to_use_negative:
- The candidate TF subset exceeds ~5 regulators (k > 5), where the 2^k mixture and
  EM fitting overfit — the paper states decoding high-order logic with >5 regulators
  'remains challenging, where overfitting is a major concern'
- The gene set is large-scale without prior TF selection — scGATE is applicable only
  to a small (<200) subset of genes with a refined candidate TF list based on external
  hints
- Data are pseudotime-sorted along differentiation trajectories or require temporal
  delay modeling — scGATE does not yet handle these
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3307
tools:
- name: scGATE
  role: 'Package implementing the mixture-model gate inference: Hill transformation,
    zero-inflated likelihood, EM prior estimation, and Bayes-factor selection'
  repo: https://github.com/CompBioIPM/scGATE
- name: VGAM
  role: R package used to fit the zero-inflated (dropout point mass + normal) distribution
    over logic-combination outputs
- name: truncnorm
  role: R package providing the truncated normal density for the non-zero component
    of the mixture likelihood
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:af81f48cd3c7743f34b73808b9fbd6d107fa2b6e4ec57f4fb3a40c098a8469f3
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# zero-inflated mixture-likelihood fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

scGATE's core statistical routine: instead of computing a separate likelihood for each Boolean rule (AND, OR, XOR), it fits a zero-inflated mixture density — a point mass at zero for dropouts plus a normal density centered on each TF combination's Hill-transformed output — to a target gene's scRNA-seq profile, then estimates prior weights w_v over the 2^k logic combinations by EM and selects the gate a posteriori via a Bayes factor.

## When to use

Use when modeling continuous scRNA-seq expression of a target gene as a mixture over candidate TF logic combinations, where scRNA-seq dropouts require a point mass at zero and per-rule likelihoods (as in Loregic/LogicTRN) would be computationally prohibitive.

## When NOT to use

- The candidate TF subset exceeds ~5 regulators (k > 5), where the 2^k mixture and EM fitting overfit — the paper states decoding high-order logic with >5 regulators 'remains challenging, where overfitting is a major concern'
- The gene set is large-scale without prior TF selection — scGATE is applicable only to a small (<200) subset of genes with a refined candidate TF list based on external hints
- Data are pseudotime-sorted along differentiation trajectories or require temporal delay modeling — scGATE does not yet handle these

## Inputs

- Preprocessed scRNA-seq expression matrix with counts rescaled into the (0,1) interval (cells in rows, genes in columns), produced by scRNA_seq_preprocessing()
- Optional base GRN TF-target candidate list (e.g. .parquet from scATAC-seq + TF binding motifs) restricting which TF subsets are enumerated
- Parameter settings: run_mode ('simple'/'complex'), h_set for the Hill function, number_of_em_iterations (default 3), max_num_regulators (default 3), top_gates (default 1), weight_threshold (5%)

## Outputs

- gates_results.csv with columns gene_name, -log10 L0, -log10 L1, log10 BF, logic_gate (top Bayesian gate per target gene)
- Posterior assignment of each observation to its maximum-posterior logic combination, with active (ON) combinations defined by the >5% weight threshold

## How to apply

First transform TF and target expression profiles with the Hill activation function H(tf) = (k_sat^h + 1) * tf^h / (k_sat^h + tf^h), searching over the h parameter space per the run_mode setting ('simple' is faster; 'complex' is more precise). For each target gene, enumerate the 2^k logic combinations (k <= 3 default, up to 5 recommended) and compute combination outputs as products of H(tf) and [1 - H(tf)] terms. Fit the zero-inflated distribution — point mass at zero for dropouts plus a normal density centered on each combination's output with variance sigma^2 — using the VGAM and truncnorm packages in R, and estimate the prior weights w_v with the EM algorithm (number_of_em_iterations = 3 by default). Assign each observation to its maximum-posterior logic combination, mark a combination ON only if it accounts for more than 5% of the target gene's observations (weight_threshold), and compute the Bayes factor BF = L1/L0 where L0 uses uniform priors w_v = 1/2^k. Keep max_num_regulators <= 3 (at most 5) to avoid overfitting, and write the top gate per target to gates_results.csv.

## Related tools

- **scGATE** (Package implementing the mixture-model gate inference: Hill transformation, zero-inflated likelihood, EM prior estimation, and Bayes-factor selection) — https://github.com/CompBioIPM/scGATE
- **VGAM** (R package used to fit the zero-inflated (dropout point mass + normal) distribution over logic-combination outputs)
- **truncnorm** (R package providing the truncated normal density for the non-zero component of the mixture likelihood)

## Examples

```
gates <- scGATE_logic(data = data, base_GRN = NA, h_set = NA, number_of_em_iterations = 3, max_num_regulators = 3, abs_cor = NA, top_gates = 1, run_mode = "simple")
```

## Evaluation signals

- Output file gates_results.csv contains the required columns gene_name, -log10 L0, -log10 L1, log10 BF, logic_gate
- Every reported gate has a positive log10 BF (L1 vs the uniform-prior L0) and a non-empty logic_gate string
- On the BoolODE synthetic toggle-switch Cluster I data, inferred gates match ground truth: gE -> ~gF with log10 BF 442.47 and gE1 -> gE.~gE2 with log10 BF 286.50
- Each combination marked active generates more than 5% of the target gene's observations (weight_threshold check)
- Inferred gates reproduce the designed AND logic of the toggle switch, e.g. gE AND NOT(gE2) regulating gE1 in cluster I

## Limitations

- Decoding Boolean logic with more than 5 transcriptional regulators per gate is unreliable due to overfitting; keep max_num_regulators <= 3 by default
- The EM defaults to 3 iterations and 'simple' run_mode searches a reduced Hill h-parameter space, trading precision for speed
- Dropout-heavy genes or genes with low variability should be removed first (sd of sqrt-variance > 0.20 filter) or the mixture likelihood is poorly identified

## Evidence

- [methods] we fit a zero-inflated distribution to calculate the probability of $H(t_{s})$ being generated from the $v$th logic combination.: [evidence span withheld — anchor status: not_found]
- [methods] we recommend fitting logic gates among subsets with up to $k=5$ factors of the candidate TF lists. This approach helps prevent overfitting: [evidence span withheld — anchor status: not_found]
