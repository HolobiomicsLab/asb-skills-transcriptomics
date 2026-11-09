---
name: scatac-seq-accessible-peak-identification
description: Use when scRNA-seq data must be paired with scATAC-seq peaks and TF motif
  scans to filter out non-relevant TFs and construct a base GRN parquet (e.g. Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet)
  for downstream scGATE logic-gate inference.
when_to_use_negative:
- scGATE downstream inference is currently applicable only to a rather small (<200)
  subset of genes with a refined list of candidate regulatory TFs — this skill is
  not for large-scale de novo GRN inference from scratch
- The data lacks paired scATAC-seq / motif evidence for the tissue context, so no
  external-hint base GRN can be generated (scGATE currently must be applied to a small
  pre-selected set of TFs based on prior biological knowledge)
- Gene identifiers in the parquet base GRN cannot be made consistent with the paired
  scRNA-seq expression matrix
edam_operation: http://edamontology.org/operation_0362
edam_topics: []
tools:
- name: scGATE
  role: Consumes the parquet base GRN via read_base_GRN() to summarize it into the
    per-target-gene candidate TF pool for logic-gate inference
  repo: github.com/CompBioIPM/scGATE
- name: arrow
  role: R package used to load the parquet base GRN file with read_parquet()
- name: R
  role: Runtime (version 4.1.3) in which scGATE and arrow are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:89d753667d7461d067afee52ef3de67ba701db406d9f216edf63f86532253d0f
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
  - arrow
  - scGATE
  edam_operation: http://edamontology.org/operation_0362
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# scatac-seq-accessible-peak-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify accessible chromatin peaks from scATAC-seq, link them to nearby target genes, and annotate them with TF DNA binding motifs to build the per-target-gene candidate TF pool (base GRN parquet) that scGATE consumes as its refined TF list.

## When to use

Use when scRNA-seq data must be paired with scATAC-seq peaks and TF motif scans to filter out non-relevant TFs and construct a base GRN parquet (e.g. Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet) for downstream scGATE logic-gate inference.

## When NOT to use

- scGATE downstream inference is currently applicable only to a rather small (<200) subset of genes with a refined list of candidate regulatory TFs — this skill is not for large-scale de novo GRN inference from scratch
- The data lacks paired scATAC-seq / motif evidence for the tissue context, so no external-hint base GRN can be generated (scGATE currently must be applied to a small pre-selected set of TFs based on prior biological knowledge)
- Gene identifiers in the parquet base GRN cannot be made consistent with the paired scRNA-seq expression matrix

## Inputs

- scATAC-seq data for the tissue/cell-type context (e.g. Cusanovich 2018 Spleen, GSE111586; Buenrostro 2018, GSE96772)
- TF DNA binding motif database
- Paired scRNA-seq expression matrix (e.g. Tabula Muris Spleen-10X_P4_7 ExpressionData.csv) for gene-identifier consistency

## Outputs

- Peak-level TF–peak assignment table (peaks annotated with TFs whose motifs they contain)
- Peak-based TF–gene candidate table in parquet format (e.g. Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet)
- Summarized TF–target candidate edge dataframe returned by scGATE read_base_GRN()

## How to apply

Acquire scATAC-seq data for the tissue/cell-type context (e.g. Cusanovich 2018 Spleen from GSE111586, with base GRN intermediates at Zenodo doi:10.5281/zenodo.8353409) and identify accessible chromatin peaks across cells/clusters. Scan TF DNA binding motifs against those peak DNA sequences so each peak is annotated with the TFs whose motifs it contains, yielding a peak-level TF–peak assignment table. Link accessible peaks to their nearby target genes — keeping only peaks meeting co-accessibility and TSS-proximity criteria — to build a peak-based TF–gene candidate table in parquet format (analogous to Buenrostro2018_base_GRN_dataframe.parquet). Summarize the parquet into a TF–target edge dataframe with scGATE's read_base_GRN(), loading via arrow::read_parquet in R 4.1.3; downstream scGATE further removes edges with low absolute Pearson correlations (abs_cor parameter).

## Related tools

- **scGATE** (Consumes the parquet base GRN via read_base_GRN() to summarize it into the per-target-gene candidate TF pool for logic-gate inference) — github.com/CompBioIPM/scGATE
- **arrow** (R package used to load the parquet base GRN file with read_parquet())
- **R** (Runtime (version 4.1.3) in which scGATE and arrow are executed)

## Examples

```
candidate_tf_target <- as.data.frame(read_parquet("Buenrostro2018_base_GRN_dataframe.parquet")); candidate_tf_target <- read_base_GRN(candidate_tf_target)
```

## Evaluation signals

- The produced parquet base GRN file loads successfully with arrow::read_parquet in R
- read_base_GRN() returns a TF–target candidate list whose gene identifiers are consistent with the paired scRNA-seq expression matrix
- The TF–target pairs structurally match the reference base GRN parquet deposited at Zenodo doi:10.5281/zenodo.8353409 (e.g. Buenrostro2018_base_GRN_dataframe.parquet, Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet)
- Included scATAC-seq peaks satisfy co-accessibility and TSS-proximity criteria before motif scanning
- Downstream scGATE accepts the summarized base_GRN without erroring in scGATE_logic()/scGATE_edge() runs

## Limitations

- scGATE is currently applicable only to a rather small (<200) subset of genes with a refined list of candidate regulatory TFs based on external hints — not large-scale inference from scratch
- Decoding high-order Boolean logic involving more than 5 transcriptional regulators remains challenging due to overfitting; the peak-derived candidate pool should be kept small
- No software version numbers beyond scGATE 0.1.0 / R 4.1.3 are stated for the ATAC/motif processing steps in this section

## Evidence
