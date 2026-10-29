---
name: differential-utr-length-testing
description: Use when SingleCellExperiment objects from scUTRquant are available and
  the goal is to call genes with significant 3'UTR length changes (DUL) between two
  cell types and compare them to DGE gene sets.
when_to_use_negative:
- The counts are gene-level only (no 3'UTR isoform matrix), e.g., a standard 10x gene-cell
  count matrix not processed through scUTRquant
- The question is de novo cleavage-site discovery rather than differential isoform
  usage testing on an existing UTRome annotation
- Fewer than two comparable cell groups (or bulk RNA-seq data) — the WUI framework
  was developed and applied for scRNA-seq cell-type pair comparisons
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scUTRboot
  role: Provides the weighted UTR isoform (WUI) statistical tests used to call significant
    differences in 3'UTR isoform usage across cell-type pairs
  repo: https://github.com/mfansler/scutrboot
- name: scUTRquant
  role: Generates the SingleCellExperiment objects with simultaneous gene and 3'UTR
    isoform quantification that serve as input to the differential testing
  repo: https://github.com/Mayrlab/scUTRquant
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:a1f356c77363490bc99d1fc7648b3886438d87b7c549b39571246c75cc3eadc3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfansler/scutrboot
  tools:
  - scUTRboot
  - scUTRquant
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# differential-utr-length-testing

## Summary

Statistical testing workflow for identifying genes with differential 3'UTR isoform usage/length (DUL) between cell-type pairs from scRNA-seq, using scUTRboot weighted UTR isoform (WUI) tests, and checking independence from differential gene expression (DGE).

## When to use

Use when SingleCellExperiment objects from scUTRquant are available and the goal is to call genes with significant 3'UTR length changes (DUL) between two cell types and compare them to DGE gene sets.

## When NOT to use

- The counts are gene-level only (no 3'UTR isoform matrix), e.g., a standard 10x gene-cell count matrix not processed through scUTRquant
- The question is de novo cleavage-site discovery rather than differential isoform usage testing on an existing UTRome annotation
- Fewer than two comparable cell groups (or bulk RNA-seq data) — the WUI framework was developed and applied for scRNA-seq cell-type pair comparisons

## Inputs

- scUTRquant-generated SingleCellExperiment objects (sparse gene-level and 3'UTR isoform count matrices)
- Cell metadata (colData) with cell-type annotations
- Multi-UTR gene list (e.g., from the UTRome atlas restricting to genes with more than two tandem isoforms)

## Outputs

- Table of DGE calls per cell-type pair (Welch t-test hits with |fold change| > 1.5, q < 0.05)
- Table of DUL calls per cell-type pair (WUI test hits with |dWUI| > 0.10, q < 0.05)
- Tabulated DGE-DUL overlap fractions and chi-square test results (e.g., written to dge_dul_overlap_fractions.tsv)

## How to apply

Load the scUTRquant-generated SingleCellExperiment objects and extract the sparse gene-level and 3'UTR isoform count matrices with cell metadata from colData. Aggregate isoform counts per cell type and restrict to multi-UTR genes that are co-expressed in both members of each cell-type pair. Call differentially expressed genes with Welch t-tests on gene-level counts (|fold change| > 1.5, q < 0.05 after multiple-testing correction) and call differentially used/lengthened 3' ends with scUTRboot weighted UTR isoform (WUI) tests (|dWUI| > 0.10, q < 0.05). Compute per-pair overlap fractions between DGE and DUL gene sets and assess independence with a chi-square test of independence; the paper reports 0-7.4% overlap, consistent with independence in 14/17 pairs, showing mRNA abundance and 3'UTR length are largely independent regulatory axes.

## Related tools

- **scUTRboot** (Provides the weighted UTR isoform (WUI) statistical tests used to call significant differences in 3'UTR isoform usage across cell-type pairs) — https://github.com/mfansler/scutrboot
- **scUTRquant** (Generates the SingleCellExperiment objects with simultaneous gene and 3'UTR isoform quantification that serve as input to the differential testing) — https://github.com/Mayrlab/scUTRquant

## Evaluation signals

- Each reported DGE-DUL overlap fraction falls within the expected 0-7.4% range
- Chi-square tests of independence show non-significant overlap in at least 14/17 cell-type pairs
- All WUI test hits satisfy |dWUI| > 0.10 and q < 0.05 after multiple-testing correction
- All DGE hits satisfy |fold change| > 1.5 and q < 0.05
- Only multi-UTR genes co-expressed in both members of each cell-type pair are included in the DUL test set

## Limitations

- Most reads from 10x Genomics data do not span mRNA 3' end cleavage sites, preventing exact de novo mapping of 3' ends — testing is limited to isoforms in the curated UTRome/CS catalog
- Genes with only one major 3' end or insufficient expression cannot be tested for differential usage
- Fewer than 20% of 3'UTR changes regulate mRNA or protein abundance, so DUL calls should not be interpreted as direct expression changes

## Evidence

- [methods] WUI test thresholds on multi-UTR genes: [evidence span withheld — anchor status: not_found]
- [abstract] Independent axes finding: "mRNA abundance and mRNA length are two largely independent axes of gene regulation that together determine the amount and spatial organization of protein synthesis."
- [results] DGE-DUL overlap range: [evidence span withheld — anchor status: not_found]
- [intro] scUTRboot role per article intro: "and a statistical testing package, called scUTRboot"
- [readme] Coupled pipeline for significant 3'UTR differences: "Coupled with scUTRboot, significant differences in 3′UTRs across samples are identified, which facilitates the integration of 3′UTR quantification into standard scRNA-seq data analysis"
