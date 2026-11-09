---
name: transcriptome-versus-genome-reference-prediction
description: Use when benchmarking phasiRNA prediction tools' runtime on the same
  sRNA library (e.g., rice SRR5049781) across cDNA and gDNA references, or when deciding
  whether multithreading (e.g., -j 4) is needed to make genome-based prediction tractable.
when_to_use_negative:
- When Bowtie index building or sRNA mapping time is itself part of the benchmark
  — this skill explicitly excludes the shared preprocess/mapping step from all timings.
- When benchmarking 24-nt phasiRNA prediction, since the workflow fixes -pl 21.
- When the goal is comparing prediction accuracy (sensitivity/precision of PHAS loci)
  rather than runtime.
edam_operation: http://edamontology.org/operation_3793
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
tools:
- name: PhasiHunter
  role: Primary multithreaded phasiRNA prediction tool timed on transcriptome and
    genome references via its phase module (-cm/-c, -gm/-g, -pl 21, -j 1/-j 4)
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bowtie
  role: Used inside phasiHunter preprocess to build the shared sRNA-to-reference mapping
    files excluded from benchmark timings
- name: unitas
  role: Comparison tool run single-threaded for 21-nt phasiRNA prediction on both
    references
- name: PhaseTank
  role: Comparison tool run single-threaded for 21-nt phasiRNA prediction on both
    references
- name: PhasiRNAnalyzer
  role: Comparison tool run single-threaded for 21-nt phasiRNA prediction on both
    references
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:7981b33967adb195f1fa61225aabfc612a73714ac61ed6c1ffb23f9b1502b107
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
  edam_operation: http://edamontology.org/operation_3793
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# transcriptome-versus-genome-reference-prediction

## Summary

Benchmarking skill for comparing 21-nt phasiRNA prediction runtime across tools (PhasiHunter, unitas, PhaseTank, PhasiRNAnalyzer) against both transcriptome (cDNA) and genome (gDNA) references, using precomputed Bowtie mapping files so index preparation is excluded from all timings.

## When to use

Use when benchmarking phasiRNA prediction tools' runtime on the same sRNA library (e.g., rice SRR5049781) across cDNA and gDNA references, or when deciding whether multithreading (e.g., -j 4) is needed to make genome-based prediction tractable.

## When NOT to use

- When Bowtie index building or sRNA mapping time is itself part of the benchmark — this skill explicitly excludes the shared preprocess/mapping step from all timings.
- When benchmarking 24-nt phasiRNA prediction, since the workflow fixes -pl 21.
- When the goal is comparing prediction accuracy (sensitivity/precision of PHAS loci) rather than runtime.

## Inputs

- rice sRNA library SRR5049781.fastq.gz (PhasiHunter demo data test_osa)
- reference transcriptome FASTA (oryza_sativa_cdna.fa)
- reference genome FASTA (oryza_sativa_gdna.fa)
- preprocessed Bowtie mapping files (.map) from phasiHunter preprocess for cdna and gdna
- installed copies of unitas, PhaseTank, and PhasiRNAnalyzer

## Outputs

- elapsed runtime per tool per reference (single-threaded) for 21-nt phasiRNA prediction
- four-threaded PhasiHunter genome reference runtime
- comparison table with one row per tool per reference plus the multithreaded entry

## How to apply

Obtain the PhasiHunter demo dataset (test_osa) containing the rice sRNA library SRR5049781.fastq.gz plus oryza_sativa_cdna.fa and oryza_sativa_gdna.fa references, and run phasiHunter preprocess (raw mode with -mi 19 -ma 25 -e 1 -n 1000000) once to produce shared cdna/gdna .map files so Bowtie indexing and mapping are excluded from all timed runs. Run the phasiHunter phase module single-threaded (-j 1) with -pl 21 and -me b against the transcriptome via -cm/-c and against the genome via -gm/-g, timing each; then run unitas, PhaseTank, and PhasiRNAnalyzer single-threaded for 21-nt prediction on both references with the same mapping step excluded. Finally, rerun phasiHunter phase on the genome with -j 4 to measure the multithreaded speedup. Compile runtimes per tool per reference and accept results that approximately reproduce ≈21.41/17.56/35.24/181.93 min (cDNA) and ≈91.03/158.30/315.55/919.32 min (gDNA), with ≈45 min for four-thread genome-based PhasiHunter.

## Related tools

- **PhasiHunter** (Primary multithreaded phasiRNA prediction tool timed on transcriptome and genome references via its phase module (-cm/-c, -gm/-g, -pl 21, -j 1/-j 4)) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bowtie** (Used inside phasiHunter preprocess to build the shared sRNA-to-reference mapping files excluded from benchmark timings)
- **unitas** (Comparison tool run single-threaded for 21-nt phasiRNA prediction on both references)
- **PhaseTank** (Comparison tool run single-threaded for 21-nt phasiRNA prediction on both references)
- **PhasiRNAnalyzer** (Comparison tool run single-threaded for 21-nt phasiRNA prediction on both references)

## Examples

```
phasiHunter phase -gm /home/user/test_osa/SRR5049781_processed_gdna.map -g /home/user/test_osa/oryza_sativa_gdna.fa -fa /home/user/test_osa/SRR5049781_trimmed_format_filter.fa -o phase_o.txt -me b -pl 21 -j 4
```

## Evaluation signals

- Single-threaded transcriptome runtimes approximately match 21.41 (PhasiHunter), 17.56 (unitas), 35.24 (PhaseTank), and 181.93 (PhasiRNAnalyzer) min.
- Single-threaded genome runtimes approximately match 91.03 (PhasiHunter), 158.30 (PhaseTank benchmark order: PhasiHunter fastest), 315.55, and 919.32 min.
- Four-threaded PhasiHunter genome runtime is ≈45 min, showing near 2× speedup over the 91.03 min single-threaded run.
- All tools use the same preprocessed .map inputs so no tool timing includes Bowtie indexing/mapping.
- All predictions use phase length 21 (-pl 21) and identical prediction-method settings (-me b default with hypergeometric + phase score).

## Limitations

- Runtimes depend on hardware; only approximate reproduction of reported values is expected.
- The benchmark excludes the shared Bowtie index/mapping step, so end-to-end pipeline wall-clock time will be longer.
- PhasiHunter is a Linux CLI tool requiring external dependencies (Bowtie, Biopython, Bedtools, Dnapi, Trim_galore, Seqkit, Perl5, Fasta36, TarHunter); a Docker image is provided to simplify setup.
- PhasiHunter version details are incomplete — only update log entry 1.0.1 (25/02/2024) is available.

## Evidence

- [results] For genome-based 21-nt phasiRNA prediction, PhasiHunter was fastest at 91.03 min in single-threaded mode, followed by unitas, PhaseTank, and PhasiRNAnalyzer at 158.30, 315.55, and 919.32 min respectively.: [evidence span withheld — anchor status: not_found]
- [methods] Preprocessed map files shared across tools exclude Bowtie indexing/mapping from timed runs.: "raw(mode): trim adaptor --> normalization --> length and abundance filter --> mapping"
- [readme] Phase module accepts transcriptome and genome map/reference pairs and a parallel thread count.: "-cm: file  --  map file based on reference transcriptome sequence"
- [readme] Phase module parallelism parameter default and semantics.: "-j:  int   --  parallel number, default=1"
- [abstract] PhasiHunter's multithreaded design motivates the runtime comparison.: "the excellent data structure and parallel computing architecture allow PhasiHunter to predict phasiRNAs and their regulatory pathways with high efficiency"
