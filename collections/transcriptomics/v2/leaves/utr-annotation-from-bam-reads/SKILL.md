---
name: utr-annotation-from-bam-reads
description: Use when you have an aligned scRNA-seq BAM (e.g. 10x Chromium, UTR-enriched)
  plus a GFF/GTF gene model and need to annotate or reannotate 3' UTRs beyond the
  canonical annotation.
when_to_use_negative:
- The BAM is not from a UTR-enriched single-cell protocol (e.g. 10x Chromium) — signal
  concentrated at UTR distal ends is what the method relies on
- You already have a validated, complete 3' UTR annotation and do not want it extended,
  overridden or augmented
- You need bulk-3'-seq-style UTR quantification rather than annotation of UTR coordinates
  from scRNA-seq coverage peaks
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3170
tools:
- name: peaks2utr
  role: Primary CLI that calls MACS3 peaks, filters them, detects polyA-tail truncation
    points and writes 3' UTR features into the output GFF3
  repo: https://github.com/haessar/peaks2utr
- name: MACS3
  role: Peak caller used internally to find broad peaks of significant read coverage
    in the BAM
- name: bedtools
  role: Required system dependency for the peaks2utr installation
- name: GenomeTools
  role: Optional post-processing/normalization of the output GFF3 annotation file
  repo: https://github.com/genometools/genometools
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:677b573cd8624ce5ef86b4c9f76e9a16e2ff51a9475a164ed86a5391d846e9ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - GETUTR
  - GenomeTools
  - MACS3
  - Python 3
  - UTRme
  - bedtools
  - peaks2utr
  - pip
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# utr-annotation-from-bam-reads

## Summary

Annotate 3' UTRs by running peaks2utr, a Python CLI that calls broad read-coverage peaks with MACS3 on an aligned BAM (e.g. 10x Chromium scRNA-seq) and refines UTR end positions using polyA-tail truncation points, extending or overriding a canonical GFF/GTF annotation.

## When to use

Use when you have an aligned scRNA-seq BAM (e.g. 10x Chromium, UTR-enriched) plus a GFF/GTF gene model and need to annotate or reannotate 3' UTRs beyond the canonical annotation.

## When NOT to use

- The BAM is not from a UTR-enriched single-cell protocol (e.g. 10x Chromium) — signal concentrated at UTR distal ends is what the method relies on
- You already have a validated, complete 3' UTR annotation and do not want it extended, overridden or augmented
- You need bulk-3'-seq-style UTR quantification rather than annotation of UTR coordinates from scRNA-seq coverage peaks

## Inputs

- Canonical gene models in GFF3 or GTF format (existing 3' UTRs optional), e.g. Tb927_01_v5.1.gff or WormBase chrI GFF
- Aligned sequencing reads in BAM format, favouring UTR-enriched runs such as 10x Chromium

## Outputs

- GFF3 annotation file (or GTF with --gtf) containing original features plus novel 3' UTR features with source=peaks2utr, written to -o output or default *.new.<ext>

## How to apply

Install peaks2utr via pip on a Debian/Ubuntu system with bedtools (`apt-get install bedtools`), optionally GenomeTools for post-processing the output GFF3. Verify the install with `peaks2utr-check`, which runs the bundled Tb927_01_v5.1.gff demo and should emit a file containing the original annotations plus 3' UTRs with source=peaks2utr. Then invoke `peaks2utr <GFF_IN> <BAM_IN> [options]`, e.g. with `-p 4` parallel workers and `-o output.gff3`; peaks2utr uses MACS3 to call broad peaks of significant read coverage, filters those peaks by a set of criteria, and detects reads with soft-clipped bases and polyA-tails of a given length, tallying their end bases as truncation points whose co-occurrence determines the precise UTR end base. Decide whether to extend, override or ignore pre-existing 3' UTR annotations in the input GFF via the optional arguments. Judge success by the output GFF3 containing original features plus 3' UTR features with source=peaks2utr, and by comparing predicted UTR ends against a canonical annotation (e.g. WormBase chrI) with a 50 bp end-position tolerance.

## Related tools

- **peaks2utr** (Primary CLI that calls MACS3 peaks, filters them, detects polyA-tail truncation points and writes 3' UTR features into the output GFF3) — https://github.com/haessar/peaks2utr
- **MACS3** (Peak caller used internally to find broad peaks of significant read coverage in the BAM)
- **bedtools** (Required system dependency for the peaks2utr installation)
- **GenomeTools** (Optional post-processing/normalization of the output GFF3 annotation file) — https://github.com/genometools/genometools

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3
```

## Evaluation signals

- peaks2utr-check demo run produces Tb927_01_v5.1.new.gff containing original annotations plus 3' UTRs with source 'peaks2utr'
- Output GFF3 contains all original input features in addition to newly annotated 3' UTR features sourced peaks2utr
- On C. elegans chromosome I, roughly 60% of predicted UTRs should match the WormBase canonical annotation within a 50 bp end-position tolerance (vs ~6% for GETUTR/UTRme), with 16 UTRs matched by all three tools and no UTR-length overprediction
- On the T. brucei 10x Chromium BAM, tallies should reproduce 9888 total 3' UTRs covering ~84% of 11703 genes, split as 3709 new / 5347 altered / 832 unchanged relative to canonical

## Limitations

- Installation instructions assume a Debian/Ubuntu system with root privileges; no software version numbers are stated in the methods
- The method favours BAM files from runs like 10x Chromium where signal is inherently concentrated at UTR distal ends; other library types may not yield usable truncation-point signal
- peaks2utr's behaviour toward pre-existing 3' UTRs (extend, override, ignore) must be tuned explicitly; default handling may not suit every annotation
- No changelog was found in the discussion section, so tracking releases relies on the PyPI version badge

## Evidence

- [intro] peaks2utr uses MACS to call broad peaks of significant read coverage, and use peaks passing a set of criteria as a basis to annotate novel 3' UTRs.: "peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file"
- [intro] Truncation-point mechanism for precise UTR end determination.: "Reads containing soft-clipped bases and polyA-tails of a given length are detected, and their end bases tallied as "truncation points". When piled up, each co-occurring truncation point is used to"
- [methods] CLI installation via pip.: "Install latest release with:
```
pip install peaks2utr
```"
- [methods] bedtools is the required system dependency.: "[bedtools](https://bedtools.readthedocs.io/en/latest/content/installation.html)
```
apt-get install bedtools
```"
- [readme] Installation verification demo with expected output file.: [evidence span withheld — anchor status: not_found]
- [readme] Output specification: GFF3 (or GTF) including original features plus 3' UTRs with source=peaks2utr.: "Outputs a GFF3 annotation file (or GTF with option `--gtf`) including original features plus 3' UTR features with `source=peaks2utr`."
- [readme] Tunability toward pre-existing 3' UTR annotations.: [evidence span withheld — anchor status: external_doc]
- [intro] Suitability of 10x Chromium BAMs because signal concentrates at UTR distal ends.: "This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs."
