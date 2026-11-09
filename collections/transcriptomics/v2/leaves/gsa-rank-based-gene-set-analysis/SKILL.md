---
name: gsa-rank-based-gene-set-analysis
description: Use when bulk RNA-seq DE contrasts (e.g., 17-AAG-sensitive vs -resistant
  HMCLs, 17-AAG response, combination response) need to be converted into ranked gene
  lists and gene-set/upstream-regulator signals to interpret drug-response mechanisms.
when_to_use_negative:
- Input is single-cell RNA-seq or CyTOF data used for combination screening by target-gene
  enrichment — this skill operates on bulk RNA-seq differential-expression contrasts
- Input is already a ranked gene list or a gene-set enrichment result with no underlying
  count matrix or sample grouping available
- The goal is two-drug combination prediction via the greedy set-cover algorithm rather
  than post-hoc interpretation of a treatment contrast
edam_operation: http://edamontology.org/operation_0331
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: secDrug
  role: R package providing the CombinationDrugMyeloma dataset object `d` and the
    AssignGroup(x, l) helper used to define sensitive-vs-resistant comparison groups
    for the DE contrasts fed into GSA
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: R
  role: Language in which DESeq2/edgeR normalization, negative-binomial DE fitting,
    GSA, and limma linear modeling are run
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: CombinationDrugMyeloma
  role: Dataset object `d` holding the HMCL bulk RNA-seq count matrices and sample
    metadata analyzed per contrast
  repo: github.com/Ujjal-Mukherjee/secDrug
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41408-022-00636-2
    title: 'secDrug: a pipeline to discover novel drug combinations to kill drug-resistant
      multiple myeloma cells using a greedy set cover algorithm and single-cell multi-omics'
  source_content_hash: sha256:22b0b99915076c8a66a2be2bcc661c66dbdff416a2d98e42b84fde44b8f36064
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CombinationDrugMyeloma
  - R
  - secDrug
  edam_operation: http://edamontology.org/operation_0331
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# gsa-rank-based-gene-set-analysis

## Summary

Rank-based Gene Set Analysis (GSA) applied to bulk RNA-seq differential-expression results from 17-AAG / 17-AAG + Ixazomib-treated myeloma cell lines to rank gene sets and identify upstream regulators (e.g., CEBPB, ERBB2, CCND1, let-7) underlying drug-response signatures.

## When to use

Use when bulk RNA-seq DE contrasts (e.g., 17-AAG-sensitive vs -resistant HMCLs, 17-AAG response, combination response) need to be converted into ranked gene lists and gene-set/upstream-regulator signals to interpret drug-response mechanisms.

## When NOT to use

- Input is single-cell RNA-seq or CyTOF data used for combination screening by target-gene enrichment — this skill operates on bulk RNA-seq differential-expression contrasts
- Input is already a ranked gene list or a gene-set enrichment result with no underlying count matrix or sample grouping available
- The goal is two-drug combination prediction via the greedy set-cover algorithm rather than post-hoc interpretation of a treatment contrast

## Inputs

- Bulk RNA-seq raw count matrices for HMCLs at untreated baseline, 0.5 µM 17-AAG single-agent, and 17-AAG + 15 nM Ixazomib combination (24 h)
- Sample metadata mapping samples to treatment group and sensitive/resistant status
- secDrug packaged dataset object `d` (CombinationDrugMyeloma) and AssignGroup(x, l) helper outputs defining comparison groups
- Gene-set collections / upstream regulator sets used by GSA

## Outputs

- Per-contrast DEG summary table (deg_summary_table.csv) with significant gene counts at p < 0.05 and |fold-change| > 2, FDR < 0.05 subsets
- Ranked gene lists (ranked_gene_lists.csv) per contrast (17-AAG-sensitive vs -resistant, 17-AAG response, combination response)
- GSA-ranked gene sets with upstream regulators (e.g., CEBPB, ERBB2, CCND1, let-7)

## How to apply

Load the deposited bulk RNA-seq count matrices and sample metadata from the secDrug repository (github.com/Ujjal-Mukherjee/secDrug), using the packaged dataset object `d` (CombinationDrugMyeloma) and the AssignGroup(x, l) helper to define sensitive-vs-resistant comparison groups. Normalize raw counts and fit negative-binomial differential-expression models in R with DESeq2 and edgeR for each contrast, producing signed, ranked gene lists. Apply GSA (rank-based gene set analysis) alongside limma linear modeling to the ranked lists to identify enriched gene sets and upstream regulators such as CEBPB, ERBB2, CCND1, and let-7. The rationale is that rank-based methods avoid arbitrary per-gene cutoffs while preserving the full ordering of the DE signal, letting pathway-level aggregation reveal regulators of the drug-response phenotype. Compile per-contrast DEG counts and write the summary to deg_summary_table.csv and the ranked lists to ranked_gene_lists.csv. A DEG is counted as significant at p < 0.05; report a stricter hit set at |fold-change| > 2 with FDR < 0.05.

## Related tools

- **secDrug** (R package providing the CombinationDrugMyeloma dataset object `d` and the AssignGroup(x, l) helper used to define sensitive-vs-resistant comparison groups for the DE contrasts fed into GSA) — github.com/Ujjal-Mukherjee/secDrug
- **R** (Language in which DESeq2/edgeR normalization, negative-binomial DE fitting, GSA, and limma linear modeling are run) — github.com/Ujjal-Mukherjee/secDrug
- **CombinationDrugMyeloma** (Dataset object `d` holding the HMCL bulk RNA-seq count matrices and sample metadata analyzed per contrast) — github.com/Ujjal-Mukherjee/secDrug

## Examples

```
groups <- AssignGroup(x, l); dds <- DESeqDataSetFromMatrix(counts(d), groups, ~1); dds <- DESeq(dds)
```

## Evaluation signals

- DEG summary reproduces the reported counts: 421 DE genes for 17-AAG-sensitive vs -resistant, 1449 for 17-AAG response, and 3974 for the 17-AAG + Ixazomib combination (p < 0.05)
- Combination contrast yields 853 genes meeting |fold-change| > 2 with FDR < 0.05
- Top upstream regulators CEBPB, ERBB2, CCND1, and let-7 appear in the ranked gene-list outputs (ranked_gene_lists.csv)
- ranked_gene_lists.csv and deg_summary_table.csv are produced with entries for all three contrasts (sensitive-vs-resistant, 17-AAG response, combination response)
- GSA ranking is computed from the full signed DE statistic per gene (rank-based), not from an arbitrary cutoff subset

## Limitations

- Pipeline is limited to two-drug combinations; model systems for testing more than two secDrugs are not yet developed
- Testing of predicted secDrugs in a wider panel of HMCLs representing refractory and clonally derived acquired resistant cell lines is not yet done
- Supplementary Methods, Tables, and Figures are referenced but not included/verifiable in the provided text

## Evidence

- [abstract] workflow: genome-wide transcriptome analysis: "genome-wide transcriptome analysis (bulk RNA sequencing)"
- [results] task finding: combination DEG counts: [evidence span withheld — anchor status: not_found]
- [methods] task workflow: GSA and regulator identification: [evidence span withheld — anchor status: not_found]
- [methods] task workflow: normalization and DE modeling: [evidence span withheld — anchor status: not_found]
- [intro] article intro: functional and sequencing characterization of secondary drugs: "we added functional assays, next-generation RNA sequencing, CRISPR-based gene editing, and high-dimensional mass cytometry (CyTOF/cytometry time of flight) in primary bone marrow cells"
