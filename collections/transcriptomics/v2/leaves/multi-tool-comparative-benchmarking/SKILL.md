---
name: multi-tool-comparative-benchmarking
description: Use when comparing execution time of phasiRNA prediction tools on the
  same sRNA library and reference sequences, and you need an apples-to-apples runtime
  comparison that isolates the prediction step from shared preprocessing.
when_to_use_negative:
- When the goal is to compare prediction accuracy or sensitivity (the sets of predicted
  phasiRNAs/PHAS loci) rather than wall-clock runtime — this skill only benchmarks
  execution time.
- 'When timing the full end-to-end pipeline: the benchmark deliberately excludes the
  shared bowtie index preparation and sRNA mapping step, so it is the wrong method
  for measuring preprocess-inclusive cost.'
- When input is raw FASTQ data from your own library rather than the preprocessed
  test_osa .map files — you would first need phasiHunter preprocess and would be timing
  a different workflow.
tools:
- name: PhasiHunter
  role: Benchmarked tool under test; its phase module (-j 1 and -j 4, -pl 21) is timed
    against transcriptome and genome references
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bowtie
  role: Shared dependency providing the common indexing/mapping step whose cost is
    excluded from all timings
- name: unitas
  role: Comparison tool run in single-threaded mode for 21-nt phasiRNA prediction
    on both references
- name: PhaseTank
  role: Comparison tool run in single-threaded mode for 21-nt phasiRNA prediction
    on both references
- name: PhasiRNAnalyzer
  role: Comparison tool run in single-threaded mode for 21-nt phasiRNA prediction
    on both references
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:645e269bd038b1a98b0edafa2dffd3c19e968861aa92c676b409cfa7c3ba0a99
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuangLab-CBI/PhasiHunter
  tools:
  - Bowtie
  - PhaseTank
  - PhasiHunter
  - PhasiRNAnalyzer
  - unitas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Multi-tool comparative benchmarking

## Summary

Benchmark runtime of PhasiHunter against unitas, PhaseTank, and PhasiRNAnalyzer for 21-nt phasiRNA prediction on transcriptome (cDNA) and genome (gDNA) references, excluding the shared bowtie indexing/mapping step from all timings, and quantify multithreaded speedup (-j 4 to ~45 min).

## When to use

Use when comparing execution time of phasiRNA prediction tools on the same sRNA library and reference sequences, and you need an apples-to-apples runtime comparison that isolates the prediction step from shared preprocessing.

## When NOT to use

- When the goal is to compare prediction accuracy or sensitivity (the sets of predicted phasiRNAs/PHAS loci) rather than wall-clock runtime — this skill only benchmarks execution time.
- When timing the full end-to-end pipeline: the benchmark deliberately excludes the shared bowtie index preparation and sRNA mapping step, so it is the wrong method for measuring preprocess-inclusive cost.
- When input is raw FASTQ data from your own library rather than the preprocessed test_osa .map files — you would first need phasiHunter preprocess and would be timing a different workflow.

## Inputs

