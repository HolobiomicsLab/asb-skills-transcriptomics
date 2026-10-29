---
name: gff3-genomic-feature-annotation-writing
description: Use when you must write a GFF3 annotation file that preserves existing
  gene model features and appends newly called 3' UTR features derived from BAM-aligned
  reads.
when_to_use_negative:
- Input is not aligned sequencing reads (BAM) or a canonical GFF/GTF annotation —
  peaks2utr's peak-based 3' UTR inference requires both.
- You only need to reformat or validate an existing annotation without adding new
  features; use GenomeTools or a generic GFF3 parser instead.
- The reads lack 3'-end enrichment (e.g. not from protocols like 10x Chromium where
  signal concentrates at UTR distal ends), reducing the reliability of called 3' UTR
  boundaries.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: peaks2utr
  role: Writes the output GFF3/GTF annotation with original features plus newly annotated
    3' UTR features
  repo: https://github.com/haessar/peaks2utr
- name: GenomeTools
  role: Optional post-processing of the output GFF3
  repo: https://github.com/genometools/genometools
- name: MACS3
  role: Called internally by peaks2utr to identify broad peaks of significant read
    coverage that become candidate 3' UTR features
  repo: https://pypi.org/project/MACS3/
- name: bedtools
  role: Required dependency used for interval operations underlying peak-to-gene matching
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:a09183af9ee480887bf9648fd08f1510a9be83ff2d2d47ab04dacae1a7cdda54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - GenomeTools
  - MACS3
  - Python
  - bedtools
  - peaks2utr
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gff3-genomic-feature-annotation-writing

## Summary

Write augmented GFF3 (or GTF) genomic annotations in which newly inferred 3' UTR features, tagged with source 'peaks2utr', are emitted alongside all original input features, as implemented by the peaks2utr tool.

## When to use

Use when you must write a GFF3 annotation file that preserves existing gene model features and appends newly called 3' UTR features derived from BAM-aligned reads.

## When NOT to use

- Input is not aligned sequencing reads (BAM) or a canonical GFF/GTF annotation — peaks2utr's peak-based 3' UTR inference requires both.
- You only need to reformat or validate an existing annotation without adding new features; use GenomeTools or a generic GFF3 parser instead.
- The reads lack 3'-end enrichment (e.g. not from protocols like 10x Chromium where signal concentrates at UTR distal ends), reducing the reliability of called 3' UTR boundaries.

## Inputs

- Gene models in GFF3 or GTF format (GFF_IN), existing 3' UTRs optional
- Aligned sequencing reads in BAM format (BAM_IN)

## Outputs

- GFF3 annotation file containing original features plus new 3' UTR features with source=peaks2utr (default suffix *.new.<ext>)
- GTF annotation file instead of GFF3 when run with --gtf

## How to apply

Run peaks2utr from the command line as `peaks2utr <GFF_IN> <BAM_IN> [options]`, passing gene models in GFF3 or GTF format (existing 3' UTRs optional) and aligned reads in BAM format. The tool calls broad peaks of significant read coverage via MACS, filters peaks to those on the same strand as a gene within the maximum distance, truncates passing UTRs at piled-up SPAT truncation points, and writes a GFF3 output containing the original features plus 3' UTR features with source=peaks2utr. By default the output uses the original filename with a `*.new.<ext>` suffix; an explicit path can be given with `-o`/`--output`, and `--gtf` switches output format to GTF. Optionally post-process the output GFF3 with GenomeTools (`apt-get install genometools`) to normalise the annotation. Features can be tuned to extend, override or ignore any pre-existing 3' UTR annotations in the input GFF.

## Related tools

- **peaks2utr** (Writes the output GFF3/GTF annotation with original features plus newly annotated 3' UTR features) — https://github.com/haessar/peaks2utr
- **GenomeTools** (Optional post-processing of the output GFF3) — https://github.com/genometools/genometools
- **MACS3** (Called internally by peaks2utr to identify broad peaks of significant read coverage that become candidate 3' UTR features) — https://pypi.org/project/MACS3/
- **bedtools** (Required dependency used for interval operations underlying peak-to-gene matching)

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3
```

## Evaluation signals

- The output file exists with the expected name: default is the original filename with a `*.new.<ext>` suffix, e.g. Tb927_01_v5.1.new.gff after running on the demo inputs.
- The output contains all original annotations from the input GFF plus additional 3' UTR features whose source field is 'peaks2utr'.
- Each newly written 3' UTR lies on the same strand as its matched gene and within the user-defined maximum distance of that gene.
- UTR end coordinates coincide with piled-up truncation points from reads with soft-clipped bases and polyA-tails where such evidence exists (SPAT truncation).
- Running `peaks2utr-check` completes successfully, producing Tb927_01_v5.1.new.gff from the bundled demo GFF and slice BAM files.
- If GenomeTools is used for post-processing, the resulting GFF3 parses without errors in `gt gff3`.

## Limitations

- Installation instructions assume a Debian / Ubuntu system with root privileges (bedtools required, GenomeTools optional).
- No software version numbers are stated in the methods; the approach favours BAM files from 10x Chromium-style runs where signal is concentrated at UTR distal ends.
- Pre-existing 3' UTR annotations are handled by tunable behaviour (extend, override or ignore), so output depends on the chosen mode.
- No changelog is available for the repository.

## Evidence

- [readme] Outputs a GFF3 annotation file (or GTF with option `--gtf`) including original features plus 3' UTR features with `source=peaks2utr`.: "Outputs a GFF3 annotation file (or GTF with option `--gtf`) including original features plus 3' UTR features with `source=peaks2utr`."
- [readme] Output file naming rule: "Output file name can be specified with `-o` or `--output`; by default outputs to original filename with a `*.new.<ext>` suffix."
- [intro] peaks2utr uses MACS to call broad peaks of significant read coverage in the BAM file: "peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file"
- [methods] GenomeTools for post-processing of output gff3: "[GenomeTools](https://github.com/genometools/genometools#building-and-installation) (for post-processing of output gff3)"
- [readme] peaks2utr can be tuned to extend, override or ignore any pre-existing 3' UTR annotations in the input GFF file.: "peaks2utr can be tuned to extend, override or ignore any pre-existing 3' UTR ann"
- [readme] Demo run produces output containing original annotations plus new UTRs: "When complete, you should see a file `Tb927_01_v5.1.new.gff` which contains original annotations as well as 3' UTRs with source "peaks2utr"."
