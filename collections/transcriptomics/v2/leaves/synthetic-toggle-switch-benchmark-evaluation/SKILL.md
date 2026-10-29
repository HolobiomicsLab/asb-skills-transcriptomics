---
name: synthetic-toggle-switch-benchmark-evaluation
description: Use when validating a logic-based GRN inference method on synthetic scRNA-seq
  data where the ground-truth Boolean regulatory rules are known (e.g. BoolODE toggle-switch
  simulations), and you need to confirm recovered gates match designed AND/OR/NOT
  logic.
when_to_use_negative:
- The target dataset is a real, large-scale scRNA-seq experiment without a refined
  candidate TF list — the authors state scGATE is applicable only to a small (<200)
  subset of genes with pre-selected TFs, so synthetic-benchmark settings like max_num_regulators=3
  with no base GRN would not transfer
- The data are pseudotime-sorted along differentiation trajectories or involve temporal
  delays — the paper notes scGATE does not yet handle pseudotime-sorted single-cell
  data
- The goal is to decode high-order Boolean logic involving more than 5 transcriptional
  regulators — the article flags overfitting as a major concern above k=5
edam_operation: http://edamontology.org/operation_2492
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: scGATE
  role: 'Core inference tool: scRNA_seq_preprocessing() for rescaling simulated counts
    and scGATE_logic() for Bayesian EM fitting of zero-inflated mixture likelihoods
    over 2^k logic combinations per candidate TF subset'
  repo: github.com/CompBioIPM/scGATE
- name: R
  role: Runtime environment (R version 4.1.3) in which the scGATE package is installed
    and the benchmark pipeline is executed
  repo: github.com/CompBioIPM/scGATE
- name: BoolODE
  role: Generator of the synthetic toggle-switch scRNA-seq ground-truth data used
    as the benchmark input
- name: VGAM
  role: R dependency used by scGATE for fitting the zero-inflated (dropout + normal)
    distribution in the EM likelihood
  repo: github.com/CompBioIPM/scGATE
- name: truncnorm
  role: R dependency for truncated-normal components used in scGATE's zero-inflated
    distribution fitting
  repo: github.com/CompBioIPM/scGATE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:6871badf7091675c2c096241930e9b559ca532c79e0aa195083475fffcdc7b53
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CompBioIPM/scGATE
  tools:
  - BoolODE
  - R
  - VGAM
  - scGATE
  - truncnorm
  edam_operation: http://edamontology.org/operation_2492
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# synthetic-toggle-switch-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks scGATE's ability to infer context-specific Boolean logic gates from synthetic BoolODE toggle-switch scRNA-seq data with no a priori network structure, validating inferred gates (e.g. gE1 = gE AND NOT gE2) against the simulated ground-truth rules and reported log10 Bayes Factor scores.

## When to use

Use when validating a logic-based GRN inference method on synthetic scRNA-seq data where the ground-truth Boolean regulatory rules are known (e.g. BoolODE toggle-switch simulations), and you need to confirm recovered gates match designed AND/OR/NOT logic.

## When NOT to use

- The target dataset is a real, large-scale scRNA-seq experiment without a refined candidate TF list — the authors state scGATE is applicable only to a small (<200) subset of genes with pre-selected TFs, so synthetic-benchmark settings like max_num_regulators=3 with no base GRN would not transfer
- The data are pseudotime-sorted along differentiation trajectories or involve temporal delays — the paper notes scGATE does not yet handle pseudotime-sorted single-cell data
- The goal is to decode high-order Boolean logic involving more than 5 transcriptional regulators — the article flags overfitting as a major concern above k=5

## Inputs

- BoolODE synthetic toggle-switch count matrix (ClusterI.csv, genes x cells with counts) from the scGATE example_data folder
- Ground-truth BoolODE toggle-switch regulatory rules (e.g. gE = NOT gF, gE1 = gE AND NOT gE2)
- scGATE R package (scGATE_0.1.0.tar.gz) with dependencies VGAM and truncnorm

## Outputs

- scgate_synthetic_gates.csv with columns gene_name, -log10 L0, -log10 L1, log10 BF, logic_gate
- Per-target-gene inferred logic_gate strings (e.g. ~gF, gE.~gE2) with log10 Bayes Factor confidence scores
- Cluster II gate calls (e.g. gC AND NOT gC2 regulating gC1) for cross-cluster comparison

