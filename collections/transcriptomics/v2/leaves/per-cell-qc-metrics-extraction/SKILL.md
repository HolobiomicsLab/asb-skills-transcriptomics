---
name: per-cell-qc-metrics-extraction
description: Use when two cellranger count runs on identical reads must be compared
  by per-cell QC summary statistics to quantify the effect of swapping the reference
  annotation (e.g. canonical vs. peaks2utr-augmented GFF3).
when_to_use_negative:
- There is only a single Cell Ranger run with no second reference/condition to compare
  against
- The BAM reads are not from a 3'-enriched single-cell protocol — peaks2utr favours
  BAM files from runs like 10x Chromium where signal is concentrated at UTR distal
  ends
- The per-cell metrics table already exists (e.g. the cellranger_per_cell_metrics_comparison.csv
  has been produced) and only downstream interpretation is needed
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: peaks2utr
  role: Augments the canonical T. brucei GFF3 with 3' UTR features (source=peaks2utr)
    derived from the 10x Chromium BAM, creating the reference used for the improved
    Cell Ranger run
  repo: https://github.com/haessar/peaks2utr
- name: GenomeTools
  role: Post-processing of the peaks2utr output gff3 before reference building
  repo: https://github.com/genometools/genometools
- name: Cell Ranger
  role: Builds the canonical and peaks2utr-augmented references (mkref) and runs count
    on identical reads to generate the metrics_summary per-cell QC statistics
- name: MACS3
  role: Called internally by peaks2utr to detect broad peaks of significant read coverage
    in the BAM used for UTR annotation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:e1b156171cc9696194f89c6e1adafc03382f6b136d92512bc4db1d1ee985501a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - Cell Ranger
  - GenomeTools
  - MACS3
  - peaks2utr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# per-cell-qc-metrics-extraction

## Summary

Extracts and compares per-cell quality metrics (median genes per cell, median UMIs per cell) from paired Cell Ranger 6.1.1 runs — one against a canonical reference, one against a peaks2utr-augmented reference — to quantify how 3' UTR annotation changes alter T. brucei 10x Chromium scRNA-seq capture efficiency.

## When to use

Use when two cellranger count runs on identical reads must be compared by per-cell QC summary statistics to quantify the effect of swapping the reference annotation (e.g. canonical vs. peaks2utr-augmented GFF3).

## When NOT to use

- There is only a single Cell Ranger run with no second reference/condition to compare against
- The BAM reads are not from a 3'-enriched single-cell protocol — peaks2utr favours BAM files from runs like 10x Chromium where signal is concentrated at UTR distal ends
- The per-cell metrics table already exists (e.g. the cellranger_per_cell_metrics_comparison.csv has been produced) and only downstream interpretation is needed

## Inputs

- 10x Chromium T. brucei scRNA-seq reads (aligned BAM input for peaks2utr)
- Canonical T. brucei gene models in GFF3 or GTF format
- peaks2utr-augmented GFF3 with 3' UTR features (source=peaks2utr)
- Cell Ranger 6.1.1 custom references built via cellranger mkref
- metrics_summary outputs from each cellranger count run

## Outputs

- Per-run median genes per cell and median UMIs per cell
- cellranger_per_cell_metrics_comparison.csv with side-by-side metrics and computed fold-improvement

## How to apply

First generate the augmented annotation by running peaks2utr on the canonical T. brucei GFF3/GTF and the aligned 10x Chromium BAM, producing a GFF3 that retains original features plus 3' UTR features with source=peaks2utr (optionally post-processed with GenomeTools). Build two Cell Ranger 6.1.1 custom references with cellranger mkref: one from the canonical GFF3 and one from the peaks2utr-augmented GFF3. Run cellranger count on the identical reads against each reference with all other parameters matched, then extract median genes per cell and median UMIs per cell from each run's metrics_summary output. Tabulate the two conditions side by side, compute the fold-improvement attributable to the extended 3' UTR annotation, and write cellranger_per_cell_metrics_comparison.csv.

## Related tools

- **peaks2utr** (Augments the canonical T. brucei GFF3 with 3' UTR features (source=peaks2utr) derived from the 10x Chromium BAM, creating the reference used for the improved Cell Ranger run) — https://github.com/haessar/peaks2utr
- **GenomeTools** (Post-processing of the peaks2utr output gff3 before reference building) — https://github.com/genometools/genometools
- **Cell Ranger** (Builds the canonical and peaks2utr-augmented references (mkref) and runs count on identical reads to generate the metrics_summary per-cell QC statistics)
- **MACS3** (Called internally by peaks2utr to detect broad peaks of significant read coverage in the BAM used for UTR annotation)

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3
```

## Evaluation signals

- Median genes per cell rises from 632 (canonical reference) to 889 (peaks2utr reference), a ~33% improvement
- Median UMIs per cell rises from 797 to 1111, a ~39% improvement
- Both cellranger count runs used identical reads and matched parameters, differing only in the reference annotation
- The peaks2utr output GFF3 contains 3' UTR features with source=peaks2utr in addition to the original features
- Results are tabulated in cellranger_per_cell_metrics_comparison.csv with the fold-improvement computed

## Limitations

- peaks2utr favours BAM files from 10x Chromium-type runs where signal is inherently concentrated at the distal ends of 3' or 5' UTRs; it is not suited to data without this UTR enrichment
- Installation of peaks2utr and dependencies assumes a Debian/Ubuntu system with root privileges (apt-get install bedtools, genometools)
- No software version numbers were stated for the peaks2utr release used, and no changelog is available
- GenomeTools is optional and only needed for post-processing of the output gff3

## Evidence

- [abstract] UTR enrichment in 10x Chromium justifies the augmented reference: "uses the UTR enrichment of single-cell technologies, such as 10× Chromium, to accurately annotate 3' UTRs for a given canonical annotation"
- [readme] Augmented GFF3 output structure: "Outputs a GFF3 annotation file (or GTF with option `--gtf`) including original features plus 3' UTR features with `source=peaks2utr`."
- [readme] Input data suitability: "This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs."
- [results] Expected per-cell metric improvements: [evidence span withheld — anchor status: not_found]
