---
name: bayesian-mixture-model-em-inference
description: Use when inferring cell-type-specific TF-gene networks and Boolean logic
  gates from a small, pre-selected set of TFs and target genes in single-cell expression
  data, given a candidate base GRN.
when_to_use_negative:
- The TF set is large (>5 regulators per target) or the gene set exceeds ~200 genes
  with unrefined candidates — overfitting of high-order Boolean logic is a major concern
  and scGATE should not be used for large-scale inference from scratch
- Data are pseudotime-sorted along differentiation trajectories with temporal delays
  — scGATE does not yet handle temporally ordered single-cell data
- Only a plain directed TF-gene network is needed with no combinatorial logic — scGATE_edge()
  is a simpler fit, and other tools may suffice
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scGATE
  role: 'Implements the Bayesian EM inference: scGATE_logic()/scGATE_gate() fit zero-inflated
    mixture likelihoods over Boolean logic combinations and rank gates by Bayes Factor'
  repo: https://github.com/CompBioIPM/scGATE
- name: R 4.1.3
  role: Language environment in which the scGATE Bayesian EM code is written and executed
    (Windows/Linux)
- name: VGAM
  role: R dependency used for zero-inflated distribution fitting within the mixture
    likelihood
- name: truncnorm
  role: R dependency for truncated normal terms in the Hill-function-based model likelihood
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:8df8939d832bae6b3f6c568e8766699e4cd3305b000be39fa2247d12e76f0bee
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
  - R 4.1.3
  - VGAM
  - arrow
  - doParallel
  - doSNOW
  - foreach
  - scGATE
  - truncnorm
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# bayesian-mixture-model-em-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A Bayesian EM-based procedure used by scGATE to fit zero-inflated mixture likelihoods over Boolean logic combinations (AND/OR/XOR on TF activities) and infer, a posteriori, the most probable logic gate and TF-target edges from continuous scRNA-seq data without binarization.

## When to use

Use when inferring cell-type-specific TF-gene networks and Boolean logic gates from a small, pre-selected set of TFs and target genes in single-cell expression data, given a candidate base GRN.

## When NOT to use

- The TF set is large (>5 regulators per target) or the gene set exceeds ~200 genes with unrefined candidates — overfitting of high-order Boolean logic is a major concern and scGATE should not be used for large-scale inference from scratch
- Data are pseudotime-sorted along differentiation trajectories with temporal delays — scGATE does not yet handle temporally ordered single-cell data
- Only a plain directed TF-gene network is needed with no combinatorial logic — scGATE_edge() is a simpler fit, and other tools may suffice

## Inputs

- scRNA-seq expression matrix (cells x genes), counts normalized to the (0,1) interval after scRNA_seq_preprocessing()
- Base GRN of candidate TF-target links derived from scATAC-seq data and TF DNA binding motifs (.parquet, e.g. Buenrostro2018_base_GRN_dataframe.parquet)
- Pre-selected TF list (e.g. Gata1, Fli1, Klf1, Spi1, Zfpm1, Tal1, Gata2 for MegE differentiation)
- Parameters: number_of_em_iterations, top_gates, max_num_regulators, h_set, abs_cor, run_mode

## Outputs

- Ranked Boolean logic gates per target gene with log10 Bayes Factor confidence scores (e.g. Fli1 = Gata1 AND NOT Klf1)
- Directed TF-target network inferred from the highest-Bayes-Factor gates
- Logic combination or partition explaining a percentage of target expression above weight_threshold

## How to apply

Preprocess the scRNA-seq matrix with scRNA_seq_preprocessing() so normalized counts lie in the (0,1) interval (set library_size_normalization='False' if already normalized upstream), and restrict genes to highly variable ones. Optionally supply a base GRN (from scATAC-seq plus TF binding motifs) via read_base_GRN() to narrow candidate TFs per target. Run scGATE_logic() (or scGATE_gate()), which models continuous TF and target profiles with a Hill activation function, fits a zero-inflated distribution to each candidate logic combination, and uses EM (number_of_em_iterations, default 3; the haematopoiesis task uses 10) to update prior probabilities over TF combinations, selecting the most probable Boolean rule a posteriori. Use run_mode='complex' for more precise Hill parameter exploration, top_gates=1 (or 50 in simple mode for >10 TF panels), max_num_regulators=2 with h_set=c(1.25, 2.25) when constraining gate order. Rank gates and edges by log10 Bayes Factor, where each candidate TF is scored by the BF of the most significant gate including it; optionally filter edges with abs_cor on absolute Pearson correlation.

## Related tools

- **scGATE** (Implements the Bayesian EM inference: scGATE_logic()/scGATE_gate() fit zero-inflated mixture likelihoods over Boolean logic combinations and rank gates by Bayes Factor) — https://github.com/CompBioIPM/scGATE
- **R 4.1.3** (Language environment in which the scGATE Bayesian EM code is written and executed (Windows/Linux))
- **VGAM** (R dependency used for zero-inflated distribution fitting within the mixture likelihood)
- **truncnorm** (R dependency for truncated normal terms in the Hill-function-based model likelihood)

## Examples

```
gates <- scGATE_logic(data = data, base_GRN = base_grn, number_of_em_iterations = 10, top_gates = 1, run_mode = "complex")
```

## Evaluation signals

- The inferred gate for the Gata1/Klf1 regulation of Fli1 should be Fli1 = Gata1 AND NOT Klf1 with the highest log10 Bayes Factor among the 16 candidate gates
- Preprocessed expression values must lie strictly in the (0,1) interval after scRNA_seq_preprocessing()
- Bayes Factors should be computed after the specified number of EM iterations (default 3; 10 in the haematopoiesis reproduction) and each TF scored by the BF of its most significant gate
- Compare the inferred directed network against the deposited ground-truth GRN from Zenodo (10.5281/zenodo.8353409) using AUROC and AUPRC to reproduce the reported benchmark
- Gates should be restricted to subsets of fewer than 5 TFs to avoid overfitting

## Limitations

- Decoding high-order Boolean logic involving more than 5 transcriptional regulators per target remains challenging due to overfitting
- scGATE is currently applicable to a small (<200) subset of genes with a refined candidate TF list based on external hints, not to large-scale inference from scratch
- Does not yet handle pseudotime-sorted single-cell data along differentiation trajectories or temporal delays
- Large protein complex interactions are missed both in the model and in the ground-truth networks

## Evidence

- [methods] Gate subset size limited to fewer than 5 TFs: [evidence span withheld — anchor status: not_found]
- [methods] Zero-inflated distribution over logic combinations: [evidence span withheld — anchor status: not_found]
