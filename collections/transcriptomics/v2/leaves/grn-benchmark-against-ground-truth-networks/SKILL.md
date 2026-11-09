---
name: grn-benchmark-against-ground-truth-networks
description: Use when a Bayes-Factor-ranked scGATE edge list (from/to/BF_score) must
  be scored against ground-truth networks such as Cus_KO/Cus_Chip or ChIP-Atlas-derived
  GRNs using precision-recall/AUPR, or when comparing scGATE against other network-inference
  tools on single-cell multi-omics data.
when_to_use_negative:
- 'The inference target is a large-scale, de novo network: the paper states scGATE
  ''is currently applicable to a rather small (<200) subset of genes with a refined
  list of candidate regulatory TFs based on external hints'', so benchmarking on unfiltered
  genome-wide gene sets is out of scope.'
- The goal is inference rather than evaluation, and no external-hint base GRN (scATAC-seq
  + TF motifs) is available to constrain candidate TF–target pairs.
- Ground truth comes from large protein complex interactions, which the authors note
  'will be missed both in our current model as well as the ground-truth', making such
  edges unreliable for scoring.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: scGATE
  role: Source of the Bayes-Factor-ranked TF–target edge list (scGATE_edge mode, 3
    EM iterations, h_act=7) that is benchmarked against the ground-truth GRNs
  repo: https://github.com/CompBioIPM/scGATE
- name: R 4.1.3
  role: Environment in which the scGATE pipeline, edge ranking, and benchmark metric
    computation are executed
- name: arrow
  role: R package used to read the base GRN .parquet files (e.g. Buenrostro2018_base_GRN_dataframe.parquet)
    before read_base_GRN() summarization
- name: VGAM
  role: scGATE dependency for fitting the zero-inflated distributions underlying the
    Bayes Factor edge scores being benchmarked
- name: truncnorm
  role: scGATE dependency supporting the truncated-normal components of the expression
    model behind the ranked edges
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:cec0d895e81eb96331fb5695f2fcd9f45f7596f082b8953d227005c3beec5a3d
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
  - R 4.1.3
  - VGAM
  - arrow
  - scGATE
  - truncnorm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# grn-benchmark-against-ground-truth-networks

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks a scGATE-inferred TF–target network against curated gold-standard GRNs (e.g. Cus_KO and Cus_Chip for human haematopoiesis, or ChIP-Atlas-derived ChIP-seq networks) by computing precision–recall/AUPR on the Bayes-Factor-ranked edge list, to reproduce the paper's claim that scGATE outperforms existing GRN-inference tools.

## When to use

Use when a Bayes-Factor-ranked scGATE edge list (from/to/BF_score) must be scored against ground-truth networks such as Cus_KO/Cus_Chip or ChIP-Atlas-derived GRNs using precision-recall/AUPR, or when comparing scGATE against other network-inference tools on single-cell multi-omics data.

## When NOT to use

- The inference target is a large-scale, de novo network: the paper states scGATE 'is currently applicable to a rather small (<200) subset of genes with a refined list of candidate regulatory TFs based on external hints', so benchmarking on unfiltered genome-wide gene sets is out of scope.
- The goal is inference rather than evaluation, and no external-hint base GRN (scATAC-seq + TF motifs) is available to constrain candidate TF–target pairs.
- Ground truth comes from large protein complex interactions, which the authors note 'will be missed both in our current model as well as the ground-truth', making such edges unreliable for scoring.

## Inputs

- Ranked scGATE edge list (columns 'from', 'to', 'BF_score'), e.g. buenrostro2018_edge_list.csv
- Ground-truth network files (Cus_KO, Cus_Chip from the scGATE GitHub; ChIP-Atlas-derived ChIP-seq GRNs for mouse tissues)
- Buenrostro2018 human haematopoiesis scRNA-seq expression matrix and TF list (example_data folder)
- Base GRN parquet derived from scATAC-seq peaks and TF binding motifs (arrow::read_parquet format)

