---
name: single-cell-long-read-exon-quantification
description: Use when you have full-length spliced long reads assigned to single cells/cell
  types and need cell-type-resolved exon usage quantification plus exon-wise differential
  exon usage testing between biological groups.
when_to_use_negative:
- When the goal is differential isoform expression at the whole-transcript level —
  the article reports 'Isoform tests for DIE show better performance than exon tests',
  so IsoQuant with typeOfTest='Iso' is preferred when isoform assignments are available
- When reads are short-read-only and not full-length spliced long reads, since inclusion/exclusion
  counts per cell require long-read exon traversals
- When cell-type labels are absent — ExonQuant groups by a groupingFactor (e.g. 'Celltype')
  that must already exist in the AllInfo file
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scisorseqr
  role: R package providing ExonQuant and DiffSplicingAnalysis for exon quantification
    and differential exon usage testing in single cells
  repo: https://github.com/noush-joglekar/scisorseqr
- name: R
  role: Runtime environment for scisorseqr functions
- name: STARlong
  role: Upstream splice-aware aligner for PacBio long reads feeding the AllInfo file
    (optional alternative to Minimap2)
  repo: https://github.com/alexdobin/STAR
- name: Minimap2
  role: Upstream long-read aligner (for Oxford Nanopore or PacBio reads) whose alignments
    feed the mapping/filtering steps before ExonQuant
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:0da6866e236cc773f5f1d45547ac92fac2b821f6df4b3b445701840bd1a183b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noush-joglekar/scisorseqr
  tools:
  - Minimap2
  - R
  - STARlong
  - scisorseqr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# single-cell-long-read-exon-quantification

## Summary

Quantify per-cell-type exon inclusion/exclusion from single-cell long-read data (ScISOrSeq) using the scisorseqr ExonQuant function, then test differential exon usage between cell groups (e.g. HIPP vs PFC) with 2x2 contingency-table chi-square tests, delta-PSI filtering and Benjamini-Yekutieli FDR correction.

## When to use

Use when you have full-length spliced long reads assigned to single cells/cell types and need cell-type-resolved exon usage quantification plus exon-wise differential exon usage testing between biological groups.

## When NOT to use

- When the goal is differential isoform expression at the whole-transcript level — the article reports 'Isoform tests for DIE show better performance than exon tests', so IsoQuant with typeOfTest='Iso' is preferred when isoform assignments are available
- When reads are short-read-only and not full-length spliced long reads, since inclusion/exclusion counts per cell require long-read exon traversals
- When cell-type labels are absent — ExonQuant groups by a groupingFactor (e.g. 'Celltype') that must already exist in the AllInfo file

## Inputs

- LongReadInfo/AllInfo file (concatenated per-read information from InfoPerLongRead, with Celltype assignments)
- Tab-separated config file specifying cell-type group comparisons (e.g. HIPP vs PFC)
- Full-length spliced long reads (PacBio or ONT) assigned to single cells via short-read Seurat clustering

## Outputs

- Exon inclusion/exclusion quantification table per cell-type group (ExonQuantOutput folder)
- Differential exon usage results TSV with columns Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR
- List of significant genes (31/2132, 1.45%) passing dPI >= 0.1 and BY-corrected FDR

## How to apply

Run ExonQuant on the LongReadInfo/AllInfo file produced by InfoPerLongRead, setting groupingFactor (e.g. 'Celltype') and a minimum threshold (10 in the paper) so exons are counted as included/excluded per cell-type group. Build a tab-separated config file defining the pairwise comparisons (e.g. HIPP vs PFC), then run DiffSplicingAnalysis with typeOfTest='Exon', minNumReads=25, and is.hier=FALSE so each config line triggers a per-comparison exon test. For each exon a 2x2 chi-square test on inclusion/exclusion read counts is computed along with the delta-PSI (dPI) shift; exon p-values are corrected with Benjamini-Yekutieli for dependent tests. Report genes with FDR significance and dPI >= 0.1; in the paper this yielded 31 significant genes (1.45%) out of 2132 tested exons/genes in the HIPP-vs-PFC comparison.

## Related tools

- **scisorseqr** (R package providing ExonQuant and DiffSplicingAnalysis for exon quantification and differential exon usage testing in single cells) — https://github.com/noush-joglekar/scisorseqr
- **R** (Runtime environment for scisorseqr functions)
- **STARlong** (Upstream splice-aware aligner for PacBio long reads feeding the AllInfo file (optional alternative to Minimap2)) — https://github.com/alexdobin/STAR
- **Minimap2** (Upstream long-read aligner (for Oxford Nanopore or PacBio reads) whose alignments feed the mapping/filtering steps before ExonQuant) — https://github.com/lh3/minimap2

## Examples

```
DiffSplicingAnalysis(configFile = config, typeOfTest = 'Exon', minNumReads = 25, is.hier = FALSE)
```

## Evaluation signals

- The ExonQuantOutput folder exists and contains per-exon inclusion/exclusion counts grouped by the specified groupingFactor with threshold=10 respected
- The DiffSplicingAnalysis output TSV has exactly the columns Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR
- Only genes/exons with dPI >= 0.1 are reported as significant; FDR column values match a Benjamini-Yekutieli correction of the raw exon p-values
- Exons failing minNumReads=25 per comparison are excluded from testing; the number of tested genes (n=2132 in the paper's HIPP vs PFC run) is reported
- Significant gene fraction is in a plausible range (paper reports 1.45%, i.e. 31 of 2132)

## Limitations

- Exon-wise tests can only flag local exon usage shifts and cannot attribute them to full isoforms; isoform-level tests are reported to perform better ('Isoform tests for DIE show better performance than exon tests')
- Genes/exons with too few long reads (below threshold=10 for quantification or minNumReads=25 for testing) are dropped, so results depend on long-read throughput and matrix sparsity
- Exons with dPI < 0.1 are filtered out even if statistically significant, limiting detection of small shifts
- scisorseqr has only been tested on a CentOS x86_64 machine per the README

## Evidence

- [methods] ExonQuant call: "ExonQuant(AllInfoFile = 'LongReadInfo/AllInfo', groupingFactor = "Celltype", threshold = 10)"
- [methods] DiffSplicingAnalysis call: "DiffSplicingAnalysis(configFile = config, typeOfTest = 'Iso', minNumReads = 25, is.hier = FALSE)"
- [results] Exon-wise result: [evidence span withheld — anchor status: not_found]
- [abstract] Isoform tests outperform exon tests: "Isoform tests for DIE show better performance than exon tests."
- [readme] scisorseqr scope: [evidence span withheld — anchor status: external_doc]
