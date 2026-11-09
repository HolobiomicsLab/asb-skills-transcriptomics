---
name: tp53-intron-retention-interpretation
description: Use when a single sample shows a SCISSOR-detected local shape change
  near a TP53 splice junction and you must test whether a junction-proximal insertion
  is associated with retained-intron reads versus exonic reads.
when_to_use_negative:
- When the goal is unsupervised cohort-level discovery of shape changes (GSC/LSC)
  rather than read-level validation of an already-called insertion in a single sample
- When there is no candidate insertion or splice-site mutation at the junction to
  test against intron retention
- When the gene's reads cannot be unambiguously split into intronic (retained-intron)
  and exonic classes at the junction
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0091
tools:
- name: SCISSOR
  role: Provides junction-read utilities (extract_splicedReads/get_SRcount) and genomic
    range building (get_Ranges/build_gaf) to extract TP53 junction-spanning reads
    from the BAM
  repo: https://github.com/hyochoi/SCISSOR
- name: Rsamtools
  role: Loads and reads the sample BAM alignment for pileup and read extraction
- name: GenomicRanges
  role: Defines the TP53 exon-intron junction intervals used to select overlapping
    reads
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:4d83c4820e5abab56e07e46750c8f698af8f9315fbe3c86dfbfdb9dda152fcd4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - GenomicRanges
  - Rsamtools
  - SCISSOR
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# tp53-intron-retention-interpretation

## Summary

Read-level validation that a candidate RNA insertion at a TP53 exon-intron junction functionally drives intron retention, by tabulating insertion presence among intronic vs exonic junction reads and testing the 2x2 contingency table with Fisher's exact test.

## When to use

Use when a single sample shows a SCISSOR-detected local shape change near a TP53 splice junction and you must test whether a junction-proximal insertion is associated with retained-intron reads versus exonic reads.

## When NOT to use

- When the goal is unsupervised cohort-level discovery of shape changes (GSC/LSC) rather than read-level validation of an already-called insertion in a single sample
- When there is no candidate insertion or splice-site mutation at the junction to test against intron retention
- When the gene's reads cannot be unambiguously split into intronic (retained-intron) and exonic classes at the junction

## Inputs

- Sample-level BAM alignment of RNA-seq reads (e.g., TCGA tumor BAM such as TCGA-BA-6871-01A)
- TP53 genomic ranges from a GTF gene annotation file (e.g., Homo_sapiens.GRCh37.87.gtf)
- Known insertion variant position at the TP53 exon-intron junction

## Outputs

- tp53_intron_retention_counts.tsv containing the 2x2 count table (intronic/exonic x insertion present/absent), odds ratio, and Fisher's exact P-value
- Per-read classification of junction-overlapping reads as intronic vs exonic with insertion status

## How to apply

Load the sample BAM (e.g., TCGA-BA-6871-01A) with Rsamtools and define TP53 genomic ranges (from a GTF such as Homo_sapiens.GRCh37.87.gtf via SCISSOR's get_Ranges/build_gaf) around the affected exon-intron junction. Extract junction-spanning reads using SCISSOR's junction-read utilities (extract_splicedReads/get_SRcount), then classify each read as intronic (retained-intron side) or exonic and count insertion-positive versus insertion-negative reads in each class. In the reference case this yields 38/38 intronic reads carrying the insertion with 0% VAF among exonic reads. Assemble the 2x2 table (intronic/exonic x insertion present/absent), apply Fisher's exact test in R, and confirm P < 0.001 before writing counts, odds ratio, and P-value to tp53_intron_retention_counts.tsv. The rationale is that a functional splice-disrupting insertion should be enriched in the reads covering the retained intron rather than in canonically spliced exonic reads.

## Related tools

- **SCISSOR** (Provides junction-read utilities (extract_splicedReads/get_SRcount) and genomic range building (get_Ranges/build_gaf) to extract TP53 junction-spanning reads from the BAM) — https://github.com/hyochoi/SCISSOR
- **Rsamtools** (Loads and reads the sample BAM alignment for pileup and read extraction)
- **GenomicRanges** (Defines the TP53 exon-intron junction intervals used to select overlapping reads)

## Examples

```
install_github("hyochoi/SCISSOR"); library(SCISSOR); regions <- build_gaf(Gene=Gene, GTF.file="./Homo_sapiens.GRCh37.87.gtf")
```

## Evaluation signals

- All intronic junction-spanning reads carry the insertion (38/38 in the reference case), while exonic VAF is 0%
- Fisher's exact test on the 2x2 table yields P < 0.001
- The output tp53_intron_retention_counts.tsv contains the count table, odds ratio, and P-value
- Junction read extraction reproduces the expected intronic/exonic read classes when re-run with the same TP53 ranges and BAM

## Limitations

- LSC detection results can depend on window size choice, which may affect which junction region is flagged for validation
- Variable fragment length in TCGA RNA-seq may introduce artifactual signals that mimic shape changes
- Tuning parameters such as log normalization and a normality cutoff value likely require optimization when a different wet lab RNA-seq protocol is used

## Evidence

- [methods] SCISSOR junction-read utilities and ranges: "We first obtain pileup data matrix using `build_pileup` and genomic ranges using `get_Ranges`"
- [readme] SCISSOR structural alteration scope: "`SCISSOR` aims for unsupervised screening of a range of structural alterations in RNA-seq data."
- [results] Shape change detection procedure: "SCISSOR implements a two-step procedure, global and local, taking advantage of low-dimensional transformations and sparsity"
- [results] TP53 aberrations recaptured: "In 452 samples, SCISSOR identified 26 GSCs and 24 LSCs capturing a rich set of known (82%) and previously unrecognized (18%) genetic aberrations in TP53"
