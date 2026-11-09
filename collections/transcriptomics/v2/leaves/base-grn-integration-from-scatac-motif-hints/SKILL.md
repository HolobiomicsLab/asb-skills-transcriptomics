---
name: base-grn-integration-from-scatac-motif-hints
description: Use when scRNA-seq expression data must be combined with a scATAC-seq/motif-derived
  base GRN (.parquet) to filter candidate TF-target pairs prior to directed TF-gene
  network or Boolean logic gate inference with scGATE.
when_to_use_negative:
- No external scATAC-seq/motif-derived base GRN exists and the TF set has not been
  pre-selected from prior biological knowledge — scGATE logic inference is only reliable
  on a small (<200) gene subset with refined candidate TFs
- The goal is decoding Boolean logic among more than 5 regulators per target — overfitting
  is a major concern beyond k=5 candidate TFs
- The expression data are pseudotime-sorted along differentiation trajectories or
  require modeling of temporal delays — scGATE does not handle these
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: scGATE
  role: Provides read_base_GRN(), scRNA_seq_preprocessing(), scGATE_edge() and scGATE_logic()
    functions that consume the scATAC/motif-derived base GRN and infer the directed
    TF-target network
  repo: https://github.com/CompBioIPM/scGATE
- name: arrow
  role: R package used to read the base GRN .parquet files via read_parquet()
- name: VGAM
  role: scGATE dependency used in fitting the zero-inflated distribution over logic
    combinations
- name: truncnorm
  role: scGATE dependency for the truncated-normal components of the expression likelihood
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:b41921c0fab52bbb6fc69c13d399114253a2d608176892e612c5d1c12f4b71e1
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# base-grn-integration-from-scatac-motif-hints

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates a base gene regulatory network derived from external hints (scATAC-seq peak accessibility plus TF DNA binding motifs, supplied as .parquet dataframes) with scRNA-seq data to narrow candidate TFs per target gene before scGATE network inference. This filtering step restricts scGATE to a small pre-selected TF set, which is required for reliable Boolean logic gate inference.

## When to use

Use when scRNA-seq expression data must be combined with a scATAC-seq/motif-derived base GRN (.parquet) to filter candidate TF-target pairs prior to directed TF-gene network or Boolean logic gate inference with scGATE.

## When NOT to use

- No external scATAC-seq/motif-derived base GRN exists and the TF set has not been pre-selected from prior biological knowledge — scGATE logic inference is only reliable on a small (<200) gene subset with refined candidate TFs
- The goal is decoding Boolean logic among more than 5 regulators per target — overfitting is a major concern beyond k=5 candidate TFs
- The expression data are pseudotime-sorted along differentiation trajectories or require modeling of temporal delays — scGATE does not handle these

## Inputs

- Base GRN dataframe in .parquet format derived from scATAC-seq peaks and TF binding motifs (e.g. Buenrostro2018_base_GRN_dataframe.parquet)
- Processed scRNA-seq expression matrix (cells in rows, genes in columns)
- Optional TF list (tf_names) to restrict candidate regulators

## Outputs

- Summarized candidate TF-target pair list (candidate_tf_target) consumed by scGATE_edge()/scGATE_logic()
- Ranked directed TF-target edge list with columns 'from', 'to', 'BF_score' (e.g. buenrostro2018_edge_list.csv)

## How to apply

Read the base GRN file with arrow::read_parquet (e.g. 'Buenrostro2018_base_GRN_dataframe.parquet' or 'Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet'), then summarize candidate TF-target pairs via scGATE's read_base_GRN() function. Optionally restrict the expression matrix to a curated TF list by passing tf_list to scRNA_seq_preprocessing() (set library_size_normalization='True' for raw UMI counts, 'False' if normalization was already applied upstream). Apply Pearson-correlation edge filtration through the abs_cor parameter (range 0-1, default 0 = no filtration) to further remove edges with low absolute TF-target correlation. Feed the resulting candidate_tf_target into scGATE_edge(data, base_GRN = candidate_tf_target, h_act = 7) for edge-mode inference or scGATE_logic()/scGATE_gate() for logic-gate mode; scGATE_gate() is preferred when the base GRN is available from external sources such as scATAC-seq and TF motif data. When building ground-truth GRNs from ChIP-seq (e.g. ChIP-Atlas), exclude data with fewer than 50 peaks and non-physiological conditions such as gene knockouts or AAV treatment.

## Related tools

- **scGATE** (Provides read_base_GRN(), scRNA_seq_preprocessing(), scGATE_edge() and scGATE_logic() functions that consume the scATAC/motif-derived base GRN and infer the directed TF-target network) — https://github.com/CompBioIPM/scGATE
- **arrow** (R package used to read the base GRN .parquet files via read_parquet())
- **VGAM** (scGATE dependency used in fitting the zero-inflated distribution over logic combinations)
- **truncnorm** (scGATE dependency for the truncated-normal components of the expression likelihood)

## Examples

```
candidate_tf_target <- as.data.frame(read_parquet("Buenrostro2018_base_GRN_dataframe.parquet")); candidate_tf_target <- read_base_GRN(candidate_tf_target); edges <- scGATE_edge(data = data, base_GRN = candidate_tf_target, h_act = 7)
```

## Evaluation signals

- The .parquet base GRN loads without error and read_base_GRN() returns a summarized TF-target pair table suitable as base_GRN input
- Expression matrix values fall within the (0,1) interval after scRNA_seq_preprocessing() rescaling
- Ranked edge list contains the expected schema ('from', 'to', 'BF_score') and BF scores are positive for retained edges
- On Buenrostro2018, benchmark metrics (AUROC/AUPRC) computed against Cus_KO and Cus_Chip ground-truth networks reproduce the reported scGATE performance
- Abs_cor filtration removes only edges with low absolute Pearson TF-target correlation; setting abs_cor = 0 leaves the edge set unchanged

## Limitations

- scGATE is currently applicable to a rather small (<200) subset of genes with a refined list of candidate regulatory TFs, not to large-scale inference from scratch
- Large protein complex interactions are missed both in the current model and in the ground-truth networks
- Data quality of the base GRN depends on external hints; ground truth built from ChIP-Atlas required excluding ChIP-seq datasets with fewer than 50 peaks or non-physiological conditions (knockouts, AAV treatment)

## Evidence

- [abstract] We have applied assay for transposase-accessible chromatin with sequencing (scATAC-seq) data and TF DNA binding motifs to filter out non-relevant TFs in gene regulations: "We have applied assay for transposase-accessible chromatin with sequencing (scATAC-seq) data and TF DNA binding motifs to filter out non-relevant TFs in gene regulations"
- [readme] read_base_GRN parquet: "candidate_tf_target <- as.data.frame(read_parquet("Buenrostro2018_base_GRN_dataframe.parquet"))
candidate_tf_target <- read_base_GRN(candidate_tf_target)"
- [readme] abs_cor description: "abs_cor                 This parameter varies in the (0, 1) interval and further removes edges with low absolute Pearson correlations between TFs and their targets."
- [results] ChIP-Atlas exclusion: "Data with fewer than 50 peaks and data obtained under non-physiological conditions, such as gene knockouts or adeno-associated virus treatment, were excluded"
