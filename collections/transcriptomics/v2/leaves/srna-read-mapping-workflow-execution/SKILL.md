---
name: srna-read-mapping-workflow-execution
description: Use when running the PhasiHunter preprocess module to produce .map files
  from sRNA FASTQ data against transcriptome/genome references, or when benchmarking
  phasiRNA prediction tools with the Bowtie index/mapping step held constant.
when_to_use_negative:
- When the analysis goal is degradome-based target or initiator prediction (phasihunter
  deg / target modules) rather than sRNA mapping and runtime comparison
- When the raw FASTQ has already been trimmed, normalized, and filtered — use preprocess
  mode m (mapping only) or skip to the phase module instead of rerunning raw mode
- When per-tool end-to-end wall-clock times including index building are desired;
  this skill deliberately excludes the shared Bowtie indexing/mapping step from timing
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: PhasiHunter
  role: Provides the preprocess (raw/clean/mapping modes) and phase submodules used
    to build shared .map artifacts and to time phasiRNA prediction with -j 1 or -j
    4
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bowtie
  role: Short-read aligner invoked by the preprocess module to map filtered sRNA reads
    onto the cDNA and gDNA reference indices
- name: Trim_galore
  role: Adapter trimming dependency used in the raw-mode preprocessing path (trim
    adaptor -> normalization -> filter -> mapping)
  repo: https://github.com/FelixKrueger/TrimGalore
- name: unitas
  role: Comparison tool run single-threaded for 21-nt phasiRNA prediction on both
    references, timed after the shared mapping step
- name: PhaseTank
  role: Comparison tool run single-threaded for 21-nt phasiRNA prediction on both
    references, timed after the shared mapping step
- name: PhasiRNAnalyzer
  role: Comparison tool run single-threaded for 21-nt phasiRNA prediction on both
    references, timed after the shared mapping step
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:38cc475656fcec363680ea1dc5c32c8ad1842cffadce049afa7920378ef06d89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuangLab-CBI/PhasiHunter
  tools:
  - Bowtie
  - PhaseTank
  - PhasiHunter
  - PhasiRNAnalyzer
  - Trim_galore
  - unitas
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# srna-read-mapping-workflow-execution

## Summary

Executes the sRNA-seq read preprocessing/mapping stage of the PhasiHunter pipeline (adapter trimming, normalization, length/abundance filtering, Bowtie mapping) so that shared mapping artifacts (cdna/gdna .map files) can feed downstream phasiRNA prediction and be excluded from runtime benchmarking of PhasiHunter, unitas, PhaseTank, and PhasiRNAnalyzer.

## When to use

Use when running the PhasiHunter preprocess module to produce .map files from sRNA FASTQ data against transcriptome/genome references, or when benchmarking phasiRNA prediction tools with the Bowtie index/mapping step held constant.

## When NOT to use

- When the analysis goal is degradome-based target or initiator prediction (phasihunter deg / target modules) rather than sRNA mapping and runtime comparison
- When the raw FASTQ has already been trimmed, normalized, and filtered — use preprocess mode m (mapping only) or skip to the phase module instead of rerunning raw mode
- When per-tool end-to-end wall-clock times including index building are desired; this skill deliberately excludes the shared Bowtie indexing/mapping step from timing

## Inputs

- sRNA-seq FASTQ or FASTQ.gz file (e.g. SRR5049781.fastq.gz)
- Reference transcriptome FASTA (oryza_sativa_cdna.fa) and genome FASTA (oryza_sativa_gdna.fa)
- Prebuilt Bowtie index prefixes (e.g. oryza_sativa_cdna_index, oryza_sativa_gdna_index)
- Length- and abundance-filtered sRNA FASTA for mapping-only mode (m)

## Outputs

- sRNA mapping file against transcriptome reference (SRR5049781_processed_cdna.map)
- sRNA mapping file against genome reference (SRR5049781_processed_gdna.map)
- Per-tool elapsed runtimes (min) for phasiRNA prediction, tabulated per tool per reference type plus four-thread PhasiHunter on the genome

## How to apply

