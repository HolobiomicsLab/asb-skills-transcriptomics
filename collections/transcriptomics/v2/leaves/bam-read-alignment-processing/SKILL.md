---
name: bam-read-alignment-processing
description: Use when a GFF3/GTF canonical annotation must be extended with 3' UTRs
  using aligned reads (BAM) enriched at transcript 3' ends, e.g. 10x Chromium scRNA-seq
  BAMs with ~10^8 reads.
when_to_use_negative:
- Input reads are not 3'-enriched — the method favours 10x Chromium-style BAMs where
  signal is inherently concentrated at the distal ends of UTRs
- The target is 5' UTR annotation or full-length transcript structure rather than
  3' UTR end determination
- No canonical GFF3/GTF annotation is available to anchor and compare the inferred
  3' UTRs against
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: peaks2utr
  role: Python CLI that processes the BAM with MACS peak calling and polyA/soft-clip
    truncation-point tallying to annotate 3' UTRs onto the canonical annotation
  repo: https://github.com/haessar/peaks2utr
- name: MACS3
  role: Called by peaks2utr to detect broad peaks of significant read coverage in
    the BAM file
- name: bedtools
  role: Required system dependency of peaks2utr for interval operations on the alignment
    data
  repo: https://bedtools.readthedocs.io/en/latest/content/installation.html
- name: GenomeTools
  role: Optional post-processing of the output GFF3 produced from the BAM analysis
  repo: https://github.com/genometools/genometools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:963a534df1796b1bdac75b0d0154b080fa0e82fe895312db781955d29d46cbe3
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bam-read-alignment-processing

## Summary

Processes aligned sequencing reads in BAM format to extract 3' UTR signal — MACS-called broad coverage peaks plus polyA-tail/soft-clip truncation points — as the read-side input for peaks2utr 3' UTR annotation against a canonical GFF3/GTF annotation.

## When to use

Use when a GFF3/GTF canonical annotation must be extended with 3' UTRs using aligned reads (BAM) enriched at transcript 3' ends, e.g. 10x Chromium scRNA-seq BAMs with ~10^8 reads.

## When NOT to use

- Input reads are not 3'-enriched — the method favours 10x Chromium-style BAMs where signal is inherently concentrated at the distal ends of UTRs
- The target is 5' UTR annotation or full-length transcript structure rather than 3' UTR end determination
- No canonical GFF3/GTF annotation is available to anchor and compare the inferred 3' UTRs against

## Inputs

- Aligned sequencing reads in BAM format (e.g. a ~140-million-read 12.4 GB 10x Chromium scRNA-seq BAM)
- Canonical gene models in GFF3 or GTF format (existing 3' UTRs optional)

## Outputs

- GFF3 annotation file (or GTF with '--gtf') containing original features plus 3' UTR features with source=peaks2utr
- Optionally a GenomeTools-normalized GFF3

## How to apply

Install peaks2utr ('pip install peaks2utr') together with the bedtools system dependency ('apt-get install bedtools'), then run 'peaks2utr <GFF_IN> <BAM_IN> [options]' with the canonical annotation (GFF3 or GTF, existing 3' UTRs optional) and the aligned-read BAM. Internally, peaks2utr uses MACS to call broad peaks of significant read coverage in the BAM, filters those peaks by a set of criteria as a basis for novel 3' UTRs, and detects reads with soft-clipped bases and polyA-tails of a given length, tallying their end bases as truncation points; piled-up co-occurring truncation points determine the precise UTR end base. The approach favours 10x Chromium BAMs where signal is inherently concentrated at the distal ends of 3' or 5' UTRs. Tune the tool to extend, override or ignore pre-existing 3' UTRs in the input annotation, and use parallel workers (e.g. '-p 4') and '-o' for the output GFF3. Optionally post-process the resulting GFF3 with GenomeTools to normalize the annotation file.

## Related tools

- **peaks2utr** (Python CLI that processes the BAM with MACS peak calling and polyA/soft-clip truncation-point tallying to annotate 3' UTRs onto the canonical annotation) — https://github.com/haessar/peaks2utr
- **MACS3** (Called by peaks2utr to detect broad peaks of significant read coverage in the BAM file)
- **bedtools** (Required system dependency of peaks2utr for interval operations on the alignment data) — https://bedtools.readthedocs.io/en/latest/content/installation.html
- **GenomeTools** (Optional post-processing of the output GFF3 produced from the BAM analysis) — https://github.com/genometools/genometools

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3
```

## Evaluation signals

- Verify installation first with 'peaks2utr-check', which should produce Tb927_01_v5.1.new.gff containing original annotations plus 3' UTRs with source=peaks2utr
- Confirm the output GFF3 contains all original input features plus new 3' UTR features tagged source=peaks2utr
- Tally inferred 3' UTRs against the canonical annotation and classify as new, altered, or unchanged — in the T. brucei benchmark this yields 9888 total 3' UTRs (3709 new, 5347 altered, 832 unchanged) covering ~84% of 11703 genes
- Validate the output GFF3 structure, e.g. by parsing it with GenomeTools after post-processing
- Check that per-gene UTR end coordinates agree with piled-up truncation points from polyA-tail/soft-clipped reads at the distal 3' ends

## Limitations

- Installation instructions assume a Debian / Ubuntu system with root privileges (bedtools installed via apt-get)
- No software version numbers stated for the tools in the methods; no changelog available
- Depends on 3'-end read enrichment; BAMs without polyA-tail/soft-clip signal at transcript distal ends will not support precise UTR end determination
- Pre-existing 3' UTR handling (extend, override or ignore) must be tuned explicitly; default behaviour may conflict with canonical annotations

## Evidence

- [readme] peaks2utr is a Python command-line tool that annotates 3' untranslated regions (UTR) for a given set of aligned sequencing reads in BAM format: "peaks2utr is a Python command-line tool that annotates 3' untranslated regions (UTR) for a given set of aligned sequencing reads in BAM format"
- [intro] MACS broad peak calling on BAM: "peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file"
- [intro] Truncation-point detection from polyA/soft-clipped reads: "Reads containing soft-clipped bases and polyA-tails of a given length are detected, and their end bases tallied as "truncation points"."
- [intro] 10x Chromium signal concentration rationale: "This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs."
- [methods] bedtools required dependency: "[bedtools](https://bedtools.readthedocs.io/en/latest/content/installation.html)
```
apt-get install bedtools
```"
- [readme] Output format specification: "Outputs a GFF3 annotation file (or GTF with option `--gtf`) including original features plus 3' UTR features with `source=peaks2utr`."
