---
name: assembled-transcript-correctness-scoring
description: Use when you have assembled transcript GTF files from one or more assembly
  conditions and need to quantify how many assembled transcripts match reference annotations
  (correct transcripts) and compute precision per condition.
when_to_use_negative:
- Assemblies were produced without a reference annotation available to score against
  (e.g. purely de novo comparison without a ground-truth transcript set).
- The goal is to compare assemblers rather than quantify correctness of a single assembly
  against reference transcripts.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0080
tools:
- name: Tiglon
  role: Genome-guided transcriptome assembler producing the GTF assemblies that are
    scored for correctness under different alignment-integration conditions
  repo: github.com/yutingsdu/Tiglon
- name: Hisat2
  role: Splice-aware aligner generating one condition's BAM alignments used to build
    assemblies for comparison
  repo: https://github.com/DaehwanKimLab/hisat2
- name: STAR
  role: Splice-aware aligner generating the second condition's BAM alignments used
    to build assemblies for comparison
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2022.104067
    title: Tiglon enables accurate transcriptome assembly via integrating mappings
      of different aligners
  source_content_hash: sha256:eb7a2498a978330ed27eab0c2262667afa641b1d812f460bdaf64a8b49d6f660
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hisat2
  - STAR
  - Star
  - Tiglon
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# Assembled transcript correctness scoring

## Summary

Scores transcriptome assemblies (e.g. Tiglon GTF outputs from Hisat2-only, Star-only, or merged Hisat2&Star alignments) against NCBI RefSeq reference annotations to count correct transcripts and compute assembly precision, enabling quantitative comparison of assembly conditions.

## When to use

Use when you have assembled transcript GTF files from one or more assembly conditions and need to quantify how many assembled transcripts match reference annotations (correct transcripts) and compute precision per condition.

## When NOT to use

- Assemblies were produced without a reference annotation available to score against (e.g. purely de novo comparison without a ground-truth transcript set).
- The goal is to compare assemblers rather than quantify correctness of a single assembly against reference transcripts.

## Inputs

- Tiglon output GTF assemblies (tiglon_outdir/Tiglon.gtf) per condition
- Reference transcript annotations (NCBI RefSeq, e.g. for GRCh38)
- Alignment BAM files for Hisat2-only, Star-only, and merged Hisat2&Star conditions

## Outputs

- Correct-transcript counts and assembly precision per condition
- Per-group percentage improvements (e.g. ablation_improvements.csv)

## How to apply

Run the assembler (e.g. Tiglon -B bamFiles_list -s <strand> -o Tiglon_outdir -p 2) under each condition — Hisat2-only, Star-only, and merged Hisat2&Star BAM lists — collecting each tiglon_outdir/Tiglon.gtf assembly. Score every GTF against NCBI RefSeq reference transcript annotations to count correct transcripts and compute assembly precision per condition. Then, for each species group (H1-H8 human, M1-M4 mouse), calculate the percentage improvement in correct transcripts and precision of the merged-alignments assembly over each single-aligner assembly, writing per-group improvements to a CSV (e.g. ablation_improvements.csv).

## Related tools

- **Tiglon** (Genome-guided transcriptome assembler producing the GTF assemblies that are scored for correctness under different alignment-integration conditions) — github.com/yutingsdu/Tiglon
- **Hisat2** (Splice-aware aligner generating one condition's BAM alignments used to build assemblies for comparison) — https://github.com/DaehwanKimLab/hisat2
- **STAR** (Splice-aware aligner generating the second condition's BAM alignments used to build assemblies for comparison) — https://github.com/alexdobin/STAR

## Examples

```
Tiglon -B bamFiles_list -s <first> -o Tiglon_outdir -p 2
```

## Evaluation signals

- Human group improvements fall within ~±0.5 percentage points of reported values: 16.11% and 13.98% more correct transcripts, 6.47% and 6.91% higher precision (merged vs Hisat2-only and Star-only).
- Mouse group improvements fall within ~±0.5 percentage points of reported values: 9.19%/7.26% more correct transcripts and 12.17%/10.05% higher precision.
- Each condition's Tiglon.gtf exists and is non-empty before scoring.
- Precision and correct-transcript counts are computed consistently per species group (H1-H8 human, M1-M4 mouse) and written to ablation_improvements.csv.

## Limitations

- Correct-transcript scoring depends on the reference annotation chosen (NCBI RefSeq); assemblies of transcripts absent from RefSeq will not be credited.
- Improvement percentages are dataset- and species-specific; the reported values (e.g. 16.11%/13.98% for human) may not transfer to other datasets.

## Evidence

- [results] Tiglon with Hisat2&Star improved precision by 6.47% and 6.91% and the number of correct transcripts by 16.11% and 13.98% over Tiglon with Hisat2-only and Star-only alignments, respectively: [evidence span withheld — anchor status: not_found]
- [methods] Run Tiglon on the Hisat2-only list: [evidence span withheld — anchor status: not_found]
- [methods] collecting each run's tiglon_outdir/Tiglon.gtf assembly: [evidence span withheld — anchor status: not_found]
- [readme] The software takes a file that lists alignment files generated by different aligners as input: "The software takes a file that lists alignment files generated by different aligners as input, and outputs all assembled candidate transcripts in GTF format."
- [methods] Minimum assembled transcript length, default: 500: "--min_trans_length/-L <int>    	: Minimum assembled transcript length, default: 500."
