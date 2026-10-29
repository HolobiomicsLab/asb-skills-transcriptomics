---
name: nanopore-drna-basecalling
description: Use when raw ONT dRNA fast5/pod5 signals from SQK-RNA002 or SQK-RNA004
  chemistries must be basecalled while simultaneously detecting modified bases (I,
  m6A, m5C, Psi) without re-squiggling or separate modification models.
when_to_use_negative:
- The goal is modified-base detection from already-basecalled reads via re-squiggling
  and separate modification models — NanoSpeech avoids this two-step pre-processing
  by design
- Chemistry- or kit-specific ONT models from Dorado/Guppy are required for comparison
  benchmarks where the state-of-the-art vendor baseline is the reference output
- Data are non-ONT (e.g. Illumina, PacBio) or non-direct-RNA (cDNA sequencing) reads,
  since the transformer is trained on raw dRNA ionic current chunks
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: NanoSpeech
  role: Modification-aware transformer basecaller with expanded dictionary (A,C,G,U
    + I or other modified bases) that converts raw dRNA fast5/pod5 signals into fasta/fastq
    in a single step
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: minimap2
  role: Maps basecalled fasta/fastq reads to the reference transcriptome using the
    map-ont preset (-ax map-ont, --secondary=no, --MD)
- name: samtools
  role: Filters the minimap2 SAM (view -b -F 2304), sorts to BAM, and indexes alignments
    before modification detection
- name: Dorado
  role: State-of-the-art ONT modification-aware basecaller used as a benchmark baseline
    against NanoSpeech basecalling accuracy and modification detection
  repo: https://github.com/nanoporetech/dorado
- name: ModiKit
  role: Post-processing tool used with Dorado to extract modification calls; NanoSpeech
    avoids this additional post-processing step
- name: m6ABasecaller
  role: Prior m6A-aware RNA basecaller predicting 5 nucleosides (A,C,G,U,m6A) from
    fast5 files, cited as a comparable modification-aware basecalling approach
  repo: https://github.com/nanoalab/m6ABasecaller
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:9e041053ec8bb045a5b463c8ad445b5f4dda94aea16cf91511aca78df5702ebc
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
  - m6ABasecaller
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# nanopore-dRNA-basecalling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Basecall Oxford Nanopore direct RNA raw signal data (fast5/pod5) with the modification-aware transformer basecaller NanoSpeech, which emits canonical bases plus modified nucleotides (e.g. inosine, m6A, m5C, Psi) in a single step, then map and post-process the calls for per-read and genome-space modification detection.

## When to use

Use when raw ONT dRNA fast5/pod5 signals from SQK-RNA002 or SQK-RNA004 chemistries must be basecalled while simultaneously detecting modified bases (I, m6A, m5C, Psi) without re-squiggling or separate modification models.

## When NOT to use

- The goal is modified-base detection from already-basecalled reads via re-squiggling and separate modification models — NanoSpeech avoids this two-step pre-processing by design
- Chemistry- or kit-specific ONT models from Dorado/Guppy are required for comparison benchmarks where the state-of-the-art vendor baseline is the reference output
- Data are non-ONT (e.g. Illumina, PacBio) or non-direct-RNA (cDNA sequencing) reads, since the transformer is trained on raw dRNA ionic current chunks

## Inputs

- ONT direct RNA raw signal data as fast5 files (SQK-RNA002 chemistry) or pod5/fast5 files (SQK-RNA004 chemistry)
- NanoSpeech model weights in .h5 format with paired .h5.cfg configuration file (e.g. NanoSpeech_Inosine_m23M_e71_d19Mrec.h5 for inosine, RNA004 multi-mod weights)
- Reference transcriptome or synthetic construct (e.g. E. coli AmpD IVT) sequence in fasta format

## Outputs

- Basecalled fasta/fastq file with modification indexes embedded in read headers
- Filtered, sorted, indexed BAM alignment produced via minimap2 and samtools
- Per-read modification BED file ({bam}.per_read.bed, 7 columns) and genome-space aggregated modification table ({bam}.genome_space.tsv, 6 columns: region, start, strand, I_count, depth, I_freq)

