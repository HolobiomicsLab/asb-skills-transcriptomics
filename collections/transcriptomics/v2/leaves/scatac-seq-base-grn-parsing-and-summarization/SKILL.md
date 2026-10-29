---
name: scatac-seq-base-grn-parsing-and-summarization
description: Use when a base GRN built from scATAC-seq data and TF binding motif analysis
  (a .parquet file of candidate TF–target links, e.g. Cusanovich2018_Spleen or Buenrostro2018)
  must be loaded and summarized before running scGATE on scRNA-seq counts.
when_to_use_negative:
- No base GRN from external hints (scATAC-seq/motifs) is available — read_base_GRN()
  is not needed and scGATE can be run with base_GRN left empty (NA), though the README
  notes scGATE_gate() is more suitable for small networks or when a base GRN is available
- The task is large-scale de novo GRN inference across thousands of genes without
  a refined candidate TF list — the article states scGATE is currently applicable
  to a rather small (<200) subset of genes with a refined list of candidate regulatory
  TFs
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGATE
  role: Provides read_base_GRN() to summarize the scATAC-seq/motif-derived base GRN,
    and scGATE_edge()/scGATE_logic() that consume the summarized candidate_tf_target
    as base_GRN input
  repo: github.com/CompBioIPM/scGATE
- name: arrow
  role: Reads the .parquet base GRN file into an R data.frame prior to read_base_GRN()
    summarization (install.packages("arrow"))
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:7d91d1614bc01060181271e9326d6a8d2193e9bb1509d3ba0789af0d4537fea6
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
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# scatac-seq-base-grn-parsing-and-summarization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parses a base gene regulatory network (GRN) derived from scATAC-seq peaks and TF DNA binding motifs — supplied as a .parquet dataframe (e.g. Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet) — using arrow::read_parquet() and scGATE's read_base_GRN() summarization, so the refined candidate TF–target pairs can be passed as the base_GRN argument to scGATE_edge()/scGATE_logic() for context-specific network inference.

## When to use

Use when a base GRN built from scATAC-seq data and TF binding motif analysis (a .parquet file of candidate TF–target links, e.g. Cusanovich2018_Spleen or Buenrostro2018) must be loaded and summarized before running scGATE on scRNA-seq counts.

## When NOT to use

- No base GRN from external hints (scATAC-seq/motifs) is available — read_base_GRN() is not needed and scGATE can be run with base_GRN left empty (NA), though the README notes scGATE_gate() is more suitable for small networks or when a base GRN is available
- The task is large-scale de novo GRN inference across thousands of genes without a refined candidate TF list — the article states scGATE is currently applicable to a rather small (<200) subset of genes with a refined list of candidate regulatory TFs

## Inputs

- .parquet base GRN dataframe derived from scATAC-seq peaks and TF DNA binding motifs (e.g. Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet, Buenrostro2018_base_GRN_dataframe.parquet)
- Raw scRNA-seq count matrix with cells in rows and genes in columns (e.g. Tabula_Muris2018_Spleen-10X_P4_7_ExpressionData.csv)
- Optional TF candidate list (e.g. Tabula_Muris2018_Spleen-10X_P4_7_tf_lists.txt)

## Outputs

- Summarized candidate TF–target dataframe (candidate_tf_target) usable as the base_GRN argument of scGATE_edge()/scGATE_logic()
- Downstream BF-ranked directed TF–gene network from scGATE_edge() (ranked_edge_list)

## How to apply

Read the .parquet base GRN file with the arrow package via read_parquet() and coerce it to a data.frame (e.g. candidate_tf_target <- as.data.frame(read_parquet("Buenrostro2018_base_GRN_dataframe.parquet"))). Summarize it with scGATE's read_base_GRN(candidate_tf_target), which condenses the scATAC-seq/motif-derived candidate TF–target pairs into the format scGATE expects. Optionally intersect candidate TFs with a TF list file such as Tabula_Muris2018_Spleen-10X_P4_7_tf_lists.txt, since scGATE recommends a small, pre-selected TF set based on prior biological knowledge. Pass the summarized object as base_GRN to scGATE_edge(data, base_GRN = candidate_tf_target, h_act = 7) with the default number_of_em_iterations = 3 and max_num_regulators = 3, after preprocessing the scRNA-seq counts with scRNA_seq_preprocessing(data, library_size_normalization="True", tf_list=tf_names). Evaluate the resulting BF-ranked directed network against ChIP-Atlas-derived ground-truth GRNs per cell type (AUROC, EPR), expecting on the Tabula Muris Spleen benchmark that scGATE wins 7/8 datasets and that top edges match reference values such as Ctcf→Rps19 with BF ≈ 2002.419.

## Related tools

- **scGATE** (Provides read_base_GRN() to summarize the scATAC-seq/motif-derived base GRN, and scGATE_edge()/scGATE_logic() that consume the summarized candidate_tf_target as base_GRN input) — github.com/CompBioIPM/scGATE
- **arrow** (Reads the .parquet base GRN file into an R data.frame prior to read_base_GRN() summarization (install.packages("arrow")))

## Examples

```
candidate_tf_target <- as.data.frame(read_parquet("Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet")); candidate_tf_target <- read_base_GRN(candidate_tf_target); edges <- scGATE_edge(data = data, base_GRN = candidate_tf_target, h_act = 7)
```

## Evaluation signals

- The parquet loads via arrow::read_parquet() and read_base_GRN() returns a summarized candidate_tf_target dataframe accepted directly by scGATE_edge() without errors
- On the Tabula Muris Spleen-10X_P4_7 run, the head of ranked_edge_list matches the reference output: Ctcf→Rps19 BF ≈ 2002.419, Batf→Rps19 BF ≈ 2001.388, Stat5b→Rplp0 BF ≈ 1840.046
- The computed average number of regulating TFs per target gene reproduces the reported ~3.39 on the Tabula Muris benchmark
- The final network scores against ChIP-Atlas-derived ground-truth GRNs at the reported benchmark level (AUROC and EPR wins in 7 of 8 scRNA-seq datasets)
- Candidate TF pairs in the summarized base GRN are consistent with the supplied TF list (tf_list argument of scRNA_seq_preprocessing()) rather than including all data columns as TFs

## Limitations

- scGATE is currently applicable only to a rather small (<200) subset of genes with a refined list of candidate regulatory TFs based on external hints; decoding high-order Boolean logic involving >5 regulators is challenging due to overfitting
- The base GRN must have been generated beforehand from external hints (scATAC-seq plus TF binding motif analyses); read_base_GRN() does not create it
- Interactions such as those in large protein complexes will be missed both in the scGATE model and in the ChIP-Atlas-derived ground-truth networks used for evaluation

## Evidence
