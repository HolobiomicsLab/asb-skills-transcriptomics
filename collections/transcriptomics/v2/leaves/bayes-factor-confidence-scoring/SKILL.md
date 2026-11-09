---
name: bayes-factor-confidence-scoring
description: Use when scGATE has fit a zero-inflated mixture likelihood to a target
  gene's expression and you need a confidence score to select or rank the most probable
  Boolean logic gate among candidate regulatory TFs.
when_to_use_negative:
- Candidate TF lists per target exceed k=5 regulators (>5 factors) — overfitting makes
  Bayes-factor ranking unreliable, per the article's discussion
- You only need a directed TF-target edge network without logic gates — use scGATE_edge()
  instead of scoring gates
- The dataset is a large-scale unfiltered GRN inference from scratch — scGATE (and
  hence BF scoring) is applicable only to a small (<200) subset of genes with a refined
  candidate TF list
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scGATE
  role: Computes per-gene L0, L1 and log10 Bayes factor via scGATE_logic() and writes
    the top gates with the logic_gate label
  repo: github.com/CompBioIPM/scGATE
- name: VGAM
  role: Fits the zero-inflated distribution used to compute the mixture likelihood
    L1 underlying the Bayes factor
- name: truncnorm
  role: Provides truncated normal densities for the zero-inflated likelihood used
    in L0/L1 computation
- name: R 4.1.3
  role: Language in which scGATE and its Bayes-factor scoring are implemented and
    tested (Windows/Linux)
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:e5de9d861e74b438e9a51d76d7fffca34777b26709bd0e2bfd045ff651e78774
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
  - R 4.1.3
  - VGAM
  - arrow
  - doParallel
  - doSNOW
  - foreach
  - scGATE
  - truncnorm
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Bayes factor confidence scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ranks inferred Boolean logic gates (TF combinations regulating a target gene) by a Bayes factor BF = L1/L0, where L1 is the mixture-model likelihood under EM-updated priors and L0 is the likelihood under uniform priors w_v = 1/2^k; reported as log10 BF per target gene.

## When to use

Use when scGATE has fit a zero-inflated mixture likelihood to a target gene's expression and you need a confidence score to select or rank the most probable Boolean logic gate among candidate regulatory TFs.

## When NOT to use

- Candidate TF lists per target exceed k=5 regulators (>5 factors) — overfitting makes Bayes-factor ranking unreliable, per the article's discussion
- You only need a directed TF-target edge network without logic gates — use scGATE_edge() instead of scoring gates
- The dataset is a large-scale unfiltered GRN inference from scratch — scGATE (and hence BF scoring) is applicable only to a small (<200) subset of genes with a refined candidate TF list

## Inputs

- Normalized scRNA-seq expression matrix (values in (0,1)) from scRNA_seq_preprocessing()
- Hill-function-transformed TF/target profiles H(tf) = (k_sat^h + 1) * tf^h / (k_sat^h + tf^h)
- EM-estimated prior weights w_v over 2^k enumerated logic combinations (AND, OR, XOR, NOT forms)
- Zero-inflated mixture likelihood components (point mass at zero plus truncated normal densities from VGAM/truncnorm)

## Outputs

- gates_results.csv with columns gene_name, -log10 L0, -log10 L1, log10 BF, logic_gate
- Top Boolean logic gate per target gene ranked by log10 Bayes factor
- sensitivity_analysis.csv aggregating log10 BF per cell count and TF-set size (for saturation analyses)

## How to apply

After running scGATE_logic() with number_of_em_iterations (default 3) and top_gates (default 1), compute BF = L1/L0 where L1 uses EM-estimated prior weights w_v over the 2^k TF combinations and L0 uses uniform priors w_v = 1/2^k; report log10 L0, log10 L1, and log10 BF per target gene. Select the logic combination with maximum posterior Pr(v*|H(t_s), theta_hat), marking a combination active only if it generates more than 5% of the target gene's observations. Report the top gates per gene (columns gene_name, -log10 L0, -log10 L1, log10 BF, logic_gate) to gates_results.csv, keeping max_num_regulators <= 3 to avoid overfitting. A positive log10 BF indicates the inferred gate is better supported than the uniform-prior null model; the metric also serves as the quality signal in downsampling sensitivity analyses (Figure 5C), where it plateaus near ~3000 cells for 15 regulatory TFs.

## Related tools

- **scGATE** (Computes per-gene L0, L1 and log10 Bayes factor via scGATE_logic() and writes the top gates with the logic_gate label) — github.com/CompBioIPM/scGATE
- **VGAM** (Fits the zero-inflated distribution used to compute the mixture likelihood L1 underlying the Bayes factor)
- **truncnorm** (Provides truncated normal densities for the zero-inflated likelihood used in L0/L1 computation)
- **R 4.1.3** (Language in which scGATE and its Bayes-factor scoring are implemented and tested (Windows/Linux))

## Examples

```
gates <- scGATE_logic(data = data, base_GRN = NA, h_set = NA, number_of_em_iterations = 3, max_num_regulators = 3, abs_cor = 0, top_gates = 1, run_mode = "simple"); write.csv(gates, "gates_results.csv")
```

## Evaluation signals

- gates_results.csv contains a positive log10 BF and a non-empty logic_gate string for each reported target gene
- Inferred gates reproduce the qualitative pattern of the BoolODE synthetic example (e.g. ~gF for gE, gE.~gE2 for gE1)
- Active logic combinations generate more than 5% of the target gene's observations (weight_threshold check)
- In downsampling sensitivity analyses, log10 BF quality plateaus near the reported ~3000-cell subsample (Figure 5C behavior)
- Output columns match the expected schema: gene_name, -log10 L0, -log10 L1, log10 BF, logic_gate

## Limitations

- Decoding high-order Boolean logic involving more than 5 transcriptional regulators remains challenging due to overfitting
- scGATE must currently be applied to a small pre-selected set of TFs based on prior biological knowledge; it is not designed for large-scale inference from scratch
- scGATE does not yet handle pseudotime-sorted single-cell data along differentiation trajectories or temporal delays

## Evidence

- [methods] Compute the Bayes factor BF = L1/L0, where L0 is the base likelihood under uniform priors w_v = 1/2^k: [evidence span withheld — anchor status: not_found]
- [methods] we recommend fitting logic gates among subsets with up to $k=5$ factors of the candidate TF lists. This approach helps prevent overfitting: [evidence span withheld — anchor status: not_found]
- [methods] Mark a combination active (ON) only if it generates more than 5% of the target gene's observations: [evidence span withheld — anchor status: not_found]