## How to apply

Load the BoolODE synthetic toggle-switch Cluster I count matrix (ClusterI.csv, 14 genes including gA, gB, gC/gC1/gC2, gD/gD1/gD2, gE/gE1/gE2, gF/gF1/gF2) from the scGATE example_data folder. Preprocess with scRNA_seq_preprocessing() setting library_size_normalization="False" since simulated counts are rescaled into the (0,1) interval via quantile normalization, then remove low-variability genes by keeping those with standard deviation (sqrt of variance across cells) above 0.20. Run scGATE_logic() with run_mode="simple", top_gates=1, max_num_regulators=3 (candidate TF subsets up to k=3, i.e. 2^3 logic combinations per gate) and the default 3 EM iterations, where prior probabilities w_v over logic combinations are fitted with a zero-inflated (dropout + normal) mixture likelihood. Extract the inferred logic_gate string and log10 BF per target gene and compare against the BoolODE ground truth (e.g. gE = NOT gF, gF = NOT gE, gE1 = gE AND NOT gE2); gates are judged correctly recovered when log10 BF is positive and the inferred gate string matches the reference (gE -> ~gF with log10 BF 442.47; gE1 -> gE.~gE2 with log10 BF 286.50).

## Related tools

- **scGATE** (Core inference tool: scRNA_seq_preprocessing() for rescaling simulated counts and scGATE_logic() for Bayesian EM fitting of zero-inflated mixture likelihoods over 2^k logic combinations per candidate TF subset) — github.com/CompBioIPM/scGATE
- **R** (Runtime environment (R version 4.1.3) in which the scGATE package is installed and the benchmark pipeline is executed) — github.com/CompBioIPM/scGATE
- **BoolODE** (Generator of the synthetic toggle-switch scRNA-seq ground-truth data used as the benchmark input)
- **VGAM** (R dependency used by scGATE for fitting the zero-inflated (dropout + normal) distribution in the EM likelihood) — github.com/CompBioIPM/scGATE
- **truncnorm** (R dependency for truncated-normal components used in scGATE's zero-inflated distribution fitting) — github.com/CompBioIPM/scGATE

## Examples

```
gates <- scGATE_logic(data = normalized_counts, base_GRN = NA, number_of_em_iterations = 3, max_num_regulators = 3, top_gates = 1, run_mode = "simple")
```

## Evaluation signals

- scgate_synthetic_gates.csv contains exactly the columns gene_name, -log10 L0, -log10 L1, log10 BF, logic_gate
- All reported gates have positive log10 BF (confidence above the null model)
- Inferred gate for gE matches the ground truth ~gF with log10 BF ≈ 442.47
- Inferred gate for gE1 matches the ground truth gE.~gE2 (gE AND NOT gE2) with log10 BF ≈ 286.50
- Cluster II recovery: the gate gC AND NOT(gC2) regulating gC1 is identified, confirming context-specific AND logic across both differentiation clusters

## Limitations

- Decoding high-order Boolean logic involving a greater number of transcriptional regulators (e.g. >5) governing a common target remains challenging, where overfitting is a major concern
- scGATE is currently applicable to a rather small (<200) subset of genes with a refined list of candidate regulatory TFs based on external hints
- Large protein complex interactions will be missed both in the current model and in the ground-truth networks
- The Bayesian EM fit uses a fixed small number of EM iterations (default 3), and run_mode='simple' trades precision for speed in the Hill-function parameter space

## Evidence

- [other] Run scGATE_logic() on the preprocessed matrix with run_mode="simple", top_gates=1, max_num_regulators=3 (candidate TF subsets up to k=3, i.e. 2^3 logic combinations per gate) and the default 3 EM iterations: [evidence span withheld — anchor status: not_found]
- [methods] we fit a zero-inflated distribution to calculate the probability of $H(t_{s})$ being generated from the $v$th logic combination.: [evidence span withheld — anchor status: not_found]
- [methods] we recommend fitting logic gates among subsets with up to $k=5$ factors of the candidate TF lists. This approach helps prevent overfitting: [evidence span withheld — anchor status: not_found]
