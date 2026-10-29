---
name: cnn-embedding-layer-design
description: Use when building or re-instantiating a transformer basecaller that must
  project STFT spectrogram-like inputs of ONT dRNA current chunks into per-frame embeddings
  before attention encoder modules.
when_to_use_negative:
- Input signals are already basecalled — modification-unaware basecallers (e.g. Dorado)
  followed by re-squiggling and post-hoc modification models is the alternative pipeline
  this CNN embedding approach replaces, not complements
- The target is canonical-only (A, C, G, U) basecalling — the expanded-vocabulary
  decoder with N modification tokens adds unnecessary complexity
- You are using a deprecated single-modification model — the README notes the inosine-only
  version will be discontinued in favour of the multi-mod version
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0092
tools:
- name: NanoSpeech
  role: Basecaller whose transformer architecture (CNN embedding front end, attention
    encoders, expanded-vocabulary decoder) this skill reconstructs; models are launched
    from the basecaller CLI
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: Keras / TensorFlow
  role: Framework in which the three ReLU CNN embedding layers, multi-head attention
    encoders, and categorical cross-entropy compilation are implemented, mirroring
    the keras.io transformer ASR example
  repo: https://github.com/keras-team/keras-io
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:6614310f7f90272531f2e7b5a34093beaf2c0aa3ad8990cea13bfc4afd44715b
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
  - Keras / TensorFlow
  - NanoSpeech
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cnn-embedding-layer-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Design the CNN front end of the NanoSpeech transformer basecaller: three Keras/TensorFlow CNN embedding layers with ReLU that convert STFT spectrograms of padded picoampere-scale ionic-current chunks into per-frame embeddings for downstream multi-head attention encoding over an expanded basecalling vocabulary.

## When to use

Use when building or re-instantiating a transformer basecaller that must project STFT spectrogram-like inputs of ONT dRNA current chunks into per-frame embeddings before attention encoder modules.

## When NOT to use

- Input signals are already basecalled — modification-unaware basecallers (e.g. Dorado) followed by re-squiggling and post-hoc modification models is the alternative pipeline this CNN embedding approach replaces, not complements
- The target is canonical-only (A, C, G, U) basecalling — the expanded-vocabulary decoder with N modification tokens adds unnecessary complexity
- You are using a deprecated single-modification model — the README notes the inosine-only version will be discontinued in favour of the multi-mod version

## Inputs

- Padded picoampere-scale raw current chunks from ONT dRNA fast5/pod5 files (default 3000-sample chunks)
- Model configuration file (models/*.h5.cfg) accompanying each h5 weights file
- Pretrained transformer weights (.h5) to initialize the architecture

## Outputs

- Per-frame embeddings from three CNN layers with ReLU, suitable as input to multi-head attention encoders
- nano_speech_model.py defining the full encoder-decoder architecture parameterized by the .h5.cfg settings
- Decoder output layer over vocabulary {A, C, G, U, N1..Nn, <, >, -} for kmer basecalling

## How to apply

Convert padded pA-scale raw current chunks (default chunk length 3000, outliers clipped to a 20-185 min-max current range with the mean) into time-frequency inputs via STFT, following the keras.io transformer ASR example that NanoSpeech adapts. Project the spectrogram through exactly three CNN embedding layers with ReLU activations to obtain per-frame embeddings. Read the transformer initialization parameters (hidden sizes, attention head counts, layer counts) from the .h5.cfg file accompanying each weights file in ./models, e.g. NanoSpeech_Inosine_m23M_e71_d19Mrec_SINGLE_MOD_VER.h5. Feed the embeddings to multi-head attention encoder modules with dense projections and dropout regularization, then attach the decoder emitting kmers over the expanded vocabulary {A, C, G, U, N1..Nn, <, >, -}. Compile with categorical cross-entropy and Adam under a custom learning-rate schedule, and save the full definition as nano_speech_model.py so the architecture can be re-instantiated from the .h5.cfg parameters.

## Related tools

- **NanoSpeech** (Basecaller whose transformer architecture (CNN embedding front end, attention encoders, expanded-vocabulary decoder) this skill reconstructs; models are launched from the basecaller CLI) — https://github.com/F0nz0/NanoSpeech_basecaller
- **Keras / TensorFlow** (Framework in which the three ReLU CNN embedding layers, multi-head attention encoders, and categorical cross-entropy compilation are implemented, mirroring the keras.io transformer ASR example) — https://github.com/keras-team/keras-io

## Examples

```
python3 NanoSpeech.py -d {fast5_folder_path} -o {fasta/fastq_output} -m {model_weights in ./models/} -t {number_of_threads} -c 20-185 -cl 3000
```

## Evaluation signals

- nano_speech_model.py executes without error and instantiates the full architecture purely from the .h5.cfg parameters
- The output-layer vocabulary contains both canonical bases (A, C, G, U) and modification tokens (N1..Nn) plus the <, >, - special symbols
- STFT spectrogram inputs pass through exactly three CNN layers with ReLU and emerge as per-frame embeddings matching the attention encoder input dimensionality
- Model compiles under TensorFlow 2.7 with categorical cross-entropy loss and Adam optimizer with the custom learning-rate schedule
- Basecalling output is valid FASTA/FASTQ with modification-position indices in read headers (e.g. inosine candidate adenosine indexes for the inosine model)

## Limitations

- Tested on CentOS-7 with an NVIDIA A100-PCIE-40GB GPU, CUDA 11.8, cuDNN 8.6 and TensorFlow v2.7.0; other stacks are untested
- At least 3 GB of GPU memory is needed per parallel basecalling thread
- Primarily validated on synthetic constructs as a proof of concept; NanoSpeech models for highly similar chemical structures are still under development
- Adequate training datasets for transformer-based ONT dRNA basecallers were missing, and public-data quality, missing benchmarks, and unavailable training sets limit reproducibility

## Evidence

- [abstract] a modification-aware basecaller for the ab initio simultaneous detection of multiple modified bases using a transformer model: "a modification-aware basecaller for the ab initio simultaneous detection of multiple modified bases using a transformer model"
- [readme] NanoSpeech is based on a Transformer model with expanded dictionaries: "NanoSpeech is based on a Transformer model with expanded dictionaries with respect to canonical-nucleotides only (A,C,G,U,I=Inosine or other additional modifications)"
