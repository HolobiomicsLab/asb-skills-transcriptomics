---
name: bulk-hic-dci-overlap-analysis
description: Use when scHi-C DCI calls (e.g., from scHiCDiff.ZINB pvalue.adj.FDR)
  need external validation against bulk Hi-C DCI sets and differential TAD regions
  before being reported as reliable findings.
when_to_use_negative:
- Input data is simulated scHi-C (e.g., the 1CDX1 chr1 sample data) where ground-truth
  embedded DCIs can be evaluated directly, making bulk-overlap validation unnecessary
- The goal is DCI detection itself rather than post-hoc validation of already-called
  single-cell DCIs — use scHiCDiff.KS/CVM/NB/ZINB detection directly instead
- No bulk Hi-C DCI calls or differential TAD annotations are available for the conditions
  being compared, since the overlap fractions cannot be computed
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3912
tools:
- name: scHiCDiff
  role: R package providing the scHiCDiff.ZINB function used to call the single-cell
    DCIs that are subsequently validated against bulk Hi-C DCIs and differential TADs
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: Implementation language in which scHiCDiff is run and the coordinate-overlap
    and TAD-annotation steps are performed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:502d3452f2cf1edccc7612fdea524f8c7d84f304650133985db332e3fb185d76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3912
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bulk-hic-dci-overlap-analysis

## Summary

Validation skill for scHi-C differential chromatin interaction (DCI) calls: intersects single-cell ZINB-detected DCIs with DCIs identified by bulk Hi-C methods and annotates them against differential TAD regions to confirm biological support, as done in Supplementary Note 6 of the scHiCDiff paper.

## When to use

Use when scHi-C DCI calls (e.g., from scHiCDiff.ZINB pvalue.adj.FDR) need external validation against bulk Hi-C DCI sets and differential TAD regions before being reported as reliable findings.

## When NOT to use

- Input data is simulated scHi-C (e.g., the 1CDX1 chr1 sample data) where ground-truth embedded DCIs can be evaluated directly, making bulk-overlap validation unnecessary
- The goal is DCI detection itself rather than post-hoc validation of already-called single-cell DCIs — use scHiCDiff.KS/CVM/NB/ZINB detection directly instead
- No bulk Hi-C DCI calls or differential TAD annotations are available for the conditions being compared, since the overlap fractions cannot be computed

## Inputs

- Non-negative matrix of scHi-C normalized read counts (count.table; rows = bin pairs, columns = samples/cells), e.g. read via readRDS from an .rds file
- Factor vector (group) giving the two conditions to compare, corresponding to count.table columns
- Set of DCIs identified by bulk Hi-C methods (genomic bin-pair coordinates)
- Set of differential TAD regions (genomic intervals)

## Outputs

- ZINB DCI results data frame with columns bin_1, bin_2, mu_1, mu_2, theta_1, theta_2, pi_1, pi_2, norm_total_mean_1, norm_total_mean_2, norm_foldChange, chi2LR1, pvalue, pvalue.adj.FDR, Remark
- Overlap summary CSV (zinb_bulk_tad_overlap.csv) reporting the fraction of ZINB DCIs supported by bulk Hi-C DCIs and the fraction located within differential TAD regions

## How to apply

Load the real-data scHi-C normalized count matrix (count.table: rows = bin pairs, columns = samples/cells) and the two-condition factor vector (group) into R via readRDS. Run scHiCDiff.ZINB(count.table, group) to fit the zero-inflated negative binomial likelihood-ratio test per bin pair, producing mu_1/mu_2, theta_1/theta_2, pi_1/pi_2, chi2LR1, pvalue, and Benjamini & Hochberg adjusted pvalue.adj.FDR. Call significant DCIs from pvalue.adj.FDR, then use the bin_1/bin_2 genomic coordinates to (1) intersect the ZINB DCIs with DCIs identified by bulk Hi-C methods to compute the supported fraction, and (2) annotate the bin pairs against differential TAD regions to compute the fraction of DCIs inside differential TADs. Write the overlap summary to a CSV (zinb_bulk_tad_overlap.csv). The ZINB model is the appropriate detection function here because it is 'specially designated for the interaction comparison at single-cell level by taking the excessive zeros feature into consideration'.

## Related tools

- **scHiCDiff** (R package providing the scHiCDiff.ZINB function used to call the single-cell DCIs that are subsequently validated against bulk Hi-C DCIs and differential TADs) — https://github.com/wmalab/scHiCDiff
- **R** (Implementation language in which scHiCDiff is run and the coordinate-overlap and TAD-annotation steps are performed)

## Examples

```
count.table <- as.matrix(readRDS("path/sampledata/sim 1 .rds")); group <- factor(c(rep(1,50), rep(2,50))); result.zinb <- scHiCDiff.ZINB(count.table, group)
```

## Evaluation signals

- A majority of ZINB-detected DCIs are supported by DCIs identified by bulk Hi-C methods (the paper's reported real-data finding)
- A majority of ZINB-detected DCIs are located within differential TAD regions
- The ZINB result data frame contains the expected columns (mu_1, mu_2, theta_1, theta_2, pi_1, pi_2, chi2LR1, pvalue, pvalue.adj.FDR) with pvalue.adj.FDR computed by Benjamini & Hochberg FDR
- zinb_bulk_tad_overlap.csv is produced and its overlap fractions are consistent with the paper's Supplementary Note 6 reporting
- Each DCI bin pair carries valid bin_1/bin_2 genomic coordinates that successfully intersect the bulk DCI and differential TAD interval sets

## Limitations

- Validation depends on external bulk Hi-C DCI and differential TAD region sets; without these reference sets the support fractions cannot be computed
- No version numbers for scHiCDiff or R are stated in the article or README; the developmental version (scHiCDiff_1.0.tar.gz, installed from source) is what is documented
- Only the overlap outcome (majority supported) is reported as the success criterion; no quantitative threshold (e.g., a specific FDR cutoff) for calling significant DCIs is stated in the provided context

## Evidence

- [supplementary] A majority of the ZINB-detected DCIs in the real-data analyses were supported by DCIs identified by bulk Hi-C methods and were located within differential TAD regions: [evidence span withheld — anchor status: not_found]
- [intro] zero-inflated Negative Binomial(ZINB) regression models is specially designated for the interaction comparison at single-cell level: "zero-inflated Negative Binomial(ZINB) regression models is specially designated for the interaction comparison at single-cell level by taking the excessive zeros feature into consideration"
- [readme] The detection function will return a data frame containing the differential chromatin interaction (DCI) analysis results, rows are bin pairs: "The detection function will return a data frame containing the differential chromatin interaction (DCI) analysis results, rows are bin pairs and columns lists the related statistics."
- [readme] pvalue.adj.FDR       Adjusted P value of H0's pvalue using Benjamini & Hochberg's method.: "pvalue.adj.FDR       Adjusted P value of H0's pvalue using Benjamini & Hochberg's method."
