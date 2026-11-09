---
name: bayesian-posterior-rule-inference
description: Use when you have a preprocessed (0,1)-rescaled scRNA-seq matrix for
  one cell type/context, a small candidate TF list (k<=5) per target gene, and you
  need the combinatorial Boolean rule (not just directed edges) linking TFs to each
  target.
when_to_use_negative:
- Target genes have a large unrefined candidate TF set (>5 regulators) — the article
  warns overfitting makes logic inference unreliable beyond ~5 TFs and recommends
  <5
- The goal is only directed TF-gene edges without logic — use scGATE_edge() instead
  of gate inference
- The gene set is large-scale (>200 genes) without prior biological knowledge to pre-select
  TFs — scGATE is stated to be applicable only to a small (<200) subset with refined
  candidate regulators
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scGATE
  role: Provides scGATE_logic(), the EM/Bayesian inference of Boolean logic gates
    over candidate TF subsets with log10 BF scoring
  repo: github.com/CompBioIPM/scGATE
- name: VGAM
  role: R dependency used for zero-inflated distribution fitting in the mixture likelihood
    over logic combinations
- name: truncnorm
  role: R dependency for truncated-normal components of the zero-inflated likelihood
    on Hill-transformed expression
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:eb03e12708d70c89f6484a19cd3a074fbe11ebe4c9181a731e0bcae7573d16c4
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
  - scGATE
  - truncnorm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# bayesian-posterior-rule-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infer the most probable Boolean logic gate (AND/OR/XOR combinations) governing a target gene from single-cell expression data by EM-updating prior probabilities w_v over 2^k logic combinations under a zero-inflated (dropout + normal) mixture likelihood, then selecting gates by Bayes Factor instead of enumerating per-rule likelihoods. This is the core inference step of scGATE and drastically reduces time complexity.

## When to use

Use when you have a preprocessed (0,1)-rescaled scRNA-seq matrix for one cell type/context, a small candidate TF list (k<=5) per target gene, and you need the combinatorial Boolean rule (not just directed edges) linking TFs to each target.

## When NOT to use

- Target genes have a large unrefined candidate TF set (>5 regulators) — the article warns overfitting makes logic inference unreliable beyond ~5 TFs and recommends <5
- The goal is only directed TF-gene edges without logic — use scGATE_edge() instead of gate inference
- The gene set is large-scale (>200 genes) without prior biological knowledge to pre-select TFs — scGATE is stated to be applicable only to a small (<200) subset with refined candidate regulators

## Inputs

- Preprocessed scRNA-seq expression matrix with normalized counts in the (0,1) interval, cells as rows and genes as columns (output of scRNA_seq_preprocessing())
- Optional base GRN (TF-gene candidate edges from scATAC-seq + TF binding motifs, .parquet summarized via read_base_GRN()); leave NA if unavailable
- Candidate TF list per target gene (k <= 5 regulators; synthetic toggle-switch case used k = 3 subsets, e.g. gE/gE1/gE2)
- Optional Louvain cluster assignment restricting the matrix to one cell-type context

## Outputs

- Per-target-gate table (e.g. scgate_synthetic_gates.csv) with columns gene_name, -log10 L0, -log10 L1, log10 BF, logic_gate
- Boolean logic_gate strings over the candidate TF subset (e.g. gE AND NOT(gE2) written as gE.~gE2)
- log10 Bayes Factor confidence scores per target gene

## How to apply

Rescale expression into the (0,1) interval with scGATE's scRNA_seq_preprocessing() (set library_size_normalization='False' for simulated count data that already need quantile-style rescaling, 'True' for raw UMI counts), and drop low-variability genes (e.g. keep genes whose SD sqrt(variance across cells) exceeds ~0.20). For each target gene, fit a zero-inflated mixture to the Hill-transformed activity H(t_s) for each of the 2^k logic combinations of the candidate TF subset, with EM (default number_of_em_iterations=3) updating prior probabilities w_v over combinations; use run_mode='simple' for speed and max_num_regulators=3 (up to 5) to avoid overfitting. Because the Bayesian framework infers the rule after model fitting, no separate likelihood formulation per logic gate is needed. Report the top_gates (default 1) logic_gate string per target together with its log10 Bayes Factor (log10 BF = -log10 L0 vs -log10 L1), keeping only gates with positive log10 BF. Validate against ground truth by comparing gate strings (e.g. gE -> ~gF, log10 BF 442.47; gE1 -> gE.~gE2, log10 BF 286.50 on BoolODE toggle-switch data) and checking the output schema (gene_name, -log10 L0, -log10 L1, log10 BF, logic_gate).

## Related tools

- **scGATE** (Provides scGATE_logic(), the EM/Bayesian inference of Boolean logic gates over candidate TF subsets with log10 BF scoring) — github.com/CompBioIPM/scGATE
- **VGAM** (R dependency used for zero-inflated distribution fitting in the mixture likelihood over logic combinations)
- **truncnorm** (R dependency for truncated-normal components of the zero-inflated likelihood on Hill-transformed expression)

## Examples

```
gates <- scGATE_logic(data = data, base_GRN = NA, h_set = NA, number_of_em_iterations = 3, max_num_regulators = 3, abs_cor = NA, top_gates = 1, run_mode = "simple")
```

## Evaluation signals

- Output table contains the exact schema: gene_name, -log10 L0, -log10 L1, log10 BF, logic_gate
- All reported gates have positive log10 BF (e.g. gE -> ~gF at 442.47, gE1 -> gE.~gE2 at 286.50 on the BoolODE Cluster I benchmark)
- Inferred gate strings match the designed ground-truth rules (gE = NOT gF, gF = NOT gE, gE1 = gE AND NOT gE2 in the toggle-switch simulation)
- top_gates parameter respected: exactly the requested number (default 1) of logic combinations reported per target gene
- max_num_regulators kept <= 5 and number_of_em_iterations at default 3 unless explicitly overridden

## Limitations

- Decoding high-order Boolean logic involving more than 5 transcriptional regulators remains challenging due to overfitting
- scGATE must be applied to a small (<200) subset of genes with refined candidate TFs based on prior biological knowledge, not large-scale inference from scratch
- Does not yet handle pseudotime-sorted single-cell data along differentiation trajectories or temporal delays

## Evidence

- [methods] Zero-inflated likelihood per logic combination: [evidence span withheld — anchor status: not_found]
