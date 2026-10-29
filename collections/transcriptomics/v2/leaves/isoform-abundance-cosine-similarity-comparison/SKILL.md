---
name: isoform-abundance-cosine-similarity-comparison
description: Use when benchmarking isoform detection and quantification accuracy of
  long-read transcript tools against a set of known reference isoforms (e.g. SIRV
  spike-ins with PacBio data), needing detection counts plus cosine similarity of
  abundance profiles.
when_to_use_negative:
- The dataset has no known reference isoforms or expected abundances to compare against
  (no ground truth for exact matching or cosine similarity)
- Input is short-read sequencing data — this skill is grounded in long-read (PacBio
  Iso-Seq, ONT) isoform analysis with GMAP alignments
- The goal is de novo transcriptome discovery/annotation genome-wide (e.g. a bambu/SQANTI3
  reference-guided workflow) rather than per-gene or spike-in benchmarking against
  a known isoform panel
tools:
- name: VIsoQLR
  role: Tool under benchmark; detects and quantifies isoforms from aligned reads,
    with a configurable read threshold (default 2%)
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: FLAIR
  role: Comparator run with default parameters and the provided SIRV GTF for isoform
    correction
- name: StringTie2
  role: Comparator run in long-read mode ('-L', '-f 0') on the same BAM to prevent
    filtering of low-confidence transcripts
- name: GMAP
  role: Aligner producing the shared BAM/BAI alignments that all compared tools consume
- name: TBLabFJD/VIsoQLR_benchmark scripts
  role: R/shell scripts (isoform_analysis.sh, isoform_comparison.R) performing the
    isoform comparison across the three tools
  repo: https://github.com/TBLabFJD/VIsoQLR_benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:bb72a3f07d632e4652c77d5053bf991719f6a65e47dc7a6bff8c1975d20e3bf5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - FLAIR
  - GMAP
  - StringTie2
  - TBLabFJD/VIsoQLR_benchmark scripts
  - VIsoQLR
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# isoform-abundance-cosine-similarity-comparison

## Summary

Benchmarks long-read isoform detection/quantification tools (VIsoQLR, FLAIR, StringTie2) by comparing each tool's assembled isoform set against known reference isoforms and computing the cosine similarity between predicted and reference isoform abundance profiles, as done on SIRV spike-ins with GMAP-aligned PacBio Iso-Seq reads.

## When to use

Use when benchmarking isoform detection and quantification accuracy of long-read transcript tools against a set of known reference isoforms (e.g. SIRV spike-ins with PacBio data), needing detection counts plus cosine similarity of abundance profiles.

## When NOT to use

- The dataset has no known reference isoforms or expected abundances to compare against (no ground truth for exact matching or cosine similarity)
- Input is short-read sequencing data — this skill is grounded in long-read (PacBio Iso-Seq, ONT) isoform analysis with GMAP alignments
- The goal is de novo transcriptome discovery/annotation genome-wide (e.g. a bambu/SQANTI3 reference-guided workflow) rather than per-gene or spike-in benchmarking against a known isoform panel

## Inputs

- GMAP-aligned long-read alignments (BAM plus BAI index) for all tools being compared
- Reference GTF annotation of known isoforms (e.g. SIRV GTF with 68 isoforms and their expected abundances)
- Assembled isoform outputs per tool: VIsoQLR isoform calls/quantifications, FLAIR output (default parameters + GTF), StringTie2 output ('-L -f 0')
- Comparison scripts from TBLabFJD/VIsoQLR_benchmark (isoform_comparison.R)

## Outputs

- Per-tool isoform detection counts matched exactly against the known isoform set (e.g. x/68 SIRV isoforms)
- Per-tool cosine similarity between predicted and reference isoform abundance profiles (expected FLAIR 0.68, StringTie2 0.42)
- Ranking of tools by isoform detection and quantification accuracy

## How to apply

Assemble isoforms from the same GMAP-aligned long-read BAM (plus BAI index) with each comparator: FLAIR with default parameters and the reference GTF, and StringTie2 in long-read mode ('-L', '-f 0' to prevent filtering of low-confidence transcripts). Match each tool's assembled isoform set exactly against the known reference isoforms (68 SIRV isoforms in the paper) and report detection counts as a fraction of the total. Then compute the cosine similarity between each tool's predicted isoform abundance profile and the reference abundances to capture quantification quality on top of detection. In the SIRV benchmark this yields FLAIR 37/68 (54%) with cosine similarity 0.68, StringTie2 12/68 (18%) with cosine similarity 0.42, versus VIsoQLR at 49/68 (72%) with 99% exon-coordinate accuracy. The comparison is implemented in R via 'isoform_comparison.R' from the TBLabFJD/VIsoQLR_benchmark repository, which compares VIsoQLR, FLAIR and StringTie2 outputs.

## Related tools

- **VIsoQLR** (Tool under benchmark; detects and quantifies isoforms from aligned reads, with a configurable read threshold (default 2%)) — https://github.com/TBLabFJD/VIsoQLR
- **FLAIR** (Comparator run with default parameters and the provided SIRV GTF for isoform correction)
- **StringTie2** (Comparator run in long-read mode ('-L', '-f 0') on the same BAM to prevent filtering of low-confidence transcripts)
- **GMAP** (Aligner producing the shared BAM/BAI alignments that all compared tools consume)
- **TBLabFJD/VIsoQLR_benchmark scripts** (R/shell scripts (isoform_analysis.sh, isoform_comparison.R) performing the isoform comparison across the three tools) — https://github.com/TBLabFJD/VIsoQLR_benchmark

## Examples

```
Rscript isoform_comparison.R  # from TBLabFJD/VIsoQLR_benchmark, after running FLAIR and StringTie2 (-L -f 0) on the GMAP-aligned SIRV BAM
```

## Evaluation signals

- Detection counts reproduce the benchmark: FLAIR 37/68 (54%) and StringTie2 12/68 (18%) of SIRV isoforms, versus VIsoQLR 49/68 (72%)
- Cosine similarities between predicted and reference abundance profiles equal 0.68 for FLAIR and 0.42 for StringTie2
- VIsoQLR detections achieve 99% accuracy of exon coordinates, as reported for the SIRV analysis
- All tools are run on the identical GMAP-aligned BAM (plus BAI) so differences are attributable to the isoform callers, not alignment
- Comparison is performed by isoform_comparison.R rather than ad-hoc matching, ensuring consistent isoform-comparison logic across tools

## Limitations

- No software version numbers were reported for VIsoQLR, StringTie2, or FLAIR in the benchmark, complicating exact reproduction
- StringTie2 extends isoform boundaries (an artifact not present in the BAM files) and does not call isoforms lacking certain exons, biasing detection counts
- VIsoQLR's automatic analysis misses isoforms whose reads fall below the default 2% read threshold; lowering the threshold detects more isoforms at the cost of more spurious calls
- Very low-abundance isoforms (e.g. 4% and 1.7%) may be overrepresented by LRS and are hardly distinguishable from noise in reference electropherograms

## Evidence

- [readme] Benchmark scripts repository: [evidence span withheld — anchor status: external_doc]
- [readme] Comparison script: [evidence span withheld — anchor status: not_found]
- [discussion] VIsoQLR outperforms comparators on SIRVs: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
- [readme] PacBio SIRV source data: [evidence span withheld — anchor status: external_doc]
- [discussion] Benchmark scope: "we have performed a benchmark to determine the quality of the isoforms detected by VIsoQLR, StringTie2 and FLAIR software in a typical LRS experiment."
