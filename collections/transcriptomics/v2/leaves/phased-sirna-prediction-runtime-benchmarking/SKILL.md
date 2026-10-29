---
name: phased-sirna-prediction-runtime-benchmarking
description: Use when comparing computational efficiency of phasiRNA/phased siRNA
  prediction tools, or when deciding whether PhasiHunter's parallel (-j) mode is needed
  for large genome-based 21-nt phasiRNA prediction workloads.
when_to_use_negative:
- When the goal is to compare prediction accuracy/sensitivity (number of phasiRNAs
  or PHAS loci recovered) rather than elapsed runtime
- When the bowtie indexing/mapping step must be included in the measured pipeline
  time — this benchmark deliberately shares and excludes it
- When benchmarking 24-nt phasiRNA prediction — the reported timings are specific
  to 21-nt phasiRNA prediction (-pl 21)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: PhasiHunter
  role: Tool under benchmark; run via its phase module (-pl 21) in single-threaded
    (-j 1) and four-thread (-j 4) modes against cdna and gdna references
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bowtie
  role: Shared sRNA mapping engine whose index preparation and mapping step is excluded
    from all tool timings
- name: unitas
  role: Comparator tool run single-threaded for 21-nt phasiRNA prediction on both
    references
- name: PhaseTank
  role: Comparator tool run single-threaded for 21-nt phasiRNA prediction on both
    references
- name: PhasiRNAnalyzer
  role: Comparator tool run single-threaded for 21-nt phasiRNA prediction on both
    references
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:3a83c4577db149c80a4b59ad02f56bdd0d29645cb3ccb0a5527c701f25835b45
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# phased-sirna-prediction-runtime-benchmarking

## Summary

Benchmarks the wall-clock runtime of 21-nt phasiRNA prediction tools (PhasiHunter, unitas, PhaseTank, PhasiRNAnalyzer) on rice sRNA data against transcriptome and genome references, excluding the shared bowtie index/mapping step, and quantifies PhasiHunter's multithreaded speedup.

## When to use

Use when comparing computational efficiency of phasiRNA/phased siRNA prediction tools, or when deciding whether PhasiHunter's parallel (-j) mode is needed for large genome-based 21-nt phasiRNA prediction workloads.

## When NOT to use

- When the goal is to compare prediction accuracy/sensitivity (number of phasiRNAs or PHAS loci recovered) rather than elapsed runtime
- When the bowtie indexing/mapping step must be included in the measured pipeline time — this benchmark deliberately shares and excludes it
- When benchmarking 24-nt phasiRNA prediction — the reported timings are specific to 21-nt phasiRNA prediction (-pl 21)

## Inputs

