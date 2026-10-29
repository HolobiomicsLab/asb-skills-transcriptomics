---
name: myeloma-drug-response-genomics
description: Use when untreated-vs-treated bulk RNA-seq count matrices from drug-sensitive
  and drug-resistant HMCLs (e.g. 0.5 µM 17-AAG ± 15 nM Ixazomib at 24 h) must be converted
  into per-contrast DEG counts and ranked gene lists.
when_to_use_negative:
- The analysis goal is single-cell (scRNAseq or CyTOF) target-enrichment screening
  rather than bulk differential-expression signatures
- The input is already a summarized DEG table or ranked list, i.e. differential expression
  has already been fit and thresholds applied
- The pipeline question is secDrug combination prediction via the greedy set cover
  algorithm on pharmacogenomics data, not validation of a specific treatment contrast
  by RNA-seq
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: R
  role: Language in which raw counts are normalized and negative-binomial differential-expression
    models (DESeq2, edgeR), GSA, and limma linear models are fit
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: secDrug
  role: R-software package whose bulk RNA-seq validation workflow and outputs (DEG
    summary, ranked gene lists) this skill reproduces
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: CombinationDrugMyeloma
  role: Packaged dataset object `d` supplying the count matrices and sample metadata
    for the untreated / 17-AAG / 17-AAG + Ixazomib contrasts
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: AssignGroup
  role: Helper function used to assign HMCLs into sensitive-vs-resistant comparison
    groups before model fitting
  repo: github.com/Ujjal-Mukherjee/secDrug
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41408-022-00636-2
    title: 'secDrug: a pipeline to discover novel drug combinations to kill drug-resistant
      multiple myeloma cells using a greedy set cover algorithm and single-cell multi-omics'
  source_content_hash: sha256:414c9dbe64fb058a1841fc013f527a63b7f0ce44a99cf10629893cb1b7acd6bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AssignGroup
  - CombinationDrugMyeloma
  - R
  - secDrug
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# myeloma-drug-response-genomics

## Summary

Reproduce bulk RNA-seq differential-expression signatures of drug (combination) response in the secDrug myeloma pipeline, e.g. 17-AAG and 17-AAG + Ixazomib in human myeloma cell lines (HMCLs), using DESeq2/edgeR negative-binomial models, GSA/limma ranked gene lists, and explicit significance cutoffs (p < 0.05; |fold-change| > 2 with FDR < 0.05).

## When to use

Use when untreated-vs-treated bulk RNA-seq count matrices from drug-sensitive and drug-resistant HMCLs (e.g. 0.5 µM 17-AAG ± 15 nM Ixazomib at 24 h) must be converted into per-contrast DEG counts and ranked gene lists.

## When NOT to use

- The analysis goal is single-cell (scRNAseq or CyTOF) target-enrichment screening rather than bulk differential-expression signatures
- The input is already a summarized DEG table or ranked list, i.e. differential expression has already been fit and thresholds applied
- The pipeline question is secDrug combination prediction via the greedy set cover algorithm on pharmacogenomics data, not validation of a specific treatment contrast by RNA-seq

## Inputs

- Bulk RNA-seq raw count matrices for untreated baseline, 0.5 µM 17-AAG single-agent, and 17-AAG + 15 nM Ixazomib combination at 24 h
- Sample metadata annotating HMCL drug-response states (sensitive vs resistant; untreated vs treated)
- Packaged dataset object `d` (CombinationDrugMyeloma) from the secDrug R package

## Outputs

- deg_summary_table.csv with per-contrast DEG counts (expected: 421 for 17-AAG-sensitive vs -resistant, 1449 for 17-AAG response, 3974 for the combination)
- ranked_gene_lists.csv with per-contrast ranked gene lists from GSA and limma
- Upstream-regulator identifications (e.g., CEBPB, ERBB2, CCND1, let-7)

## How to apply

