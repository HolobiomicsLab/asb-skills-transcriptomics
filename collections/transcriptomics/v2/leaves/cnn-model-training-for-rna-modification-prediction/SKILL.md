---
name: cnn-model-training-for-rna-modification-prediction
description: Use when binary-classifying RNA modification sites from uridine-centered
  sequence fragments plus predicted secondary structure, or when reproducing/ablation-testing
  sequence-only vs structure-only CNN baselines.
when_to_use_negative:
- The goal is a direct comparison to iPseU-CNN — no source code or webserver was available
  for it, making direct comparison impossible
- Input fragments are not uridine-centered 51-mers from an Ψ benchmark; the encoding
  assumes exactly 51 positions with uridine at the center
- Fully imbalanced evaluation only, if the 3–4% accuracy/F1 improvement benchmark
  on the balanced dataset is the validation target
edam_operation: http://edamontology.org/operation_0253
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MU-PseUDeep
  role: Dual-input CNN architecture (sequence + secondary structure branches) being
    trained for Ψ site prediction and ablated into sequence-only and structure-only
    CNNs
  repo: https://github.com/smk5g5/MU-PseUDeep
- name: Hyperas
  role: Wraps the Keras CNN definition so hyperparameters (dropouts, optimizer settings)
    can be templated and tuned
  repo: https://github.com/maxpumperla/hyperas
- name: Hyperopt
  role: TPE search backend invoked by Hyperas to optimize the CNN hyperparameters
  repo: https://github.com/hyperopt/hyperopt
- name: Keras / TensorFlow
  role: Implements and fits the 1D CNN branches with SGD, binary cross-entropy, early
    stopping, and checkpoint callbacks
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:76e8c256dffa49ef462753800f4f419512cbc325175b7acc540ed541df5367ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smk5g5/MU-PseUDeep
  tools:
  - Hyperas
  - Hyperopt
  - Keras / TensorFlow
  - MU-PseUDeep
  - TensorFlow
  edam_operation: http://edamontology.org/operation_0253
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# cnn-model-training-for-rna-modification-prediction

## Summary

Train 1D convolutional neural networks (Keras/TensorFlow, hyperparameter-tuned with Hyperas/Hyperopt) on one-of-K encoded RNA sequence (51 × 4) and secondary-structure (51 × 6) matrices to classify pseudouridine (Ψ) sites, following the MU-PseUDeep dual-input architecture.

## When to use

Use when binary-classifying RNA modification sites from uridine-centered sequence fragments plus predicted secondary structure, or when reproducing/ablation-testing sequence-only vs structure-only CNN baselines.

## When NOT to use

- The goal is a direct comparison to iPseU-CNN — no source code or webserver was available for it, making direct comparison impossible
- Input fragments are not uridine-centered 51-mers from an Ψ benchmark; the encoding assumes exactly 51 positions with uridine at the center
- Fully imbalanced evaluation only, if the 3–4% accuracy/F1 improvement benchmark on the balanced dataset is the validation target

## Inputs

- Ψ-centered 51 bp RNA fragments (Ψ plus 25 flanking bases) for human (hg19), mouse (mm10), yeast (sacCer3) from RMBase v2.0
- One-of-K encoded sequence matrix of size 51 × 4
- One-of-K encoded secondary-structure matrix of size 51 × 6 (dot-bracket notation from RNAshapes)
- Binary class labels (positive Ψ site vs sampled negative region without experimentally validated Ψ sites)

## Outputs

- Trained sequence-only, structure-only, and dual-input 1D CNN models (Keras checkpoints)
- Test-fold accuracy, F1, sensitivity (and MCC for imbalanced data) per model
- Ablation comparison table (dual-input vs sequence-only vs structure-only CNNs)

## How to apply

Encode each 51 bp Ψ fragment (Ψ plus 25 flanking bases from RMBase v2.0 for human/mouse/yeast) as a one-of-K binary matrix of size 51 × 4 for sequence and 51 × 6 for RNAshapes-derived secondary structure. Build two 1D CNN branches per input — seq_1 (filters 5, kernel 10) and seq_2 (filters 9, kernel 4) for sequence; sec_1 (filters 5, kernel 10) and sec_2 (filters 9, kernel 4) for structure — with glorot_normal init, PRelu activations, and tuned dropout (66.5%/3.8% sequence; 74.5%/36.9% structure). Compile with SGD (lr 0.0137) and binary cross-entropy, fit at batch size 32 for up to 500 epochs with early-stopping patience 20 and model checkpointing, applying bootstrapping by binning negative samples so each bin matches the positive class size. Tune hyperparameters with Hyperas/Hyperopt (TPE), then evaluate accuracy, F1, and sensitivity per ablation; success is the dual-input model beating the sequence-only CNN by ~3–4% accuracy/F1 and up to 9% sensitivity on the balanced dataset.

## Related tools

- **MU-PseUDeep** (Dual-input CNN architecture (sequence + secondary structure branches) being trained for Ψ site prediction and ablated into sequence-only and structure-only CNNs) — https://github.com/smk5g5/MU-PseUDeep
- **Hyperas** (Wraps the Keras CNN definition so hyperparameters (dropouts, optimizer settings) can be templated and tuned) — https://github.com/maxpumperla/hyperas
- **Hyperopt** (TPE search backend invoked by Hyperas to optimize the CNN hyperparameters) — https://github.com/hyperopt/hyperopt
- **Keras / TensorFlow** (Implements and fits the 1D CNN branches with SGD, binary cross-entropy, early stopping, and checkpoint callbacks)

## Examples

```
best_run = optim.minimize(model=create_model, data=data, algo=tpe.suggest, max_evals=10, trials=Trials())
```

## Evaluation signals

- Comparison table shows the dual-input model improving over the sequence-only CNN by approximately 3–4% accuracy and F1 (and up to 9% sensitivity) on the balanced dataset
- Sequence-only CNN outperforms the structure-only CNN, matching the reported ablation ordering
- Training runs terminate via early stopping (patience 20) within the 500-epoch cap and restore the checkpointed best weights
- Encoded input shapes are exactly 51 × 4 (sequence) and 51 × 6 (secondary structure) one-of-K binary matrices
- On imbalanced data, the combined model exceeds the sequence CNN by ~2% accuracy, ~2% F1, up to 4% MCC, and up to almost 7% sensitivity

## Limitations

- No source code or webserver was available for iPseU-CNN, so a direct comparison with that sequence-only CNN approach was impossible
- Secondary structure must be predicted beforehand (RNAshapes dot-bracket notation), adding dependence on an external structure predictor
- Negative-sample bootstrapping (binning negatives to the positive class size) means results depend on the sampling scheme across bins

## Evidence

- [methods] One-of-K encoding of sequence and secondary structure: "Sequence data was converted into a one-of-K encoding binary matrix of size 51 × 4"
- [methods] Dual 1D CNN per input: "For each input (sequence and secondary structure), a pair of 1D CNN was used"
- [methods] Hyperparameter tuning: "A hyperparameter optimization of various hyperparameters was carried out using Hyperas"
- [results] Ablation improvement finding: "The improvement was by 3–4% for accuracy and F1 and up to 9% for sensitivity in the balanced dataset in comparison to sequence CNN"
- [readme] Hyperas role: [evidence span withheld — anchor status: not_found]
