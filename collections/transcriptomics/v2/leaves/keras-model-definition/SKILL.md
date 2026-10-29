---
name: keras-model-definition
description: Use when reimplementing or validating a dual-branch 1D CNN for binary
  classification of RNA modification sites from one-of-K encoded sequence and structure
  matrices, or when a model must match a published Keras parameter count.
when_to_use_negative:
- Task requires a single-input sequence-only CNN; the article's sequence-only baseline
  lacks the secondary-structure branch and does not achieve the combined model's 3–4%
  accuracy/F1 improvement
- Input sequences are not 51-nt uridine-centered fragments; the 51×4 / 51×6 input
  shapes assume a fixed window with uridine at the center
- A tree-based or SVM method is preferred over deep learning (e.g. reproducing XG-PseU,
  PseUI, or iRNA-PseU baselines)
edam_operation: http://edamontology.org/operation_3901
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
tools:
- name: MU-PseUDeep
  role: Published dual-input 1D CNN architecture being reconstructed; parameter-count
    and hyperparameter reference
  repo: https://github.com/smk5g5/MU-PseUDeep
- name: Keras
  role: Framework used to define and instantiate the two-branch Conv1D model with
    a TensorFlow backend
- name: TensorFlow
  role: Backend for the Keras model definition and training
- name: Hyperas
  role: Wrapper used for hyperparameter optimization of filters, dropout rates, regularizers,
    and learning rate
  repo: https://github.com/maxpumperla/hyperas
- name: Hyperopt
  role: Underlying search library driven by Hyperas for tuning the model hyperparameters
  repo: https://github.com/hyperopt/hyperopt
- name: RNAshapes
  role: Generates the dot-bracket secondary structure notation that is one-of-K encoded
    into the 51×6 structure input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:098c26ae66218d7dfa423947568ea664db7efc5a119b3de381ec3a60a8ff1051
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smk5g5/MU-PseUDeep
  tools:
  - Hyperas
  - Hyperopt
  - Keras
  - MU-PseUDeep
  - RNAshapes
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3901
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# keras-model-definition

## Summary

Reconstruct a dual-input 1D CNN in Keras (TensorFlow backend) following the MU-PseUDeep architecture, which encodes RNA sequence (51×4 one-of-K) and RNAshapes secondary-structure context (51×6) in two parallel convolutional branches whose concatenated feature maps feed a 512-neuron softplus dense layer and a binary classification head.

## When to use

Use when reimplementing or validating a dual-branch 1D CNN for binary classification of RNA modification sites from one-of-K encoded sequence and structure matrices, or when a model must match a published Keras parameter count.

## When NOT to use

- Task requires a single-input sequence-only CNN; the article's sequence-only baseline lacks the secondary-structure branch and does not achieve the combined model's 3–4% accuracy/F1 improvement
- Input sequences are not 51-nt uridine-centered fragments; the 51×4 / 51×6 input shapes assume a fixed window with uridine at the center
- A tree-based or SVM method is preferred over deep learning (e.g. reproducing XG-PseU, PseUI, or iRNA-PseU baselines)

## Inputs

- One-of-K encoded binary matrix of RNA sequence, shape 51×4 (25 bases upstream + uridine + 25 bases downstream)
- One-of-K encoded binary matrix of RNAshapes secondary-structure dot-bracket notation, shape 51×6
- Binary labels for pseudouridine (Ψ) site vs. non-site

## Outputs

- Compiled Keras (TensorFlow backend) dual-branch 1D CNN model with trainable parameter count of 661,118
- Binary classification scores for candidate uridine sites

## How to apply

Build two input branches accepting one-of-K encoded binary matrices of size 51×4 (RNA sequence) and 51×6 (RNAshapes secondary-structure dot-bracket context). For each branch, add a first Conv1D layer with 5 filters, kernel size 10, 'glorot_normal' initializer, and 'PRelu' activation with tuned kernel-regularizer and dropout (seq_1: 0.0321/66.5%; sec_1: 0.00109/74.5%), then a second Conv1D layer with 9 filters, kernel size 4, 'PRelu' activation and tuned regularizer/dropout (seq_2: 0.01608/3.8%; sec_2: 0.0340/36.9%). Concatenate all four conv outputs and pass them to a 512-neuron dense layer with 'softplus' activation followed by the binary classification output head. Compile with the SGD optimizer (learning rate 0.0137) and binary cross-entropy loss. Hyperparameters (filter counts, dropout, regularizers, learning rate) were selected via Hyperas/Hyperopt optimization.

## Related tools

- **MU-PseUDeep** (Published dual-input 1D CNN architecture being reconstructed; parameter-count and hyperparameter reference) — https://github.com/smk5g5/MU-PseUDeep
- **Keras** (Framework used to define and instantiate the two-branch Conv1D model with a TensorFlow backend)
- **TensorFlow** (Backend for the Keras model definition and training)
- **Hyperas** (Wrapper used for hyperparameter optimization of filters, dropout rates, regularizers, and learning rate) — https://github.com/maxpumperla/hyperas
- **Hyperopt** (Underlying search library driven by Hyperas for tuning the model hyperparameters) — https://github.com/hyperopt/hyperopt
- **RNAshapes** (Generates the dot-bracket secondary structure notation that is one-of-K encoded into the 51×6 structure input)

## Examples

```
model = Model(inputs=[seq_in, struct_in], outputs=out); model.compile(optimizer=SGD(learning_rate=0.0137), loss='binary_crossentropy')
```

## Evaluation signals

- model.summary() must report exactly 661,118 total trainable parameters
- Both input tensors must have shapes (None, 51, 4) and (None, 51, 6)
- Each branch must contain exactly two Conv1D layers (5 filters/kernel 10, then 9 filters/kernel 4) with PReLU activations
- Concatenated feature maps must feed a Dense(512, activation='softplus') layer before the binary output head
- Model must compile with SGD (lr=0.0137) and binary cross-entropy loss without errors

## Limitations

- Fixed 51-nt input window limits applicability to uridine-centered fragments; other window sizes require architectural changes
- Parameter count (661,118) is only reproducible with the exact filter counts, kernel sizes, and dense layer size reported
- Reported hyperparameters (dropouts, regularizers, learning rate 0.0137) were tuned by Hyperas for this specific task and may not transfer to other datasets
- Direct comparison to iPseU-CNN was impossible because no source code or webserver was available for that approach

## Evidence

- [methods] Two-branch architecture with per-branch Conv1D layers: "For each input (sequence and secondary structure), a pair of 1D CNN was used"
- [methods] One-of-K encoding of the sequence input: "Sequence data was converted into a one-of-K encoding binary matrix of size 51 × 4"
- [abstract] Dual-input design capturing sequence and structure: "which inputs the raw RNA sequence and the predicted secondary structure to two sets of convolutional neural networks"
- [methods] Secondary structure encoding source: "the abstract secondary structure dot-bracket notation was generated using the RNAshapes package"
- [methods] Hyperparameter tuning framework: "A hyperparameter optimization of various hyperparameters was carried out using Hyperas"
- [readme] Repository hosting the reference model: "Prediction of Pseudouridine Sites using deep learning"
