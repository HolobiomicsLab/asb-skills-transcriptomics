---
name: candidate-tf-pool-filtering
description: Use when scRNA-seq data alone must be augmented with context-specific
  regulatory hints — specifically when scATAC-seq peak data and TF motif annotations
  are available and a filtered candidate TF–target table is needed before logic-gate
  inference.
when_to_use_negative:
- No scATAC-seq data or TF motif annotations exist for the context — read_base_GRN()
  can be skipped and base_GRN left empty (NA), but the filtering skill itself cannot
  be applied
- Goal is large-scale de novo GRN inference over thousands of genes — scGATE with
  a refined candidate TF list is applicable only to a rather small (<200) subset of
  genes
- Target genes lack nearby accessible peaks or motif matches — the filtering would
  yield an empty candidate TF pool for those genes
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: scGATE
  role: Provides read_base_GRN() to summarize the parquet base GRN into a TF–target
    candidate list; scGATE_0.1.0.tar.gz installed from source in R
  repo: https://github.com/CompBioIPM/scGATE
- name: arrow
  role: R package used to read the peak-based base GRN parquet file via read_parquet()
- name: R
  role: Runtime environment (R 4.1.3) in which the scGATE package and arrow are used
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:4ca5e01f688d3464df41f1ea7ab86855b713c7dcd9bfe3f9110d7160de35df92
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
  - arrow
  - scGATE
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# candidate-tf-pool-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds the refined per-target-gene candidate TF pool that scGATE consumes by intersecting scATAC-seq accessible peaks (co-accessibility and TSS-proximity criteria) with TF DNA binding motif scans, producing a peak-based TF–gene base GRN in parquet format that is summarized with read_base_GRN().

## When to use

Use when scRNA-seq data alone must be augmented with context-specific regulatory hints — specifically when scATAC-seq peak data and TF motif annotations are available and a filtered candidate TF–target table is needed before logic-gate inference.

## When NOT to use

- No scATAC-seq data or TF motif annotations exist for the context — read_base_GRN() can be skipped and base_GRN left empty (NA), but the filtering skill itself cannot be applied
- Goal is large-scale de novo GRN inference over thousands of genes — scGATE with a refined candidate TF list is applicable only to a rather small (<200) subset of genes
- Target genes lack nearby accessible peaks or motif matches — the filtering would yield an empty candidate TF pool for those genes

## Inputs

- scATAC-seq data for the tissue/cell-type context (e.g. Cusanovich2018 Spleen, GSE111586) with accessible chromatin peaks
- TF DNA binding motif database for scanning peak sequences
- Peak-based TF–gene candidate base GRN dataframe in .parquet format (e.g. Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet)
- Paired scRNA-seq expression matrix (e.g. Tabula_Muris2018_Spleen-10X_P4_7_ExpressionData.csv) for gene-identifier consistency checking

## Outputs

- Per-target-gene candidate regulatory TF list (refined TF pool) consumed by scGATE_logic()/scGATE_edge()
- Summarized TF–target edge dataframe produced by read_base_GRN()
- Base GRN parquet file structurally matching the reference at Zenodo doi:10.5281/zenodo.8353409

## How to apply

First acquire the scATAC-seq data for the matching tissue/cell-type context (e.g. Cusanovich 2018 Spleen, GSE111586) and identify accessible chromatin peaks across cells/clusters. Scan TF DNA binding motifs against those peak sequences so each peak is annotated with the TFs whose motifs it contains, yielding a peak-level TF–peak assignment table. Link accessible peaks to nearby target genes — using the co-accessibility and TSS-proximity criteria applied in the base GRN stage — to build a peak-based TF–gene candidate table in parquet format (analogous to Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet). In R, load the parquet with arrow::read_parquet and summarize it into a TF–target edge dataframe with scGATE's read_base_GRN() function; this summarized base GRN is then passed as the base_GRN argument to scGATE_logic()/scGATE_edge(). Validate that the parquet loads with arrow::read_parquet, that gene identifiers are consistent with the paired scRNA-seq expression matrix, and that TF–target pairs structurally match the reference base GRN parquet deposited at Zenodo doi:10.5281/zenodo.8353409.

## Related tools

- **scGATE** (Provides read_base_GRN() to summarize the parquet base GRN into a TF–target candidate list; scGATE_0.1.0.tar.gz installed from source in R) — https://github.com/CompBioIPM/scGATE
- **arrow** (R package used to read the peak-based base GRN parquet file via read_parquet())
- **R** (Runtime environment (R 4.1.3) in which the scGATE package and arrow are used)

## Examples

```
candidate_tf_target <- as.data.frame(read_parquet("Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet")); candidate_tf_target <- read_base_GRN(candidate_tf_target)
```

## Evaluation signals

- The produced parquet file loads successfully with arrow::read_parquet()
- read_base_GRN() returns a TF–target candidate list whose gene identifiers are consistent with the paired scRNA-seq expression matrix
- TF–target pairs structurally match the reference base GRN parquet deposited at Zenodo doi:10.5281/zenodo.8353409
- Downstream scGATE_logic()/scGATE_edge() runs accept the summarized dataframe as the base_GRN argument without identifier errors
- Filtered TF pool is limited to TFs whose motifs are present in accessible peaks linked to each target gene (non-relevant TFs removed)

## Limitations

- scGATE is currently applicable to a rather small (<200) subset of genes with a refined list of candidate regulatory TFs based on external hints — not large-scale inference from scratch
- scGATE must be applied to a small pre-selected set of TFs based on prior biological knowledge
- The base GRN must already exist in .parquet format from external hints like scATAC-seq and TF binding motif analyses; this skill consumes rather than generates raw peak calls in the scGATE pipeline
- Decoding high-order Boolean logic involving more than 5 transcriptional regulators remains challenging due to overfitting, so candidate TF subsets should stay within the recommended k<=5

## Evidence
