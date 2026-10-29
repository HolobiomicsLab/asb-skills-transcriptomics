---
name: fishers-exact-contingency-testing
description: Use when discrete, mutually exclusive counts form a 2x2 table needing
  an exact P-value — e.g. insertion-carrying vs wild-type reads split by intronic/exonic
  class at a SCISSOR-flagged junction, or co-altered gene counts between cohorts.
when_to_use_negative:
- The variable is continuous base-level coverage or pileup intensity rather than discrete
  counts — SCISSOR's Scissor function on build_pileup output (shape change detection)
  is the appropriate route, not an exact test on counts.
- Counts are not independent or marginals are not verified against the extracted read
  set (e.g. paired reads from one fragment counted twice), which violates the exact
  test's assumptions.
- The goal is cohort-level shape outlier discovery rather than testing a pre-specified
  contingency table of co-altered gene counts.
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_3308
tools:
- name: SCISSOR
  role: Provides junction-read extraction utilities (extract_splicedReads/get_SRcount,
    build_pileup/get_Ranges) that produce the read-level counts classified into the
    contingency table
  repo: https://github.com/hyochoi/SCISSOR
- name: Rsamtools
  role: R/Bioconductor dependency used to access the BAM alignment when extracting
    junction-overlapping reads
- name: GenomicRanges
  role: R/Bioconductor dependency used to represent the TP53 genomic ranges used as
    inputs for the Scissor function
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:e2bb38f0cf29b6e4cf0410d7ed82e5a509d353f0d90e8f205496238decab6ba2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - GenomicRanges
  - Rsamtools
  - SCISSOR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# fishers-exact-contingency-testing

## Summary

Exact test on 2x2 (r x c) contingency tables of discrete counts, used in the SCISSOR framework to test read-level insertion support at a flagged splice junction (intronic vs exonic x insertion present vs absent) and to compare the number of co-altered genes in every pair of cohorts, yielding odds ratios and exact P-values.

## When to use

Use when discrete, mutually exclusive counts form a 2x2 table needing an exact P-value — e.g. insertion-carrying vs wild-type reads split by intronic/exonic class at a SCISSOR-flagged junction, or co-altered gene counts between cohorts.

## When NOT to use

- The variable is continuous base-level coverage or pileup intensity rather than discrete counts — SCISSOR's Scissor function on build_pileup output (shape change detection) is the appropriate route, not an exact test on counts.
- Counts are not independent or marginals are not verified against the extracted read set (e.g. paired reads from one fragment counted twice), which violates the exact test's assumptions.
- The goal is cohort-level shape outlier discovery rather than testing a pre-specified contingency table of co-altered gene counts.

## Inputs

- BAM alignment file (e.g. TCGA-BA-6871-01A RNA-seq BAM)
- TP53 genomic ranges (from build_gaf/GTF via get_Ranges)
- 2x2 contingency table of counts (intronic/exonic x insertion present/absent) built from classified junction reads

## Outputs

- Exact P-value from fisher.test (expected P < 0.001 for the TP53 junction finding)
- Odds ratio for the 2x2 table
- Count table written to tp53_intron_retention_counts.tsv

## How to apply

Extract reads spanning the TP53 exon-intron junction from the sample BAM (e.g. TCGA-BA-6871-01A) using SCISSOR's junction-read utilities (extract_splicedReads/get_SRcount) with TP53 genomic ranges from build_gaf/get_Ranges, then classify each junction-overlapping read as intronic (retained-intron side) or exonic and tally insertion-carrying versus wild-type reads per class. Assemble the 2x2 contingency table (intronic/exonic x insertion present/absent — expected 38/38 intronic reads carry the insertion, with exonic VAF = 0%) and run fisher.test() in R on that table. Report the odds ratio and exact P-value together with the count table written to tp53_intron_retention_counts.tsv; confirm P < 0.001 only after checking that table marginals equal the number of extracted junction reads. The same exact-test machinery is applied at the cohort level in SCISSOR, where Fisher's exact test is applied to the number of co-altered genes in every pair of cohorts for unsupervised concordance analysis.

## Related tools

- **SCISSOR** (Provides junction-read extraction utilities (extract_splicedReads/get_SRcount, build_pileup/get_Ranges) that produce the read-level counts classified into the contingency table) — https://github.com/hyochoi/SCISSOR
- **Rsamtools** (R/Bioconductor dependency used to access the BAM alignment when extracting junction-overlapping reads)
- **GenomicRanges** (R/Bioconductor dependency used to represent the TP53 genomic ranges used as inputs for the Scissor function)

## Examples

```
fisher.test(matrix(c(38, 0, 0, n_exonic), nrow=2, byrow=TRUE, dimnames=list(c("intronic","exonic"), c("insertion","no_insertion"))))
```

## Evaluation signals

- 2x2 table marginals sum to the number of junction-overlapping reads extracted from the BAM
- Expected cell counts reproduce the finding: 38/38 intronic reads carry the insertion and 0% VAF among exonic reads
- fisher.test returns P < 0.001 for the assembled table
- Output file tp53_intron_retention_counts.tsv contains the count table, odds ratio, and P-value
- At cohort level, every cohort pair yields one Fisher's exact test on co-altered gene counts

## Limitations

- Only 59% of splice site mutations annotated in the public HNSC repository were associated with a statistically significant shape change, so a non-significant test does not rule out a real event.
- Variable fragment length may introduce artifactual signals detectable by SCISSOR, which can distort read-level counts feeding the contingency table.
- SCISSOR tuning parameters (log normalization, normality cutoff) likely require optimization for different RNA-seq protocols, affecting which junctions get flagged for testing.

## Evidence

- [results] Fisher's exact test on co-altered genes in cohort pairs: "Fisher’s exact test applied to the number of co-altered genes in every pair of the cohorts"
- [methods] Pileup and genomic ranges feed the Scissor function: "We first obtain pileup data matrix using `build_pileup` and genomic ranges using `get_Ranges` that will be used as inputs for the function `Scissor`"
- [other] TP53 junction insertion finding reproduced by the task: [evidence span withheld — anchor status: not_found]
- [readme] SCISSOR works from a base-level pileup of aligned reads: "`SCISSOR` considers a novel shape property of aligned short read data through a base-level pileup file."
- [results] Not all splice site mutations reach significance: "only 59% of mutations annotated as splice site mutations in the public repository of HNSC mutations were associated with a statistically significant shape change"
