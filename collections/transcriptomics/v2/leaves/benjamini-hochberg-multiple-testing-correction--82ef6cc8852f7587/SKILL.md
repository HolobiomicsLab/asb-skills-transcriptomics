---
name: benjamini-hochberg-multiple-testing-correction
description: Use when you have thousands of per-gene isoform (or TSS/polyA) chi-square
  p-values from a scisorseqr region-vs-region comparison and need FDR-controlled DIE
  calls with an effect-size (delta-PI) filter.
when_to_use_negative:
- When testing exon-wise inclusion/exclusion, where tests are dependent — use Benjamini–Yekutieli
  correction instead, as the paper does for the 31-gene exon result
- When input is raw per-gene isoform counts rather than per-gene chi-square test p-values;
  the correction operates on the p-value/dPI table, not on the counts directly
- When only a single pre-specified hypothesis is tested and no multiple-testing correction
  is needed
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scisorseqr
  role: Generates the per-gene p-values and dPI values (DiffSplicingAnalysis) that
    the BH/BY correction is applied to
  repo: https://github.com/noush-joglekar/scisorseqr
- name: R
  role: Package language in which the correction and downstream filtering of the results
    TSV are implemented
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:7700fd325bc6768efb1443e629408386cba7819b3ed5edbfe300edddcfa0da8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noush-joglekar/scisorseqr
  tools:
  - R
  - scisorseqr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# benjamini-hochberg-multiple-testing-correction

## Summary

Apply Benjamini–Hochberg FDR correction to the per-gene 2x2 chi-square p-values produced by scisorseqr's DiffSplicingAnalysis, then combine FDR <= 0.05 with a delta-PI >= 0.1 effect-size cutoff to call differentially isoform-expressed genes between HIPP and PFC.

## When to use

Use when you have thousands of per-gene isoform (or TSS/polyA) chi-square p-values from a scisorseqr region-vs-region comparison and need FDR-controlled DIE calls with an effect-size (delta-PI) filter.

## When NOT to use

- When testing exon-wise inclusion/exclusion, where tests are dependent — use Benjamini–Yekutieli correction instead, as the paper does for the 31-gene exon result
- When input is raw per-gene isoform counts rather than per-gene chi-square test p-values; the correction operates on the p-value/dPI table, not on the counts directly
- When only a single pre-specified hypothesis is tested and no multiple-testing correction is needed

## Inputs

- Per-gene isoform count table (bulk HIPP vs PFC) from www.isoformAtlas.com, from which n x 2 chi-square p-values and delta-PI are computed
- Per-gene TSS and polyA-site count tables for the same comparison
- Exon inclusion/exclusion quantifications from ExonQuantOutput (ExonQuant with groupingFactor='Celltype', threshold=10) when running the exon-wise test

## Outputs

- Results TSV with columns Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR
- List of significant DIE genes (FDR <= 0.05 and dPI >= 0.1), e.g. 395 genes out of 3958 tested with 141 flagged for differential TSS/polyA usage
- For exon-wise dependent testing, 31 significant genes out of 2132 after Benjamini–Yekutieli correction

## How to apply

Run DiffSplicingAnalysis on the per-gene isoform count tables (typeOfTest='Iso', minNumReads=25, is.hier=FALSE), which yields a p-value per gene from n x 2 chi-square tests on isoform proportions between hippocampus and prefrontal cortex. Apply Benjamini–Hochberg step-up correction over the 3958 tested genes to convert p-values to an FDR column, and retain genes with FDR <= 0.05 AND delta-PI (change in percent-isoform usage) >= 0.1, recovering the published 395 significant genes (9.06%). Apply the identical framework to the TSS and polyA-site count tables and flag the 141 genes whose DIE is attributable to differential TSS or polyA usage. Important decision point: when the tests are dependent — as in the exon-wise inclusion/exclusion test over 2132 exons/genes — switch to Benjamini–Yekutieli correction, which yields 31 significant genes (1.45%) for HIPP-vs-PFC.

## Related tools

- **scisorseqr** (Generates the per-gene p-values and dPI values (DiffSplicingAnalysis) that the BH/BY correction is applied to) — https://github.com/noush-joglekar/scisorseqr
- **R** (Package language in which the correction and downstream filtering of the results TSV are implemented)

## Examples

```
DiffSplicingAnalysis(configFile = config, typeOfTest = 'Iso', minNumReads = 25, is.hier = FALSE)
```

## Evaluation signals

- Gene-wise isoform test must recover 395 significant genes out of 3958 tested (9.06%) at FDR <= 0.05 with delta-PI >= 0.1
- Of the significant genes, 141 must be flagged for differential TSS or polyA-site usage under the same framework
- Results TSV must contain the columns Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR
- Exon-wise test must use Benjamini–Yekutieli (not BH) and yield 31 significant genes out of 2132 tested (1.45%)
- No gene should pass with FDR > 0.05 or dPI < 0.1

## Limitations

- BH assumes independent or positively correlated tests; the paper explicitly switches to Benjamini–Yekutieli correction for dependent exon-wise tests
- The gene-wise finding was validated for bulk HIPP-vs-PFC at postnatal day 7; whether the observed DIE patterns persist in disease case-control studies, across adult brain regions, and across species requires further studies
- Long-read clustering discrepancies may stem from sparsity of the isoform matrix due to lower long-read throughput, affecting the underlying p-values being corrected

## Evidence

- [methods] BH parameters and dPI threshold: [evidence span withheld — anchor status: not_found]
- [results] Gene-wise result reproduces 395 genes: [evidence span withheld — anchor status: not_found]
- [results] Exon-wise dependent tests need BY: [evidence span withheld — anchor status: not_found]
- [methods] DiffSplicingAnalysis generates the p-values: [evidence span withheld — anchor status: not_found]
- [readme] scisorseqr is the R package: [evidence span withheld — anchor status: not_found]
