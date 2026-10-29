---
name: bulk-rna-seq-expression-normalization
description: Use when normalized bulk RNA-seq profiles must be converted into a z-scaled
  markers_expression matrix as query input for deCS correlation-based cell type annotation
  (e.g., Figure S12 iPSC-NPC/neuron differentiation-quality assessment).
when_to_use_negative:
- Input is single-cell scRNA-seq clusters requiring deCS.fisher marker-gene enrichment
  rather than correlation on expression profiles
- Query profiles are gene lists (marker lists) rather than expression matrices
- Genes are not human gene symbols (e.g., mouse orthologs or Ensembl IDs without symbol
  mapping), since deCS example scripts require human gene symbols
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: deCS
  role: provides example preprocessing scripts and reference panels (MonacoImmune_main,
    BlueprintEncode_main, Human_cell_atlas_of_fetal) that define the scale (z-score/t-score)
    to which the normalized query matrix must conform before deCS.correlation
  repo: https://github.com/bsml320/deCS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:d4d63f796c29d80b8582fdbedd8cebe892cfb0f33a597090e8ba9bd9e961b795
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - deCS
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# bulk-rna-seq-expression-normalization

## Summary

Preprocess bulk RNA-seq count matrices (e.g., hiPSC-NPC and hiPSC-neuron samples from Stertz et al.) into a scaled sample-level expression matrix via log-transformation followed by z-scaling, so that Pearson correlation-based cell type annotation with deCS.correlation operates on comparable profiles with human gene symbols.

## When to use

Use when normalized bulk RNA-seq profiles must be converted into a z-scaled markers_expression matrix as query input for deCS correlation-based cell type annotation (e.g., Figure S12 iPSC-NPC/neuron differentiation-quality assessment).

## When NOT to use

- Input is single-cell scRNA-seq clusters requiring deCS.fisher marker-gene enrichment rather than correlation on expression profiles
- Query profiles are gene lists (marker lists) rather than expression matrices
- Genes are not human gene symbols (e.g., mouse orthologs or Ensembl IDs without symbol mapping), since deCS example scripts require human gene symbols

## Inputs

- bulk RNA-seq raw count matrix (genes x samples) with human gene symbols, e.g., hiPSC-NPC (n=13) and hiPSC-neuron (n=11) samples
- deCS-provided example preprocessing scripts

## Outputs

- markers_expression: log-transformed, z-scaled sample-level expression matrix (genes x samples) ready for deCS.correlation
- annotated sample table (Query, Max_correlation, Top1, Top2, Top3, Cell_labels), e.g., written to hipsc_decs_result.txt with 24 rows

## How to apply

Start from the bulk RNA-seq count matrices for hiPSC-NPC (n=13) and hiPSC-neuron (n=11) samples, ensuring row names are human gene symbols as required by deCS example preprocessing scripts. Log-transform the expression values (analogous to deCS's log2(AverageExpression + 1) handling of cluster-average profiles) and then z-scale across samples with scale() applied per gene to obtain the markers_expression matrix. This mirrors the deCS PBMC workflow in which a cluster/sample-average log expression matrix is row-wise scaled before calling deCS.correlation(markers_expression, ref_panel). The rationale is that deCS reference panels (MonacoImmune_main, BlueprintEncode_main, Human_cell_atlas_of_fetal) store pre-calculated cell type specificity t-scores/z-scores, so the query must be on a matching scale for Pearson correlation coefficients to be meaningful. Feed the resulting matrix, not raw counts, into deCS.correlation, optionally with cor_threshold to flag low-confidence calls as 'Undetermined cells'.

## Related tools

- **deCS** (provides example preprocessing scripts and reference panels (MonacoImmune_main, BlueprintEncode_main, Human_cell_atlas_of_fetal) that define the scale (z-score/t-score) to which the normalized query matrix must conform before deCS.correlation) — https://github.com/bsml320/deCS

## Examples

```
markers_expression <- t(scale(t(log2(expr_counts + 1)))) ; hipsc_decs_result <- deCS.correlation(markers_expression, Human_cell_atlas_of_fetal)
```

## Evaluation signals

- Output annotation table contains exactly 24 rows (13 NPC + 11 neuron samples)
- NPC samples are assigned in Cell_labels to progenitor/hESC-like cell types (fetal progenitor cells or hESCs) in 12 of 13 samples, matching the paper's reported result
- Neuron samples map to neuronal cell types in the Cell_labels column
- markers_expression matrix has row-wise z-scores (per-gene mean ~0, sd ~1) and retains human gene symbol row names
- Pearson Max_correlation values are reported for every query sample and low-confidence calls are correctly labeled 'Undetermined cells' when cor_threshold is set

## Limitations

- deCS does not take tissue information (except HCAF) into account, which may bias annotation results of the normalized query profiles
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types, so normalized bulk profiles can only be matched to existing reference panels

## Evidence

- [other] Normalize and scale the sample-level expression profiles (log-transform plus z-scaling) to obtain a markers_expression matrix suitable for correlation-based annotation.: [evidence span withheld — anchor status: not_found]
- [readme] README describes scaled expression matrix input for correlation analysis.: "Here, `markers_expression` is scaled gene expression matrix for human scRNA-seq or bulk RNA-seq"
- [readme] deCS computes Pearson/Spearman correlation against pre-calculated reference panels.: "we calculate pearson correlation coefficient (PCC) or Spearman's rank correlation coefficient with each of the cell type in the reference dataset"
- [methods] cor_threshold guards against mis-annotation.: "give `cor_threshold` to avoid mis-annotation (regard as "Undetermined cells")"
- [readme] Reference panels are pre-calculated cell type specificity scores.: "`ref_panel` is pre-calculated cell type specificity score reference panel"
