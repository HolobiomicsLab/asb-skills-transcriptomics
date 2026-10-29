---
name: gff3-annotation-parsing
description: Use when a workflow requires reading gene models from a GFF3/GTF canonical
  annotation, merging tool-annotated 3' UTR features (source=peaks2utr) with original
  features, or comparing predicted UTRs against a canonical GFF truth set.
when_to_use_negative:
- The annotation is already a peaks2utr output GFF3 with source=peaks2utr 3' UTR features
  and no further UTR annotation is needed.
- Input gene models are not in GFF/GTF format (e.g. raw BED or SAM) — convert first
  or the tool cannot accept them.
- The task is de novo transcript discovery without a canonical annotation to anchor
  UTR annotation against.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: peaks2utr
  role: Reads GFF/GTF canonical annotation as input and writes an augmented GFF3 (or
    GTF) with 3' UTR features
  repo: https://github.com/haessar/peaks2utr
- name: GenomeTools
  role: Optional post-processing and normalization of the output gff3 annotation
  repo: https://github.com/genometools/genometools
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:0ae4889c3e5ba205b0a440551e7049c85e46cccf761d2968c15b0de04835a367
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - GETUTR
  - GenomeTools
  - Python 3
  - UTRme
  - bedtools
  - peaks2utr
  - pip
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# gff3-annotation-parsing

## Summary

Parse, validate, and manipulate GFF3 (or GTF) gene-model annotations — e.g. canonical annotations like WormBase chrI or T. brucei GFF3 — as both input to and output of peaks2utr-style 3' UTR annotation pipelines, optionally normalizing with GenomeTools.

## When to use

Use when a workflow requires reading gene models from a GFF3/GTF canonical annotation, merging tool-annotated 3' UTR features (source=peaks2utr) with original features, or comparing predicted UTRs against a canonical GFF truth set.

## When NOT to use

- The annotation is already a peaks2utr output GFF3 with source=peaks2utr 3' UTR features and no further UTR annotation is needed.
- Input gene models are not in GFF/GTF format (e.g. raw BED or SAM) — convert first or the tool cannot accept them.
- The task is de novo transcript discovery without a canonical annotation to anchor UTR annotation against.

## Inputs

- Canonical gene-model annotation in GFF3 or GTF format (e.g. WormBase chrI GFF, Tb927_01_v5.1.gff)
- peaks2utr output GFF3 containing original features plus 3' UTR features with source=peaks2utr

## Outputs

- Annotated GFF3 file (or GTF with --gtf) with original features plus 3' UTR features with source=peaks2utr
- Normalized annotation file after GenomeTools post-processing
- Per-gene UTR classification counts (new / altered / unchanged) relative to the canonical annotation

## How to apply

Take the canonical annotation (e.g. WormBase chrI GFF, T. brucei GFF3/GTF such as Tb927_01_v5.1.gff) as GFF_IN to the pipeline; peaks2utr accepts either GFF3 or GTF, with existing 3' UTRs optional and tunable to extend, override or ignore pre-existing 3' UTR annotations. Run the annotation and read the output GFF3, which retains all original features plus new 3' UTR features flagged with source=peaks2utr (use --gtf for GTF output instead). Optionally post-process and normalize the resulting gff3 with GenomeTools. For benchmarking, compare predicted UTR end positions against the canonical annotation within a 50 bp tolerance (e.g. ~60% agreement for peaks2utr vs ~6% for GETUTR/UTRme on C. elegans chromosome I), and tally new (3709), altered (5347), and unchanged (832) UTRs against canonical gene models.

## Related tools

- **peaks2utr** (Reads GFF/GTF canonical annotation as input and writes an augmented GFF3 (or GTF) with 3' UTR features) — https://github.com/haessar/peaks2utr
- **GenomeTools** (Optional post-processing and normalization of the output gff3 annotation) — https://github.com/genometools/genometools

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3
```

## Evaluation signals

- Output GFF3 contains all original input features plus 3' UTR features with source=peaks2utr
- peaks2utr-check demo on Tb927_01_v5.1.gff produces Tb927_01_v5.1.new.gff with original annotations plus 3' UTRs
- Against the C. elegans chrI truth set, ~60% of predicted UTRs agree with the canonical annotation within a 50 bp end-position tolerance (vs ~6% for GETUTR/UTRme), with 16 UTRs shared across all three tools
- Genome-scale T. brucei run yields 9888 total 3' UTRs covering ~84% of 11703 genes: 3709 new, 5347 altered, 832 unchanged
- GenomeTools post-processing of the output gff3 completes without parser/validation errors

## Limitations

- Installation instructions assume a Debian / Ubuntu system with root privileges; other systems require following linked dependency instructions
- No software version numbers are stated in the methods and no changelog was found
- Existing 3' UTR annotations in the input GFF can be extended, overridden or ignored — the tuning choice affects comparison against the canonical annotation

## Evidence

- [intro] peaks2utr is a Python command-line tool that annotates 3' untranslated regions (UTR) for a given set of aligned sequencing reads in BAM format, and canonical annotation in GFF or GTF format.: "peaks2utr is a Python command-line tool that annotates 3' untranslated regions (UTR) for a given set of aligned sequencing reads in BAM format, and canonical annotation in GFF or GTF format."
- [readme] peaks2utr can be tuned to extend, override or ignore any pre-existing 3' UTR annotations in the input GFF file.: [evidence span withheld — anchor status: external_doc]
- [readme] Outputs a GFF3 annotation file (or GTF with option `--gtf`) including original features plus 3' UTR features with `source=peaks2utr`.: "Outputs a GFF3 annotation file (or GTF with option `--gtf`) including original features plus 3' UTR features with `source=peaks2utr`."
- [methods] [GenomeTools](https://github.com/genometools/genometools#building-and-installation) (for post-processing of output gff3): "[GenomeTools](https://github.com/genometools/genometools#building-and-installation) (for post-processing of output gff3)"
- [readme] GFF_IN - gene models in either GFF3 or GTF format (existing 3' UTRs optional).: "`GFF_IN` - gene models in either GFF3 or GTF format (existing 3' UTRs optional)."
- [readme] Installation instructions assume a Debian / Ubuntu system with root privileges.: "Installation instructions assume a Debian / Ubuntu system with root privileges."