Run `phasiHunter preprocess` with mode r (raw FASTQ: trim adaptor -> normalization -> length and abundance filter -> mapping) using parameters -mi 19 -ma 25 -e 1 -n 1000000 to generate the transcriptome .map file, then mode m (mapping only) with the length/abundance-filtered FASTA to generate the genome .map file via the Bowtie indices (oryza_sativa_cdna_index / oryza_sativa_gdna_index). These preprocessed .map outputs let all tools share one Bowtie indexing/mapping step so timings compare only the prediction phase (e.g. `phasiHunter phase -pl 21 -j 1` or `-j 4`). Rationale: PhasiHunter's preprocess and phase modules are separate CLI submodules, so mapping artifacts can be reused across tools; the benchmark reported PhasiHunter at 91.03 min single-threaded and ~45 min with four threads (-j 4) on the genome reference.

## Related tools

- **PhasiHunter** (Provides the preprocess (raw/clean/mapping modes) and phase submodules used to build shared .map artifacts and to time phasiRNA prediction with -j 1 or -j 4) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bowtie** (Short-read aligner invoked by the preprocess module to map filtered sRNA reads onto the cDNA and gDNA reference indices)
- **Trim_galore** (Adapter trimming dependency used in the raw-mode preprocessing path (trim adaptor -> normalization -> filter -> mapping)) — https://github.com/FelixKrueger/TrimGalore
- **unitas** (Comparison tool run single-threaded for 21-nt phasiRNA prediction on both references, timed after the shared mapping step)
- **PhaseTank** (Comparison tool run single-threaded for 21-nt phasiRNA prediction on both references, timed after the shared mapping step)
- **PhasiRNAnalyzer** (Comparison tool run single-threaded for 21-nt phasiRNA prediction on both references, timed after the shared mapping step)

## Examples

```
phasiHunter preprocess -m r -i /home/user/test_osa/SRR5049781.fastq.gz -mi 19 -ma 25 -e 1 -n 1000000 -o /home/user/test_osa/SRR5049781_processed_cdna.map -in /home/user/test_osa/index/oryza_sativa_cdna_index
```

## Evaluation signals

- The preprocess outputs are two .map files (cdna and gdna) built with identical cutoffs (-mi 19 -ma 25 -e 1 -n 1000000) and reused across all four tools
- Single-threaded genome-reference runtimes approximately match the reported order and values: PhasiHunter ≈ 91.03 min, unitas ≈ 158.30 min, PhaseTank ≈ 315.55 min, PhasiRNAnalyzer ≈ 919.32 min
- Transcriptome-reference runtimes approximate 21.41/17.56/35.24/181.93 min for PhasiHunter/unitas/PhaseTank/PhasiRNAnalyzer
- Four-threaded phasiHunter phase (-j 4) on the genome reference completes in approximately 45 min, i.e. faster than the 91.03 min single-threaded run
- `phasihunter -h` runs without error, confirming the installation and dependency environment (Bowtie, Biopython, Bedtools, Dnapi, Trim_galore, Seqkit, Perl5, Fasta36, TarHunter)

## Limitations

- Runtimes are specific to the rice (Oryza sativa) test_osa demo dataset and single-threaded vs four-thread (-j 4) configurations; absolute times will not transfer to other genomes or hardware
- Timings exclude the Bowtie indexing/mapping step, so the comparison does not reflect total end-to-end pipeline cost per tool
- Only PhasiHunter is explicitly multithreaded in this benchmark; unitas, PhaseTank, and PhasiRNAnalyzer were timed in single-threaded mode only
- No software version number for PhasiHunter is stated in the article (only update log 1.0.1 - 25/02/2024 in the README)

## Evidence

- [readme] raw(mode): trim adaptor --> normalization --> length and abundance filter --> mapping: "raw(mode): trim adaptor --> normalization --> length and abundance filter --> mapping"
- [readme] phase module parallel number parameter: "-j:  int   --  parallel number, default=1"
- [methods] PhasiHunter is a CLI program on Linux: "phasihunter is a CLI program runing on linux platform."
- [results] genome-based runtime benchmark finding: [evidence span withheld — anchor status: not_found]
- [abstract] parallel computing architecture claim: "the excellent data structure and parallel computing architecture allow PhasiHunter to predict phasiRNAs and their regulatory pathways with high efficiency"
