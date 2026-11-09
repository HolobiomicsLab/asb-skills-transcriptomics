---
name: rna-sequence-one-hot-encoding
description: Use when preparing Ψ-centered 51-bp RNA fragments (sequence plus RNAshapes
  dot-bracket structure) from RMBase v2.0 as fixed-size one-of-K binary matrices to
  feed paired 1D CNN inputs like MU-PseUDeep.
when_to_use_negative:
- Fragments are not 51-bp and uridine-centered (e.g., variable-length windows or non-Ψ
  modifications) — the fixed 51 × 4 and 51 × 6 matrices would not apply
- Input is already a derived feature vector (e.g., the pseudo-nucleotide composition
  features used by iRNA-PseU or PPUS SVMs) rather than raw fragment sequence and structure
- You need to handle class imbalance alone — one-of-K encoding does not address imbalance;
  negatives must still be bootstrapped into N bins sized to the positive class
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: MU-PseUDeep
  role: Dual-input 1D CNN (Keras/TensorFlow) that consumes the 51 × 4 sequence and
    51 × 6 structure one-of-K matrices for Ψ site prediction
  repo: https://github.com/smk5g5/MU-PseUDeep
- name: RNAshapes
  role: Generates the abstract dot-bracket secondary-structure notation that is one-of-K
    encoded into the 51 × 6 matrix
- name: Keras
  role: Framework in which the encoded 51 × 4 and 51 × 6 matrices are declared as
    inputs to the paired 1D convolutional layers
- name: Hyperas
  role: Wraps the Keras CNN consuming the encoded matrices to tune filters, kernels,
    and dropout probabilities via Hyperopt
  repo: https://github.com/maxpumperla/hyperas
- name: Hyperopt
  role: Search algorithm behind Hyperas that optimizes the CNN hyperparameters applied
    to the one-of-K encoded inputs
  repo: https://github.com/hyperopt/hyperopt
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:57485340ffeca75b3a8ec44c9405c6c576b9cd23920125168051f620baec9465
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
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# rna-sequence-one-hot-encoding

## Summary

One-of-K (one-hot) encoding of 51-bp Ψ-centered RNA fragments into 51 × 4 nucleotide binary matrices and RNAshapes dot-bracket secondary structure into 51 × 6 binary matrices, producing the fixed-width dual inputs that MU-PseUDeep's paired 1D CNNs require for pseudouridine site prediction.

## When to use

Use when preparing Ψ-centered 51-bp RNA fragments (sequence plus RNAshapes dot-bracket structure) from RMBase v2.0 as fixed-size one-of-K binary matrices to feed paired 1D CNN inputs like MU-PseUDeep.

## When NOT to use

- Fragments are not 51-bp and uridine-centered (e.g., variable-length windows or non-Ψ modifications) — the fixed 51 × 4 and 51 × 6 matrices would not apply
- Input is already a derived feature vector (e.g., the pseudo-nucleotide composition features used by iRNA-PseU or PPUS SVMs) rather than raw fragment sequence and structure
- You need to handle class imbalance alone — one-of-K encoding does not address imbalance; negatives must still be bootstrapped into N bins sized to the positive class

## Inputs

- Ψ-centered 51-nt RNA fragments (Ψ plus 25 flanking bases each side) derived from RMBase v2.0 for human (hg19), mouse (mm10), and yeast (sacCer3)
- Secondary structure dot-bracket notation for the same 51-nt fragments generated with RNAshapes
- Class labels (Ψ site vs. non-site) with negatives sampled from RNA regions without experimentally validated Ψ sites

## Outputs

- One-of-K binary sequence matrix of size 51 × 4 per fragment
- One-of-K binary secondary-structure matrix of size 51 × 6 per fragment
- Encoded tensors ready as paired inputs to the 1D CNNs in Keras/TensorFlow

## How to apply

