---
name: delta-pi-percent-isoform-usage-calculation
description: Use when comparing per-gene isoform proportions between two conditions
  (e.g., bulk or single-cell HIPP vs PFC) from long-read ScISOrSeq count tables to
  identify genes with significant shifts in isoform, TSS, or polyA-site usage.
when_to_use_negative:
- The input is short-read data without full-length isoform assignments — PI cannot
  be computed reliably from exon-level coverage alone, and the paper shows isoform
  tests outperform exon tests only when isoform counts exist
- Genes with fewer than 25 supporting long reads (minNumReads=25) — PI estimates are
  too sparse, per the paper's noted sparsity of the isoform matrix from lower long-read
  throughput
- The goal is cell-type-resolved DIE rather than bulk pairwise comparison — the hierarchical
  testing mode (is.hier=TRUE) and tree traversal functions are then more appropriate
  than this bulk pairwise setup
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: scisorseqr
  role: Runs DiffSplicingAnalysis with typeOfTest='Iso', minNumReads=25, is.hier=FALSE
    to perform per-gene n x 2 chi-square tests on isoform proportions and compute
    PI/delta-PI
  repo: github.com/noush-joglekar/scisorseqr
- name: R
  role: Implementation language for scisorseqr and for applying Benjamini-Hochberg
    FDR correction to chi-square p-values
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:ee2809196f91a4552e80bc0f41f09c2dabe3bc92020c4d1d31a058b6f119fb48
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - scisorseqr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# delta-PI percent isoform usage calculation

## Summary

Quantifies differential isoform expression (DIE) between two conditions by computing per-gene percent-isoform (PI) usage from long-read isoform count tables, testing differences with n x 2 chi-square tests (scisorseqr's DiffSplicingAnalysis), and flagging genes with Benjamini-Hochberg FDR <= 0.05 and delta-PI >= 0.1.

## When to use

Use when comparing per-gene isoform proportions between two conditions (e.g., bulk or single-cell HIPP vs PFC) from long-read ScISOrSeq count tables to identify genes with significant shifts in isoform, TSS, or polyA-site usage.

## When NOT to use

- The input is short-read data without full-length isoform assignments — PI cannot be computed reliably from exon-level coverage alone, and the paper shows isoform tests outperform exon tests only when isoform counts exist
- Genes with fewer than 25 supporting long reads (minNumReads=25) — PI estimates are too sparse, per the paper's noted sparsity of the isoform matrix from lower long-read throughput
- The goal is cell-type-resolved DIE rather than bulk pairwise comparison — the hierarchical testing mode (is.hier=TRUE) and tree traversal functions are then more appropriate than this bulk pairwise setup

## Inputs

- Per-gene isoform count table (long-read assigned reads per gene per condition, e.g. bulk HIPP and bulk PFC from isoformAtlas)
- Per-gene TSS count table
- Per-gene polyA-site count table
- scisorseqr config file specifying grouping factor and comparison

## Outputs

- Results TSV of tested genes with chi-square p-values, BH-adjusted FDR, and PI values per condition
- List of significant DIE genes (FDR <= 0.05, delta-PI >= 0.1) with delta-PI magnitudes
- Subset of DIE genes flagged as attributable to differential TSS or polyA-site usage

## How to apply

Load the per-gene isoform count table (here, bulk HIPP vs bulk PFC from www.isoformAtlas.com) and run scisorseqr's DiffSplicingAnalysis with typeOfTest='Iso', minNumReads=25, and is.hier=FALSE, which performs per-gene n x 2 chi-square tests on isoform proportions between conditions. Apply Benjamini-Hochberg correction at FDR <= 0.05 and require delta-PI (change in percent-isoform usage) >= 0.1 to call a gene differentially isoform-expressed. The identical chi-square framework is repeated on per-gene TSS and polyA-site count tables to attribute DIE to differential transcription-start-site or polyA usage. Rationale: isoform tests outperform exon-level tests for DIE, and the percent-isoform change cutoff ensures the effect size is biologically meaningful, not just statistically significant. Decision point: minNumReads=25 guards against testing genes with too few long reads to estimate stable PI.

## Related tools

- **scisorseqr** (Runs DiffSplicingAnalysis with typeOfTest='Iso', minNumReads=25, is.hier=FALSE to perform per-gene n x 2 chi-square tests on isoform proportions and compute PI/delta-PI) — github.com/noush-joglekar/scisorseqr
- **R** (Implementation language for scisorseqr and for applying Benjamini-Hochberg FDR correction to chi-square p-values)

## Examples

```
DiffSplicingAnalysis(configFile = config, typeOfTest = 'Iso', minNumReads = 25, is.hier = FALSE)
```

## Evaluation signals

- Results TSV must contain 395 significant genes out of 3958 tested genes for bulk HIPP vs PFC (9.06% tested), matching the published regional DIE result
- 141 of the 395 significant genes must be flagged as having DIE attributable to differential TSS or polyA-site usage
- All reported genes must satisfy both BH-adjusted FDR <= 0.05 and delta-PI >= 0.1
- Only genes meeting minNumReads=25 enter the chi-square tests (3958 tested from the tested universe)
- Percent-isoform values per condition must be proportions in [0,1] that sum to 1 within each gene

## Limitations

- The isoform count matrix can be sparse due to lower long-read throughput, which limits the power and stability of PI estimates
- minNumReads=25 excludes lowly expressed genes, so DIE calls are restricted to well-covered genes
- Some significant DIE genes have isoforms novel to GENCODE, so annotation-based interpretations may be incomplete
- Whether these DIE patterns persist in disease case-control studies, across adult brain regions, and across species is untested

## Evidence

- [discussion] Region-specific isoform testing revealed 395 genes with DIE between hippocampus and PFC, partially caused by isoforms novel to GENCODE.: "Region-specific isoform testing revealed 395 genes with DIE between hippocampus and PFC, partially caused by isoforms novel to GENCODE."
- [methods] DiffSplicingAnalysis with typeOfTest='Iso' and minNumReads=25: "DiffSplicingAnalysis(configFile = config, 
                     typeOfTest = 'Iso', 
                     minNumReads = 25, is.hier = FALSE)"
- [abstract] Isoform tests for DIE show better performance than exon tests.: "Isoform tests for DIE show better performance than exon tests."
- [readme] scisorseqr is a linux based R-package for analyzing differential isoform expression in single cells.: "scisorseqr is a linux based R-package for analyzing differential isoform
expression in single cells."