Load the deposited bulk RNA-seq count matrices and sample metadata from the secDrug repository (github.com/Ujjal-Mukherjee/secDrug) via the packaged dataset object `d` (CombinationDrugMyeloma), and use the AssignGroup(x, l) helper to define sensitive-vs-resistant comparison groups. Normalize raw counts and fit negative-binomial differential-expression models in R with DESeq2 and edgeR for each contrast (17-AAG-sensitive vs -resistant lines, 17-AAG response, combination response). Apply GSA (rank-based gene set analysis) and limma linear modeling to generate ranked gene lists and identify upstream regulators (e.g., CEBPB, ERBB2, CCND1, let-7). Compile per-contrast DEG counts and validate against the reported counts — 421 DE genes for 17-AAG-sensitive vs -resistant, 1449 for 17-AAG response, and 3974 for the combination — reporting genes at p < 0.05 and the stringent subset at |fold-change| > 2 with FDR < 0.05.

## Related tools

- **R** (Language in which raw counts are normalized and negative-binomial differential-expression models (DESeq2, edgeR), GSA, and limma linear models are fit) — github.com/Ujjal-Mukherjee/secDrug
- **secDrug** (R-software package whose bulk RNA-seq validation workflow and outputs (DEG summary, ranked gene lists) this skill reproduces) — github.com/Ujjal-Mukherjee/secDrug
- **CombinationDrugMyeloma** (Packaged dataset object `d` supplying the count matrices and sample metadata for the untreated / 17-AAG / 17-AAG + Ixazomib contrasts) — github.com/Ujjal-Mukherjee/secDrug
- **AssignGroup** (Helper function used to assign HMCLs into sensitive-vs-resistant comparison groups before model fitting) — github.com/Ujjal-Mukherjee/secDrug

## Examples

```
library(secDrug); data(d); groups <- AssignGroup(d, l); dds <- DESeq2::DESeqDataSetFromMatrix(counts(d), colData=groups, design=~response)
```

## Evaluation signals

- deg_summary_table.csv reproduces the reported counts: 421 DE genes for 17-AAG-sensitive vs -resistant, 1449 for 17-AAG response, and 3974 for the combination
- The combination contrast reports 3974 genes significant at p < 0.05, of which 853 show |fold-change| > 2 with FDR < 0.05
- Top regulators CEBPB, ERBB2, CCND1, and let-7 appear in the ranked_gene_lists.csv outputs
- Group assignments produced by AssignGroup(x, l) cover all three treatment arms (untreated baseline, 0.5 µM 17-AAG, 17-AAG + 15 nM Ixazomib at 24 h)
- Differential expression was fit independently with both DESeq2 and edgeR negative-binomial models per contrast

## Limitations

- DEG counts depend on the exact contrast definition; only three contrasts (17-AAG-sensitive vs -resistant, 17-AAG response, combination response) are validated against reported numbers
- Reported significance is two-tiered (p < 0.05 per contrast; the more stringent subset additionally requires |fold-change| > 2 and FDR < 0.05), so naive single-threshold comparisons will not reproduce the 3974 / 853 split
- Pipeline validation in this skill is limited to in vitro HMCL bulk RNA-seq; the underlying pipeline has no software/code availability details described for the in silico portion in the discussion, and supplementary methods are not verifiable in the provided text

## Evidence

- [results] A total of 3974 genes changed significantly between untreated and 17-AAG + Ixazomib combination-treated samples (p < 0.05), of which 853 showed |fold-change| > 2 with FDR < 0.05.: [evidence span withheld — anchor status: not_found]
- [intro] we added functional assays, next-generation RNA sequencing, CRISPR-based gene editing, and high-dimensional mass cytometry (CyTOF: "we added functional assays, next-generation RNA sequencing, CRISPR-based gene editing, and high-dimensional mass cytometry (CyTOF"
- [abstract] we developed a universally applicable R-software package for predicting novel secondary therapies in chemotherapy-resistant cancers: "we developed a universally applicable R-software package for predicting novel secondary therapies in chemotherapy-resistant cancers"
- [methods] - `AssignGroup(x, l)`: [evidence span withheld — anchor status: not_found]
