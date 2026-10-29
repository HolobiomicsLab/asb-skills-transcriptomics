---
name: peak-to-gene-regulatory-linking
description: Use when you have scATAC-seq data (e.g. Cusanovich 2018 Spleen, GSE111586)
  plus a TF motif catalogue and need to filter out non-relevant TFs and build a per-target-gene
  candidate TF pool (base GRN parquet) for downstream logic-gate GRN inference.
when_to_use_negative:
- scGATE is applicable only to a small (<200) subset of genes with a refined list
  of candidate regulatory TFs — do not use this base-GRN pipeline for large-scale
  inference from scratch without external hints
- No scATAC-seq or motif data are available for the tissue/cell-type context — the
  base GRN cannot be built and scGATE should be run with base_GRN = NA
- The goal is pseudotime-sorted single-cell data along differentiation trajectories
  or modeling temporal delays — scGATE does not yet handle this
- You expect to decode Boolean logic among more than 5 TFs per target — overfitting
  is a major concern and this candidate-pool construction supports subsets of up to
  k=5
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0203
tools:
- name: scGATE
  role: Provides read_base_GRN() to summarize the peak-based base GRN parquet into
    a TF–target candidate dataframe consumed by downstream gate/edge inference
  repo: https://github.com/CompBioIPM/scGATE
- name: arrow
  role: R package used to load the base GRN parquet file via arrow::read_parquet before
    summarization
  repo: https://github.com/CompBioIPM/scGATE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:5652bb60f33e2f1bbc71ffb595f3cfc36cf2649e7450d793ce8660026a1b46cf
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# peak-to-gene-regulatory-linking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs a scATAC-seq-derived base gene regulatory network: accessible chromatin peaks passing co-accessibility and TSS-proximity criteria are scanned for TF DNA binding motifs and linked to nearby target genes, producing a peak-based TF–gene candidate table (parquet) that scGATE consumes as its refined candidate TF pool per target gene.

## When to use

Use when you have scATAC-seq data (e.g. Cusanovich 2018 Spleen, GSE111586) plus a TF motif catalogue and need to filter out non-relevant TFs and build a per-target-gene candidate TF pool (base GRN parquet) for downstream logic-gate GRN inference.

## When NOT to use

- scGATE is applicable only to a small (<200) subset of genes with a refined list of candidate regulatory TFs — do not use this base-GRN pipeline for large-scale inference from scratch without external hints
- No scATAC-seq or motif data are available for the tissue/cell-type context — the base GRN cannot be built and scGATE should be run with base_GRN = NA
- The goal is pseudotime-sorted single-cell data along differentiation trajectories or modeling temporal delays — scGATE does not yet handle this
- You expect to decode Boolean logic among more than 5 TFs per target — overfitting is a major concern and this candidate-pool construction supports subsets of up to k=5

## Inputs

- scATAC-seq data for the tissue/cell-type context (e.g. Cusanovich 2018 Spleen, GSE111586)
- TF DNA binding motif catalogue / motif scanning results
- Peak coordinates and target gene annotations (for TSS-proximity peak–gene linking)
- Co-accessibility information across cells/clusters

## Outputs

- Peak-level TF–peak assignment table (peaks annotated with TFs whose motifs they contain)
- Peak-based TF–gene candidate table in parquet format (e.g. Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet)
- Summarized TF–target candidate edge dataframe returned by scGATE read_base_GRN()
- Validated parquet file structurally matching the reference base GRN at Zenodo doi:10.5281/zenodo.8353409

## How to apply

Acquire scATAC-seq data for the tissue/cell-type context (e.g. Cusanovich2018 Spleen base GRN intermediates at Zenodo doi:10.5281/zenodo.8353409) and identify accessible chromatin peaks across cells/clusters. Scan TF DNA binding motifs against the accessible peak sequences so each peak is annotated with the TFs whose motifs it contains, yielding a peak-level TF–peak assignment table. Link accessible peaks to nearby target genes to build a peak-based TF–gene candidate dataframe, stored in parquet format analogous to Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet. Summarize this base GRN parquet into a TF–target edge dataframe with the scGATE read_base_GRN() function (after loading with arrow::read_parquet in R 4.1.3). The rationale is that external chromatin-accessibility and motif evidence constrains the combinatorial search to biologically plausible regulators before scRNA-seq-based refinement (e.g. abs_cor Pearson-correlation edge filtration) and Boolean gate fitting.

## Related tools

- **scGATE** (Provides read_base_GRN() to summarize the peak-based base GRN parquet into a TF–target candidate dataframe consumed by downstream gate/edge inference) — https://github.com/CompBioIPM/scGATE
- **arrow** (R package used to load the base GRN parquet file via arrow::read_parquet before summarization) — https://github.com/CompBioIPM/scGATE

## Examples

```
candidate_tf_target <- as.data.frame(read_parquet("Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet")); candidate_tf_target <- read_base_GRN(candidate_tf_target)
```

## Evaluation signals

- The produced parquet file must load successfully with arrow::read_parquet in R 4.1.3
- read_base_GRN() must return a TF–target candidate list whose gene identifiers are consistent with the paired scRNA-seq expression matrix
- The TF–target pairs should structurally match the reference base GRN parquet deposited at Zenodo doi:10.5281/zenodo.8353409 (e.g. Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet)
- Candidate TF subsets per target should be practical for gate fitting (scGATE recommends subsets with up to k=5 factors) to prevent overfitting
- Downstream abs_cor filtration should be able to further remove edges with low absolute Pearson correlations between TFs and targets

## Limitations

- scGATE is currently applicable to a rather small (<200) subset of genes with a refined list of candidate regulatory TFs based on external hints, not large-scale inference from scratch
- Decoding high-order Boolean logic involving more than 5 transcriptional regulators remains challenging due to overfitting
- Large protein complex interactions are missed both in the current model and in the ground-truth networks
- Context specificity depends on the quality of external cues (scATAC-seq and motifs); no software version numbers are stated for tools used in the base GRN stage

## Evidence

- [methods] The base GRN stage identifies scATAC-seq peaks meeting co-accessibility and TSS-proximity criteria, then scans those peak DNA sequences for TF binding motifs: [evidence span withheld — anchor status: not_found]
