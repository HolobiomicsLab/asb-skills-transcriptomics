---
name: gene-regulatory-network-inference
description: Use when scRNA-seq (optionally with scATAC-seq/motif hints) must be turned
  into a cell-type-specific directed TF-gene network whose target genes are explained
  by Boolean logic combinations of <5 candidate regulators.
when_to_use_negative:
- 'Large-scale inference from scratch: scGATE is applicable only to a small (<200)
  subset of genes with a refined list of candidate TFs based on external hints, not
  genome-wide network discovery.'
- 'Pseudotime-sorted single-cell data along differentiation trajectories or temporal-delay
  modeling: scGATE does not yet handle temporal information.'
- 'Targets regulated by more than ~5 TFs: decoding high-order Boolean logic beyond
  k=5 regulators is unreliable due to overfitting.'
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scGATE
  role: Core R package (v0.1.0) providing scRNA_seq_preprocessing(), read_base_GRN(),
    scGATE_logic() and scGATE_edge() for zero-inflated Bayesian logic-gate GRN inference
  repo: https://github.com/CompBioIPM/scGATE
- name: R 4.1.3
  role: Runtime environment in which scGATE is installed from source (scGATE_0.1.0.tar.gz)
    and executed on Windows or Linux
- name: VGAM
  role: R dependency used by scGATE for zero-inflated distribution fitting during
    EM inference
- name: truncnorm
  role: R dependency for truncated-normal likelihood terms in the Hill-function mixture
    model
- name: arrow
  role: R package used to read .parquet base GRN files with read_parquet()
- name: doParallel / foreach / doSNOW
  role: R packages enabling parallel computing for scGATE runs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:7600bd5863641eb90926f3d2ac42109ebd6f0022c80d0a24f1d8609ed1050cd0
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
  - GRNBOOST2
  - MINI-EX
  - PIDC
  - R
  - R 4.1.3
  - RSpectra
  - Rcpp
  - SCING
  - SCopeLoomR
  - VGAM
  - arrow
  - conda
  - doParallel
  - doParallel / foreach / doSNOW
  - doSNOW
  - foreach
  - igraph
  - ppcor
  - pySCENIC
  - scGATE
  - scPlant
  - scTenifoldNet
  - truncnorm
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# gene-regulatory-network-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infer directed TF–target gene regulatory networks together with Boolean logic gates (AND, OR, XOR combinations) from single-cell RNA-seq data using scGATE, a Bayesian zero-inflated mixture model that fits Hill activation functions without binarizing expression.

## When to use

Use when scRNA-seq (optionally with scATAC-seq/motif hints) must be turned into a cell-type-specific directed TF-gene network whose target genes are explained by Boolean logic combinations of <5 candidate regulators.

## When NOT to use

- Large-scale inference from scratch: scGATE is applicable only to a small (<200) subset of genes with a refined list of candidate TFs based on external hints, not genome-wide network discovery.
- Pseudotime-sorted single-cell data along differentiation trajectories or temporal-delay modeling: scGATE does not yet handle temporal information.
- Targets regulated by more than ~5 TFs: decoding high-order Boolean logic beyond k=5 regulators is unreliable due to overfitting.

## Inputs

- Normalized scRNA-seq expression matrix (cells × genes), rescaled to the (0,1) interval via scRNA_seq_preprocessing()
- Base GRN of candidate TF-target links in .parquet format, generated from scATAC-seq data and TF DNA binding motifs (e.g. Buenrostro2018_base_GRN_dataframe.parquet)
- Candidate TF list for a pre-selected context (e.g. Gata1, Fli1, Klf1, Spi1, Zfpm1, Tal1, Gata2 for MegE differentiation)
- Optional ground-truth GRN file (e.g. base_grn_mouse_blood_cell_differentiation_toggle_switch.csv from Zenodo 10.5281/zenodo.8353409) for benchmarking

## Outputs

