---
name: genome-annotation-statistics-verification
description: Use when reproducing a published annotation result (e.g. peaks2utr 3'
  UTR counts on T. brucei scRNA-seq) and you must confirm your output GFF3/GTF matches
  reported totals and new/altered/unchanged partitions.
when_to_use_negative:
- The goal is de novo gene prediction or annotating features other than 3' UTRs —
  peaks2utr only annotates 3' untranslated regions from BAM reads against a given
  canonical annotation.
- No canonical GFF/GTF annotation is available to serve as the baseline for classifying
  UTRs as new, altered, or unchanged.
- The input BAM is not from a protocol where signal concentrates at UTR distal ends
  (e.g. not 10x Chromium scRNA-seq), since truncation-point pileups driving the counts
  depend on soft-clipped polyA-tail reads.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: peaks2utr
  role: Generates the 3' UTR annotation (GFF3/GTF output with source=peaks2utr) whose
    counts are verified; invoked as peaks2utr <GFF_IN> <BAM_IN> with -p for parallel
    workers and -o for output
  repo: https://github.com/haessar/peaks2utr
- name: GenomeTools
  role: Optional post-processing/normalization of the output GFF3 prior to tallying
    and comparison
  repo: https://github.com/genometools/genometools#building-and-installation
- name: bedtools
  role: Required system dependency for running peaks2utr, installed with 'apt-get
    install bedtools'
  repo: https://bedtools.readthedocs.io/en/latest/content/installation.html
- name: MACS3
  role: Internal dependency used by peaks2utr to call broad peaks of significant read
    coverage in the BAM that form the basis for 3' UTR annotation
  repo: https://pypi.org/project/MACS3/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:9c9847e2d2abe3eadb46e9989bf7c1183de6600376ed6278c0b21c57460887d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - GenomeTools
  - MACS3
  - bedtools
  - peaks2utr
  - pip
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# genome-annotation-statistics-verification

## Summary

Verify a reproducible genome annotation run by tallying the annotated features (e.g. 3' UTRs emitted by peaks2utr with source=peaks2utr) against published counts, classifying them as new, altered, or unchanged relative to a canonical annotation, and checking gene coverage proportions.

## When to use

Use when reproducing a published annotation result (e.g. peaks2utr 3' UTR counts on T. brucei scRNA-seq) and you must confirm your output GFF3/GTF matches reported totals and new/altered/unchanged partitions.

## When NOT to use

- The goal is de novo gene prediction or annotating features other than 3' UTRs — peaks2utr only annotates 3' untranslated regions from BAM reads against a given canonical annotation.
- No canonical GFF/GTF annotation is available to serve as the baseline for classifying UTRs as new, altered, or unchanged.
- The input BAM is not from a protocol where signal concentrates at UTR distal ends (e.g. not 10x Chromium scRNA-seq), since truncation-point pileups driving the counts depend on soft-clipped polyA-tail reads.

## Inputs

- Output GFF3 (or GTF with --gtf) annotation file from peaks2utr, containing original features plus 3' UTRs with source=peaks2utr
- Canonical T. brucei annotation in GFF3/GTF format (reference for new/altered/unchanged classification)
- Reference figures to verify against: 9888 total 3' UTRs; 3709 new; 5347 altered; 832 unchanged; ~84% of 11703 genes covered

## Outputs

- Counts of new, altered, and unchanged 3' UTRs relative to the canonical annotation
- Proportion of available genes covered by the annotated 3' UTRs
- Pass/fail verdict on reproduction of the published annotation statistics

## How to apply

Run the annotation tool on the reference inputs, then validate the output GFF3 by confirming it retains the original features and adds 3' UTR features with source=peaks2utr. Parse the output and compare each predicted 3' UTR's coordinates against the canonical annotation to classify it as new, altered (overlapping but different boundaries), or unchanged, then tally the counts. For the T. brucei Briggs et al. 2021 10x Chromium BAM (~140 million reads, 12.4 GB), the expected result is 9888 total 3' UTRs over ~84% of 11703 genes — roughly 60% more than canonical — partitioned as 3709 new, 5347 altered, and 832 unchanged. Optionally normalize the output GFF3 with GenomeTools before comparison so that equivalent annotations are structurally comparable. Report the coverage proportion of available genes as an additional sanity signal.

## Related tools

- **peaks2utr** (Generates the 3' UTR annotation (GFF3/GTF output with source=peaks2utr) whose counts are verified; invoked as peaks2utr <GFF_IN> <BAM_IN> with -p for parallel workers and -o for output) — https://github.com/haessar/peaks2utr
- **GenomeTools** (Optional post-processing/normalization of the output GFF3 prior to tallying and comparison) — https://github.com/genometools/genometools#building-and-installation
- **bedtools** (Required system dependency for running peaks2utr, installed with 'apt-get install bedtools') — https://bedtools.readthedocs.io/en/latest/content/installation.html
- **MACS3** (Internal dependency used by peaks2utr to call broad peaks of significant read coverage in the BAM that form the basis for 3' UTR annotation) — https://pypi.org/project/MACS3/

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3
```

## Evaluation signals

- Output GFF3 contains the original input features plus 3' UTR features with source=peaks2utr
- Total 3' UTR count equals 9888 when run on the T. brucei (Briggs et al. 2021) scRNA-seq dataset against the canonical annotation
- Partition counts match 3709 new, 5347 altered, and 832 unchanged relative to the canonical annotation
- 3' UTRs cover approximately 84% of the 11703 available genes (~60% more than the canonical annotation)
- peaks2utr-check demo run completes and yields Tb927_01_v5.1.new.gff with 3' UTRs from source peaks2utr before running the full dataset

## Limitations

- No software version numbers are stated in the methods ('Install latest release with:'), so exact reproducibility of counts may depend on the release used.
- Installation instructions assume a Debian/Ubuntu system with root privileges; other systems require alternate dependency setup.
- No changelog is available for the repository, complicating version tracking.
- The full-scale run requires a very large input BAM (~140 million reads, 12.4 GB), so verification may be resource-intensive unless parallel workers (-p 4) are used.

## Evidence

- [readme] peaks2utr-check demo yields annotated output: "When complete, you should see a file `Tb927_01_v5.1.new.gff` which contains original annotations as well as 3' UTRs with source "peaks2utr"."
- [methods] install latest release: "Install latest release with:
```
pip install peaks2utr
```"
- [methods] output format description: "Outputs a GFF3 annotation file (or GTF with option `--gtf`) including original features plus 3' UTR features with `source=peaks2utr`."
- [intro] peaks calling approach: "peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file"
- [abstract] 10x Chromium rationale: "uses the UTR enrichment of single-cell technologies, such as 10× Chromium, to accurately annotate 3' UTRs"
- [readme] GenomeTools post-processing: "[GenomeTools](https://github.com/genometools/genometools#building-and-installation) (for post-processing of output gff3)"
