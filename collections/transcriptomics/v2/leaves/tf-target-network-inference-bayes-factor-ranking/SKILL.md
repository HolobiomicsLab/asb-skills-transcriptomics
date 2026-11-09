---
name: tf-target-network-inference-bayes-factor-ranking
description: Use when you have a small (<200 gene) scRNA-seq matrix, a candidate TF
  list, and an optional scATAC-seq/motif-derived base GRN parquet, and you need a
  directed, BF-ranked TF–target network for a specific tissue or cell type.
when_to_use_negative:
- Input spans more than ~200 genes or lacks a refined candidate TF list — the paper
  states scGATE 'is currently applicable to a rather small (<200) subset of genes
  with a refined list of candidate regulatory TFs based on external hints'
- Data are pseudotime-sorted along differentiation trajectories or you need to model
  temporal delays — scGATE does not yet handle these
- You expect to capture regulation by large protein complexes or high-order logic
  (>5 regulators) — overfitting makes this unreliable and such interactions are missed
  by the model and the ground truth
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scGATE
  role: 'Core inference tool: read_base_GRN(), scRNA_seq_preprocessing(), and scGATE_edge()/scGATE_logic()
    produce the BF-ranked directed network (version 0.1.0, R 4.1.3)'
  repo: github.com/CompBioIPM/scGATE
- name: arrow
  role: Reads the base GRN .parquet files (e.g. Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet)
    into an R data frame before read_base_GRN() summarization
- name: VGAM
  role: R dependency used for zero-inflated distribution fitting during logic-combination
    likelihood computation
- name: truncnorm
  role: R dependency for truncated-normal components in the Hill-function-based model
    fitting
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:a9359242813f887a1d8bf1359603dabcc9a04fca9f62a0f78f0155f196dc686e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CompBioIPM/scGATE
  tools:
  - R 4.1.3
  - VGAM
  - arrow
  - doParallel
  - doSNOW
  - foreach
  - scGATE
  - truncnorm
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# tf-target-network-inference-bayes-factor-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infer directed TF–gene regulatory networks from preprocessed scRNA-seq count matrices with scGATE (R 4.1.3), fitting a Hill activation function within a Bayesian EM framework and ranking candidate edges by Bayes Factor (BF_score). A base GRN from scATAC-seq peaks and TF binding motifs constrains candidate TFs per target gene.

## When to use

Use when you have a small (<200 gene) scRNA-seq matrix, a candidate TF list, and an optional scATAC-seq/motif-derived base GRN parquet, and you need a directed, BF-ranked TF–target network for a specific tissue or cell type.

## When NOT to use

- Input spans more than ~200 genes or lacks a refined candidate TF list — the paper states scGATE 'is currently applicable to a rather small (<200) subset of genes with a refined list of candidate regulatory TFs based on external hints'
- Data are pseudotime-sorted along differentiation trajectories or you need to model temporal delays — scGATE does not yet handle these
- You expect to capture regulation by large protein complexes or high-order logic (>5 regulators) — overfitting makes this unreliable and such interactions are missed by the model and the ground truth

## Inputs

- Raw scRNA-seq UMI count matrix as CSV with cells in rows and genes in columns (e.g. Tabula_Muris2018_Spleen-10X_P4_7_ExpressionData.csv)
- Base GRN of candidate TF–target pairs from scATAC-seq peaks and TF binding motifs in .parquet format (e.g. Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet, Buenrostro2018_base_GRN_dataframe.parquet)
- Optional list of candidate transcription factors (e.g. Tabula_Muris2018_Spleen-10X_P4_7_tf_lists.txt)
- Optional ground-truth GRN (e.g. ChIP-Atlas-derived mouse TF ChIP-seq networks, Cus_KO / Cus_Chip networks) for benchmarking

## Outputs

- Directed TF–target network as a ranked edge list with columns 'from', 'to', 'BF_score' (e.g. saved as buenrostro2018_edge_list.csv)
- Boolean logic gates per target gene when using scGATE_logic()/scGATE_gate() (top_gates default 1, ranked by Bayes Factor)
- Benchmark metrics (AUROC, EPR, AUPR) and average number of regulating TFs per target gene when evaluated against a ground truth

## How to apply

In R 4.1.3 with scGATE_0.1.0 installed (dependencies: VGAM, truncnorm, arrow, doParallel, foreach, doSNOW), first summarize the base GRN from its .parquet file using arrow::read_parquet followed by scGATE's read_base_GRN(). Preprocess raw UMI counts with scRNA_seq_preprocessing(data, library_size_normalization='True', tf_list=tf_names), which divides each cell's counts by total reads, scales by a scaling factor, and rescales into the (0,1) interval. Run scGATE_edge(data, base_GRN = candidate_tf_target, h_act = 7) with the default number_of_em_iterations = 3 and max_num_regulators = 3 to obtain a ranked edge list with 'from', 'to', and 'BF_score' columns; use scGATE_logic()/scGATE_gate() instead if Boolean logic gates (AND/OR/XOR among ≤5 TFs) are wanted. Optionally set abs_cor in (0,1) to further drop edges with low absolute Pearson correlation between TF and target. Benchmark the ranked edges against a ground truth (e.g. ChIP-Atlas-derived GRNs) with AUROC/EPR or AUPR metrics.

## Related tools

- **scGATE** (Core inference tool: read_base_GRN(), scRNA_seq_preprocessing(), and scGATE_edge()/scGATE_logic() produce the BF-ranked directed network (version 0.1.0, R 4.1.3)) — github.com/CompBioIPM/scGATE
- **arrow** (Reads the base GRN .parquet files (e.g. Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet) into an R data frame before read_base_GRN() summarization)
- **VGAM** (R dependency used for zero-inflated distribution fitting during logic-combination likelihood computation)
- **truncnorm** (R dependency for truncated-normal components in the Hill-function-based model fitting)

## Examples

```
scGATE_edge(data = data, base_GRN = candidate_tf_target, h_act = 7, number_of_em_iterations = 3, max_num_regulators = 3)
```

## Evaluation signals

- On Tabula Muris Spleen-10X_P4_7, head(ranked_edge_list) must match the reference output: Ctcf→Rps19 BF ≈ 2002.419, Batf→Rps19 BF ≈ 2001.388, Stat5b→Rplp0 BF ≈ 1840.046
- The average number of regulating TFs per target gene should reproduce the reported value of ~3.39
- Output edge list must contain exactly the 'from', 'to', 'BF_score' columns with positive Bayes Factor scores; input matrix must lie in the (0,1) interval after scRNA_seq_preprocessing()
- Against ChIP-Atlas ground truth on 8 Tabula Muris scRNA-seq datasets, scGATE should achieve best AUROC and EPR in 7 of 8 datasets relative to other benchmarked tools
- Default parameters must match the reference run: h_act = 7, number_of_em_iterations = 3, max_num_regulators = 3, abs_cor = 0 (no correlation filtration)

## Limitations

- Logic-gate decoding is recommended only for subsets of up to k=5 candidate TFs to prevent overfitting; >5 regulators remains challenging
- scGATE requires external hints (scATAC-seq + TF binding motifs) or prior biological knowledge to pre-select TFs; large-scale inference from scratch is not supported
- Interactions within large protein complexes are missed by both the model and ChIP-seq-derived ground truth networks
- Pseudotime-sorted single-cell data along differentiation trajectories and temporal delays are not yet handled

## Evidence

- [methods] Overfitting guard for logic gates: [evidence span withheld — anchor status: not_found]
- [results] Benchmark result on Tabula Muris: [evidence span withheld — anchor status: not_found]