## Outputs

- Precision-recall / AUPR values for scGATE-ranked edges against each ground-truth network
- Comparative benchmark table of scGATE vs. other GRN-inference tools on the same 217-gene subnetwork (25 TFs, 200 target genes)

## How to apply

First produce the ranked network to be scored: library-size normalize the scRNA-seq matrix with scRNA_seq_preprocessing(library_size_normalization='True', tf_list=tf_names) so expression is rescaled into the (0,1) interval, read the scATAC-seq/motif-derived base GRN parquet (e.g. Buenrostro2018_base_GRN_dataframe.parquet) with arrow::read_parquet and summarize it via read_base_GRN(), then run scGATE_edge(data, base_GRN=candidate_tf_target, h_act=7) with the default 3 EM iterations. Next load the ground-truth networks — for the human haematopoiesis evaluation, the Cus_KO and Cus_Chip networks hosted on the project GitHub; for mouse tissues, GRNs generated from ChIP-Atlas mouse TF ChIP-seq data, excluding datasets with fewer than 50 peaks and non-physiological conditions such as gene knockouts or AAV treatment. Compute network-reconstruction performance metrics (precision–recall/AUPR) on the scGATE-ranked edges against each ground truth, and repeat identically for competing tools on the same subnetwork (the haematopoiesis benchmark used a 217-gene subnetwork of 25 TFs and 200 target genes). Correct application is judged by metrics that reproduce the paper's reported superiority of scGATE over existing tools on human blood data (Buenrostro2018, GSE96772).

## Related tools

- **scGATE** (Source of the Bayes-Factor-ranked TF–target edge list (scGATE_edge mode, 3 EM iterations, h_act=7) that is benchmarked against the ground-truth GRNs) — https://github.com/CompBioIPM/scGATE
- **R 4.1.3** (Environment in which the scGATE pipeline, edge ranking, and benchmark metric computation are executed)
- **arrow** (R package used to read the base GRN .parquet files (e.g. Buenrostro2018_base_GRN_dataframe.parquet) before read_base_GRN() summarization)
- **VGAM** (scGATE dependency for fitting the zero-inflated distributions underlying the Bayes Factor edge scores being benchmarked)
- **truncnorm** (scGATE dependency supporting the truncated-normal components of the expression model behind the ranked edges)

## Examples

```
edges <- scGATE_edge(data = data, base_GRN = candidate_tf_target, h_act = 7); write.csv(edges, 'buenrostro2018_edge_list.csv')
```

## Evaluation signals

- The ranked edge list is saved with the exact schema (columns 'from', 'to', 'BF_score') as buenrostro2018_edge_list.csv
- Preprocessing invariant: scRNA_seq_preprocessing() with library_size_normalization='True' yields counts normalized per cell and rescaled into the (0,1) interval
- scGATE_edge ran with the default 3 EM iterations and h_act=7 on the Buenrostro2018 base GRN parquet
- Benchmark AUPR against both Cus_KO and Cus_Chip ground truths reproduces the reported scGATE performance and its superiority over existing tools on human blood data
- Mouse-tissue ground truths respect the ChIP-Atlas filtering: ChIP-seq datasets with <50 peaks or non-physiological conditions (knockouts, AAV treatment) are excluded

## Limitations

- scGATE is applicable only to a small (<200) subset of genes with a refined candidate-TF list derived from external hints (scATAC-seq/motifs); it cannot be benchmarked fairly on large-scale de novo inference.
- Logic gates are recommended only for subsets of up to k=5 candidate TFs to prevent overfitting, and high-order Boolean logic (>5 regulators) remains intractable.
- Ground-truth networks themselves are incomplete: large protein complex interactions are missed in both the model and the ground truth.
- scGATE does not handle pseudotime-sorted data along differentiation trajectories or temporal delays, constraining which differentiation benchmarks are valid.

## Evidence