## How to apply

Run NanoSpeech_multi_mod.py on the pod5/fast5 folder with the RNA004 multi-modification model weights (-m; the paired *.h5.cfg is auto-loaded), number of parallel GPU models (-t, ~3 GB GPU memory per thread, e.g. ~15 threads on an A100-40GB), and output fasta/fastq path (-o). For RNA002 inosine-only work, use the multi-species model NanoSpeech_Inosine_m23M_e71_d19Mrec.h5 with its coupled .h5.cfg; basecalled read headers carry indexes of adenosines with higher probability of being inosines. Align the basecalled fasta/fastq to the reference transcriptome with minimap2 (map-ont preset, -k 14, --secondary=no, --MD), then filter/convert with samtools view -b -F 2304 | samtools sort -O BAM and samtools index, removing the intermediate SAM. Finally run modification_detector.py / inosine_detector.py with the BAM (-b), basecalled fasta/q (-f) and a minimum modification/inosine quality threshold (-q) to produce per-read BED ({bam}.per_read.bed, 7 columns) and genome-space aggregated tables ({bam}.genome_space.tsv, 6 columns: region, start, strand, I_count, depth, I_freq).

## Related tools

- **NanoSpeech** (Modification-aware transformer basecaller with expanded dictionary (A,C,G,U + I or other modified bases) that converts raw dRNA fast5/pod5 signals into fasta/fastq in a single step) — https://github.com/F0nz0/NanoSpeech_basecaller
- **minimap2** (Maps basecalled fasta/fastq reads to the reference transcriptome using the map-ont preset (-ax map-ont, --secondary=no, --MD))
- **samtools** (Filters the minimap2 SAM (view -b -F 2304), sorts to BAM, and indexes alignments before modification detection)
- **Dorado** (State-of-the-art ONT modification-aware basecaller used as a benchmark baseline against NanoSpeech basecalling accuracy and modification detection) — https://github.com/nanoporetech/dorado
- **ModiKit** (Post-processing tool used with Dorado to extract modification calls; NanoSpeech avoids this additional post-processing step)
- **m6ABasecaller** (Prior m6A-aware RNA basecaller predicting 5 nucleosides (A,C,G,U,m6A) from fast5 files, cited as a comparable modification-aware basecalling approach) — https://github.com/nanoalab/m6ABasecaller

## Examples

```
python3 NanoSpeech_multi_mod.py -d {pod5/fast5 folderpath} -m {RNA004 multi-mod weights}.h5 -t 15 -o {output fasta/fastq}
```

## Evaluation signals

- Computed per-read identity, consensus accuracy and insertion rates are comparable to (or exceed) the deposited Dorado(+ModKit) outputs; NanoSpeech outperforms Dorado on fully modified m6A/m5C/Psi constructs while remaining comparable on short/partially modified oligos (Fig. 3A–K)
- On the I-rich coli-IVT (E. coli AmpD) sample, inosine accuracy reaches ~96% with F1-score ~95% and only a few false negatives
- More than 99% of basecalled reads align correctly to the reference and NanoSpeech mappability exceeds the Guppy baseline
- SAM filtering (-F 2304 removes supplementary alignments) yields sorted indexed BAMs that inosine_detector.py consumes without errors; outputs contain the 7-column per-read BED and 6-column genome-space TSV
- Detected inosines localize to the expected reference positions of the synthetic construct rather than appearing randomly across the transcript

## Limitations

- Primarily tested on synthetic constructs as a proof of concept; real-world performance on complex epitranscriptomes is less validated
- Inference artefacts or 'hallucinations' can arise from highly similar chemical structures or intrinsic errors in modified samples
- Streamlined NanoSpeech models for detecting highly similar chemical structures are still under development; the single-modification (inosine-only) version will be deprecated in favor of the multi-mod version
- The often questionable quality of public data for RNA modifications and the lack of reliable benchmarks impair accurate modification detection and unbiased comparisons
- Sample-level factors (number of reads, quality, contaminations, unknown IVT conditions) hamper training set generation

## Evidence
