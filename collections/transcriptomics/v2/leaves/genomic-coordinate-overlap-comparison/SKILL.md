---
name: genomic-coordinate-overlap-comparison
description: Use when comparing predicted UTR (or other interval) coordinates from
  multiple tools against a reference annotation and you need a tolerant agreement
  fraction rather than exact-match counting.
when_to_use_negative:
- The input is already a summary table of agreement fractions rather than raw coordinate
  annotations needing comparison.
- The question is about UTR annotation quality itself (peak calling, truncation-point
  detection) rather than benchmarking predicted coordinates against a truth set.
- Only one tool's predictions exist and no canonical annotation is available — no
  reference to compare against.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: peaks2utr
  role: Produces the predicted 3' UTR GFF3 annotations (source=peaks2utr) whose coordinates
    are compared against the canonical annotation and against GETUTR/UTRme predictions
  repo: https://github.com/haessar/peaks2utr
- name: bedtools
  role: Required dependency used for interval operations supporting the coordinate
    overlap comparison
  repo: https://bedtools.readthedocs.io/en/latest/content/installation.html
- name: GenomeTools
  role: Optional post-processing of the output GFF3 before comparison
  repo: https://github.com/genometools/genometools
- name: GETUTR
  role: Compared tool whose 3' UTR predictions are scored against the canonical annotation
- name: UTRme
  role: Compared tool whose 3' UTR predictions are scored against the canonical annotation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:bd97ba08b5a829907f86abcd857fbb34e845e795a8f773b603996b074ba4a4ce
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# genomic-coordinate-overlap-comparison

## Summary

Compares predicted 3' UTR genomic coordinates against a canonical annotation using a positional tolerance (50 bp end-position agreement) to quantify annotation accuracy, as done when benchmarking peaks2utr against GETUTR and UTRme on C. elegans chromosome I.

## When to use

Use when comparing predicted UTR (or other interval) coordinates from multiple tools against a reference annotation and you need a tolerant agreement fraction rather than exact-match counting.

## When NOT to use

- The input is already a summary table of agreement fractions rather than raw coordinate annotations needing comparison.
- The question is about UTR annotation quality itself (peak calling, truncation-point detection) rather than benchmarking predicted coordinates against a truth set.
- Only one tool's predictions exist and no canonical annotation is available — no reference to compare against.

## Inputs

- Predicted 3' UTR intervals per tool (GFF3/GTF, e.g. peaks2utr output with source=peaks2utr)
- Canonical reference annotation (WormBase C. elegans chrI GFF)
- Aligned reads BAM shared by all compared tools (10x Chromium chromosome I BAM)
- Tolerance threshold (50 bp end-position agreement)

## Outputs

- Per-tool agreement fraction of predicted UTRs vs canonical annotation within 50 bp (e.g. peaks2utr ~60% vs GETUTR/UTRme ~6%)
- Count of UTRs predicted by all compared tools (16 in the benchmark)
- Overprediction check result (whether predicted UTR lengths exceed the annotation)
- Summary table (utr_comparison_summary.csv)

## How to apply

Run each annotation tool (e.g. peaks2utr, GETUTR, UTRme) on the same aligned-read BAM and canonical GFF/GTF input — for the benchmark, the WormBase C. elegans chromosome I annotation and a 10x Chromium chromosome I BAM. Extract each tool's predicted 3' UTR intervals (peaks2utr emits them as GFF3 features with source=peaks2utr). Compare each tool's predicted UTR end positions against the canonical annotation with a 50 bp tolerance to count agreements, then compute the per-tool agreement fraction. Additionally intersect predictions across tools to count UTRs shared by all three (16 in the benchmark), and check whether any tool systematically overpredicts UTR lengths relative to the reference (peaks2utr did not). Write the agreement fractions, shared-UTR count, and overprediction result into a summary table such as utr_comparison_summary.csv.

## Related tools

- **peaks2utr** (Produces the predicted 3' UTR GFF3 annotations (source=peaks2utr) whose coordinates are compared against the canonical annotation and against GETUTR/UTRme predictions) — https://github.com/haessar/peaks2utr
- **bedtools** (Required dependency used for interval operations supporting the coordinate overlap comparison) — https://bedtools.readthedocs.io/en/latest/content/installation.html
- **GenomeTools** (Optional post-processing of the output GFF3 before comparison) — https://github.com/genometools/genometools
- **GETUTR** (Compared tool whose 3' UTR predictions are scored against the canonical annotation)
- **UTRme** (Compared tool whose 3' UTR predictions are scored against the canonical annotation)

## Examples

```
peaks2utr WormBase_chrI.gff chrI_10x.bam -p 4 -o peaks2utr_chri_utrs.gff3
```

## Evaluation signals

- Reproduced benchmark should show peaks2utr matching roughly 60% of its UTRs to the WormBase chrI canonical annotation within 50 bp, versus about 6% for GETUTR and UTRme.
- Exactly 16 UTRs should be matched across all three tools (peaks2utr, GETUTR, UTRme) on chromosome I.
- The overprediction check should confirm peaks2utr does not overpredict UTR lengths relative to the annotation.
- peaks2utr output GFF3 must contain original features plus 3' UTR features with source=peaks2utr.
- Summary table (utr_comparison_summary.csv) must contain per-tool agreement fractions, the shared-UTR count, and the overprediction result.

## Limitations

- Agreement is measured with a 50 bp end-position tolerance, so calls differing by more than this are counted as disagreements even if biologically plausible.
- Comparison relies on a single reference chromosome (C. elegans chromosome I) and one BAM input; results may not generalize genome-wide or to other data types.
- peaks2utr installation assumes a Debian/Ubuntu system with root privileges (bedtools via apt-get), and no software version numbers are stated in the methods.

## Evidence

- [results] peaks2utr matched ~60% of UTRs to the canonical annotation (within a 50 bp tolerance) on C. elegans chromosome I, compared to only ~6% for GETUTR and UTRme.: [evidence span withheld — anchor status: not_found]
- [readme] Outputs a GFF3 annotation file (or GTF with option --gtf) including original features plus 3' UTR features with source=peaks2utr.: "Outputs a GFF3 annotation file (or GTF with option `--gtf`) including original features plus 3' UTR features with `source=peaks2utr`."
- [methods] peaks2utr is called from the command line as: peaks2utr <GFF_IN> <BAM_IN> [options]: "peaks2utr is called from the command line as:
```
peaks2utr <GFF_IN> <BAM_IN> [options]
```"
- [readme] Installation instructions assume a Debian / Ubuntu system with root privileges.: "Installation instructions assume a Debian / Ubuntu system with root privileges. Follow the links for instructions for other systems."
