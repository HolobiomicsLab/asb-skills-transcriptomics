---
name: utr-truncation-point-pileup-annotation
description: Use when a GFF/GTF gene annotation plus a BAM of aligned reads (e.g.
  10x Chromium, where signal concentrates at UTR distal ends) must be converted into
  novel 3' UTR annotations.
when_to_use_negative:
- The input BAM is not from a 3'-enriched library — the peak-calling and truncation-point
  logic assumes signal concentrated at the distal ends of 3' or 5' UTRs (e.g. 10x
  Chromium)
- You need 5' UTR or full-length transcript isoform annotation rather than 3' UTR
  end designation
- Peak-to-gene strand/distance filtering is not applicable because the annotation
  lacks strand or gene features
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: peaks2utr
  role: Main CLI that links MACS peaks to genes and truncates designated 3' UTRs via
    truncation-point pileup
  repo: https://github.com/haessar/peaks2utr
- name: MACS3
  role: Calls broad peaks of significant read coverage in the BAM that become 3' UTR
    candidates
  repo: https://pypi.org/project/MACS3/
- name: bedtools
  role: Required dependency used by peaks2utr (e.g. for interval/strand overlap operations
    underlying peak-to-gene matching)
  repo: https://bedtools.readthedocs.io/en/latest/content/installation.html
- name: GenomeTools
  role: Optional post-processing of the output GFF3
  repo: https://github.com/genometools/genometools#building-and-installation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:a52399dee4054081ffacc2411d11f680ca714b0b61255b16581988ff685b2185
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# utr-truncation-point-pileup-annotation

## Summary

Annotates 3' UTRs by linking MACS3-called broad peaks of significant read coverage to same-strand genes within a maximum distance, then defining each passing UTR's precise end via pileup of polyA/soft-clip truncation points from the BAM.

## When to use

Use when a GFF/GTF gene annotation plus a BAM of aligned reads (e.g. 10x Chromium, where signal concentrates at UTR distal ends) must be converted into novel 3' UTR annotations.

## When NOT to use

- The input BAM is not from a 3'-enriched library — the peak-calling and truncation-point logic assumes signal concentrated at the distal ends of 3' or 5' UTRs (e.g. 10x Chromium)
- You need 5' UTR or full-length transcript isoform annotation rather than 3' UTR end designation
- Peak-to-gene strand/distance filtering is not applicable because the annotation lacks strand or gene features

## Inputs

- Canonical gene annotation in GFF3 or GTF format (existing 3' UTRs optional)
- Aligned sequencing reads in BAM format (favours 10x Chromium-style BAMs)

## Outputs

- GFF3 annotation file with original features plus novel 3' UTR features with source=peaks2utr (default output suffix *.new.<ext>)
- GTF output instead of GFF3 when --gtf is passed

## How to apply

Run peaks2utr <GFF_IN> <BAM_IN> [options]: the tool calls MACS3 to find broad 'peaks' of significant read coverage in the BAM, then filters those peaks to a set of criteria — peaks must lie on the same strand as a gene and within a user-defined maximum distance of it — to select candidates for novel 3' UTRs. For each passing peak, reads containing soft-clipped bases and polyA-tails of a given length are detected and their end bases tallied as 'truncation points'; co-occurring truncation points piled up at the same position determine the precise end base of the UTR (truncating the region, e.g. via SPAT pile-up truncation points, where possible). Pre-existing 3' UTR annotations can be tuned to be extended, overridden or ignored. The output is a GFF3 (or GTF with --gtf) containing the original features plus 3' UTR features with source=peaks2utr.

## Related tools

- **peaks2utr** (Main CLI that links MACS peaks to genes and truncates designated 3' UTRs via truncation-point pileup) — https://github.com/haessar/peaks2utr
- **MACS3** (Calls broad peaks of significant read coverage in the BAM that become 3' UTR candidates) — https://pypi.org/project/MACS3/
- **bedtools** (Required dependency used by peaks2utr (e.g. for interval/strand overlap operations underlying peak-to-gene matching)) — https://bedtools.readthedocs.io/en/latest/content/installation.html
- **GenomeTools** (Optional post-processing of the output GFF3) — https://github.com/genometools/genometools#building-and-installation

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3
```

## Evaluation signals

- Output file Tb927_01_v5.1.new.gff (or *.new.<ext> generally) exists after a run on demo inputs
- Output GFF3 contains all original input annotations plus newly added 3' UTR features with source 'peaks2utr'
- Each added 3' UTR lies on the same strand as its parent gene and within the configured maximum distance
- Each UTR end base coincides with a pileup of co-occurring truncation points from soft-clipped/polyA-tailed reads
- `peaks2utr-check` completes successfully with default parameters

## Limitations

- Approach favours BAMs from runs like 10x Chromium where signal is inherently concentrated at UTR distal ends; other library types may not yield reliable truncation points
- Installation assumes a Debian/Ubuntu system with root privileges (apt-get install bedtools)
- No software version numbers stated in the methods ('Install latest release with:') and no changelog found
- Pre-existing 3' UTR annotations may be extended, overridden or ignored depending on tuning, so output is configuration-dependent

## Evidence

- [intro] peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file, and uses those peaks that pass a set of criteria as a basis to annotate novel: "peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file, and uses those peaks that pass a set of criteria as a basis to annotate novel"
- [intro] Reads containing soft-clipped bases and polyA-tails of a given length are detected, and their end bases tallied as "truncation points". When piled up, each co-occurring truncation point is used to: "Reads containing soft-clipped bases and polyA-tails of a given length are detected, and their end bases tallied as "truncation points". When piled up, each co-occurring truncation point is used to"
- [readme] This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs.: "This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs."
- [readme] When complete, you should see a file `Tb927_01_v5.1.new.gff` which contains original annotations as well as 3' UTRs with source "peaks2utr".: "When complete, you should see a file `Tb927_01_v5.1.new.gff` which contains original annotations as well as 3' UTRs with source "peaks2utr"."
- [methods] peaks2utr can be tuned to extend, override or ignore any pre-existing 3' UTR annotations in the input GFF file.: [evidence span withheld — anchor status: external_doc]
