---
name: utr-annotation-benchmarking
description: Use when you need to measure whether improved 3' UTR annotation (e.g.
  peaks2utr output with source=peaks2utr) translates into better per-cell genes captured
  and UMI counts in a 10x Chromium scRNA-seq quantification pipeline.
when_to_use_negative:
- The 3' UTR annotation has not yet been generated — this skill benchmarks an existing
  peaks2utr-augmented GFF3, it does not produce UTR annotations (run peaks2utr on
  the GFF + BAM first).
- The read data are not from a UTR-enriched single-cell protocol such as 10x Chromium;
  peaks2utr's signal depends on reads concentrating at UTR distal ends, so per-cell
  metric gains would not be interpretable for bulk or non-polyA-capture data.
- The goal is de novo gene discovery or transcript assembly rather than comparing
  quantification impact of two annotation versions of the same gene models.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3170
tools:
- name: peaks2utr
  role: Generates the augmented GFF3 (original features plus 3' UTR features with
    source=peaks2utr) whose quantification impact this skill benchmarks
  repo: https://github.com/haessar/peaks2utr
- name: Cell Ranger
  role: mkref builds canonical and augmented custom references; count quantifies the
    identical 10x Chromium reads against each to produce per-cell metrics
- name: GenomeTools
  role: Optional post-processing of the peaks2utr output GFF3 before reference building
  repo: https://github.com/genometools/genometools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:5d0ce94c51c405a8a63e52c6e8176b38f1c85b40534bb715f893eaf97d89174e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - Cell Ranger
  - GenomeTools
  - peaks2utr
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# utr-annotation-benchmarking

## Summary

Benchmarking skill that quantifies the downstream effect of peaks2utr-extended 3' UTR annotations by rebuilding a Cell Ranger 6.1.1 custom reference from the augmented GFF3 and comparing per-cell scRNA-seq metrics against the canonical annotation.

## When to use

Use when you need to measure whether improved 3' UTR annotation (e.g. peaks2utr output with source=peaks2utr) translates into better per-cell genes captured and UMI counts in a 10x Chromium scRNA-seq quantification pipeline.

## When NOT to use

- The 3' UTR annotation has not yet been generated — this skill benchmarks an existing peaks2utr-augmented GFF3, it does not produce UTR annotations (run peaks2utr on the GFF + BAM first).
- The read data are not from a UTR-enriched single-cell protocol such as 10x Chromium; peaks2utr's signal depends on reads concentrating at UTR distal ends, so per-cell metric gains would not be interpretable for bulk or non-polyA-capture data.
- The goal is de novo gene discovery or transcript assembly rather than comparing quantification impact of two annotation versions of the same gene models.

## Inputs

- Canonical T. brucei gene model annotation in GFF3/GTF format (e.g. Tb927_01_v5.1.gff)
- peaks2utr-augmented GFF3 containing original features plus 3' UTR features with source=peaks2utr
- 10x Chromium scRNA-seq reads aligned to BAM (the input peaks2utr used, e.g. Tb927_01_v5.1.slice.bam-style file)
- Cell Ranger 6.1.1 reference package files built by cellranger mkref

## Outputs

- cellranger_per_cell_metrics_comparison.csv tabulating median genes per cell and median UMIs per cell for canonical vs peaks2utr-augmented references
- Two Cell Ranger count outputs (metrics_summary per run)
- Fold-improvement figures attributable to the extended 3' UTR annotation

## How to apply

Build two Cell Ranger 6.1.1 custom references with cellranger mkref: one from the canonical T. brucei GFF3 and one from the peaks2utr-augmented GFF3 in which 3' UTR features with source=peaks2utr have been merged into the original gene models (optionally post-processed with GenomeTools). Run cellranger count on the identical 10x Chromium reads against both references, keeping every parameter other than the reference matched between the two runs. Extract median genes per cell and median UMIs per cell from each run's metrics_summary output, tabulate the two conditions side by side, and compute the fold improvement attributable to the extended 3' UTR annotation. The rationale is that correct UTR annotation is crucial for transcriptomic analysis yet mostly overlooked, and peaks2utr exploits the UTR enrichment inherent to 10x Chromium reads, so any gain in per-cell metrics directly evidences annotation quality.

## Related tools

- **peaks2utr** (Generates the augmented GFF3 (original features plus 3' UTR features with source=peaks2utr) whose quantification impact this skill benchmarks) — https://github.com/haessar/peaks2utr
- **Cell Ranger** (mkref builds canonical and augmented custom references; count quantifies the identical 10x Chromium reads against each to produce per-cell metrics)
- **GenomeTools** (Optional post-processing of the peaks2utr output GFF3 before reference building) — https://github.com/genometools/genometools

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3 && cellranger mkref --gff=output.gff3 && cellranger count --id=pb2utr_run --transcriptome=./output
```

## Evaluation signals

- Median genes per cell rises from 632 (canonical reference) to 889 with the peaks2utr-updated reference, a ~33% improvement matching the reported figures.
- Median UMIs per cell rises from 797 to 1111, a ~39% improvement matching the reported figures.
- The peaks2utr count run uses the identical reads and all parameters matched to the baseline run except the reference, so any metric delta is attributable to the annotation.
- The augmented GFF3 passed to cellranger mkref contains original features plus 3' UTR features with source=peaks2utr (and has optionally been post-processed with GenomeTools).
- The comparison is written to cellranger_per_cell_metrics_comparison.csv with the two conditions tabulated side by side.

## Limitations

- Benchmark outcomes are reference-build- and pipeline-version-dependent (figures reported for Cell Ranger 6.1.1); other Cell Ranger versions may shift the per-cell metrics.
- peaks2utr can be tuned to extend, override or ignore pre-existing 3' UTR annotations, so the augmented reference content depends on how the tool was configured before benchmarking.
- Improvement figures are specific to the T. brucei / 10x Chromium dataset; they do not guarantee similar gains for other organisms or library chemistries.

## Evidence

- [abstract] uses the UTR enrichment of single-cell technologies, such as 10× Chromium, to accurately annotate 3' UTRs for a given canonical annotation: "uses the UTR enrichment of single-cell technologies, such as 10× Chromium, to accurately annotate 3' UTRs for a given canonical annotation"
- [abstract] Correct UTR annotation is crucial: "Correct annotation of UTRs is crucial in transcriptomic analysis to accurately capture the expression of each gene yet is mostly overlooked in annotation pipelines"
- [readme] peaks2utr can be tuned to extend, override or ignore any pre-existing 3' UTR annotations: [evidence span withheld — anchor status: external_doc]
- [methods] Outputs a GFF3 annotation file (or GTF with option --gtf) including original features plus 3' UTR features with source=peaks2utr.: "Outputs a GFF3 annotation file (or GTF with option `--gtf`) including original features plus 3' UTR features with `source=peaks2utr`."
- [other] Benchmark task validation figures: [evidence span withheld — anchor status: not_found]
