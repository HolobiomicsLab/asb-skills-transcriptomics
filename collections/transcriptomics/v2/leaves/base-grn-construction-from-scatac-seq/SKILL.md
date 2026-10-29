---
name: base-grn-construction-from-scatac-seq
description: 'Use when preparing scGATE inputs from paired scRNA-seq + scATAC-seq
  data: you need a base GRN parquet (e.g. Buenrostro2018_base_GRN_dataframe.parquet)
  of candidate TF–target edges derived from scATAC-seq peaks and TF motifs before
  running scGATE_edge()/scGATE_gate().'
when_to_use_negative:
- You have no paired scATAC-seq data or TF motif annotations and must rely on prior
  biological knowledge to pre-select TFs — scGATE then runs without a base GRN (base_GRN
  = NA).
- 'You need large-scale network inference from scratch: the article notes scGATE is
  applicable only to a small (<200) subset of genes with a refined candidate TF list
  derived from external hints.'
- Pseudotime-sorted single-cell data along differentiation trajectories — scGATE does
  not yet handle temporal ordering or delays.
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0204
tools:
- name: scGATE
  role: provides read_base_GRN() to summarize the parquet base GRN into a TF–target
    candidate dataframe used as base_GRN input
  repo: https://github.com/CompBioIPM/scGATE
- name: arrow
  role: R package used to load the base GRN parquet file via read_parquet()
- name: R 4.1.3
  role: runtime environment in which scGATE and the parquet-reading workflow execute
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:45305b406a85dcb3e3abcf5015416202587626ba6b539d86b89a0a8c23d7795e
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
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# base-grn-construction-from-scatac-seq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs the per-target-gene candidate TF pool (base GRN) that scGATE consumes by identifying accessible chromatin peaks meeting co-accessibility and TSS-proximity criteria in scATAC-seq data, scanning peak DNA sequences for TF binding motifs, and linking peaks to nearby genes, stored as a parquet TF–target dataframe.

## When to use

Use when preparing scGATE inputs from paired scRNA-seq + scATAC-seq data: you need a base GRN parquet (e.g. Buenrostro2018_base_GRN_dataframe.parquet) of candidate TF–target edges derived from scATAC-seq peaks and TF motifs before running scGATE_edge()/scGATE_gate().

## When NOT to use

- You have no paired scATAC-seq data or TF motif annotations and must rely on prior biological knowledge to pre-select TFs — scGATE then runs without a base GRN (base_GRN = NA).
- You need large-scale network inference from scratch: the article notes scGATE is applicable only to a small (<200) subset of genes with a refined candidate TF list derived from external hints.
- Pseudotime-sorted single-cell data along differentiation trajectories — scGATE does not yet handle temporal ordering or delays.

## Inputs

- scATAC-seq data for the tissue/cell-type context (e.g. Cusanovich2018 Spleen, GSE111586)
- TF DNA binding motif databases for scanning accessible peak sequences
- paired scRNA-seq expression matrix (e.g. Tabula_Muris2018_Spleen-10X_P4_7_ExpressionData.csv) for gene-identifier consistency
- intermediate/base GRN parquet file (e.g. Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet, Buenrostro2018_base_GRN_dataframe.parquet)

## Outputs

- parquet peak-based TF–gene candidate table (base GRN dataframe)
- TF–target candidate list/edge dataframe returned by read_base_GRN(), consumable as scGATE base_GRN argument

## How to apply

Acquire the tissue-matched scATAC-seq data (e.g. Cusanovich 2018 Spleen, GSE111586; base GRN intermediates also at Zenodo doi:10.5281/zenodo.8353409) and identify accessible chromatin peaks across cells/clusters. Scan TF DNA binding motifs against the accessible peaks so each peak is annotated with the TFs whose motifs it contains, yielding a peak-level TF–peak assignment table. Link accessible peaks to their nearby target genes to build a peak-based TF–gene candidate table (e.g. Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet) with peaks meeting co-accessibility and TSS-proximity criteria. In R (version 4.1.3), load the parquet with arrow::read_parquet and summarize it into a TF–target edge dataframe using the scGATE package's read_base_GRN() function; this summarized candidate_tf_target then feeds scGATE_edge() or scGATE_logic().

## Related tools

- **scGATE** (provides read_base_GRN() to summarize the parquet base GRN into a TF–target candidate dataframe used as base_GRN input) — https://github.com/CompBioIPM/scGATE
- **arrow** (R package used to load the base GRN parquet file via read_parquet())
- **R 4.1.3** (runtime environment in which scGATE and the parquet-reading workflow execute)

## Examples

```
candidate_tf_target <- as.data.frame(read_parquet("Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet")); candidate_tf_target <- read_base_GRN(candidate_tf_target)
```

## Evaluation signals

- The produced parquet file loads with arrow::read_parquet.
- read_base_GRN() returns a TF–target candidate list whose gene identifiers are consistent with the paired scRNA-seq expression matrix (e.g. Tabula_Muris2018_Spleen-10X_P4_7_ExpressionData.csv).
- TF–target pairs structurally match the reference base GRN parquet deposited at Zenodo doi:10.5281/zenodo.8353409.
- Only TFs whose motifs occur in accessible peaks passing co-accessibility and TSS-proximity criteria appear as candidate regulators for each target gene.
- Downstream scGATE_edge()/scGATE_logic() accepts the candidate_tf_target dataframe as base_GRN without identifier errors.

## Limitations

- Base GRN construction requires scATAC-seq data matched to the tissue/cell-type context; without external hints, TFs must be pre-selected on prior biological knowledge.
- Downstream scGATE decoding is limited to logic gates among subsets of fewer than 5 TFs to avoid overfitting; >5 regulators remains challenging.
- Base GRN filtering is a hint, not ground truth: edges are additionally pruned by absolute Pearson correlation (abs_cor parameter) between TFs and targets.

## Evidence

- [full_text] The base GRN stage identifies scATAC-seq peaks meeting co-accessibility and TSS-proximity criteria, then scans those peak DNA sequences for TF binding motifs: [evidence span withheld — anchor status: not_found]
- [abstract] we have applied assay for transposase-accessible chromatin with sequencing (scATAC-seq) data and TF DNA binding motifs to filter out non-relevant TFs in gene regulations: "We have applied assay for transposase-accessible chromatin with sequencing (scATAC-seq) data and TF DNA binding motifs to filter out non-relevant TFs in gene regulations"
- [readme] candidate_tf_target <- as.data.frame(read_parquet("/example_data/Buenrostro2018_base_GRN_dataframe.parquet")): "candidate_tf_target <- as.data.frame(read_parquet("Buenrostro2018_base_GRN_dataframe.parquet"))
candidate_tf_target <- read_base_GRN(candidate_tf_target)"
- [discussion] scGATE is currently applicable to a rather small (<200) subset of genes with a refined list of candidate regulatory TFs based on external hints: "scGATE is currently applicable to a rather small (<200) subset of genes with a refined list of candidate regulatory TFs based on external hints"
