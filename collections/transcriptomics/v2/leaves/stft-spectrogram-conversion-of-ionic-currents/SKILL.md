---
name: stft-spectrogram-conversion-of-ionic-currents
description: Use when raw nanopore dRNA signals (fast5/pod5 chunks, e.g. 3000-sample
  chunks, pA scale) must be preprocessed into spectrogram-like inputs for a speech-to-text-style
  transformer that calls canonical and modified bases.
when_to_use_negative:
- Data are already basecalled reads (fasta/fastq/BAM) rather than raw current signals
  — re-squiggling or modification-detection post-processing (e.g. ModiKit) is the
  appropriate path instead
- Working with latest ONT chemistry reads for which ONT's Dorado modification-aware
  basecaller is available and supported
- Signal chunks are truncated or unpadded in a way inconsistent with the chunk length
  the model was trained on
edam_operation: null
edam_topics: []
tools:
- name: NanoSpeech
  role: Basecaller whose pipeline performs the STFT conversion, CNN embedding, and
    transformer decoding of raw dRNA currents
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: TensorFlow
  role: Framework used to implement the STFT/CNN/transformer architecture and compile
    it with categorical cross-entropy loss and the Adam optimizer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:de028d820a573201bbff8740130a71d213f429d556d1c1f9f2cf8146a59aba99
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
  - NanoSpeech
  - TensorFlow
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# stft-spectrogram-conversion-of-ionic-currents

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts padded picoampere-scale ionic current chunks from ONT direct RNA reads into time-frequency spectrograms via STFT so a transformer basecaller (NanoSpeech) can consume them like audio features for kmer basecalling over an expanded nucleotide vocabulary.

## When to use

Use when raw nanopore dRNA signals (fast5/pod5 chunks, e.g. 3000-sample chunks, pA scale) must be preprocessed into spectrogram-like inputs for a speech-to-text-style transformer that calls canonical and modified bases.

## When NOT to use

- Data are already basecalled reads (fasta/fastq/BAM) rather than raw current signals — re-squiggling or modification-detection post-processing (e.g. ModiKit) is the appropriate path instead
- Working with latest ONT chemistry reads for which ONT's Dorado modification-aware basecaller is available and supported
- Signal chunks are truncated or unpadded in a way inconsistent with the chunk length the model was trained on

## Inputs

- Raw ionic current chunks from ONT dRNA fast5/pod5 reads (padded pA-scale signal, default 3000-sample chunks)
- Pretrained NanoSpeech model weights (.h5) with matching architecture configuration (.h5.cfg)

## Outputs

- STFT time-frequency spectrogram representations per current chunk
- CNN-embedded per-frame feature sequences feeding the transformer encoder
- Basecalled kmer sequences over vocabulary {A, C, G, U, N1..Nn, <, >, -} emitted as fasta/fastq with modification candidate indexes in read headers

## How to apply

Chunk the raw current signal (default chunks_len=3000 in NanoSpeech) and pad to a fixed length, then compute a short-time Fourier transform to obtain a time-frequency spectrogram, mirroring the keras.io transformer ASR example the library adapts. Before transformation, optionally clip outlying current values to the mean within a min-max window (default '20-185' pA) to suppress spikes. Project the resulting spectrogram through three CNN embedding layers with ReLU to obtain per-frame embeddings, which are then fed to multi-head attention encoder blocks with dense/regularization sublayers. The decoder emits kmers over the expanded vocabulary {A, C, G, U, N1..Nn, <, >, -}, trained with categorical cross-entropy under Adam with a custom learning-rate schedule. Model architecture parameters should be read from the .h5.cfg file accompanying each .h5 weights file rather than hard-coded.

## Related tools

- **NanoSpeech** (Basecaller whose pipeline performs the STFT conversion, CNN embedding, and transformer decoding of raw dRNA currents) — https://github.com/F0nz0/NanoSpeech_basecaller
- **TensorFlow** (Framework used to implement the STFT/CNN/transformer architecture and compile it with categorical cross-entropy loss and the Adam optimizer)

## Examples

```
python3 NanoSpeech.py -d {fast5_folder_path} -o {out.fastq} -m models/NanoSpeech_Inosine_m23M_e71_d19Mrec_SINGLE_MOD_VER.h5 -c 20-185 -t 15
```

## Evaluation signals

- The saved nano_speech_model.py script executes without error and instantiates the full architecture from the .h5.cfg parameters
- The output layer vocabulary contains both canonical bases (A, C, G, U) and modification tokens (N1..Nn)
- Basecalled output is a fasta/fastq file where reads carry indexes of adenosines with high modification probability in the header
- Per-read quality and modification calls agree with minimap2-aligned reference positions at a rate consistent with the reported accuracy comparable to state-of-the-art ONT basecallers

## Limitations

- Primarily tested on synthetic constructs as a proof of concept; robustness on diverse real-world dRNA data is less established
- Adequate training datasets for ONT dRNA transformers were missing, and questionable quality of public RNA modification data and lack of reliable benchmarks can lead to inference artefacts or 'hallucinations' for highly similar chemical structures
- At least 3 GB of GPU memory per parallel basecalling thread is required (e.g. ~15 threads on an NVIDIA A100 40GB)

## Evidence

- [other] NanoSpeech converts padded pA-scale current chunks into spectrograms via STFT, embeds them through three CNN layers with ReLU: [evidence span withheld — anchor status: not_found]