- PhasiHunter demo dataset test_osa (https://cbi.njau.edu.cn/PhasiHunter_demo_data/test_osa.tar.gz) with rice sRNA library SRR5049781
- Preprocessed mapping files: SRR5049781_processed_cdna.map and SRR5049781_processed_gdna.map from phasiHunter preprocess
- Reference transcriptome fasta (oryza_sativa_cdna.fa) and reference genome fasta (oryza_sativa_gdna.fa)
- sRNA fasta (SRR5049781_trimmed_format_filter.fa) for the phase module -fa input

## Outputs

- Runtime table with one row per tool (PhasiHunter, unitas, PhaseTank, PhasiRNAnalyzer) per reference type (transcriptome, genome)
- Separate entry for four-threaded PhasiHunter on the genome reference (~45 min vs ~91.03 min single-threaded)

## How to apply

Obtain the PhasiHunter demo dataset (test_osa) containing the rice sRNA library (SRR5049781), the oryza_sativa_cdna and oryza_sativa_gdna references, and the preprocessed .map files from phasiHunter preprocess so the shared bowtie indexing/mapping is excluded from timing. Run phasiHunter phase in single-threaded mode (-j 1) for 21-nt phasiRNA prediction (-pl 21) against the transcriptome (-cm/-c with the cdna map and fasta) and then against the genome (-gm/-g with the gdna map and fasta), recording elapsed time for each. Run unitas, PhaseTank, and PhasiRNAnalyzer in single-threaded mode on both references under the identical exclusion of bowtie indexing/mapping. Finally run phasiHunter phase on the genome reference with four parallel threads (-j 4) to measure the speedup. Compile one row per tool per reference plus the four-thread entry, and validate against the reported values (≈21.41/17.56/35.24/181.93 min on transcriptome; ≈91.03/158.30/315.55/919.32 min on genome; ≈45 min for four-thread PhasiHunter).

## Related tools

- **PhasiHunter** (Benchmarked tool under test; its phase module (-j 1 and -j 4, -pl 21) is timed against transcriptome and genome references) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bowtie** (Shared dependency providing the common indexing/mapping step whose cost is excluded from all timings)
- **unitas** (Comparison tool run in single-threaded mode for 21-nt phasiRNA prediction on both references)
- **PhaseTank** (Comparison tool run in single-threaded mode for 21-nt phasiRNA prediction on both references)
- **PhasiRNAnalyzer** (Comparison tool run in single-threaded mode for 21-nt phasiRNA prediction on both references)

## Examples

```
/usr/bin/time phasiHunter phase -cm /home/user/test_osa/SRR5049781_processed_cdna.map -c /home/user/test_osa/oryza_sativa_cdna.fa -gm /home/user/test_osa/SRR5049781_processed_gdna.map -g /home/user/test_osa/oryza_sativa_gdna.fa -fa /home/user/test_osa/SRR5049781_trimmed_format_filter.fa -pl 21 -me b -j 4
```

## Evaluation signals

- Measured transcriptome runtimes approximately reproduce 21.41 (PhasiHunter), 17.56 (unitas), 35.24 (PhaseTank), and 181.93 (PhasiRNAnalyzer) minutes in single-threaded mode.
- Measured genome runtimes approximately reproduce 91.03, 158.30, 315.55, and 919.32 minutes for PhasiHunter, unitas, PhaseTank, and PhasiRNAnalyzer respectively.
- Four-threaded PhasiHunter on the genome (-j 4) drops from ~91.03 min to ~45 min, confirming the parallel-computing speedup claim.
- All four tools use the same shared bowtie index/mapping outputs (cdna and gdna .map files) and none of the timings include that preprocessing step.
- Every tool is run in single-threaded mode (-j 1) with phase length 21 (-pl 21) so the comparison is configuration-matched.

## Limitations

- Runtimes are machine-dependent; absolute minutes will vary with hardware, so validation must accept approximate reproduction of the reported values rather than exact matches.
- The benchmark excludes bowtie indexing/mapping, so it does not reflect total end-to-end pipeline cost.
- The PhasiHunter README lists no complete changelog (only update log entry '1.0.1 - 25/02/2024'), so the exact software version used for the benchmark is not documented; no discussion-section limitations were extractable.
- Timing comparison covers 21-nt phasiRNA prediction only; the 24-nt mode is not benchmarked here.

## Evidence

- [abstract] Parallel architecture claim from abstract: "the excellent data structure and parallel computing architecture allow PhasiHunter to predict phasiRNAs and their regulatory pathways with high efficiency"
- [methods] Phase module parallel parameter: "-j:  int   --  parallel number, default=1"
- [readme] Phase length parameter for the 21-nt benchmark: "-pl: int   --  phase length, 21 | 24, default=21"
- [readme] Prediction method parameter used in the benchmarked phase runs: "-me: str   --  phasiRNA prediction method, h(hypergeometric test) | p(phase score) | b (both), default=b"
- [methods] PhasiHunter runs as a CLI on Linux, the benchmarking environment: "phasihunter is a CLI program runing on linux platform"
