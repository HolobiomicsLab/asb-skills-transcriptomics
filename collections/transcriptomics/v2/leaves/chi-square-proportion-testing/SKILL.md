---
name: chi-square-proportion-testing
description: Use when comparing categorical read-assignment proportions (isoform,
  TSS, or polyA-site usage) between two biological groups per gene, and you need a
  gene-level significance call with FDR control and a minimum effect size (delta-PI).
when_to_use_negative:
- The question is differential gene or exon expression magnitude rather than a shift
  in isoform/TSS/polyA proportions — use ExonQuant or standard DE tools instead
- A cell-type-hierarchical comparison is required; the article's non-hierarchical
  setting (is.hier=FALSE) applies to the bulk two-region test
- Genes have fewer than minNumReads=25 long reads, where chi-square proportion tests
  are underpowered/unreliable
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_0203
tools:
- name: scisorseqr
  role: R package providing DiffSplicingAnalysis for per-gene chi-square tests on
    isoform/TSS/polyA proportions
  repo: github.com/noush-joglekar/scisorseqr
- name: R
  role: Runtime environment and language for scisorseqr and downstream BH correction
    / delta-PI filtering
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:67f07ff15a66d40597afcf71a04e46f7d3ef1c00157b4936792d33190b98b186
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - scisorseqr
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# chi-square-proportion-testing

## Summary

Per-gene n x 2 chi-square testing of isoform (and TSS/polyA-site) proportion differences between conditions, implemented via scisorseqr's DiffSplicingAnalysis, with Benjamini-Hochberg correction and a delta-PI effect-size filter. This is the statistical core of the HIPP-vs-PFC differential isoform expression (DIE) test in the postnatal mouse brain isoform atlas.

## When to use

Use when comparing categorical read-assignment proportions (isoform, TSS, or polyA-site usage) between two biological groups per gene, and you need a gene-level significance call with FDR control and a minimum effect size (delta-PI).

## When NOT to use

- The question is differential gene or exon expression magnitude rather than a shift in isoform/TSS/polyA proportions — use ExonQuant or standard DE tools instead
- A cell-type-hierarchical comparison is required; the article's non-hierarchical setting (is.hier=FALSE) applies to the bulk two-region test
- Genes have fewer than minNumReads=25 long reads, where chi-square proportion tests are underpowered/unreliable

## Inputs

- Per-gene isoform count table (genes x isoforms x condition), e.g. bulk HIPP vs PFC from www.isoformAtlas.com
- Per-gene TSS usage count table for the same genes/conditions
- Per-gene polyA-site usage count table for the same genes/conditions
- scisorseqr configuration file specifying the two-group comparison

## Outputs

- Results TSV of per-gene chi-square tests with p-values, BH-adjusted FDR, and delta-PI per gene
- Set of significant DIE genes (expected 395 of 3958 for HIPP-vs-PFC at FDR <= 0.05, delta-PI >= 0.1)
- Subset of DIE genes attributable to differential TSS or polyA-site usage (141 genes)

## How to apply

Load the per-gene isoform count table (e.g. bulk hippocampus vs prefrontal cortex counts from www.isoformAtlas.com) and run scisorseqr's DiffSplicingAnalysis with typeOfTest='Iso', minNumReads=25, and is.hier=FALSE, which performs a per-gene n x 2 chi-square test on isoform proportions between the two regions. Apply Benjamini-Hochberg multiple-testing correction at FDR <= 0.05 and require delta-PI >= 0.1 (change in percent-isoform usage) to flag differentially isoform-expressed genes. Repeat the identical chi-square framework on per-gene TSS and polyA-site count tables to attribute DIE to differential transcription-start-site or polyA usage. minNumReads=25 guards against testing genes with too few long reads, where chi-square approximation is unreliable. As a sanity anchor, the published HIPP-vs-PFC bulk test recovered 395 significant genes out of 3958 tested (9.06%), of which 141 showed differential TSS or polyA-site usage.

## Related tools

- **scisorseqr** (R package providing DiffSplicingAnalysis for per-gene chi-square tests on isoform/TSS/polyA proportions) — github.com/noush-joglekar/scisorseqr
- **R** (Runtime environment and language for scisorseqr and downstream BH correction / delta-PI filtering)

## Examples

```
DiffSplicingAnalysis(configFile = config, typeOfTest = 'Iso', minNumReads = 25, is.hier = FALSE)
```

## Evaluation signals

- Results TSV contains exactly 395 significant genes out of 3958 tested (9.06%) for HIPP vs PFC at FDR <= 0.05 and delta-PI >= 0.1
- 141 of the significant genes are flagged as having DIE attributable to differential TSS or polyA-site usage under the same chi-square framework
- No gene with fewer than 25 reads per condition is called significant (minNumReads=25 filter respected)
- Adjusted p-values are monotonically consistent with Benjamini-Hochberg correction over the 3958 tested genes
- Reported delta-PI for every flagged gene is >= 0.1

## Limitations

- Chi-square proportion tests on sparse long-read counts are underpowered; sparsity of the isoform matrix due to lower long-read throughput can cause discrepancies
- The published result is for bulk HIPP vs PFC at postnatal day 7 in mouse; generalizability to adult brain regions, disease case-control settings, or other species is untested
- Significance depends on the delta-PI >= 0.1 threshold; smaller isoform-usage shifts will not be reported even if statistically significant

## Evidence

- [methods] DiffSplicingAnalysis call parameters: "DiffSplicingAnalysis(configFile = config, typeOfTest = 'Iso', minNumReads = 25, is.hier = FALSE)"
- [discussion] 395 DIE genes between HIPP and PFC: "Region-specific isoform testing revealed 395 genes with DIE between hippocampus and PFC, partially caused by isoforms novel to GENCODE."
- [readme] scisorseqr purpose and applicability: [evidence span withheld — anchor status: external_doc]
- [abstract] Isoform tests outperform exon tests: "Isoform tests for DIE show better performance than exon tests."
