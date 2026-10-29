---
name: single-cell-hic-cell-quality-control
description: Use when preparing raw single-cell Hi-C count data (e.g. oocyte vs zygote
  cells) for scHiCDiff DCI testing and you need to filter low-contact cells and restrict
  contacts to a short genomic distance range.
when_to_use_negative:
- Input is already a QC-filtered, normalized count.table ready for scHiCDiff detection
  functions — re-filtering could discard valid cells
- Analysis targets long-range (>10 Mb) chromatin contacts; the 0-10 Mb distance restriction
  would remove the bin pairs of interest
- Bulk Hi-C contact matrices are supplied rather than single-cell data; the >5000
  contacts-per-cell threshold is defined for single-cell libraries
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3688
- http://edamontology.org/topic_0602
tools:
- name: scHiCDiff
  role: R package whose detection functions (scHiCDiff.KS/CVM/NB/ZINB) consume the
    QC-filtered count.table; the QC step defines the cell and distance-range inputs
    those functions expect
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: Language used to load the count table, apply the >5000 nondiagonal contact
    and 0-10 Mb range filters, and assemble count.table and group before calling scHiCDiff
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:96ef9f517459cb0273070b04966f274433314b1fc44582b3333c9004a637946d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3688
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-hic-cell-quality-control

## Summary

Quality-control filter for single-cell Hi-C datasets prior to differential chromatin interaction (DCI) detection: retain only cells with sufficient contact counts and restrict bin pairs to a defined genomic distance range. Applied to the Flyamer 2017 mouse oocyte vs zygote scHi-C data before running scHiCDiff.

## When to use

Use when preparing raw single-cell Hi-C count data (e.g. oocyte vs zygote cells) for scHiCDiff DCI testing and you need to filter low-contact cells and restrict contacts to a short genomic distance range.

## When NOT to use

- Input is already a QC-filtered, normalized count.table ready for scHiCDiff detection functions — re-filtering could discard valid cells
- Analysis targets long-range (>10 Mb) chromatin contacts; the 0-10 Mb distance restriction would remove the bin pairs of interest
- Bulk Hi-C contact matrices are supplied rather than single-cell data; the >5000 contacts-per-cell threshold is defined for single-cell libraries

## Inputs

- Normalized single-cell Hi-C count table (matrix of bin pairs x cells/samples, e.g. from the Flyamer 2017 mouse oocyte vs zygote dataset)
- Group factor vector labeling the two conditions to be compared (oocyte vs zygote)

## Outputs

- Filtered count.table matrix containing only cells with >5000 nondiagonal intra-chromosomal contacts and only bin pairs within the 0-10 Mb contact distance range
- Matching group factor vector for the retained cells

## How to apply

Load the normalized single-cell Hi-C count table (rows = bin pairs, columns = samples/cells) and retain only cells with more than 5000 total nondiagonal intra-chromosomal contacts. Restrict bin pairs to contacts within the 0–10 Mb range, filtering out longer-range interactions. Assemble the filtered matrix as count.table and a group factor vector labeling the two conditions (oocyte vs zygote). These QC criteria were the ones applied to the real Flyamer 2017 scHi-C datasets before running the scHiCDiff.KS, scHiCDiff.CVM, scHiCDiff.NB and scHiCDiff.ZINB detection functions.

## Related tools

- **scHiCDiff** (R package whose detection functions (scHiCDiff.KS/CVM/NB/ZINB) consume the QC-filtered count.table; the QC step defines the cell and distance-range inputs those functions expect) — https://github.com/wmalab/scHiCDiff
- **R** (Language used to load the count table, apply the >5000 nondiagonal contact and 0-10 Mb range filters, and assemble count.table and group before calling scHiCDiff)

## Examples

```
count.table <- as.matrix(readRDS("path/sampledata/sim 1 .rds")); group <- factor(c(rep(1,50), rep(2,50)))
```

## Evaluation signals

- Every retained cell has more than 5000 total nondiagonal intra-chromosomal contacts in the filtered count table
- All retained bin pairs span at most 10 Mb (contacts within the 0-10 Mb range); no longer-range pairs remain
- Diagonal (self-ligating) entries are excluded from the contact total used for the cell-level threshold
- Dimensions of filtered count.table match the number of retained cells; group factor vector length equals the number of retained columns
- Downstream scHiCDiff detection functions return a data frame with rows of bin pairs and the required statistics columns (bin_1, bin_2, pvalue, adjusted FDR p-value)

## Limitations

- The >5000 nondiagonal contact and 0-10 Mb distance criteria were specified for the Flyamer 2017 mouse oocyte vs zygote scHi-C data; other datasets may require recalibrated thresholds
- Filtering to the 0-10 Mb range removes long-range interactions, so DCIs at larger genomic distances will not be detected downstream
- No version numbers for scHiCDiff or R are stated in the article or README, so exact software provenance must be recorded separately

## Evidence

- [other] cell QC threshold: [evidence span withheld — anchor status: not_found]
- [readme] detection inputs: "count.table      A non-negative  matrix of scHi-C normalized read counts.The rows of the 
                 matrix are bin pair and columns are samples/cells."
- [readme] detection output schema: "The detection function will return a data frame containing the differential chromatin interaction (DCI) analysis results, rows are bin pairs and columns lists the related statistics."
- [intro] detection functions: "the other type is the detection function (scHiCDiff.KS, scHiCDiff.CVM, scHiCDiff.NB and scHiCDiff.ZINB)"
