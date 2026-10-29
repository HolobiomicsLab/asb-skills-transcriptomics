---
name: 10x-chromium-single-cell-read-counting
description: Use when 10x Chromium scRNA-seq reads must be counted against competing
  reference annotations to measure how a modified GFF3 (e.g., extended 3' UTRs) changes
  per-cell genes and UMI metrics.
when_to_use_negative:
- The biological question is UTR identification itself — use peaks2utr on BAM + canonical
  GFF3 instead of re-running Cell Ranger
- Reads are not from a 3'-tagged single-cell chemistry (e.g., bulk RNA-seq or full-length
  protocols) where signal is not concentrated at 3' UTR distal ends
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell Ranger
  role: mkref builds the two custom references; count quantifies per-cell genes and
    UMIs against each reference on identical reads
- name: peaks2utr
  role: Generates the augmented GFF3 reference whose 3' UTR features (source=peaks2utr)
    drive the improved per-cell counts
  repo: https://github.com/haessar/peaks2utr
- name: GenomeTools
  role: Post-processes the peaks2utr output GFF3 before it is packaged into the Cell
    Ranger reference
  repo: https://github.com/genometools/genometools
- name: 10x Chromium
  role: Source scRNA-seq chemistry whose 3'-enriched read signal is exploited by the
    augmented annotation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:0ec5e67b25b0d34ad14dbfb629a5daac820844355d93b288bb06b1044da7423c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - 10x Chromium
  - Cell Ranger
  - GenomeTools
  - peaks2utr
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# 10x Chromium single-cell read counting

## Summary

Re-runs Cell Ranger 6.1.1 count on identical 10x Chromium T. brucei scRNA-seq reads against two GFF3-based references (canonical vs peaks2utr-augmented 3' UTR annotation) to quantify per-cell gains in genes captured and UMIs attributable to extended 3' UTR annotation.

## When to use

Use when 10x Chromium scRNA-seq reads must be counted against competing reference annotations to measure how a modified GFF3 (e.g., extended 3' UTRs) changes per-cell genes and UMI metrics.

## When NOT to use

- The biological question is UTR identification itself — use peaks2utr on BAM + canonical GFF3 instead of re-running Cell Ranger
- Reads are not from a 3'-tagged single-cell chemistry (e.g., bulk RNA-seq or full-length protocols) where signal is not concentrated at 3' UTR distal ends

## Inputs

- 10x Chromium scRNA-seq reads (BAM or FASTQ) aligned to the T. brucei genome
- Canonical T. brucei gene models in GFF3 (or GTF) format
- peaks2utr-augmented GFF3 annotation with 3' UTR features (source=peaks2utr) merged with original gene models

## Outputs

- Two cellranger count runs (one per reference) with metrics_summary outputs
- cellranger_per_cell_metrics_comparison.csv tabulating median genes per cell (632 vs 889) and median UMIs per cell (797 vs 1111) with fold improvement

## How to apply

Build two Cell Ranger 6.1.1 custom references with cellranger mkref: one from the canonical T. brucei GFF3 and one from the peaks2utr-augmented GFF3, whose output contains original gene models plus 3' UTR features with source=peaks2utr. Run cellranger count on the identical 10x Chromium BAM/FASTQ inputs against both references, keeping all parameters matched between the two runs so any metric difference is attributable to the annotation alone. Extract median genes per cell and median UMIs per cell from each run's metrics_summary output, tabulate the conditions side by side, and compute the fold improvement. For the T. brucei case, the expected result is median genes/cell rising from 632 to 889 (~33%) and median UMIs/cell from 797 to 1111 (~39%), because 10x Chromium signal is inherently concentrated at the distal ends of the 3' UTRs, so longer UTR annotation recovers more reads per gene.

## Related tools

- **Cell Ranger** (mkref builds the two custom references; count quantifies per-cell genes and UMIs against each reference on identical reads)
- **peaks2utr** (Generates the augmented GFF3 reference whose 3' UTR features (source=peaks2utr) drive the improved per-cell counts) — https://github.com/haessar/peaks2utr
- **GenomeTools** (Post-processes the peaks2utr output GFF3 before it is packaged into the Cell Ranger reference) — https://github.com/genometools/genometools
- **10x Chromium** (Source scRNA-seq chemistry whose 3'-enriched read signal is exploited by the augmented annotation)

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3; cellranger mkref --gff=output.gff3 --fasta=genome.fa; cellranger count --id=peaks2utr_run --transcriptome=peaks2utr_ref
```

## Evaluation signals

- Median genes per cell increases from 632 (canonical reference) to 889 (peaks2utr-augmented reference), a ~33% improvement
- Median UMIs per cell increases from 797 to 1111, a ~39% improvement
- Both count runs used identical input reads and matched parameters, so metric deltas are attributable to the annotation change alone
- The augmented reference GFF3 contains original features plus 3' UTR features with source=peaks2utr, verified via peaks2utr-check/demo output naming convention

## Limitations

- Improvement figures are specific to the T. brucei dataset and Cell Ranger 6.1.1; they are not guaranteed for other organisms, chemistries, or Cell Ranger versions
- peaks2utr favours BAM files from runs like 10x Chromium where signal is concentrated at UTR distal ends; alignments without that enrichment pattern may not yield usable 3' UTR extensions
- No software version numbers were stated in the article's installation instructions, so exact peaks2utr/GenomeTools versions must be pinned independently

## Evidence

- [results] Rerunning Cell Ranger 6.1.1 with the peaks2utr-updated T. brucei reference increased genes captured per cell from 632 to 889 (~33%) and UMI counts per cell from 797 to 1111 (~39%): [evidence span withheld — anchor status: not_found]
- [abstract] peaks2utr uses UTR enrichment of single-cell technologies such as 10x Chromium to accurately annotate 3' UTRs: "uses the UTR enrichment of single-cell technologies, such as 10× Chromium, to accurately annotate 3' UTRs for a given canonical annotation"
- [readme] This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs.: "This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs."
- [methods] Outputs a GFF3 annotation file (or GTF with option --gtf) including original features plus 3' UTR features with source=peaks2utr.: "Outputs a GFF3 annotation file (or GTF with option `--gtf`) including original features plus 3' UTR features with `source=peaks2utr`."
