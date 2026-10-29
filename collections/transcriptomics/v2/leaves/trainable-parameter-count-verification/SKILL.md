---
name: trainable-parameter-count-verification
description: Use when re-implementing a published neural architecture in Keras/TensorFlow
  and you need objective confirmation that layer configurations (filters, kernel sizes,
  dense units) reproduce the paper's reported model exactly.
when_to_use_negative:
- You are training or fine-tuning an intentionally modified architecture (e.g., different
  dense width or added layers), where the paper's parameter count is no longer the
  correct invariant
- You only need to verify hyperparameters such as learning rate, dropout rates, or
  kernel regularizer weights — these do not affect the parameter count and require
  different checks
- The source paper reports no architecture details or parameter totals, leaving no
  reference value to verify against
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MU-PseUDeep
  role: The published dual-input CNN architecture whose reconstructed Keras model
    must yield 661,118 trainable parameters
  repo: https://github.com/smk5g5/MU-PseUDeep
- name: Keras
  role: Framework used to instantiate the model and expose the parameter count via
    count_params()/model.summary()
- name: TensorFlow
  role: Backend on which the Keras model is instantiated for the parameter count check
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:d8cc8bee7fe9335d77604541b4345a56104993cae766ec1319fb0e7b2e7af009
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smk5g5/MU-PseUDeep
  tools:
  - Keras
  - MU-PseUDeep
  - TensorFlow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Trainable Parameter Count Verification

## Summary

A structural sanity check that validates a reconstructed deep learning model (e.g., the MU-PseUDeep dual-branch 1D CNN) by confirming its total trainable parameter count matches the value reported in the source paper (661,118), catching silent architecture errors such as wrong filter sizes, kernel dimensions, or dense-layer widths.

## When to use

Use when re-implementing a published neural architecture in Keras/TensorFlow and you need objective confirmation that layer configurations (filters, kernel sizes, dense units) reproduce the paper's reported model exactly.

## When NOT to use

- You are training or fine-tuning an intentionally modified architecture (e.g., different dense width or added layers), where the paper's parameter count is no longer the correct invariant
- You only need to verify hyperparameters such as learning rate, dropout rates, or kernel regularizer weights — these do not affect the parameter count and require different checks
- The source paper reports no architecture details or parameter totals, leaving no reference value to verify against

## Inputs

- Fully assembled Keras/TensorFlow model (two 1D-CNN branches over 51×4 and 51×6 one-of-K matrices, concatenated and fed to a 512-neuron dense layer)
- Published parameter count to check against (661,118 for MU-PseUDeep)

## Outputs

- Total trainable parameter count reported by model.count_params() or model.summary()
- Pass/fail assertion comparing the reconstructed count to the published value

## How to apply

Instantiate the full model first: build two input branches accepting one-of-K encoded binary matrices of size 51×4 (RNA sequence) and 51×6 (RNAshapes secondary structure), each passing through a pair of 1D CNN layers (first: filter size 5, kernel size 10; second: filter size 9, kernel size 4, both with 'PRelu' activation), concatenate the four conv outputs into a 512-neuron dense layer with 'softplus' activation, and add the binary classification head. Compile with SGD (learning rate 0.0137) and binary cross-entropy so the optimizer slot parameters are counted. Then call Keras' parameter-counting routine (e.g., model.count_params() or read model.summary()) and assert the result equals 661,118 as stated in the paper. If the count differs, audit each layer's filter count, kernel size, and dense width before training, since parameter count is a sensitive invariant that changes with any architectural deviation.

## Related tools

- **MU-PseUDeep** (The published dual-input CNN architecture whose reconstructed Keras model must yield 661,118 trainable parameters) — https://github.com/smk5g5/MU-PseUDeep
- **Keras** (Framework used to instantiate the model and expose the parameter count via count_params()/model.summary())
- **TensorFlow** (Backend on which the Keras model is instantiated for the parameter count check)

## Examples

```
model = Model([seq_in, sec_in], out); model.compile(optimizer=SGD(lr=0.0137), loss='binary_crossentropy'); assert model.count_params() == 661118
```

## Evaluation signals

- model.count_params() returns exactly 661,118 trainable parameters
- model.summary() lists two branches, each with two 1D CNN layers (filter sizes 5 and 9; kernel sizes 10 and 4), a concatenation node, a 512-unit dense layer, and a binary output head
- Changing any single filter count or dense width alters the count, confirming the check is sensitive to the intended architecture
- Compiling with the SGD optimizer (lr 0.0137) and binary cross-entropy does not change the trainable parameter total

## Limitations

- A matching parameter count confirms layer sizes but not the presence of specific activations (PRelu/softplus), initializers (glorot_normal), or regularizer weights — those must be checked separately
- Non-trainable parameters or optimizer slot variables may be included depending on the counting method used, so the counting routine must match what the paper reported
- The check assumes the paper's stated total (661,118) is itself accurate; it cannot detect if the original paper miscounted

## Evidence

- [methods] pair of 1D CNN per input branch: "For each input (sequence and secondary structure), a pair of 1D CNN was used"
- [methods] one-of-K encoded inputs: "Sequence data was converted into a one-of-K encoding binary matrix of size 51 × 4"
- [readme] architecture diagram in repository README: "Deep learning model architecture for pseudouridine RNA modification prediction."
