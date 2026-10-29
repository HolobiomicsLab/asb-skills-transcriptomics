---
name: transformer-architecture-implementation
description: Use when reconstructing a modification-aware ONT direct-RNA basecaller
  from archived .h5 weights and their .h5.cfg files, or when adapting keras.io speech-to-text
  transformer code to raw ionic current signals.
when_to_use_negative:
- When the goal is running inference with an already-trained basecaller (e.g. the
  NanoSpeech.py CLI) rather than rebuilding the model definition from .h5.cfg parameters
- When input is already basecalled FASTQ/FASTA or aligned BAM data — this skill operates
  on raw ionic current chunks and model-definition files
- When the target is DNA sequencing with canonical-vocabulary basecalling (e.g. Dorado
  duplex/modified basecalling workflows) rather than an expanded RNA modification
  vocabulary
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: NanoSpeech
  role: Basecaller whose transformer model definition (STFT input, CNN embedding,
    multi-head attention encoder, expanded-vocabulary kmer decoder) is reconstructed
    by this skill
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: TensorFlow
  role: Framework used to build and compile the transformer with categorical cross-entropy
    and Adam; tested at v2.7.0 with CUDA 11.8 and cuDNN 8.6
- name: Keras
  role: 'Reference implementation source: the keras.io transformer ASR example that
    NanoSpeech''s architecture mirrors'
  repo: https://github.com/keras-team/keras-io
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:c6960a2199d68c9c608e8a8bf0e71fbc3d869599a50ff1b9743cc5254c34452c
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
  - Keras
  - NanoSpeech
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# transformer-architecture-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implement the NanoSpeech encoder-decoder transformer basecaller that converts STFT spectrograms of padded picoampere-scale ionic current chunks into base-called kmers over an expanded vocabulary that includes modified-base tokens (A, C, G, U, N1..Nn, <, >, -).

## When to use

Use when reconstructing a modification-aware ONT direct-RNA basecaller from archived .h5 weights and their .h5.cfg files, or when adapting keras.io speech-to-text transformer code to raw ionic current signals.

## When NOT to use

- When the goal is running inference with an already-trained basecaller (e.g. the NanoSpeech.py CLI) rather than rebuilding the model definition from .h5.cfg parameters
- When input is already basecalled FASTQ/FASTA or aligned BAM data — this skill operates on raw ionic current chunks and model-definition files
- When the target is DNA sequencing with canonical-vocabulary basecalling (e.g. Dorado duplex/modified basecalling workflows) rather than an expanded RNA modification vocabulary

## Inputs

- Transformer initialization parameters from NanoSpeech .h5.cfg configuration files accompanying .h5 weights files
- Padded picoampere-scale raw ionic current chunks from ONT dRNA fast5/pod5 reads (default 3000-sample chunks, clip range e.g. 20-185)

## Outputs

- nano_speech_model.py script that instantiates the full encoder-decoder transformer architecture from the .h5.cfg parameters
- A compiled Keras/TensorFlow model with a decoder output layer over the expanded vocabulary {A, C, G, U, N1..Nn, <, >, -}

## How to apply

Read the transformer initialization parameters from the model configuration file (models/*.h5.cfg) that accompanies each h5 weights file, and instantiate the architecture in a saved nano_speech_model.py. Convert padded pA-scale current chunks (default chunk length 3000, outliers clipped to a min-max range such as 20-185 by replacing with the mean) into time-frequency spectrograms via STFT, following the keras.io transformer ASR example the library adapts. Project the spectrogram inputs through three CNN embedding layers with ReLU, then stack multi-head attention encoder modules, each with dense projection and dropout/normalization regularization sublayers, and attach a decoder that emits kmers over the expanded vocabulary {A, C, G, U, N1..Nn, <, >, -}, where N tokens represent modified bases such as inosine. Compile the model with categorical cross-entropy loss, the Adam optimizer, and a custom learning-rate schedule under TensorFlow (v2.7.0 in the tested environment).

## Related tools

- **NanoSpeech** (Basecaller whose transformer model definition (STFT input, CNN embedding, multi-head attention encoder, expanded-vocabulary kmer decoder) is reconstructed by this skill) — https://github.com/F0nz0/NanoSpeech_basecaller
- **TensorFlow** (Framework used to build and compile the transformer with categorical cross-entropy and Adam; tested at v2.7.0 with CUDA 11.8 and cuDNN 8.6)
- **Keras** (Reference implementation source: the keras.io transformer ASR example that NanoSpeech's architecture mirrors) — https://github.com/keras-team/keras-io

## Examples

```
git clone https://github.com/F0nz0/NanoSpeech_basecaller.git && conda create --name NanoSpeech python=3.8 && conda install cudatoolkit=11.8 cudnn=8.3.2 -c nvidia -y
```

## Evaluation signals

- The saved nano_speech_model.py executes without error and instantiates the full architecture from the .h5.cfg parameters
- The output decoder layer vocabulary contains both canonical bases (A, C, G, U) and modification tokens (N1..Nn, <, >, -)
- The three CNN embedding layers and stacked multi-head attention encoder modules match the layer counts/dims specified in the .h5.cfg configuration
- The model compiles with categorical cross-entropy loss and the Adam optimizer under TensorFlow 2.7.0 without raising shape errors when fed STFT spectrograms of 3000-sample chunks

## Limitations

- Streamlined NanoSpeech models enabling the detection of highly similar chemical structures are still under development
- The single-modification (Inosine-only) version of NanoSpeech.py is deprecated in favor of the multi-mod version
- The architecture depends on adequate training datasets, which the paper notes are currently missing for ONT dRNA

## Evidence

- [other] NanoSpeech converts padded pA-scale current chunks into spectrograms via STFT, embeds them through three CNN layers with ReLU, feeds multi-head attention encoders plus dense DNN modules (dropout/normalisation), and decodes kmers over the vocabulary: [evidence span withheld — anchor status: not_found]
- [other] Read the transformer initialization parameters from the NanoSpeech model configuration file (models/*.h5.cfg) that accompanies each h5 weights file: [evidence span withheld — anchor status: not_found]
