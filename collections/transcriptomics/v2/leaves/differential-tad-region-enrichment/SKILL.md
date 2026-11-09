---
name: differential-tad-region-enrichment
description: Use when validating scHi-C DCI calls (e.g. from scHiCDiff.ZINB) against
  orthogonal bulk Hi-C DCI sets and differential topologically associating domain
  (TAD) regions to assess biological support of single-cell detections.
when_to_use_negative:
- Input is a bulk Hi-C dataset rather than single-cell interaction counts — bulk comparisons
  are only used here as an external validation reference, not as the detection input
- You need to detect DCIs de novo rather than validate already-called ZINB DCIs against
  external annotations
- No differential TAD region or bulk Hi-C DCI reference sets are available for the
  analyzed conditions, so no supported-fraction can be computed
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3934
tools:
- name: scHiCDiff
  role: Provides scHiCDiff.ZINB for the zero-inflated negative binomial likelihood-ratio
    test producing per-bin-pair pvalues and pvalue.adj.FDR used as the DCI calls to
    be validated against bulk Hi-C DCIs and differential TADs
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: Implementation language in which scHiCDiff is run, e.g. loading count.table
    via readRDS and calling scHiCDiff.ZINB
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:73095aaeaefee5c572b707968a69264d7205d92498b2c41517b38a2770edcf27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# differential-tad-region-enrichment

## Summary

Annotate ZINB-detected differential chromatin interactions (DCIs) from single-cell Hi-C against bulk Hi-C DCIs and differential TAD regions to validate that a majority of detected DCIs are supported and located within differential TADs, as reported in Supplementary Note 6 of the scHiCDiff evaluation.

## When to use

Use when validating scHi-C DCI calls (e.g. from scHiCDiff.ZINB) against orthogonal bulk Hi-C DCI sets and differential topologically associating domain (TAD) regions to assess biological support of single-cell detections.

## When NOT to use

- Input is a bulk Hi-C dataset rather than single-cell interaction counts — bulk comparisons are only used here as an external validation reference, not as the detection input
- You need to detect DCIs de novo rather than validate already-called ZINB DCIs against external annotations
- No differential TAD region or bulk Hi-C DCI reference sets are available for the analyzed conditions, so no supported-fraction can be computed

## Inputs

- Normalized scHi-C count matrix (count.table: non-negative matrix, rows = bin pairs, columns = samples/cells)
- Two-condition factor vector (group) matching count.table columns
- DCI results data frame from scHiCDiff.ZINB with bin_1/bin_2 genomic coordinates and pvalue.adj.FDR column
- Differential TAD region annotations
- Bulk Hi-C method DCI calls

## Outputs

- Overlap summary table (e.g. zinb_bulk_tad_overlap.csv) reporting the fraction of ZINB DCIs supported by bulk Hi-C DCIs
- Fraction of ZINB-detected DCIs located within differential TAD regions

## How to apply

Run the ZINB detection per bin pair (scHiCDiff.ZINB on the normalized count.table matrix and two-condition group factor), obtaining mu_1/mu_2, theta_1/theta_2, pi_1/pi_2, chi2LR1, pvalue, and Benjamini & Hochberg adjusted pvalue (pvalue.adj.FDR). Call significant DCIs from pvalue.adj.FDR and use the bin_1/bin_2 genomic coordinates to intersect the ZINB DCIs with DCIs identified by bulk Hi-C methods, computing the supported fraction. Annotate the ZINB DCI bin pairs against differential TAD regions and compute the fraction of DCIs within differential TADs. Write the overlap summary (e.g. to zinb_bulk_tad_overlap.csv) and judge success by whether a majority of ZINB-detected DCIs are both supported by bulk Hi-C DCIs and located within differential TAD regions, consistent with the paper's real-data finding.

## Related tools

- **scHiCDiff** (Provides scHiCDiff.ZINB for the zero-inflated negative binomial likelihood-ratio test producing per-bin-pair pvalues and pvalue.adj.FDR used as the DCI calls to be validated against bulk Hi-C DCIs and differential TADs) — https://github.com/wmalab/scHiCDiff
- **R** (Implementation language in which scHiCDiff is run, e.g. loading count.table via readRDS and calling scHiCDiff.ZINB)

## Examples

```
count.table <- readRDS("path/sampledata/sim 1 .rds"); group <- factor(c(rep(1,50), rep(2,50))); result.zinb <- scHiCDiff.ZINB(count.table, group)
```

## Evaluation signals

- A majority of ZINB-detected DCIs are supported by DCIs identified by bulk Hi-C methods
- A majority of ZINB-detected DCIs are located within differential TAD regions
- DCI results data frame contains the expected ZINB columns (mu_1, mu_2, theta_1, theta_2, pi_1, pi_2, chi2LR1, pvalue, pvalue.adj.FDR)
- Called DCIs have non-empty bin_1/bin_2 coordinates permitting genomic intersection with bulk DCIs and TAD regions
- Overlap summary file (zinb_bulk_tad_overlap.csv) is produced with per-category overlap fractions consistent with the reported real-data finding

## Limitations

- Validation is reported for real-data analyses in Supplementary Note 6; the fraction supported is not guaranteed to be a majority for arbitrary datasets or parameter choices
- Requires external bulk Hi-C DCI calls and differential TAD regions for the same conditions; without them the overlap fractions cannot be computed
- No version numbers stated for scHiCDiff or R in the README, so reproducibility of the exact ZINB fit may vary across installs

## Evidence

- [readme] ZINB outputs and FDR: "pvalue.adj.FDR       Adjusted P value of H0's pvalue using Benjamini & Hochberg's method."
- [intro] ZINB design rationale: "zero-inflated Negative Binomial(ZINB) regression models is specially designated for the interaction comparison at single-cell level by taking the excessive zeros feature into consideration"
- [abstract] Main finding on ZINB DCIs: "scHiCDiff, especially the zero-inflated negative binomial model option, can effectively detect reliable and consistent single-cell DCIs between two conditions"
- [methods] R snippet for detection calls: "result.zinb <- scHiCDiff.ZINB(count.table,group)"
- [readme] Binary coordinate fields used for intersection: "bin_1,bin_2          The interacting region of the bin pair."
