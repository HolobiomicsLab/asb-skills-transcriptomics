---
name: dual-branch-cnn-architecture-implementation
description: 'Use when reproducing or porting the MU-PseUDeep pseudouridine-site classifier:
  input is paired 51×4 sequence and 51×6 RNAshapes structure one-of-K matrices that
  must be encoded by separate Conv1D branches and fused in Keras.'
when_to_use_negative:
- The task requires a sequence-only CNN without secondary-structure context (e.g.,
  replicating the paper's baseline comparison model).
- Sequences are not fixed-length, uridine-centered 51-nt fragments one-of-K encoded
  (e.g., variable-length RNAs or protein data).
- The goal is hyperparameter search over architectures rather than reproducing the
  fixed MU-PseUDeep topology (use the Hyperas/Hyperopt workflow instead).
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MU-PseUDeep
  role: The dual-branch 1D CNN architecture being reproduced; its open-source reference
    implementation this skill mirrors
  repo: https://github.com/smk5g5/MU-PseUDeep
- name: Keras
  role: Framework used to instantiate the two input branches, Conv1D layers, concatenation,
    and 512-neuron dense head
- name: TensorFlow
  role: Backend on which the compiled SGD/binary-cross-entropy model runs
- name: Hyperas
  role: Supplied the tuned kernel-regularizer weights and dropout rates embedded in
    each Conv1D layer during model construction
  repo: https://github.com/maxpumperla/hyperas
- name: hyperopt
  role: Optimization backend wrapped by Hyperas to search the architecture's hyperparameter
    space
  repo: https://github.com/hyperopt/hyperopt
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:83d39f060778b38e1f6bfaa9951418cd59e8d2057cfaec112b51633e2b4291d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smk5g5/MU-PseUDeep
  tools:
  - Hyperas
  - Keras
  - MU-PseUDeep
  - TensorFlow
  - hyperopt
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# dual-branch-cnn-architecture-implementation

## Summary

Reconstructs the MU-PseUDeep dual-input 1D CNN in Keras/TensorFlow: two Conv1D branches encode the 51×4 one-of-K RNA sequence and 51×6 RNAshapes secondary-structure matrices, whose outputs are concatenated into a 512-neuron softplus dense layer and a binary Ψ-site head (661,118 trainable parameters).

## When to use

Use when reproducing or porting the MU-PseUDeep pseudouridine-site classifier: input is paired 51×4 sequence and 51×6 RNAshapes structure one-of-K matrices that must be encoded by separate Conv1D branches and fused in Keras.

## When NOT to use

- The task requires a sequence-only CNN without secondary-structure context (e.g., replicating the paper's baseline comparison model).
- Sequences are not fixed-length, uridine-centered 51-nt fragments one-of-K encoded (e.g., variable-length RNAs or protein data).
- The goal is hyperparameter search over architectures rather than reproducing the fixed MU-PseUDeep topology (use the Hyperas/Hyperopt workflow instead).

## Inputs

- One-of-K encoded binary matrix of RNA sequence, size 51×4 (uridine-centered 51-nt fragments)
- One-of-K encoded binary matrix of RNAshapes dot-bracket secondary-structure context, size 51×6
- Hyperas-tuned hyperparameters: per-layer kernel regularizers, dropout rates (66.5%/74.5%/3.8%/36.9%), and SGD learning rate 0.0137

## Outputs

- Keras functional model (TensorFlow backend) with 661,118 trainable parameters
- Binary pseudouridine (Ψ) site classification scores for uridine-centered fragments

## How to apply

Build two Keras functional-API branches: one taking a 51×4 one-of-K matrix of the raw RNA sequence and one taking a 51×6 matrix of the RNAshapes dot-bracket secondary-structure context. Feed each branch through a first 1D CNN layer (filters=5, kernel_size=10, 'glorot_normal' initializer, 'PRelu' activation) with branch-tuned kernel_regularizer and dropout (seq_1: 0.0321/66.5%; sec_1: 0.00109/74.5%), then a second 1D CNN layer (filters=9, kernel_size=4, 'PRelu'; seq_2: 0.01608/3.8%; sec_2: 0.0340/36.9%). Concatenate all four convolutional outputs and pass them to a 512-neuron dense layer with 'softplus' activation followed by the binary Ψ-site output head. Compile with the SGD optimizer (learning rate 0.0137) and binary cross-entropy loss on a TensorFlow backend. Validate the reconstruction by confirming the total trainable parameter count of 661,118 as stated in the paper. The rationale is the paper's finding that adding the secondary-structure branch improved accuracy/F1 by 3–4% (and sensitivity up to 9%) over a sequence-only CNN.

## Related tools

- **MU-PseUDeep** (The dual-branch 1D CNN architecture being reproduced; its open-source reference implementation this skill mirrors) — https://github.com/smk5g5/MU-PseUDeep
- **Keras** (Framework used to instantiate the two input branches, Conv1D layers, concatenation, and 512-neuron dense head)
- **TensorFlow** (Backend on which the compiled SGD/binary-cross-entropy model runs)
- **Hyperas** (Supplied the tuned kernel-regularizer weights and dropout rates embedded in each Conv1D layer during model construction) — https://github.com/maxpumperla/hyperas
- **hyperopt** (Optimization backend wrapped by Hyperas to search the architecture's hyperparameter space) — https://github.com/hyperopt/hyperopt

## Examples

```
from keras.layers import Conv1D; seq_1 = Conv1D(filters=5, kernel_size=10, kernel_initializer='glorot_normal', activation='PRelu')(seq_input)
```

## Evaluation signals

- model.count_params() (or model.summary()) reports 661,118 total trainable parameters, matching the paper
- Input tensors have shape (None, 51, 4) for the sequence branch and (None, 51, 6) for the secondary-structure branch
- Concatenation merges exactly four Conv1D outputs (two per branch) into the 512-neuron 'softplus' dense layer ahead of the binary output head
- Compiled optimizer is SGD with learning_rate=0.0137 and loss='binary_crossentropy'
- Per-layer dropout rates equal the tuned values: 66.5% (seq_1), 74.5% (sec_1), 3.8% (seq_2), 36.9% (sec_2)

## Limitations

- Direct architectural comparison with iPseU-CNN was impossible because no source code or webserver was available for that approach
- The 661,118-parameter count and topology are specific to the 51×4/51×6 one-of-K encodings; changing input length or alphabet breaks validation
- Tuned values (kernel regularizers, dropout rates, lr=0.0137) came from Hyperas on this paper's human/mouse/yeast Ψ datasets and may not transfer to other data

## Evidence

- [abstract] Dual-input architecture: sequence and secondary structure fed to two CNN sets: "which inputs the raw RNA sequence and the predicted secondary structure to two sets of convolutional neural networks"
- [abstract] Open-source reference implementation available on GitHub: "The tool is open source, available at https://github.com/smk5g5/MU-PseUDeep."
- [methods] Sequence branch input is a 51×4 one-of-K binary matrix: "Sequence data was converted into a one-of-K encoding binary matrix of size 51 × 4"
- [methods] Each of the two inputs is processed by a pair of 1D CNNs: "For each input (sequence and secondary structure), a pair of 1D CNN was used"
- [methods] Branch-level regularizer and dropout values were obtained via Hyperas optimization: "A hyperparameter optimization of various hyperparameters was carried out using Hyperas"
