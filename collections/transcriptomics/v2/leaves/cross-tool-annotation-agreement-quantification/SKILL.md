---
name: cross-tool-annotation-agreement-quantification
description: Use when you have 3' UTR predictions from multiple annotation tools plus
  a trusted canonical annotation (e.g. WormBase chrI GFF) and need to quantify which
  tool agrees with the reference.
when_to_use_negative:
- There is no trusted canonical annotation to compare against, so agreement fractions
  cannot be computed
- Only one tool's predictions are available, making cross-tool agreement and shared-UTR
  counting impossible
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0622
tools:
- name: peaks2utr
  role: Tool under benchmark; annotates 3' UTRs from BAM reads and a GFF/GTF canonical
    annotation
  repo: https://github.com/haessar/peaks2utr
- name: GETUTR
  role: Competing 3' UTR prediction tool run on the same chromosome I BAM for comparison
- name: UTRme
  role: Competing 3' UTR prediction tool run on the same chromosome I BAM for comparison
- name: bedtools
  role: Required dependency for peaks2utr during the benchmark run
  repo: https://bedtools.readthedocs.io/en/latest/content/installation.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:79ada7cd043aecec36dcaad9042b881e186edfeaefc231eb2ff28adb36a983b4
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
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cross-tool-annotation-agreement-quantification

## Summary

Benchmark a 3' UTR annotation tool (peaks2utr) against competing tools (GETUTR, UTRme) by comparing each tool's predicted UTRs to a canonical reference annotation within a fixed end-position tolerance, and quantify per-tool agreement fractions and shared predictions.

## When to use

Use when you have 3' UTR predictions from multiple annotation tools plus a trusted canonical annotation (e.g. WormBase chrI GFF) and need to quantify which tool agrees best with the reference.

## When NOT to use

- There is no trusted canonical annotation to compare against, so agreement fractions cannot be computed
- Only one tool's predictions are available, making cross-tool agreement and shared-UTR counting impossible

## Inputs

- Chromosome-level 10x Chromium aligned reads BAM file (C. elegans chromosome I)
- Canonical reference annotation in GFF format (WormBase chrI GFF)
- Predicted 3' UTR sets from peaks2utr (GFF3, source=peaks2utr), GETUTR, and UTRme

## Outputs

- Per-tool agreement fractions versus the canonical annotation (50 bp end-position tolerance)
- Count of UTRs predicted by all three tools (16 on chromosome I)
- UTR-length overprediction check result for peaks2utr
- Summary table utr_comparison_summary.csv

## How to apply

Run each tool on identical input: the same chromosome-level BAM file (here the 10x Chromium C. elegans chromosome I BAM) and the same canonical annotation. For peaks2utr, call it as 'peaks2utr <GFF_IN> <BAM_IN> -p 4 -o peaks2utr_chri_utrs.gff3' on the WormBase chrI GFF annotation; run GETUTR and UTRme on the same BAM and annotation. Count each tool's predicted UTRs that match the canonical annotation end positions within a 50 bp tolerance, and report the agreement fraction (expected: ~60% for peaks2utr vs ~6% for GETUTR and UTRme). Additionally identify UTRs predicted by all three tools (16 on chromosome I) and check whether peaks2utr overpredicts UTR lengths relative to the annotation; write per-tool fractions, shared-UTR count, and overprediction status into a summary CSV (utr_comparison_summary.csv).

## Related tools

- **peaks2utr** (Tool under benchmark; annotates 3' UTRs from BAM reads and a GFF/GTF canonical annotation) — https://github.com/haessar/peaks2utr
- **GETUTR** (Competing 3' UTR prediction tool run on the same chromosome I BAM for comparison)
- **UTRme** (Competing 3' UTR prediction tool run on the same chromosome I BAM for comparison)
- **bedtools** (Required dependency for peaks2utr during the benchmark run) — https://bedtools.readthedocs.io/en/latest/content/installation.html

## Examples

```
peaks2utr <GFF_IN> <BAM_IN> -p 4 -o peaks2utr_chri_utrs.gff3
```

## Evaluation signals

- peaks2utr matches roughly 60% of its UTRs to the WormBase chrI canonical annotation within the 50 bp tolerance
- GETUTR and UTRme match only ~6% of their UTRs to the canonical annotation within 50 bp
- Exactly 16 UTRs are matched across all three tools on chromosome I
- No UTR-length overprediction by peaks2utr relative to the annotation is detected
- peaks2utr output GFF3 contains original features plus 3' UTR features with source=peaks2utr

## Limitations

- Accuracy comparison is limited to a single chromosome-level truth set (C. elegans chromosome I) with a 50 bp tolerance choice
- peaks2utr installation instructions assume a Debian/Ubuntu system with root privileges, and no software version numbers are stated in the paper
- GETUTR and UTRme were not re-optimized on the benchmark input; the 6% figure reflects their behavior on 10x Chromium BAM data

## Evidence

- [abstract] uses the UTR enrichment of single-cell technologies, such as 10× Chromium, to accurately annotate 3' UTRs for a given canonical annotation: "uses the UTR enrichment of single-cell technologies, such as 10× Chromium, to accurately annotate 3' UTRs for a given canonical annotation"
- [intro] peaks2utr is a Python command-line tool that annotates 3' untranslated regions (UTR) for a given set of aligned sequencing reads in BAM format: "peaks2utr is a Python command-line tool that annotates 3' untranslated regions (UTR) for a given set of aligned sequencing reads in BAM format"
- [methods] peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3: "peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3"
- [readme] Outputs a GFF3 annotation file (or GTF with option `--gtf`) including original features plus 3' UTR features with `source=peaks2utr`.: "Outputs a GFF3 annotation file (or GTF with option `--gtf`) including original features plus 3' UTR features with `source=peaks2utr`."
