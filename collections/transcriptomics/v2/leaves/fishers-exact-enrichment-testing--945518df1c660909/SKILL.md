---
name: fishers-exact-enrichment-testing
description: Use when a single-cell dataset has categorical clone assignments (e.g.,
  mitochondrial clones from vireoSNP at probability >0.8) and independent cell labels
  (tumor vs non-tumor, CNV subclones) and you need to test enrichment of one clone
  in one label.
when_to_use_negative:
- When clone assignments are not confident — cells with vireoSNP assignment probability
  ≤0.8 should be excluded before testing, not forced into the contingency table
- When the label is continuous or multi-class without a prespecified contrast — Fisher's
  exact test on a 2x2 table requires two clones and two label categories
- When clone-label association is already established by genotype overlap rather than
  cell-level contingency (e.g., identical variant profiles), a test would be circular
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3050
tools:
- name: vireoSNP
  role: Provides the confident per-cell clone assignments (probability >0.8) that
    populate the contingency table
  repo: https://github.com/single-cell-genetics/vireo
- name: MQuad
  role: Selects the clonally informative mtDNA variants (ΔBIC/Kneedle cutoff) that
    define the clones being tested for enrichment
  repo: https://github.com/single-cell-genetics/MQuad
- name: cellsnp-lite
  role: Generates the SNP-by-cell AD/DP matrices from the TNBC1 BAM that feed MQuad
    upstream of clone assignment
  repo: https://github.com/single-cell-genetics/cellsnp-lite
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:35e306cf125b10408dd7216ef399840d1c47c2af74dbcab32571624150734272
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/vireo
  tools:
  - MQuad
  - cellsnp-lite
  - vireoSNP
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3050
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# fishers-exact-enrichment-testing

## Summary

A two-sided Fisher's exact test to assess whether a specific mitochondrial clone (e.g., mtDNA clone 1 assigned by MQuad/vireoSNP) is enriched among annotated cells (e.g., tumor cells) relative to another clone, using a 2x2 contingency table of clone-by-label counts.

## When to use

Use when a single-cell dataset has categorical clone assignments (e.g., mitochondrial clones from vireoSNP at probability >0.8) and independent cell labels (tumor vs non-tumor, CNV subclones) and you need to test enrichment of one clone in one label.

## When NOT to use

- When clone assignments are not confident — cells with vireoSNP assignment probability ≤0.8 should be excluded before testing, not forced into the contingency table
- When the label is continuous or multi-class without a prespecified contrast — Fisher's exact test on a 2x2 table requires two clones and two label categories
- When clone-label association is already established by genotype overlap rather than cell-level contingency (e.g., identical variant profiles), a test would be circular

## Inputs

- Per-cell clone assignment table from vireoSNP (clone ID plus assignment probability, filtered at probability >0.8)
- Per-cell external label table (e.g., tumor/non-tumor calls or CNV subclone labels from the original TNBC1 study)
- 2x2 contingency table of counts (clone 1 vs clone 0 crossed with label of interest)

## Outputs

- Two-sided Fisher's exact test p-value (TNBC1 expectation: p ≈ 6e-5 for clone 1 tumor enrichment)
- Odds ratio quantifying clone-1 enrichment among tumor cells
- Confusion/overlap matrix between confidently assigned mitochondrial clones and CNV subclones

## How to apply

After clones are assigned (e.g., vireoSNP on MQuad-selected mtDNA SNPs, keeping cells with assignment probability >0.8), cross-tabulate confidently assigned cells by clone against the external label (e.g., tumor vs normal, or CNV subclone). Build a 2x2 contingency table comparing clone 1 vs clone 0 against the label of interest and run a two-sided Fisher's exact test; in the TNBC1 sample this tests enrichment of mitochondrial clone 1 among tumor cells relative to clone 0. Rationale: clone sizes are small and counts are discrete, so an exact conditional test on the hypergeometric distribution is appropriate rather than a chi-squared approximation. Interpret the odds ratio and p-value together — in TNBC1, mitochondrial clone 1 was tumor-enriched (p ≈ 6e-5) and overlapped almost exclusively with CNV subclone B, indicating a small mtDNA-defined subclone nested within a CNV clone.

## Related tools

- **vireoSNP** (Provides the confident per-cell clone assignments (probability >0.8) that populate the contingency table) — https://github.com/single-cell-genetics/vireo
- **MQuad** (Selects the clonally informative mtDNA variants (ΔBIC/Kneedle cutoff) that define the clones being tested for enrichment) — https://github.com/single-cell-genetics/MQuad
- **cellsnp-lite** (Generates the SNP-by-cell AD/DP matrices from the TNBC1 BAM that feed MQuad upstream of clone assignment) — https://github.com/single-cell-genetics/cellsnp-lite

## Examples

```
from scipy.stats import fisher_exact; odds_ratio, p = fisher_exact([[clone1_tumor, clone0_tumor], [clone1_nontumor, clone0_nontumor]], alternative='two-sided')
```

## Evaluation signals

- Confusion matrix between mtDNA clones and CNV subclones shows mitochondrial clone 1 overlapping almost exclusively with CNV subclone B
- Two-sided Fisher's exact test on clone 1 (tumor vs clone 0) yields p ≈ 6e-5 in the TNBC1 dataset
- Only cells with vireoSNP assignment probability >0.8 are counted (1097 of 1146 cells, 95.8%, in TNBC1)
- Exactly two informative mtDNA variants underlie the clone assignments before the test is run
- Contingency table row/column margins sum to the number of confidently assigned cells

## Limitations

- The test assumes independent cells and a prespecified 2x2 contrast; it cannot resolve finer substructure (TNBC1's clone 1 is a small subclone within CNV subclone B, so CNV and mtDNA resolutions differ)
- Excluding low-probability cells (probability ≤0.8) can reduce the tested population (1146 → 1097 cells), and the unassigned cells' labels are untested
- A significant p-value indicates enrichment, not causation of tumor status by the mtDNA clone

## Evidence

- [other] Two-sided Fisher's exact test for clone-1 tumor enrichment: [evidence span withheld — anchor status: not_found]
- [other] Confidence filtering before testing: [evidence span withheld — anchor status: not_found]
- [other] Expected TNBC1 result of the test: [evidence span withheld — anchor status: not_found]
- [other] Clone assignment scale entering the test: [evidence span withheld — anchor status: not_found]
