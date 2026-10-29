---
name: prelu-convolutional-layer-configuration
description: Use when reconstructing or retraining a dual-branch 1D CNN for nucleotide-modification
  site prediction from one-hot encoded sequence plus secondary-structure matrices,
  or when the target model must report 661,118 trainable parameters.
when_to_use_negative:
- Input is raw nucleotide sequence rather than a one-of-K encoded 51×4 / 51×6 matrix
  — the encoding step must happen first
- A single-modality model suffices — the paper showed the combined model's benefit
  comes from adding the secondary-structure branch; sequence-only CNNs are weaker
  (3–4% lower accuracy/F1)
- No source code or webserver exists for a competitor method (e.g. iPseU-CNN), so
  this architecture cannot be used for a direct head-to-head reconstruction of that
  tool
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: MU-PseUDeep
  role: The dual-input 1D CNN architecture being reconstructed; open-source reference
    implementation
  repo: https://github.com/smk5g5/MU-PseUDeep
- name: Keras
  role: API used to define the Conv1D/PReLU/Dense layers and compile the model
- name: TensorFlow
  role: Backend on which the Keras model is instantiated
- name: hyperas
  role: Wrapper used to tune the kernel-regularizer weights, dropout rates, and learning
    rate via Hyperopt templates
  repo: https://github.com/maxpumperla/hyperas
- name: hyperopt
  role: Underlying hyperparameter optimization engine driven by Hyperas
  repo: https://github.com/hyperopt/hyperopt
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:81d52a6a020f7ad329d0adf704ba25ee41bdf878f0644d323a2b28edbe050edf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smk5g5/MU-PseUDeep
  tools:
  - Keras
  - MU-PseUDeep
  - TensorFlow
  - hyperas
  - hyperopt
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# prelu-convolutional-layer-configuration

## Summary

Configures the dual-input 1D CNN architecture of MU-PseUDeep in Keras/TensorFlow, where PReLU-activated convolutional branches process one-of-K encoded RNA sequence (51×4) and RNAshapes secondary-structure (51×6) matrices before concatenation into a 512-neuron softplus dense layer. Reproduces the paper's exact hyperparameters, including tuned kernel regularizers and dropout rates per layer.

## When to use

Use when reconstructing or retraining a dual-branch 1D CNN for nucleotide-modification site prediction from one-hot encoded sequence plus secondary-structure matrices, or when the target model must report 661,118 trainable parameters.

## When NOT to use

- Input is raw nucleotide sequence rather than a one-of-K encoded 51×4 / 51×6 matrix — the encoding step must happen first
- A single-modality model suffices — the paper showed the combined model's benefit comes from adding the secondary-structure branch; sequence-only CNNs are weaker (3–4% lower accuracy/F1)
- No source code or webserver exists for a competitor method (e.g. iPseU-CNN), so this architecture cannot be used for a direct head-to-head reconstruction of that tool

## Inputs

- One-of-K encoded binary matrix of RNA sequence, size 51×4
- One-of-K encoded binary matrix of RNAshapes secondary-structure dot-bracket context, size 51×6
- Binary labels for pseudouridine (Ψ) site vs. non-site samples
- Tuned hyperparameter set from Hyperas/Hyperopt optimization (regularizer weights, dropout rates, learning rate 0.0137)

## Outputs

- Compiled Keras (TensorFlow backend) dual-branch 1D CNN model with 661,118 trainable parameters
- Binary classification output head for Ψ site prediction
- Trained model checkpoint usable for transcriptome-wide Ψ site scanning

## How to apply

Build two input branches accepting one-of-K encoded binary matrices of size 51×4 (RNA sequence) and 51×6 (RNAshapes dot-bracket secondary structure). For each branch add a first Conv1D layer with filter size 5, kernel size 10, 'glorot_normal' initializer and PReLU activation, using the tuned kernel-regularizer weight and dropout rate (seq_1: 0.0321/66.5%; sec_1: 0.00109/74.5%); then a second Conv1D layer with filter size 9, kernel size 4, PReLU activation, and its tuned values (seq_2: 0.01608/3.8%; sec_2: 0.0340/36.9%). Concatenate all four convolution outputs, feed a 512-neuron dense layer with 'softplus' activation, and attach a binary classification head. Compile with SGD (learning rate 0.0137) and binary cross-entropy loss; the decision point is verifying the instantiated model reports exactly 661,118 trainable parameters as stated in the paper.

## Related tools

- **MU-PseUDeep** (The dual-input 1D CNN architecture being reconstructed; open-source reference implementation) — https://github.com/smk5g5/MU-PseUDeep
- **Keras** (API used to define the Conv1D/PReLU/Dense layers and compile the model)
- **TensorFlow** (Backend on which the Keras model is instantiated)
- **hyperas** (Wrapper used to tune the kernel-regularizer weights, dropout rates, and learning rate via Hyperopt templates) — https://github.com/maxpumperla/hyperas
- **hyperopt** (Underlying hyperparameter optimization engine driven by Hyperas) — https://github.com/hyperopt/hyperopt

## Examples

```
from keras.layers import Conv1D, Input, PReLU; Conv1D(5, 10, kernel_initializer='glorot_normal', kernel_regularizer=l2(0.0321))(Input(shape=(51,4)))  # seq_1 branch; repeat per paper spec, then concatenate and Dense(512, activation='softplus')
```

## Evaluation signals

- Instantiated model.summary() reports exactly 661,118 trainable parameters
- Each branch has two Conv1D layers: layer 1 with 5 filters/kernel size 10, layer 2 with 9 filters/kernel size 4, both PReLU-activated
- Sequence branch first layer uses 'glorot_normal' initializer with kernel regularizer 0.0321 and dropout 66.5%; structure branch uses 0.00109 and 74.5%
- Dense layer after concatenation has 512 units with 'softplus' activation
- Model compiles with SGD at learning rate 0.0137 and binary cross-entropy loss without errors

## Limitations

- Requires RNAshapes-generated dot-bracket secondary-structure input; sequence-only encoding omits the structure branch and loses 3–4% accuracy/F1 (up to 9% sensitivity) per the paper's ablation
- Parameter count validation (661,118) is specific to this exact configuration; changing any filter count, kernel size, or dense width invalidates the check
- Dropout and regularizer values were tuned with Hyperas for the human/mouse/yeast Ψ datasets and may not transfer to other tasks without re-optimization

## Evidence

- [methods] For each input (sequence and secondary structure), a pair of 1D CNN was used: "For each input (sequence and secondary structure), a pair of 1D CNN was used"
- [methods] One-of-K encoding size 51 × 4: "Sequence data was converted into a one-of-K encoding binary matrix of size 51 × 4"
- [readme] README architecture statement: "Deep learning model architecture for pseudouridine RNA modification prediction."
- [abstract] Two sets of CNNs on sequence and structure: "which inputs the raw RNA sequence and the predicted secondary structure to two sets of convolutional neural networks"
