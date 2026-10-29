---
name: count-matrix-normalization
description: Use when raw bulk RNA-seq count matrices plus sample metadata must be
  normalized and tested for differential expression (DESeq2/edgeR negative-binomial
  GLMs) between untreated baseline and 17-AAG/Ixazomib-treated HMCLs at p<0.05 with
  |FC|>2, FDR<0.05 reporting.
when_to_use_negative:
- Input is already a normalized expression matrix or a published DEG table — re-normalizing
  raw counts would double-correct size factors
- The data are single-cell RNA-seq or CyTOF single-cell proteomics measurements, which
  require scRNA-seq/cytometry preprocessing rather than bulk count-matrix normalization
- The question is drug-combination prediction/ranking of secDrugs rather than calling
  differentially expressed genes
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Language in which count normalization, DESeq2/edgeR negative-binomial models,
    GSA, and limma modeling are executed
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: secDrug
  role: Package that ships the deposited CombinationDrugMyeloma dataset object `d`
    and the AssignGroup(x, l) helper used to define contrast groups
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: CombinationDrugMyeloma
  role: Packaged dataset object providing the untreated, 17-AAG, and 17-AAG + Ixazomib
    count matrices and metadata
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: DESeq2
  role: Normalization of raw counts and negative-binomial differential-expression
    model fitting for each contrast
- name: edgeR
  role: Second negative-binomial DE model used to cross-check per-contrast DEG calls
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41408-022-00636-2
    title: 'secDrug: a pipeline to discover novel drug combinations to kill drug-resistant
      multiple myeloma cells using a greedy set cover algorithm and single-cell multi-omics'
  source_content_hash: sha256:baececf0be0b4ba4c4eed39be4cf9888ece40ed21da0f806abd47a01ebc3d3fc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CombinationDrugMyeloma
  - DESeq2
  - R
  - edgeR
  - secDrug
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# count-matrix-normalization

## Summary

Normalize deposited bulk RNA-seq raw count matrices and fit negative-binomial differential-expression models in R (DESeq2, edgeR) to call DEGs between untreated baseline and drug-treated myeloma cell lines. This skill converts raw counts into per-contrast DEG tables and ranked gene lists with explicit significance thresholds.

## When to use

Use when raw bulk RNA-seq count matrices plus sample metadata must be normalized and tested for differential expression (DESeq2/edgeR negative-binomial GLMs) between untreated baseline and 17-AAG/Ixazomib-treated HMCLs at p<0.05 with |FC|>2, FDR<0.05 reporting.

## When NOT to use

- Input is already a normalized expression matrix or a published DEG table — re-normalizing raw counts would double-correct size factors
- The data are single-cell RNA-seq or CyTOF single-cell proteomics measurements, which require scRNA-seq/cytometry preprocessing rather than bulk count-matrix normalization
- The question is drug-combination prediction/ranking of secDrugs rather than calling differentially expressed genes

## Inputs

- Raw bulk RNA-seq count matrices (gene x sample) from the secDrug repository
- Sample metadata table mapping samples to treatment groups (untreated baseline, 0.5 µM 17-AAG single-agent, 17-AAG + 15 nM Ixazomib combination, 24 h)
- Group-assignment definitions via AssignGroup(x, l) for sensitive-vs-resistant HMCL contrasts

## Outputs

- Per-contrast DEG summary table written to deg_summary_table.csv (gene, log2 fold-change, p-value, FDR, group assignment)
- Ranked gene lists written to ranked_gene_lists.csv for GSA/limma upstream-regulator analysis
- Per-contrast DEG counts (e.g., 421 for 17-AAG-sensitive vs -resistant, 1449 for 17-AAG response, 3974 for the combination)

## How to apply

Load the deposited bulk RNA-seq count matrices and sample metadata from the secDrug repository using the packaged dataset object `d` (CombinationDrugMyeloma), and define sensitive-vs-resistant comparison groups with the AssignGroup(x, l) helper for contrasts such as untreated vs 17-AAG + 15 nM Ixazomib at 24 h. Normalize raw counts and fit negative-binomial differential-expression models with DESeq2 and edgeR for each contrast (17-AAG-sensitive vs -resistant lines, 17-AAG response, combination response); run both packages because normalization choices differ and cross-checking guards against size-factor artifacts. Apply GSA (rank-based gene set analysis) and limma linear modeling on the normalized data to generate ranked gene lists and identify upstream regulators. Report per-contrast DEG counts at p < 0.05, and flag high-confidence hits as |fold-change| > 2 with FDR < 0.05. Write the summary to deg_summary_table.csv and ranked lists to ranked_gene_lists.csv for downstream validation.

## Related tools

- **R** (Language in which count normalization, DESeq2/edgeR negative-binomial models, GSA, and limma modeling are executed) — github.com/Ujjal-Mukherjee/secDrug
- **secDrug** (Package that ships the deposited CombinationDrugMyeloma dataset object `d` and the AssignGroup(x, l) helper used to define contrast groups) — github.com/Ujjal-Mukherjee/secDrug
- **CombinationDrugMyeloma** (Packaged dataset object providing the untreated, 17-AAG, and 17-AAG + Ixazomib count matrices and metadata) — github.com/Ujjal-Mukherjee/secDrug
- **DESeq2** (Normalization of raw counts and negative-binomial differential-expression model fitting for each contrast)
- **edgeR** (Second negative-binomial DE model used to cross-check per-contrast DEG calls)

## Examples

```
library(DESeq2); dds <- DESeqDataSetFromMatrix(countData=counts, colData=coldata, design=~group); dds <- DESeq(dds); res <- results(dds, alpha=0.05); write.csv(res, 'deg_summary_table.csv')
```

## Evaluation signals

- Per-contrast DEG counts reproduce the reported values: 421 DE genes for 17-AAG-sensitive vs -resistant, 1449 for 17-AAG response, and 3974 for the 17-AAG + Ixazomib combination at p < 0.05
- Subset with |fold-change| > 2 and FDR < 0.05 equals 853 genes for the combination contrast
- Top upstream regulators CEBPB, ERBB2, CCND1, and let-7 appear in the ranked gene lists
- deg_summary_table.csv and ranked_gene_lists.csv exist with columns for gene, fold-change, p-value, and FDR for every contrast
- DESeq2 and edgeR produce concordant directional calls on the majority of significant genes per contrast

## Limitations

- DEG counts are specific to the deposited conditions (untreated, 0.5 µM 17-AAG, 17-AAG + 15 nM Ixazomib, 24 h) and will not transfer to other doses, time points, or cell-line panels
- Supplementary Methods, Tables, and Figures supporting the DE analysis are referenced but not verifiable in the provided text
- Normalization artifacts or batch effects in the deposited count matrices may shift DEG counts relative to the reported thresholds

## Evidence

- [results] A total of 3974 genes changed significantly between untreated and 17-AAG + Ixazomib combination-treated samples (p < 0.05), of which 853 showed |fold-change| > 2 with FDR < 0.05.: [evidence span withheld — anchor status: not_found]
- [methods] Normalize raw counts and fit negative-binomial differential-expression models in R with DESeq2 and edgeR for each contrast: [evidence span withheld — anchor status: not_found]
- [abstract] genome-wide transcriptome analysis (bulk RNA sequencing): "genome-wide transcriptome analysis (bulk RNA sequencing)"
- [intro] we used our HMCL panel as in vitro model system representing inter-individual heterogeneity in drug-response/resistance: "we used our HMCL panel as in vitro model system representing inter-individual heterogeneity in drug-response/resistance to show that the top predicted secondary secDrugs are indeed effective against"