Start from fragments of exactly 51 nt (Ψ plus 25 upstream and 25 downstream bases) extracted with BEDTools from RMBase v2.0 annotations against hg19, mm10, or sacCer3, keeping only fragments with uridine at their center. Generate the secondary structure dot-bracket notation for each fragment using the RNAshapes package. Convert the sequence into a one-of-K binary matrix of size 51 × 4 (one channel per nucleotide per position) and the structure into a one-of-K binary matrix of size 51 × 6, so both inputs share 51 rows that align positionally for the dual-input CNN. Feed these matrices to the paired 1D CNNs of MU-PseUDeep (seq_1: filter size 5, kernel size 10; seq_2: filter size 9, kernel size 4; PRelu activation) and tune filters, kernels, and dropout via Hyperas/Hyperopt. Train with SGD (learning rate 0.0137), binary cross-entropy, batch size 32, up to 500 epochs with early-stop patience of 20, bootstrapping negatives into N bins each sized to the positive class. The rationale: fixed-width one-of-K preserves per-position categorical information for convolution kernels, and this dual encoding underlies the 3–4% accuracy/F1 gain (up to 9% sensitivity) of the combined model over the sequence-only CNN on the balanced dataset.

## Related tools

- **MU-PseUDeep** (Dual-input 1D CNN (Keras/TensorFlow) that consumes the 51 × 4 sequence and 51 × 6 structure one-of-K matrices for Ψ site prediction) — https://github.com/smk5g5/MU-PseUDeep
- **RNAshapes** (Generates the abstract dot-bracket secondary-structure notation that is one-of-K encoded into the 51 × 6 matrix)
- **Keras** (Framework in which the encoded 51 × 4 and 51 × 6 matrices are declared as inputs to the paired 1D convolutional layers)
- **Hyperas** (Wraps the Keras CNN consuming the encoded matrices to tune filters, kernels, and dropout probabilities via Hyperopt) — https://github.com/maxpumperla/hyperas
- **Hyperopt** (Search algorithm behind Hyperas that optimizes the CNN hyperparameters applied to the one-of-K encoded inputs) — https://github.com/hyperopt/hyperopt

## Examples

```
import numpy as np; X_seq = np.eye(4)[nt_idx]  # 51x4 one-of-K; X_sec = np.eye(6)[struct_idx]  # 51x6 for MU-PseUDeep dual inputs
```

## Evaluation signals

- One-of-K invariant: every row of the 51 × 4 sequence matrix (and the 51 × 6 structure matrix) contains exactly one '1' across its channels
- Dimensions match the paper exactly: 51 rows (Ψ plus 25 bases each side) × 4 nucleotide channels for sequence, and 51 × 6 for structure
- Position 26 (center) of every positive fragment is uridine, consistent with the uridine-centered fragment filter
- Sequence and structure matrices have identical row counts so per-position channels align for the dual-input CNN
- Encoded data reproduces the ablation behavior when trained with SGD (lr 0.0137), binary cross-entropy, batch size 32: sequence-only CNN beats structure-only CNN, and dual-input improves accuracy/F1 by 3–4% (sensitivity up to 9%) on the balanced dataset

## Limitations

- Matrix sizes are hard-coded to 51 positions and 4/6 channels; fragments with different flanking lengths or structure alphabets require re-encoding
- Structure features depend on RNAshapes-predicted dot-bracket notation, so structure-prediction errors propagate as noisy channels in the 51 × 6 matrix
- The paper's ablation showed the sequence-only CNN outperformed the secondary-structure-only CNN, so structure encoding alone is insufficient
- Direct comparison with the iPseU-CNN-style sequence-only design was impossible because no source code or webserver was available for that approach

## Evidence

- [methods] One-of-K sequence encoding defines the primary input tensor: "Sequence data was converted into a one-of-K encoding binary matrix of size 51 × 4"
- [methods] Structure notation source for the 51 × 6 matrix: "the abstract secondary structure dot-bracket notation was generated using the RNAshapes package"
- [methods] Fragment extraction fixing the 51-bp window: "we extracted the Ψ and surrounding 25 bases upstream and downstream nucleotides using BEDTools"
- [abstract] Dual-input CNN consuming both encoded matrices: "which inputs the raw RNA sequence and the predicted secondary structure to two sets of convolutional neural networks"
- [readme] Repository purpose statement: "Prediction of Pseudouridine Sites using deep learning"