- PhasiHunter demo dataset test_osa (https://cbi.njau.edu.cn/PhasiHunter_demo_data/test_osa.tar.gz) containing SRR5049781.fastq.gz sRNA library
- oryza_sativa_cdna.fa reference transcriptome fasta
- oryza_sativa_gdna.fa reference genome fasta
- Preprocessed bowtie mapping files (cdna and gdna .map outputs from phasiHunter preprocess)
- SRR5049781_trimmed_format_filter.fa length/abundance-filtered sRNA fasta

## Outputs

- Runtime table with one row per tool per reference (transcriptome and genome), single-threaded
- Separate runtime entry for four-threaded (-j 4) PhasiHunter on the genome reference
- Validated comparison against reported benchmark values (~21.41/17.56/35.24/181.93 min transcriptome; ~91.03/158.30/315.55/919.32 min genome; ~45 min four-thread)

## How to apply

Use the PhasiHunter demo dataset (test_osa) with the rice sRNA library SRR5049781 and the oryza_sativa_cdna and oryza_sativa_gdna references. First run the phasiHunter preprocess module (r mode for cdna, m mode for gdna; -mi 19 -ma 25 -e 1 -n 1000000) to produce the shared cdna and gdna .map files, so bowtie indexing/mapping is excluded from all tool timings. Then run phasiHunter phase with -pl 21 and -j 1 against the transcriptome (-cm/-c) and genome (-gm/-g), and run unitas, PhaseTank, and PhasiRNAnalyzer each single-threaded on both references. Finally, rerun the genome-based phasiHunter phase step with -j 4 to measure parallel speedup. Compile runtimes per tool per reference and accept results that approximately reproduce the reported values (transcriptome ≈ 21.41/17.56/35.24/181.93 min for PhasiHunter/unitas/PhaseTank/PhasiRNAnalyzer; genome ≈ 91.03/158.30/315.55/919.32 min; four-thread genome PhasiHunter ≈ 45 min).

## Related tools

- **PhasiHunter** (Tool under benchmark; run via its phase module (-pl 21) in single-threaded (-j 1) and four-thread (-j 4) modes against cdna and gdna references) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bowtie** (Shared sRNA mapping engine whose index preparation and mapping step is excluded from all tool timings)
- **unitas** (Comparator tool run single-threaded for 21-nt phasiRNA prediction on both references)
- **PhaseTank** (Comparator tool run single-threaded for 21-nt phasiRNA prediction on both references)
- **PhasiRNAnalyzer** (Comparator tool run single-threaded for 21-nt phasiRNA prediction on both references)

## Examples

```
phasiHunter phase -cm SRR5049781_processed_cdna.map -c oryza_sativa_cdna.fa -gm SRR5049781_processed_gdna.map -g oryza_sativa_gdna.fa -fa SRR5049781_trimmed_format_filter.fa -me b -pl 21 -j 1; time phasiHunter phase -gm SRR5049781_processed_gdna.map -g oryza_sativa_gdna.fa -fa SRR5049781_trimmed_format_filter.fa -me b -pl 21 -j 4
```

## Evaluation signals

- Genome-based single-threaded runtimes approximately reproduce 91.03 min (PhasiHunter), 158.30 min (unitas), 315.55 min (PhaseTank), and 919.32 min (PhasiRNAnalyzer)
- Transcriptome-based single-threaded runtimes approximately reproduce 21.41 min (PhasiHunter), 17.56 min (unitas), 35.24 min (PhaseTank), and 181.93 min (PhasiRNAnalyzer)
- Four-thread PhasiHunter genome runtime is approximately 45 min, confirming near-linear speedup from parallel computing architecture
- All timings exclude the bowtie index preparation and sRNA mapping step, which is identical across tools via the shared preprocess .map files
- PhasiHunter ranks fastest in the genome-based comparison while unitas is fastest on the transcriptome

## Limitations

- Benchmark uses a single rice (Oryza sativa) sRNA dataset (SRR5049781) and demo data, so runtimes may not transfer to other genomes or library sizes
- Hardware specifications (cores, memory, disk) are not reported in the extractable context, so absolute timings are machine-dependent and only relative comparisons should be trusted
- PhasiHunter version number is not explicitly stated (only an update log entry 1.0.1 - 25/02/2024), complicating exact reproduction
- No discussion section content is present, so limitations or reproducibility caveats cannot be extracted from that section

## Evidence

- [results] For genome-based 21-nt phasiRNA prediction, PhasiHunter was fastest at 91.03 min in single-threaded mode, followed by unitas, PhaseTank, and PhasiRNAnalyzer at 158.30, 315.55, and 919.32 min respectively.: [evidence span withheld — anchor status: not_found]
- [abstract] the excellent data structure and parallel computing architecture allow PhasiHunter to predict phasiRNAs and their regulatory pathways with high efficiency: "the excellent data structure and parallel computing architecture allow PhasiHunter to predict phasiRNAs and their regulatory pathways with high efficiency"
- [readme] -pl: int   --  phase length, 21 | 24, default=21: "-pl: int   --  phase length, 21 | 24, default=21"
- [readme] -j:  int   --  parallel number, default=1: "-j:  int   --  parallel number, default=1"
- [readme] https://cbi.njau.edu.cn/PhasiHunter_demo_data/test_osa.tar.gz: "**Download link**

`https://cbi.njau.edu.cn/PhasiHunter_demo_data/test_osa.tar.gz`"
