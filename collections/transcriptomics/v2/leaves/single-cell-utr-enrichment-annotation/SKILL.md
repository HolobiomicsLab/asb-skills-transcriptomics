---
name: single-cell-utr-enrichment-annotation
description: Use when you have aligned single-cell reads (BAM, e.g. 10x Chromium)
  plus a canonical GFF/GTF annotation and need novel or improved 3' UTR end calls,
  especially to benchmark against tools like GETUTR or UTRme.
when_to_use_negative:
- Input is bulk RNA-seq BAM with weak signal at UTR distal ends — the method relies
  on the UTR enrichment inherent to 10x Chromium-style single-cell runs.
- You need to re-annotate non-UTR features (e.g. CDS or exon boundaries) — peaks2utr
  only annotates 3' UTRs on top of the supplied annotation.
- No aligned BAM exists (e.g. only a draft genome or transcript assembly) — peaks2utr
  requires coverage peaks and read truncation points from aligned reads.
edam_operation: http://edamontology.org/operation_0372
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: peaks2utr
  role: Main CLI that annotates 3' UTRs from GFF/GTF + BAM inputs using MACS peaks
    and truncation points
  repo: https://github.com/haessar/peaks2utr
- name: MACS3
  role: Called internally by peaks2utr to identify broad peaks of significant read
    coverage in the BAM
  repo: https://pypi.org/project/MACS3/
- name: bedtools
  role: Required dependency for interval operations during UTR annotation
  repo: https://bedtools.readthedocs.io/en/latest/content/installation.html
- name: GenomeTools
  role: Optional tool for post-processing of the output GFF3
  repo: https://github.com/genometools/genometools
- name: GETUTR
  role: Comparator tool for benchmarking 3' UTR prediction accuracy on the same BAM
    and annotation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:6b58a7faf0475666e7d35cccb218260133d6e25d733410c84323c6aae350978e
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
  edam_operation: http://edamontology.org/operation_0372
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# single-cell-utr-enrichment-annotation

## Summary

Annotates 3' UTRs by exploiting the UTR enrichment of single-cell RNA-seq data (e.g. 10x Chromium BAMs), using MACS-called broad coverage peaks plus soft-clip/polyA truncation points to define precise UTR ends, and benchmarking agreement against a canonical annotation with a 50 bp end-position tolerance.

## When to use

Use when you have aligned single-cell reads (BAM, e.g. 10x Chromium) plus a canonical GFF/GTF annotation and need novel or improved 3' UTR end calls, especially to benchmark against tools like GETUTR or UTRme.

## When NOT to use

- Input is bulk RNA-seq BAM with weak signal at UTR distal ends — the method relies on the UTR enrichment inherent to 10x Chromium-style single-cell runs.
- You need to re-annotate non-UTR features (e.g. CDS or exon boundaries) — peaks2utr only annotates 3' UTRs on top of the supplied annotation.
- No aligned BAM exists (e.g. only a draft genome or transcript assembly) — peaks2utr requires coverage peaks and read truncation points from aligned reads.

## Inputs

- Canonical gene model annotation in GFF3 or GTF format (existing 3' UTRs optional)
- Aligned sequencing reads in BAM format (10x Chromium single-cell data preferred)

## Outputs

- GFF3 annotation file (or GTF with --gtf) containing original features plus new 3' UTR features with source=peaks2utr

## How to apply

Run peaks2utr with a canonical annotation (GFF/GTF) and an aligned BAM as inputs; the tool uses MACS to call broad 'peaks' of significant read coverage, filters peaks by a set of criteria, then detects reads with soft-clipped bases and polyA-tails of a given length and tallies their end bases as 'truncation points'. Piled-up co-occurring truncation points determine the precise UTR end base — this favours 10x Chromium BAMs where signal is concentrated at UTR distal ends. Decide whether to extend, override or ignore pre-existing 3' UTR annotations via tool options. To validate, compare predicted UTRs to the canonical annotation (e.g. WormBase C. elegans chrI) using a 50 bp end-position tolerance and count agreement, optionally running GETUTR/UTRme on the same inputs for comparison and checking whether predicted UTR lengths are over-extended.

## Related tools

- **peaks2utr** (Main CLI that annotates 3' UTRs from GFF/GTF + BAM inputs using MACS peaks and truncation points) — https://github.com/haessar/peaks2utr
- **MACS3** (Called internally by peaks2utr to identify broad peaks of significant read coverage in the BAM) — https://pypi.org/project/MACS3/
- **bedtools** (Required dependency for interval operations during UTR annotation) — https://bedtools.readthedocs.io/en/latest/content/installation.html
- **GenomeTools** (Optional tool for post-processing of the output GFF3) — https://github.com/genometools/genometools
- **GETUTR** (Comparator tool for benchmarking 3' UTR prediction accuracy on the same BAM and annotation)

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3
```

## Evaluation signals

- Benchmark agreement: fraction of predicted UTRs matching the canonical annotation within a 50 bp end-position tolerance — expect ~60% for peaks2utr vs ~6% for GETUTR/UTRme on C. elegans chromosome I.
- Consensus check: ~16 UTRs on C. elegans chrI should be matched by all three tools (peaks2utr, GETUTR, UTRme).
- Output schema: resulting file contains original annotation features plus 3' UTR features with source=peaks2utr (file defaults to *.new.<ext> suffix).
- No UTR-length overprediction relative to the canonical annotation (peaks2utr should not systematically extend beyond WormBase UTR ends).
- Installation sanity: running peaks2utr-check on the demo files (Tb927_01_v5.1.gff / Tb927_01_v5.1.slice.bam) produces Tb927_01_v5.1.new.gff with 3' UTRs of source peaks2utr.

## Limitations

- Installation instructions assume a Debian/Ubuntu system with root privileges (bedtools required; GenomeTools optional).
- No software version numbers are stated in the methods and no changelog was found, so pinning requires care.
- The method favours BAM files with UTR enrichment (10x Chromium); performance on data without distal-end concentrated signal is not established.
- peaks2utr can over- or under-specify UTR lengths unless tuned; the tool can be set to extend, override or ignore pre-existing 3' UTR annotations, a decision point the user controls.

## Evidence

- [abstract] uses the UTR enrichment of single-cell technologies, such as 10× Chromium, to accurately annotate 3' UTRs for a given canonical annotation: "uses the UTR enrichment of single-cell technologies, such as 10× Chromium, to accurately annotate 3' UTRs for a given canonical annotation"
- [intro] peaks2utr uses MACS to call broad peaks of significant read coverage in the BAM file, and uses those peaks that pass a set of criteria as a basis to annotate novel: "peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file, and uses those peaks that pass a set of criteria as a basis to annotate novel"
- [readme] peaks2utr can be tuned to extend, override or ignore any pre-existing 3' UTR annotations in the input GFF file.: [evidence span withheld — anchor status: external_doc]
- [methods] Install latest release with:

pip install peaks2utr: "Install latest release with:
```
pip install peaks2utr
```"
- [readme] To check that peaks2utr has installed correctly, simply run the following in your terminal to initiate a short run with default parameters: "To check that peaks2utr has installed correctly, simply run the following in your terminal to initiate a short run with default parameters"
- [readme] Installation instructions assume a Debian / Ubuntu system with root privileges.: "Installation instructions assume a Debian / Ubuntu system with root privileges. Follow the links for instructions for other systems."
