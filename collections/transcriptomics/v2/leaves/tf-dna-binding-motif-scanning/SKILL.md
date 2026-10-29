---
name: tf-dna-binding-motif-scanning
description: Use when you have scATAC-seq peaks linked to target genes and need to
  filter the universe of TFs down to a per-target-gene candidate pool before inferring
  a TF–gene network (e.g. as the base GRN input for scGATE).
when_to_use_negative:
- You already have a curated base GRN parquet from external hints — skip motif scanning
  and go straight to read_base_GRN() summarization.
- 'You need large-scale de novo inference over many genes: scGATE with its refined
  candidate TFs applies only to a small (<200) subset of genes, and >5 TFs per logic
  gate is impractical.'
- Your goal is predicting logic gates without regulatory context — leave base_GRN
  empty rather than building a motif-based one.
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scGATE
  role: Provides read_base_GRN() to read and summarize the motif-derived peak-based
    base GRN parquet into a TF–target candidate list
  repo: https://github.com/CompBioIPM/scGATE
- name: arrow
  role: R package used to load the base GRN parquet file via arrow::read_parquet before
    summarization
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:6fe49a360736accc61d705d1f22bf3d8813c581872229b895c36a23e3982ffba
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
  - arrow
  - scGATE
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# TF DNA binding motif scanning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scans accessible chromatin peak sequences from scATAC-seq data for TF DNA binding motifs to assign candidate regulatory TFs to each peak, producing a peak-based TF–gene candidate table (base GRN parquet) that scGATE consumes as its refined candidate TF pool.

## When to use

Use when you have scATAC-seq peaks linked to target genes and need to filter the universe of TFs down to a per-target-gene candidate pool before inferring a TF–gene network (e.g. as the base GRN input for scGATE).

## When NOT to use

- You already have a curated base GRN parquet from external hints — skip motif scanning and go straight to read_base_GRN() summarization.
- You need large-scale de novo inference over many genes: scGATE with its refined candidate TFs applies only to a small (<200) subset of genes, and >5 TFs per logic gate is impractical.
- Your goal is predicting logic gates without regulatory context — leave base_GRN empty rather than building a motif-based one.

## Inputs

- scATAC-seq data for the tissue/cell-type context (e.g. Cusanovich 2018 Spleen peaks, GSE111586)
- TF DNA binding motif library
- Peak-to-gene linkage (co-accessibility and TSS-proximity criteria)
- Raw scRNA-seq expression matrix with gene identifiers matching the base GRN (e.g. Tabula_Muris2018_Spleen-10X_P4_7_ExpressionData.csv)

## Outputs

- Peak-level TF–peak motif assignment table
- Base GRN parquet file of candidate TF–gene pairs (e.g. Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet)
- Summarized TF–target candidate dataframe from read_base_GRN()

## How to apply

Acquire the scATAC-seq data for the tissue/cell-type context (e.g. Cusanovich 2018 Spleen, GSE111586) and identify accessible chromatin peaks across cells/clusters, keeping peaks that meet co-accessibility and TSS-proximity criteria relative to the target gene. Scan the DNA sequences of those peaks against a library of TF DNA binding motifs so each peak is annotated with the TFs whose motifs it contains. Link the motif-annotated accessible peaks to their nearby target genes to build a peak-based TF–gene candidate dataframe, and export it in parquet format (analogous to Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet). Downstream, summarize the parquet into a TF–target candidate edge list with scGATE's read_base_GRN() (loaded via arrow::read_parquet in R 4.1.3), so scGATE_logic()/scGATE_edge() operate only on motif-supported TF–target pairs.

## Related tools

- **scGATE** (Provides read_base_GRN() to read and summarize the motif-derived peak-based base GRN parquet into a TF–target candidate list) — https://github.com/CompBioIPM/scGATE
- **arrow** (R package used to load the base GRN parquet file via arrow::read_parquet before summarization)

## Examples

```
candidate_tf_target <- as.data.frame(read_parquet("/example_data/Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet")); candidate_tf_target <- read_base_GRN(candidate_tf_target)
```

## Evaluation signals

- The produced parquet file loads successfully with arrow::read_parquet.
- read_base_GRN() returns a TF–target candidate list whose gene identifiers are consistent with the paired scRNA-seq expression matrix.
- TF–target pairs structurally match the reference base GRN parquet deposited at Zenodo doi:10.5281/zenodo.8353409 (e.g. Buenrostro2018_base_GRN_dataframe.parquet).
- Every retained candidate TF for a target gene is supported by a motif instance within an accessible peak linked to that gene (co-accessibility + TSS-proximity criteria).
- Rounds of downstream filtering with abs_cor in (0,1) remove TF–target edges with low absolute Pearson correlation, confirming the candidate pool feeds correctly into scGATE_edge().

## Limitations

- This filtering stage only narrows candidates; scGATE is currently applicable to a rather small (<200) subset of genes with a refined list of candidate regulatory TFs based on external hints.
- Decoding high-order Boolean logic involving more than 5 transcriptional regulators remains challenging due to overfitting — keep candidate subsets small (k<=5).
- The base GRN must already exist in .parquet format from external hints like scATAC-seq and TF binding motif analyses; scGATE itself does not generate it.

## Evidence

- [results] The base GRN stage identifies scATAC-seq peaks meeting co-accessibility and TSS-proximity criteria: [evidence span withheld — anchor status: not_found]
