---
name: basecaller-benchmark-evaluation
description: Use when evaluating whether a modification-aware basecaller (e.g. NanoSpeech
  inosine-only or multi-mod models) detects modified bases as accurately as a baseline
  such as Guppy or Dorado+ModiKit on synthetic or IVT ONT dRNA datasets.
when_to_use_negative:
- Input is already basecalled fasta/fastq without raw signal — modification-aware
  basecalling from signal cannot be applied
- The goal is modification detection via re-squiggling with modification-unaware basecallers
  plus ad hoc models rather than single-step ab initio basecalling
- Data come from chemistries/kits outside those validated for the available NanoSpeech
  models
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: NanoSpeech
  role: Modification-aware transformer basecaller producing fasta/fastq with per-read
    modification indexes for benchmarking
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: minimap2
  role: Aligns basecalled reads to the reference (map-ont preset, --secondary=no --MD)
    to support mappability and alignment-based metrics
- name: samtools
  role: Filters (-F 2304), sorts and indexes BAM alignments prior to modification
    detection
- name: Dorado
  role: ONT baseline basecaller whose outputs (with ModiKit post-processing) are compared
    against NanoSpeech
  repo: https://github.com/nanoporetech/dorado
- name: ModiKit
  role: Post-processing of Dorado modification calls for stoichiometry comparison
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:3eb87fd422e98854559f1fe1e10817df7504786a9aad9923092325dd7355b543
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/F0nz0/NanoSpeech_basecaller
  repo_url: https://github.com/F0nz0/NanoSpeech_basecaller
  tools:
  - Dorado
  - ModiKit
  - NanoSpeech
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# basecaller-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark modification-aware ONT basecallers (NanoSpeech vs Guppy/Dorado+ModiKit) by basecalling raw dRNA fast5/pod5 reads, aligning with minimap2, extracting per-read and genome-space modification calls, and comparing accuracy, F1, mappability, identity and modification stoichiometry against reported figures.

## When to use

Use when evaluating whether a modification-aware basecaller (e.g. NanoSpeech inosine-only or multi-mod models) detects modified bases as accurately as a baseline such as Guppy or Dorado+ModiKit on synthetic or IVT ONT dRNA datasets.

## When NOT to use

- Input is already basecalled fasta/fastq without raw signal — modification-aware basecalling from signal cannot be applied
- The goal is modification detection via re-squiggling with modification-unaware basecallers plus ad hoc models rather than single-step ab initio basecalling
- Data come from chemistries/kits outside those validated for the available NanoSpeech models

## Inputs

- Raw ONT dRNA fast5/pod5 read folders (SQK-RNA002 or SQK-RNA004 chemistry)
- NanoSpeech model weights (.h5) with coupled .h5.cfg configuration file
- Reference transcriptome or synthetic construct sequence (e.g. E. coli AmpD coli-IVT construct)
- Deposited baseline basecaller outputs (Dorado + ModiKit, or Guppy) for comparison

## Outputs

- Basecalled fasta/fastq with modification indexes in read headers
- Filtered, sorted, indexed BAM alignments
- Per-read BED of modification calls ({bam}.per_read.bed, 7 columns)
- Genome-space aggregated modification table ({bam}.genome_space.tsv, 6 columns: region, start, strand, I_count, depth, I_freq)
- Summary table of benchmark metrics (accuracy, F1, mappability, identity, consensus accuracy, insertion rate, stoichiometry)

## How to apply

Basecall raw fast5/pod5 reads with NanoSpeech (NanoSpeech.py or NanoSpeech_multi_mod.py, choosing the RNA002 multi-species inosine-only model NanoSpeech_Inosine_m23M_e71_d19Mrec.h5 with its .h5.cfg, or the RNA004 multi-modification weights; allocate ~3 GB GPU memory per parallel thread, e.g. 15 threads on an A100-40GB). Align basecalled fasta/fastq to the reference with minimap2 using appropriate presets (map-ont, -k 5 for the AmpD coli-IVT construct; -k 14 --secondary=no --MD for RNA004), filter with samtools view -b -F 2304, sort, index to BAM, then run inosine_detector.py or modification_detector.py with -b BAM, -f fasta/q and a minimum modification quality threshold (-q). Compute inosine accuracy, F1, correctly-aligned read fraction, mappability (coli-IVT), and per-read identity, consensus accuracy, insertion rate and modification stoichiometry (RNA004), comparing against deposited Dorado(+Modkit) outputs. Accept the benchmark when metrics reproduce the reported relationships, e.g. ~96% inosine accuracy and ~95% F1 on coli-IVT with >99% correctly aligned reads and NanoSpeech mappability exceeding Guppy.

## Related tools

- **NanoSpeech** (Modification-aware transformer basecaller producing fasta/fastq with per-read modification indexes for benchmarking) — https://github.com/F0nz0/NanoSpeech_basecaller
- **minimap2** (Aligns basecalled reads to the reference (map-ont preset, --secondary=no --MD) to support mappability and alignment-based metrics)
- **samtools** (Filters (-F 2304), sorts and indexes BAM alignments prior to modification detection)
- **Dorado** (ONT baseline basecaller whose outputs (with ModiKit post-processing) are compared against NanoSpeech) — https://github.com/nanoporetech/dorado
- **ModiKit** (Post-processing of Dorado modification calls for stoichiometry comparison)

## Examples

```
python3 NanoSpeech_multi_mod.py -d {raw_data_folderpath} -o {out.fasta} -m models/NanoSpeech_Inosine_m23M_e71_d19Mrec.h5 -t 15 && minimap2 -ax map-ont -k 5 --secondary=no --MD ref.fa out.fasta > out.sam
```

## Evaluation signals

- On coli-IVT (AmpD) sample, inosine accuracy reaches ~96% and F1 ~95% at expected reference positions with only a few false negatives
- More than 99% of basecalled reads align correctly to the AmpD construct reference
- NanoSpeech mappability exceeds the Guppy baseline on the coli-IVT construct (Fig. 1B-C, Supp. Figs. S5-S7)
- Per-read identity, consensus accuracy and insertion rates are comparable to state-of-the-art ONT software (Dorado) on short and partially modified RNA004 oligos
- Genome-space tables have the expected 6-column schema and per-read BEDs the expected 7-column schema

## Limitations

- Primarily tested on synthetic constructs as a proof of concept; real-world performance depends on read number, quality, contaminations and unknown IVT conditions
- Streamlined NanoSpeech models for highly similar chemical structures are still under development, risking inference artefacts or 'hallucinations'
- Lack of reliable benchmarks and unavailable training datasets of competing tools limit unbiased reproducible comparisons

## Evidence

- [abstract] NanoSpeech, a modification-aware basecaller for the ab initio simultaneous detection of multiple modified bases using a transformer model: "Here, we introduce NanoSpeech, a modification-aware basecaller for the ab initio simultaneous detection of multiple modified bases using a transformer model"
- [readme] README command for inosine-aware basecalling: "python3 NanoSpeech.py -d {fast5_folder_path} -o {fasta/fastq output file} -m {model_weigths in ./models/} -t {number of threads}"
- [methods] Filtering and BAM conversion step: "samtools view -b -F 2304 {out_filepath}.sam | samtools sort -O BAM"
- [readme] GPU memory guidance for parallel basecalling: "It's important to consider at least 3 GB of GPU memory for each parallel thread. For an NVIDIA-A100 40GPU 15 threads should be a good amount of parallel models."
- [discussion] Discussion on benchmark reliability: "the lack of reliable benchmarks for training and validation, both of which impair accurate modification detection in real-world data"
