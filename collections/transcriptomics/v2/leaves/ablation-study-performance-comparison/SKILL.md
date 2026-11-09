---
name: ablation-study-performance-comparison
description: Use when a multi-input deep learning model (e.g., sequence + structure)
  needs validation that each input branch adds value, by training matched single-input
  CNN ablations on the same encoded data and benchmark folds.
when_to_use_negative:
- When the method you want to compare against has no available source code or webserver
  — e.g., the paper notes that for iPseU-CNN 'no source code or webserver was available
  for this approach, a direct comparison is impossible'
- When the task requires only final performance benchmarking against external published
  tools (XG-PseU, PseUI, iRNA-PseU) rather than internal modality-attribution ablations
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
tools:
- name: MU-PseUDeep
  role: Dual-input CNN baseline whose results the ablations are compared against
  repo: https://github.com/smk5g5/MU-PseUDeep
- name: Hyperas
  role: Hyperparameter optimization wrapper used to tune the CNN architectures (dropout
    rates, filter sizes) prior to ablation training
  repo: https://github.com/maxpumperla/hyperas
- name: Hyperopt
  role: Search backend used by Hyperas for optimizing hyperparameters of the ablation
    CNNs
  repo: https://github.com/hyperopt/hyperopt
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:2e0ef32683c2055a85fb43d22720a261bcadad8966fe514c81933bb3b99398f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smk5g5/MU-PseUDeep
  tools:
  - Hyperas
  - Hyperopt
  - MU-PseUDeep
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ablation-study-performance-comparison

## Summary

Trains single-modality ablation models (sequence-only and secondary-structure-only 1D CNNs) alongside the dual-input model and compares accuracy, F1, and sensitivity to quantify each input modality's contribution. In MU-PseUDeep this isolates the benefit of RNA secondary structure features for pseudouridine (Ψ) site prediction.

## When to use

Use when a multi-input deep learning model (e.g., sequence + structure) needs validation that each input branch adds value, by training matched single-input CNN ablations on the same encoded data and benchmark folds.

## When NOT to use

- When the method you want to compare against has no available source code or webserver — e.g., the paper notes that for iPseU-CNN 'no source code or webserver was available for this approach, a direct comparison is impossible'
- When the task requires only final performance benchmarking against external published tools (XG-PseU, PseUI, iRNA-PseU) rather than internal modality-attribution ablations

## Inputs

- Preprocessed 51-bp RNA fragments centered on Ψ (Ψ ± 25 flanking bases) from RMBase v2.0 for human, mouse, yeast
- One-of-K encoded sequence binary matrices (51 × 4)
- One-of-K encoded secondary structure binary matrices (51 × 6, dot-bracket from RNAshapes)
- 10-fold stratified 80:20 train/test splits with sequence-identity reduction via cd-hit-est-2d

## Outputs

- Trained sequence-only 1D CNN ablation model
- Trained secondary-structure-only 1D CNN ablation model
- Comparison table of accuracy, F1, and sensitivity per ablation vs. dual-input model on held-out test folds

## How to apply

Load the preprocessed Ψ benchmark fragments (Ψ plus 25 flanking bases each side, i.e. 51 bp) from RMBase v2.0 for human (hg19), mouse (mm10), and yeast (sacCer3). Encode sequence as a one-of-K binary matrix of size 51 × 4 and secondary structure dot-bracket context (from RNAshapes) as a one-of-K binary matrix of size 51 × 6. Build a sequence-only 1D CNN (two conv layers, glorot_normal init, PRelu, dropout 66.5%/3.8%) and a structure-only 1D CNN (dropout 74.5%/36.9%), mirroring the architecture described in the paper. Fit both with SGD (learning rate 0.0137), binary cross-entropy loss, batch size 32, up to 500 epochs with early-stop patience 20 and model checkpointing, using bootstrapping bins where negative samples are divided into N bins each sized to the positive class, and the same 10-fold stratified 80:20 split with cd-hit-est-2d identity reduction as the dual model. Evaluate accuracy, F1, and sensitivity per ablation on held-out test folds and compare against the dual-input MU-PseUDeep results; the comparison table is the deliverable.

## Related tools

- **MU-PseUDeep** (Dual-input CNN baseline whose results the ablations are compared against) — https://github.com/smk5g5/MU-PseUDeep
- **Hyperas** (Hyperparameter optimization wrapper used to tune the CNN architectures (dropout rates, filter sizes) prior to ablation training) — https://github.com/maxpumperla/hyperas
- **Hyperopt** (Search backend used by Hyperas for optimizing hyperparameters of the ablation CNNs) — https://github.com/hyperopt/hyperopt

## Examples

```
best_run, best_model = optim.minimize(model=create_model, data=data, algo=tpe.suggest, max_evals=100, trials=Trials())  # Hyperas tuning of seq-only CNN
```

## Evaluation signals

- Dual-input model must improve over the sequence-only CNN by approximately 3–4% accuracy and F1 (up to 9% sensitivity) on the balanced dataset
- Sequence-only CNN must outperform the secondary-structure-only CNN
- On the imbalanced dataset the combined model should exceed the sequence CNN by ~2% accuracy, 2% F1, up to 4% MCC, and up to ~7% sensitivity
- Ablation models must use identical training protocol (SGD lr 0.0137, batch size 32, early-stop patience 20, bootstrapping bins) and identical data folds as the dual-input model
- Encodings must have correct dimensions: 51 × 4 for sequence, 51 × 6 for structure

## Limitations

- Direct comparison with iPseU-CNN is impossible because no source code or webserver was available for that approach
- Secondary-structure context is predicted (RNAshapes), not experimentally determined, so ablation gains reflect predicted structure quality
- Results are specific to Ψ site prediction on 51-bp fragments; ablation conclusions may not transfer to other modifications or fragment lengths

## Evidence

- [results] Balanced-dataset improvement of dual model over sequence CNN: "The improvement was by 3–4% for accuracy and F1 and up to 9% for sensitivity in the balanced dataset in comparison to sequence CNN"
- [methods] One-of-K encoding of sequence input: "Sequence data was converted into a one-of-K encoding binary matrix of size 51 × 4"
- [methods] Dual CNN architecture basis for ablations: "For each input (sequence and secondary structure), a pair of 1D CNN was used"
- [methods] Hyperparameter tuning of the CNNs: "A hyperparameter optimization of various hyperparameters was carried out using Hyperas"
- [results] Imbalanced-dataset comparison results: "our combined model outperformed the sequence CNN with a 2% accuracy. The improvement was also 2% for F1, up to 4% for MCC, and up to almost 7% for sensitivity"