- Ranked Boolean logic gates per target gene with log10 Bayes Factor confidence scores (e.g. Fli1 = Gata1 AND NOT Klf1)
- Directed TF-target gene network (with or without logic gates, via scGATE_logic() or scGATE_edge())
- Benchmark metrics AUROC and AUPRC against a deposited ground-truth GRN

## How to apply

Start from a normalized expression matrix rescaled into the (0,1) interval — run scRNA_seq_preprocessing() (set library_size_normalization='False' if normalization was applied upstream) and remove genes with low variability, since scGATE operates on highly variable genes per context. Restrict the analysis to a small pre-selected set of candidate TFs per target, ideally from a base GRN in .parquet format derived from scATAC-seq and TF binding motifs, summarized with read_base_GRN(); edges with low absolute Pearson correlation can be pruned via the abs_cor parameter. Fit the model with scGATE_logic() (e.g. number_of_em_iterations=10, top_gates=1, run_mode='complex' for precision; run_mode='simple' and top_gates=50 for panels with >10 TFs, or max_num_regulators=2 with h_set=c(1.25, 2.25)), which fits a zero-inflated distribution over logic combinations by EM and infers the most probable Boolean rule a posteriori via a Bayesian framework, scoring gates and edges by log10 Bayes Factor. For network-only output (no gates), use scGATE_edge(). Keep subsets to k<=5 TFs to prevent overfitting, and validate the inferred directed network against a ground-truth GRN by computing AUROC and AUPRC.

## Related tools

- **scGATE** (Core R package (v0.1.0) providing scRNA_seq_preprocessing(), read_base_GRN(), scGATE_logic() and scGATE_edge() for zero-inflated Bayesian logic-gate GRN inference) — https://github.com/CompBioIPM/scGATE
- **R 4.1.3** (Runtime environment in which scGATE is installed from source (scGATE_0.1.0.tar.gz) and executed on Windows or Linux)
- **VGAM** (R dependency used by scGATE for zero-inflated distribution fitting during EM inference)
- **truncnorm** (R dependency for truncated-normal likelihood terms in the Hill-function mixture model)
- **arrow** (R package used to read .parquet base GRN files with read_parquet())
- **doParallel / foreach / doSNOW** (R packages enabling parallel computing for scGATE runs)

## Examples

```
gates <- scGATE_logic(data = data, base_GRN = candidate_tf_target, number_of_em_iterations = 10, top_gates = 1, run_mode = "complex")
```

## Evaluation signals

- Rank gates/edges by log10 Bayes Factor and confirm the top-scoring gate reproduces known rules, e.g. Fli1 = Gata1 AND NOT Klf1 for MegE differentiation.
- Compare the inferred directed TF-target network against the deposited ground-truth GRN and check AUROC/AUPRC match reported benchmark performance.
- Verify expression matrix values lie in the (0,1) interval after scRNA_seq_preprocessing() before running scGATE_logic().
- Check that fitted logic gates involve at most k<=5 (default max_num_regulators=3) regulators per target to avoid overfitting.
- Confirm abs_cor filtration removed TF-target pairs with low absolute Pearson correlation when a nonzero threshold is set.

## Limitations

- scGATE must currently be applied to a relatively small set of pre-selected TFs based on prior biological knowledge; it cannot discover networks without candidate hints.
- Applicable only to a rather small (<200) subset of genes with refined candidate regulatory TFs.
- Decoding high-order Boolean logic involving >5 transcriptional regulators per target remains challenging due to overfitting.
- Large protein complex interactions are missed both by the model and by the ground-truth networks.
- Pseudotime-sorted data along differentiation trajectories and temporal delays are not yet handled.
- Directed TF-gene networks alone do not convey combinatorial relationships; logic-gate mode is needed for that.

## Evidence

- [methods] Recommended subset size for logic gates: [evidence span withheld — anchor status: not_found]
- [methods] Zero-inflated distribution fitting: [evidence span withheld — anchor status: not_found]
