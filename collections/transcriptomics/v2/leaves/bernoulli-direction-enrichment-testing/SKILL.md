---
name: bernoulli-direction-enrichment-testing
description: Use when an analysis reduces to binary directional calls per gene or
  exon — TSS shift direction in pairwise DIE comparisons, or same/opposite delta-PSI
  between single-cell and spatial measurements — and an exact enrichment p-value against
  a 50/50 null is required.
when_to_use_negative:
- When effect magnitudes, not just direction, are the quantity of interest — the Bernoulli
  test discards delta-PSI magnitude and must be complemented by Pearson correlation
  of continuous values
- When the underlying categories are not equiprobable under the null (directions a
  priori biased, e.g. annotational upstream-TSS preference), so the p = 0.5 binomial
  assumption is violated
- When events are independent tests of magnitude rather than binary concordance —
  use the pairwise DIE fraction framework (deltaPI = 0.1) instead
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Runs the exact two-sided Bernoulli/binomial test on direction counts and produces
    the summary CSV
- name: scisorseqr
  role: Supplies the DiffSplicingAnalysis isoform/TSS results tables whose directional
    calls feed the binomial test; also applicable to long-read spatial transcriptomics
    exon quantification
  repo: github.com/noush-joglekar/scisorseqr
- name: pandas
  role: Loads the IsoQuantOutput-style count tables and assembles the per-exon concordance
    and TSS-direction summary CSVs
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20343-5
    title: A spatially resolved brain region- and cell type-specific isoform atlas
      of the postnatal mouse brain
  source_content_hash: sha256:89724cce2f7dc7d1eb5cffca7ec3a491dd62e5d05892782cb394639537b1c7f5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DiffSplicingAnalysis
  - IsoQuant
  - R
  - pandas
  - scisorseqr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# bernoulli-direction-enrichment-testing

## Summary

An exact two-sided Bernoulli/binomial test that assesses whether binary directional outcomes in isoform regulation data (e.g. upstream vs downstream TSS shifts, same vs opposite delta-PSI direction) are enriched against a 50% null expectation, as used to validate the CPE upstream-TSS signature (70/93 genes, p = 3e-7) and single-cell-vs-spatial delta-PSI concordance (85% of 40 exons, p <= 3.5e-6).

## When to use

Use when an analysis reduces to binary directional calls per gene or exon — TSS shift direction in pairwise DIE comparisons, or same/opposite delta-PSI between single-cell and spatial measurements — and an exact enrichment p-value against a 50/50 null is required.

## When NOT to use

- When effect magnitudes, not just direction, are the quantity of interest — the Bernoulli test discards delta-PSI magnitude and must be complemented by Pearson correlation of continuous values
- When the underlying categories are not equiprobable under the null (directions a priori biased, e.g. annotational upstream-TSS preference), so the p = 0.5 binomial assumption is violated
- When events are independent tests of magnitude rather than binary concordance — use the pairwise DIE fraction framework (deltaPI = 0.1) instead

## Inputs

- Per-gene TSS-direction classification table for CPE-relevant genes (upstream/downstream labels from DiffSplicingAnalysis typeOfTest='TSS' results with Gene, pvals, dPI, maxDeltaPI_ix1, maxDeltaPI_ix2, FDR columns)
- Per-exon paired delta-PSI vectors: single-cell HIPP-vs-PFC dPSI and spatial 10X Visium + ONT long-read dPSI for 40 region-specific exons
- Null expectation (typically p = 0.5 equiprobable directions)

## Outputs

- cpe_tss_direction_summary.csv with TSS-direction fields, 70/93 upstream count, and Bernoulli p-value (<= 3e-7)
- exon_delta_psi_concordance.csv with 40 rows of exon ID, single-cell dPSI, spatial dPSI, and same-direction flag
- Two-sided exact binomial p-value and success fraction (k/n) per direction-enrichment test

## How to apply

First convert each tested event into a directional binary outcome: in the CPE analysis, classify TSS shifts of the 93 CPE-relevant genes (from DiffSplicingAnalysis TSS tests with minNumReads=25, deltaPI=0.1 threshold) as upstream or downstream; in the spatial validation, flag each of the 40 region-specific exons as same-direction if single-cell and spatial delta-PSI (HIPP minus PFC) share sign. Count successes k out of n and run a two-sided exact binomial test with expected probability 0.5 (e.g. R's binom.test). For the CPE signature this yields 70/93 upstream with p = 3e-7; for the spatial concordance this yields 34/40 same-direction (85%) with p <= 3.5e-6. Because the binomial test discards magnitude, pair it with a Pearson correlation of the continuous delta-PSI values (r^2 = 0.6, validated in [0.5, 0.7]) when effect-size agreement also matters.

## Related tools

- **R** (Runs the exact two-sided Bernoulli/binomial test on direction counts and produces the summary CSV)
- **scisorseqr** (Supplies the DiffSplicingAnalysis isoform/TSS results tables whose directional calls feed the binomial test; also applicable to long-read spatial transcriptomics exon quantification) — github.com/noush-joglekar/scisorseqr
- **pandas** (Loads the IsoQuantOutput-style count tables and assembles the per-exon concordance and TSS-direction summary CSVs)

## Examples

```
bt <- binom.test(70, 93, p = 0.5, alternative = "two.sided"); bt$p.value  # expect 3e-7
```

## Evaluation signals

- CPE TSS-direction summary reports exactly 70/93 upstream-TSS genes with Bernoulli p <= 3e-7
- exon_delta_psi_concordance.csv contains exactly 40 rows with same-direction fraction ~0.85 and binomial p <= 3.5e-6
- Pearson r^2 between single-cell and spatial delta-PSI vectors falls in [0.5, 0.7]
- Direction labels are internally consistent: each gene/exon has exactly one upstream/downstream (or same/opposite) binary call derived from the sign of dPSI or maxDeltaPI_ix
- Test is symmetric under relabeling: swapping direction labels maps k to n-k and preserves the two-sided p-value

## Limitations

- The binomial test is sensitive to sparsity: the article notes reproducibility issues 'could be partially due to the sparsity of the isoform matrix due to lower long-read throughput', which can shrink n and destabilize direction calls
- Binary concordance alone does not demonstrate quantitative agreement — the 85% same-direction result required the companion r^2 = 0.6 correlation to establish magnitude agreement
- Conclusions are drawn from small event sets (n = 93 genes, n = 40 exons); enrichment p-values assume independent, identically-distributed directional outcomes

## Evidence

- [abstract] CPE upstream-TSS preference is the biology the 70/93 Bernoulli test validates: "Cell types indigenous to one anatomic structure display distinctive DIE, e.g. the choroid plexus epithelium manifests distinct transcription-start-site usage."
- [other] deltaPI threshold converts continuous DIE results into the direction calls tested: [evidence span withheld — anchor status: not_found]
- [other] The Bernoulli test of same-direction delta-PSI changes across the 40 exons: [evidence span withheld — anchor status: not_found]
- [readme] scisorseqr generates the per-cell-type and spatial quantifications underlying the binary direction calls: [evidence span withheld — anchor status: external_doc]
