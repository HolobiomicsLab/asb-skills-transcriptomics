---
name: modified-base-vocabulary-design
description: Use when building or adapting a deep-learning basecaller for ONT direct
  RNA data and the goal is single-step detection of modified bases (e.g. m6A, m1A,
  inosine) rather than modification-unaware basecalling followed by re-squiggling.
when_to_use_negative:
- The downstream pipeline already performs modification-unaware basecalling followed
  by re-squiggling with modification-specific models (e.g. Dorado plus modification
  models) — the expanded vocabulary is meant to replace that multi-step workflow
- Only canonical bases are needed and no modified-base calling is required — a standard
  four-base vocabulary suffices and avoids training-data burden
- The target modifications are highly similar chemical structures — streamlined NanoSpeech
  models for these are still under investigation and development
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0659
tools:
- name: NanoSpeech
  role: Transformer basecaller whose output layer is defined over the expanded vocabulary
    {A,C,G,U,N1..Nn,<,>,-}; provides model definitions and pretrained h5 weights
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: NanoListener
  role: Extracts current chunks and kmers from dRNA reads (whatever ONT chemistry/kit)
    to build training datasets whose labels use the expanded vocabulary
- name: TensorFlow
  role: Framework used to define and compile the transformer (categorical cross-entropy
    loss, Adam optimizer, custom learning-rate schedule)
- name: m6ABasecaller
  role: Prior modification-aware basecaller predicting 5 nucleosides (A,C,G,U,m6A)
    — a precedent for expanded-vocabulary design
  repo: https://github.com/novoalab/m6ABasecaller
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:3a5e74aab22c1b312e5f3f4052a11819776c0b74c04410104d3a0775a8314d5d
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
  - NanoListener
  - NanoSpeech
  - TensorFlow
  - m6ABasecaller
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# modified-base-vocabulary-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Designs an expanded output-token vocabulary (canonical bases A, C, G, U plus modification tokens such as inosine, encoded as N1..Nn, together with boundary and gap symbols) so a transformer basecaller can ab initio call modified bases in a single pass instead of requiring separate basecalling, re-squiggling, and modification-detection models.

## When to use

Use when building or adapting a deep-learning basecaller for ONT direct RNA data and the goal is single-step detection of modified bases (e.g. m6A, m1A, inosine) rather than modification-unaware basecalling followed by re-squiggling.

## When NOT to use

- The downstream pipeline already performs modification-unaware basecalling followed by re-squiggling with modification-specific models (e.g. Dorado plus modification models) — the expanded vocabulary is meant to replace that multi-step workflow
- Only canonical bases are needed and no modified-base calling is required — a standard four-base vocabulary suffices and avoids training-data burden
- The target modifications are highly similar chemical structures — streamlined NanoSpeech models for these are still under investigation and development

## Inputs

- ONT dRNA raw signals (fast5/pod5) with corresponding kmer labels extracted via NanoListener
- Target modification list (e.g. inosine, m6A, m1A) to encode as vocabulary tokens
- Transformer model configuration file (models/*.h5.cfg) accompanying the h5 weights

## Outputs

- Expanded-vocabulary token definition, e.g. {A, C, G, U, N1..Nn, <, >, -}
- Compiled TensorFlow/Keras transformer basecaller definition (nano_speech_model.py) with an output layer over the expanded vocabulary
- Pretrained model weights in .h5 format with matching .cfg parameter files

## How to apply

Take the canonical nucleotide alphabet (A, C, G, U for RNA) and append one token per target modification (e.g. I for inosine, or generic N1..Nn tokens), plus structural symbols such as '<', '>', and '-' for sequence start/end and padding. Use NanoListener to extract current chunks and kmers from dRNA reads produced with whatever ONT chemistry and kit, so each kmer label in the training set carries the modified-base tokens. Initialize the transformer from the model configuration file (models/*.h5.cfg) accompanying each h5 weights file, attach the decoder that emits kmers over the expanded vocabulary, and compile with categorical cross-entropy and the Adam optimizer under TensorFlow. Validate that the output layer covers both canonical bases and modification tokens, and that basecalling accuracy remains comparable to state-of-the-art ONT software such as Dorado while modification detection works without any post-processing step like ModiKit.

## Related tools

- **NanoSpeech** (Transformer basecaller whose output layer is defined over the expanded vocabulary {A,C,G,U,N1..Nn,<,>,-}; provides model definitions and pretrained h5 weights) — https://github.com/F0nz0/NanoSpeech_basecaller
- **NanoListener** (Extracts current chunks and kmers from dRNA reads (whatever ONT chemistry/kit) to build training datasets whose labels use the expanded vocabulary)
- **TensorFlow** (Framework used to define and compile the transformer (categorical cross-entropy loss, Adam optimizer, custom learning-rate schedule))
- **m6ABasecaller** (Prior modification-aware basecaller predicting 5 nucleosides (A,C,G,U,m6A) — a precedent for expanded-vocabulary design) — https://github.com/novoalab/m6ABasecaller

## Examples

```
python3 NanoSpeech.py -d {fast5_folder_path} -o {fasta/fastq output} -m {model_weigths in ./models/} -t {number of threads}
```

## Evaluation signals

- The output layer's vocabulary contains both canonical bases (A, C, G, U) and modification tokens (e.g. I/N1..Nn) plus boundary symbols (<, >, -)
- The saved model script executes without error and instantiates the full architecture from the .h5.cfg parameters
- The model compiles under TensorFlow with categorical cross-entropy loss and the Adam optimizer
- Basecalling accuracy on unmodified reads is comparable to state-of-the-art ONT basecallers (e.g. Dorado)
- Modified bases are called ab initio without post-processing steps such as ModiKit or re-squiggling

## Limitations

- Training datasets adequate for transformer-based basecallers of ONT dRNA data are currently missing and must be generated via simulated randomers (NanoListener)
- Highly similar chemical structures and intrinsic errors in modified samples lead to inference artefacts or 'hallucinations' in the modification tokens
- Models were primarily tested on synthetic constructs (IVTs) as proof of concept; public RNA-modification data quality and unknown IVT conditions hamper training set generation

## Evidence

- [other] keras vocabulary in task card: [evidence span withheld — anchor status: not_found]
