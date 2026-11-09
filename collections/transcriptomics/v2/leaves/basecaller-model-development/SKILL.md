---
name: basecaller-model-development
description: Use when pretrained NanoSpeech model weights (.h5 with .h5.cfg files)
  must be turned into an executable Keras/TF architecture, or when developing a transformer
  basecaller that maps raw ONT dRNA signals to kmers over an expanded vocabulary.
when_to_use_negative:
- The goal is modification detection after standard basecalling via re-squiggling
  with a separate model (e.g. Dorado + modification-specific models) rather than single-step
  modification-aware basecalling
- Only canonical basecalling (A,C,G,U) is needed and no expanded vocabulary with modified-base
  tokens (N tokens, e.g. inosine) is required
- Working with DNA sequencing signals or chemistries other than ONT direct RNA (SQK-RNA002
  / SQK-RNA004) that the models were trained on
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: NanoSpeech
  role: Basecaller whose transformer architecture (STFT input, CNN embeddings, multi-head
    attention encoders, expanded-vocabulary decoder) is reconstructed and instantiated
    from .h5.cfg parameters
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: TensorFlow
  role: Framework used to define, compile (categorical cross-entropy + Adam optimizer
    with custom LR schedule) and run the transformer model
- name: Keras
  role: keras.io transformer ASR example is the reference code adapted for the encoder-decoder
    architecture
  repo: https://github.com/keras-team/keras-io
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:e56a2495a912e7f80e49b7bcbd8c3779c053fad3c69ea0a78014ecc4d1b4937a
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# basecaller-model-development

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs a modification-aware transformer basecaller (NanoSpeech-style) in TensorFlow/Keras by instantiating the architecture from model configuration parameters (.h5.cfg) that accompany pretrained h5 weight files, so raw ionic current chunks can be basecalled over an expanded nucleotide vocabulary including modified-base tokens.

## When to use

Use when pretrained NanoSpeech model weights (.h5 with .h5.cfg files) must be turned into an executable Keras/TF architecture, or when developing a transformer basecaller that maps raw ONT dRNA signals to kmers over an expanded vocabulary.

## When NOT to use

- The goal is modification detection after standard basecalling via re-squiggling with a separate model (e.g. Dorado + modification-specific models) rather than single-step modification-aware basecalling
- Only canonical basecalling (A,C,G,U) is needed and no expanded vocabulary with modified-base tokens (N tokens, e.g. inosine) is required
- Working with DNA sequencing signals or chemistries other than ONT direct RNA (SQK-RNA002 / SQK-RNA004) that the models were trained on

## Inputs

- NanoSpeech model configuration file (models/*.h5.cfg) accompanying each pretrained h5 weights file
- Pretrained transformer weights (.h5)
- Padded picoampere-scale raw ionic current chunks (default chunk length 3000) from fast5/pod5 dRNA reads

## Outputs

- Executable model definition script (nano_speech_model.py) instantiating the full transformer architecture
- Basecalled FASTA/FASTQ reads with kmers called over the expanded vocabulary {A,C,G,U,N1..Nn,<,>,-}

## How to apply

Read the transformer initialization parameters from the model configuration file (models/*.h5.cfg) shipped alongside each h5 weights file, then rebuild the architecture in nano_speech_model.py: convert padded picoampere-scale raw current chunks into time-frequency inputs via STFT (adapting the keras.io transformer ASR example), project the spectrogram through three CNN embedding layers with ReLU, stack multi-head attention encoder modules with dense projection and dropout/normalisation sublayers, and attach a decoder emitting kmers over the expanded vocabulary {A,C,G,U,N1..Nn,<,>,-}, where N tokens represent modified bases. Compile with categorical cross-entropy loss and the Adam optimizer under TensorFlow v2.7.0 with a custom learning-rate schedule. Load the pretrained h5 weights into the instantiated model. Decision point: the runtime targets CUDA 11.8 + cuDNN 8.x on an NVIDIA GPU (A100 tested); allocate at least 3 GB of GPU memory per parallel basecalling thread.

## Related tools

- **NanoSpeech** (Basecaller whose transformer architecture (STFT input, CNN embeddings, multi-head attention encoders, expanded-vocabulary decoder) is reconstructed and instantiated from .h5.cfg parameters) — https://github.com/F0nz0/NanoSpeech_basecaller
- **TensorFlow** (Framework used to define, compile (categorical cross-entropy + Adam optimizer with custom LR schedule) and run the transformer model)
- **Keras** (keras.io transformer ASR example is the reference code adapted for the encoder-decoder architecture) — https://github.com/keras-team/keras-io

## Examples

```
python3 NanoSpeech.py -d {fast5_folder_path} -o {output.fastq} -m {model_weigths in ./models/} -t {number of threads}
```

## Evaluation signals

- The saved nano_speech_model.py executes without error and instantiates the full architecture from the .h5.cfg parameters
- The output layer vocabulary contains both canonical bases (A, C, G, U) and modification tokens (N1..Nn, e.g. inosine), plus delimiter symbols <, > and -
- Pretrained h5 weights load into the instantiated model without shape mismatches
- Basecalled FASTQ output places candidate modified-base indices (e.g. inosine adenosines) in the read header as the repository documents
- GPU memory footprint stays near ~3 GB per parallel basecalling thread as expected

## Limitations

- Tested on a CentOS-7 system with NVIDIA A100-PCIE-40GB, CUDA 11.8, cuDNN 8.6, TensorFlow v2.7.0; other stacks may require adjustments
- Training datasets adequate for transformer-based basecallers of ONT dRNA data are currently missing, and the unavailability of training datasets used by current basecalling tools limits unbiased, reproducible comparisons
- Inference artefacts or 'hallucinations' arise from highly similar chemical structures, and streamlined models for highly similar modifications are still under development
- Primarily tested on synthetic constructs as a proof of concept

## Evidence

- [other] NanoSpeech converts padded pA-scale current chunks into spectrograms via STFT: [evidence span withheld — anchor status: not_found]
- [other] Read transformer init parameters from .h5.cfg: [evidence span withheld — anchor status: not_found]
- [readme] Expanded dictionary description: "NanoSpeech is based on a Transformer model with expanded dictionaries with respect to canonical-nucleotides only (A,C,G,U,I=Inosine or other additional modifications)."
- [readme] TF/CUDA runtime environment: "NanoSpeech has been tested on a CentOS-7 system with an NVIDIA A100-PCIE-40GB GPU (Compute Capability 8.0), CUDA 11.8, cuDNN 8.6 and TensorFlow v.2.7.0."
- [intro] Objective: transformer basecaller with expanded dictionary: "we developed a computational pipeline based on two brand-new programs, NanoListener and NanoSpeech. NanoListener allows the creation of robust training datasets from ONT dRNA data"
