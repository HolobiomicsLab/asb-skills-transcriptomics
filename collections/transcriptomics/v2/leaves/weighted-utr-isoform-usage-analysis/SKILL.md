---
name: weighted-utr-isoform-usage-analysis
description: Use when scUTRquant SingleCellExperiment outputs (gene + 3'UTR isoform
  count matrices) cover multi-UTR genes across two or more cell types or perturbations
  and differentially used/lengthened 3' ends must be called and compared to DGE gene
  sets.
when_to_use_negative:
- Input data are raw 10x Genomics reads that have not yet been processed through scUTRquant
  — most 10x reads do not span 3' end cleavage sites, so WUI testing needs scUTRquant
  isoform quantification first
- Genes of interest are single-UTR genes with only one major 3' end; isoform usage
  analysis requires multi-UTR genes
- Bulk RNA-seq or full-length protocols where standard 3'UTR isoform quantification
  assumptions (3'-end tag data, truncated-transcript targets) do not hold
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scUTRboot
  role: Statistical testing package that runs weighted UTR isoform (WUI) tests to
    identify significant differences in 3'UTR isoform usage across samples
- name: scUTRquant
  role: Upstream pipeline producing the SingleCellExperiment gene and 3'UTR isoform
    count matrices that the WUI tests consume
  repo: https://github.com/Mayrlab/scUTRquant
- name: kallisto-bustools
  role: Underlying quantification workflow on which scUTRquant builds for 3'-end tag
    scRNA-seq libraries
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:cfe662d5b68d5a7516312b7ac36daca923062cef17b378a885ff448c3c04ed6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - kallisto-bustools
  - scUTRboot
  - scUTRquant
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# weighted-utr-isoform-usage-analysis

## Summary

Statistical detection of differential 3'UTR isoform usage (lengthening/shortening) from scUTRquant SingleCellExperiment outputs using the scUTRboot weighted UTR isoform (WUI) test, complementing differential gene expression to show mRNA abundance and 3'UTR length are largely independent regulatory axes.

## When to use

Use when scUTRquant SingleCellExperiment outputs (gene + 3'UTR isoform count matrices) cover multi-UTR genes across two or more cell types or perturbations and differentially used/lengthened 3' ends must be called and compared to DGE gene sets.

## When NOT to use

- Input data are raw 10x Genomics reads that have not yet been processed through scUTRquant — most 10x reads do not span 3' end cleavage sites, so WUI testing needs scUTRquant isoform quantification first
- Genes of interest are single-UTR genes with only one major 3' end; isoform usage analysis requires multi-UTR genes
- Bulk RNA-seq or full-length protocols where standard 3'UTR isoform quantification assumptions (3'-end tag data, truncated-transcript targets) do not hold

## Inputs

- scUTRquant SingleCellExperiment objects containing sparse gene-level and 3'UTR isoform count matrices plus cell metadata in colData
- Cell type annotations / cell-type pair definitions from colData
- Multi-UTR gene definitions (e.g., from the human/mouse UTRome atlases)

## Outputs

- Table of differential 3'UTR usage (DUL) hits per comparison with dWUI effect sizes and q-values
- Per-pair DGE-DUL overlap fractions with chi-square test results, e.g., dge_dul_overlap_fractions.tsv

## How to apply

Load the scUTRquant-generated SingleCellExperiment objects and extract the sparse gene-level and 3'UTR isoform count matrices with cell metadata from colData. Aggregate isoform counts per cell type (pseudobulk) and restrict to multi-UTR genes that are co-expressed in both members of each comparison (e.g., each cell-type pair). Run scUTRboot weighted UTR isoform (WUI) tests on these multi-UTR genes to call differentially used or lengthened 3' ends, retaining hits with |dWUI| > 0.10 and q < 0.05 after multiple-testing correction. In parallel, call differentially expressed genes with Welch t-tests on gene-level counts (|fold change| > 1.5, q < 0.05), then compute DGE-DUL overlap fractions and test them against chance with chi-square tests of independence — the paper found only 0-7.4% overlap across 17 cell-type pairs. The rationale is that scUTRboot's WUI statistics identify significant 3'UTR differences across samples, enabling integration of 3'UTR quantification into standard scRNA-seq analysis.

## Related tools

- **scUTRboot** (Statistical testing package that runs weighted UTR isoform (WUI) tests to identify significant differences in 3'UTR isoform usage across samples)
- **scUTRquant** (Upstream pipeline producing the SingleCellExperiment gene and 3'UTR isoform count matrices that the WUI tests consume) — https://github.com/Mayrlab/scUTRquant
- **kallisto-bustools** (Underlying quantification workflow on which scUTRquant builds for 3'-end tag scRNA-seq libraries)

## Evaluation signals

- Every reported DUL hit satisfies |dWUI| > 0.10 and q < 0.05 after multiple-testing correction
- Each DGE call set satisfies |fold change| > 1.5 with q < 0.05 (Welch t-tests on gene-level counts)
- Per-pair DGE-DUL overlap fractions fall in the expected 0-7.4% range across the 17 cell-type pairs
- Chi-square tests of independence indicate DGE and DUL sets are statistically independent in 14/17 sample pairs
- Tabulated overlap fractions and chi-square results are written to dge_dul_overlap_fractions.tsv with the expected schema

## Limitations

- Most reads from 10x Genomics data do not span mRNA 3' end cleavage sites, preventing exact de novo mapping of mRNA 3' ends; quantification relies on a pre-built CS catalog (UTRome) rather than de novo discovery
- Overlap analysis is restricted to multi-UTR genes co-expressed in both members of each comparison, so single-isoform genes are excluded from DUL calls
- Fewer than 20% of 3'UTR changes regulate mRNA or protein abundance, so DUL hits should not be assumed to alter expression output

## Evidence

- [full_text] WUI test thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] scUTRboot role: "Coupled with scUTRboot, significant differences in 3′UTRs across samples are identified, which facilitates the integration of 3′UTR quantification into standard scRNA-seq data analysis"
- [abstract] independence finding: "we discover extensive 3'UTR length changes across cell types that are as widespread and coordinately regulated as gene expression changes but affect mostly different genes"
- [abstract] scUTRquant quantification: "simultaneous quantification of gene and 3'UTR isoform expression from single-cell RNA sequencing (scRNA-seq) data"
