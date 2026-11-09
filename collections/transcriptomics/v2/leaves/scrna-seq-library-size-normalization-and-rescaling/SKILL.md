---
name: scrna-seq-library-size-normalization-and-rescaling
description: Use when raw scRNA-seq UMI count matrices (e.g. Tabula Muris Spleen-10X_P4_7
  ExpressionData.csv) must be normalized and rescaled to the (0,1) interval before
  feeding scRNA_seq_preprocessing output into scGATE_edge()/scGATE_logic() network
  inference.
when_to_use_negative:
- Input matrix is already normalized or is not raw UMI counts (set library_size_normalization="False"
  or skip; double-normalizing distorts the (0,1) rescaling)
- Goal is large-scale GRN inference from scratch on many genes without a refined TF
  candidate list — the article notes scGATE is applicable only to a small (<200) subset
  of genes with refined candidate TFs
- Data are pseudotime-sorted single-cell profiles along differentiation trajectories,
  which scGATE does not yet handle
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scGATE
  role: Provides the scRNA_seq_preprocessing() function that performs library size
    normalization, scaling, and (0,1) rescaling of raw counts
  repo: github.com/CompBioIPM/scGATE
- name: R 4.1.3
  role: Runtime environment in which the scGATE preprocessing code is written and
    executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:13fe1858ce2164ca01f412ac41e835c4d1150b82b3cf9130054dbfd1cf0dd550
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# scrna-seq-library-size-normalization-and-rescaling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

scGATE-specific preprocessing of raw scRNA-seq count matrices (cells in rows, genes in columns) that performs library size normalization followed by min-max rescaling into the (0,1) interval, optionally restricted to a supplied TF list, so expression profiles are ready for Hill-function-based logic-gate inference.

## When to use

Use when raw scRNA-seq UMI count matrices (e.g. Tabula Muris Spleen-10X_P4_7 ExpressionData.csv) must be normalized and rescaled to the (0,1) interval before feeding scRNA_seq_preprocessing output into scGATE_edge()/scGATE_logic() network inference.

## When NOT to use

- Input matrix is already normalized or is not raw UMI counts (set library_size_normalization="False" or skip; double-normalizing distorts the (0,1) rescaling)
- Goal is large-scale GRN inference from scratch on many genes without a refined TF candidate list — the article notes scGATE is applicable only to a small (<200) subset of genes with refined candidate TFs
- Data are pseudotime-sorted single-cell profiles along differentiation trajectories, which scGATE does not yet handle

## Inputs

- Raw scRNA-seq UMI count matrix, cells in rows and genes in columns (e.g. CSV such as Tabula_Muris2018_Spleen-10X_P4_7_ExpressionData.csv)
- Optional TF candidate list (e.g. Tabula_Muris2018_Spleen-10X_P4_7_tf_lists.txt) passed as tf_list

## Outputs

- Normalized count matrix with values in the (0,1) interval, suitable input for scGATE_edge() / scGATE_logic()

## How to apply

Load the raw count matrix with genes in columns and cells in rows (e.g. Tabula_Muris2018_Spleen-10X_P4_7_ExpressionData.csv). Optionally pass a TF candidate list (e.g. Tabula_Muris2018_Spleen-10X_P4_7_tf_lists.txt) via the tf_list argument so only those columns are retained as candidates. Run scRNA_seq_preprocessing(data, library_size_normalization="True", tf_list=tf_names): library size normalization divides each cell's raw UMI counts by its total number of reads, then counts are scaled by a scaling factor and finally rescaled into the (0,1) interval. The (0,1)-bounded output is required because scGATE models continuous expression with a Hill activation/climbing function rather than binarizing; keeping library_size_normalization="True" (the default) is recommended unless the matrix is already normalized. The normalized matrix then feeds directly into scGATE_edge(data = data, base_GRN = candidate_tf_target, h_act = 7) or scGATE_logic().

## Related tools

- **scGATE** (Provides the scRNA_seq_preprocessing() function that performs library size normalization, scaling, and (0,1) rescaling of raw counts) — github.com/CompBioIPM/scGATE
- **R 4.1.3** (Runtime environment in which the scGATE preprocessing code is written and executed)

## Examples

```
normalized_counts <- scRNA_seq_preprocessing(data = data_scRNA_seq, library_size_normalization = "True", tf_list = NA)
```

## Evaluation signals

- All normalized values fall within the (0,1) interval as required by the scGATE_edge()/scGATE_logic() data parameter spec
- library_size_normalization flag was "True" (default), meaning each cell's UMI counts were divided by its total number of reads before scaling and rescaling
- With tf_list supplied, only listed TF columns are treated as candidates; with tf_list = NA, all columns in the data matrix are considered
- Downstream sanity check: head of the ranked_edge_list from scGATE_edge() on the Spleen-10X_P4_7 dataset should match reference output (Ctcf→Rps19 with BF ≈ 2002.419, Batf→Rps19 with BF ≈ 2001.388)

## Limitations

- scGATE is currently applicable to a rather small (<200) subset of genes with a refined list of candidate regulatory TFs based on external hints
- Decoding high-order Boolean logic involving more than 5 transcriptional regulators remains challenging due to overfitting
- scGATE does not yet handle pseudotime-sorted single-cell data along differentiation trajectories

## Evidence
