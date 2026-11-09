---
name: genomic-annotation-command-line-workflow-execution
description: Use when you have a GFF/GTF gene model plus a BAM of aligned reads (e.g.
  10x Chromium scRNA-seq) and need to annotate or refine 3' UTR features at the genome
  level.
when_to_use_negative:
- You already have a validated 3' UTR annotation and only need generic GFF format
  conversion or validation — use GenomeTools alone instead of the peaks2utr pipeline.
- Input reads lack UTR-distal signal (e.g. not 3' enriched single-cell or similar
  data), so MACS peak calls will not concentrate at UTR distal ends as the method
  expects.
- The task is 5' UTR or CDS annotation rather than 3' UTR annotation from BAM peaks.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: peaks2utr
  role: 'Main CLI pipeline: calls MACS peaks, links peaks to same-strand genes within
    maximum distance, truncates UTRs at SPAT pile-up points, and writes the annotation
    GFF3'
  repo: https://github.com/haessar/peaks2utr
- name: bedtools
  role: Required dependency for genomic interval operations during peak-to-gene matching
  repo: https://bedtools.readthedocs.io/en/latest/content/installation.html
- name: MACS3
  role: Called by peaks2utr to identify broad peaks of significant read coverage in
    the BAM
  repo: https://pypi.org/project/MACS3/
- name: GenomeTools
  role: Optional post-processing of the output GFF3
  repo: https://github.com/genometools/genometools
- name: pip
  role: Used to install the peaks2utr Python 3 (>=3.8) package
  repo: https://pypi.org/project/peaks2utr/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:9b7a59cc17ac8edb6eb373cea39861576b02581b058fe4601c4316f4e2d24a16
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
  - pip
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# genomic-annotation-command-line-workflow-execution

## Summary

Run peaks2utr, a Python 3 CLI that annotates 3' UTRs from a canonical GFF/GTF annotation and a BAM of aligned reads, leveraging MACS3-called broad peaks and SPAT truncation points. This skill covers installing the toolchain, invoking the pipeline, and validating its GFF3 output.

## When to use

Use when you have a GFF/GTF gene model plus a BAM of aligned reads (e.g. 10x Chromium scRNA-seq) and need to annotate or refine 3' UTR features at the genome level.

## When NOT to use

- You already have a validated 3' UTR annotation and only need generic GFF format conversion or validation — use GenomeTools alone instead of the peaks2utr pipeline.
- Input reads lack UTR-distal signal (e.g. not 3' enriched single-cell or similar data), so MACS peak calls will not concentrate at UTR distal ends as the method expects.
- The task is 5' UTR or CDS annotation rather than 3' UTR annotation from BAM peaks.

## Inputs

- Gene models in GFF3 or GTF format (GFF_IN; existing 3' UTRs optional)
- Aligned sequencing reads in BAM format (BAM_IN; e.g. 10x Chromium 3' enriched reads)

## Outputs

- GFF3 annotation file (or GTF with --gtf) containing original features plus new 3' UTR features with source=peaks2utr, default output name *.new.<ext>

## How to apply

Install peaks2utr via `pip install peaks2utr` (or build from source with `git clone` and `python3 -m build`) and install the required bedtools dependency (`apt-get install bedtools`); optionally add GenomeTools for output post-processing. Verify the install with `peaks2utr-check`, which runs a short demo on Tb927_01_v5.1.gff and Tb927_01_v5.1.slice.bam. Then invoke `peaks2utr <GFF_IN> <BAM_IN> [options]`: the tool uses MACS to call broad peaks of significant read coverage from the BAM, filters peaks to those on the same strand as a gene within a user-defined maximum distance, truncates passing 3' UTRs at SPAT pile-up truncation points (soft-clipped/polyA-tail end bases), and writes original features plus new 3' UTR features (source=peaks2utr) to a GFF3 (or GTF with `--gtf`). Choose `-p` for parallelism and `-o`/`--output` for the output filename; the tool can be tuned to extend, override, or ignore pre-existing 3' UTR annotations.

## Related tools

- **peaks2utr** (Main CLI pipeline: calls MACS peaks, links peaks to same-strand genes within maximum distance, truncates UTRs at SPAT pile-up points, and writes the annotation GFF3) — https://github.com/haessar/peaks2utr
- **bedtools** (Required dependency for genomic interval operations during peak-to-gene matching) — https://bedtools.readthedocs.io/en/latest/content/installation.html
- **MACS3** (Called by peaks2utr to identify broad peaks of significant read coverage in the BAM) — https://pypi.org/project/MACS3/
- **GenomeTools** (Optional post-processing of the output GFF3) — https://github.com/genometools/genometools
- **pip** (Used to install the peaks2utr Python 3 (>=3.8) package) — https://pypi.org/project/peaks2utr/

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3
```

## Evaluation signals

- peaks2utr-check completes successfully, producing Tb927_01_v5.1.new.gff from the bundled demo files
- Output GFF3 contains all original input annotations plus 3' UTR features with source 'peaks2utr'
- Each annotated 3' UTR lies on the same strand as its linked gene and within the user-defined maximum distance of a MACS-called peak
- UTR end positions coincide with piled-up SPAT truncation points where available
- File format is valid GFF3 (or GTF when --gtf is used), suitable for GenomeTools post-processing

## Limitations

- Installation instructions assume a Debian/Ubuntu system with root privileges; bedtools is a required external dependency.
- No software version numbers stated in the methods; no changelog found in the repository.
- The approach is optimized for data where signal concentrates at UTR distal ends (e.g. 10x Chromium); other BAM types may yield weaker annotations.

## Evidence

- [methods] peaks2utr is called from the command line as: "peaks2utr is called from the command line as:
```
peaks2utr <GFF_IN> <BAM_IN> [options]
```"
- [intro] MACS peak calling: "peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file"
- [intro] Truncation point method: "Reads containing soft-clipped bases and polyA-tails of a given length are detected, and their end bases tallied as "truncation points"."
- [readme] Install and verify: "To check that peaks2utr has installed correctly, simply run the following in your terminal to initiate a short run with default parameters"
- [readme] Output description: "Outputs a GFF3 annotation file (or GTF with option `--gtf`) including original features plus 3' UTR features with `source=peaks2utr`."
- [methods] Example invocation: "peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3"
