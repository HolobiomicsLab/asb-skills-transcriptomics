---
name: strand-specific-read-coverage-analysis
description: Use when aligned strand-specific reads in BAM format (e.g. 10x Chromium
  runs where signal concentrates at 3' UTR distal ends) must be converted into candidate
  intervals of significant coverage as the basis for annotating novel 3' UTRs.
when_to_use_negative:
- The goal is final 3' UTR annotation with precise end bases — peak calling alone
  only yields candidate coverage regions; the truncation-point refinement and criteria
  filtering stages are separate downstream tasks.
- The input BAM is not strand-specific or the reads cannot be reliably assigned to
  a strand — merging both strands would corrupt per-strand peak calls.
- The analysis target is narrow punctate peak detection (e.g. transcription-factor
  ChIP-seq) rather than broad extended coverage regions; peaks2utr's design favours
  10x Chromium-style signal concentrated at UTR distal ends.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: MACS3
  role: Peak caller invoked in broad-peak mode once per strand on the strand-split
    BAMs to produce the broadPeak candidate regions
  repo: https://pypi.org/project/MACS3/
- name: peaks2utr
  role: Pipeline whose first stage this skill reconstructs; it consumes the MACS-called
    broad peaks as the basis for annotating novel 3' UTRs
  repo: https://github.com/haessar/peaks2utr
- name: bedtools
  role: Required dependency of the peaks2utr installation for interval operations
    on the peak/BED coordinate sets
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:2849b96525c0f9d82ae6977034992187ffe637829285e285a220ffaed2f983da
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MACS3
  - Python 3
  - bedtools
  - peaks2utr
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# strand-specific-read-coverage-analysis

## Summary

Reconstructs the first stage of the peaks2utr 3' UTR annotation pipeline: calling broad peaks of significant read coverage with MACS3 separately on the plus- and minus-strand BAM files, yielding per-strand broadPeak BED interval sets that feed downstream UTR designation.

## When to use

Use when aligned strand-specific reads in BAM format (e.g. 10x Chromium runs where signal concentrates at 3' UTR distal ends) must be converted into candidate intervals of significant coverage as the basis for annotating novel 3' UTRs.

## When NOT to use

- The goal is final 3' UTR annotation with precise end bases — peak calling alone only yields candidate coverage regions; the truncation-point refinement and criteria filtering stages are separate downstream tasks.
- The input BAM is not strand-specific or the reads cannot be reliably assigned to a strand — merging both strands would corrupt per-strand peak calls.
- The analysis target is narrow punctate peak detection (e.g. transcription-factor ChIP-seq) rather than broad extended coverage regions; peaks2utr's design favours 10x Chromium-style signal concentrated at UTR distal ends.

## Inputs

- Aligned sequencing reads in BAM format (e.g. Tb927_01_v5.1.slice.bam), strand-split into plus- and minus-strand read files
- MACS3 installed via PyPI (https://pypi.org/project/MACS3/)

## Outputs

- Plus-strand broadPeak BED-format file of significant read-coverage peaks
- Minus-strand broadPeak BED-format file of significant read-coverage peaks

## How to apply

Split the aligned BAM input (e.g. the demo Tb927_01_v5.1.slice.bam) into plus-strand and minus-strand read files so that coverage signal from the two strands is never merged — strand separation matters because UTR candidates are strand-specific. Run MACS3 (the PyPI-available MACS implementation peaks2utr depends on) in broad-peak mode on the plus-strand BAM, then repeat as an independent run on the minus-strand BAM; broad mode is used because peaks2utr seeks extended regions of significant read coverage rather than narrow punctate peaks. Write out each strand's peak calls as broadPeak BED-format files without applying any gene-search or UTR designation, since peaks are only the raw candidate set — downstream, peaks2utr keeps only 'those peaks that pass a set of criteria' and refines end points using soft-clipped/polyA truncation points.

## Related tools

- **MACS3** (Peak caller invoked in broad-peak mode once per strand on the strand-split BAMs to produce the broadPeak candidate regions) — https://pypi.org/project/MACS3/
- **peaks2utr** (Pipeline whose first stage this skill reconstructs; it consumes the MACS-called broad peaks as the basis for annotating novel 3' UTRs) — https://github.com/haessar/peaks2utr
- **bedtools** (Required dependency of the peaks2utr installation for interval operations on the peak/BED coordinate sets)

## Examples

```
macs3 callpeak -t Tb927_01_v5.1.plus.bam --broad -n plus_strand_peaks; macs3 callpeak -t Tb927_01_v5.1.minus.bam --broad -n minus_strand_peaks
```

## Evaluation signals

- Each per-strand peak file exists and is non-empty after the MACS run.
- Each peak file parses cleanly as a broadPeak BED file (chromosome, start, end, and broad-peak columns present per interval).
- Plus-strand and minus-strand peak sets were produced by independent MACS runs — no interval in the plus-strand file originates from a merged two-strand signal.
- Peak intervals correspond to regions of significant read coverage and fall within the coordinate bounds of the input BAM's reference sequences.
- The peak sets are usable downstream: peaks passing peaks2utr's criteria serve as the basis for novel 3' UTR calls in the final GFF3 (source=peaks2utr).

## Limitations

- No software version numbers are stated for MACS3/peaks2utr in the article methods, so exact peak-calling parameters are not pinned by the source.
- The approach favours BAM files from 10x Chromium-style runs where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs; data without this enrichment may yield weak peaks.
- Peaks are only candidates — 'those peaks that pass a set of criteria' are used as a basis for annotation, so raw peak output alone is not a UTR annotation.
- Installation instructions assume a Debian / Ubuntu system with root privileges (e.g. apt-get install bedtools).

## Evidence

- [intro] peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file: "peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file"
- [intro] uses those peaks that pass a set of criteria as a basis to annotate novel: "uses those peaks that pass a set of criteria as a basis to annotate novel 3' UTRs."
- [readme] This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs.: "This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs."
- [readme] peaks2utr is a Python command-line tool that annotates 3' untranslated regions (UTR) for a given set of aligned sequencing reads in BAM format: "peaks2utr is a Python command-line tool that annotates 3' untranslated regions (UTR) for a given set of aligned sequencing reads in BAM format"
