---
name: hyperparameter-optimization-deep-learning
description: Use when training a Keras CNN (e.g., for RNA modification site prediction)
  and you need to search dropout rates, kernel regularizer weights, optimizer choice,
  and learning rate to maximize validation F1-score.
when_to_use_negative:
- The model is a classical ML method (e.g., SVM or XGBoost like iRNA-PseU or XG-PseU
  baselines) rather than a Keras neural network, since Hyperas only wraps Keras models
- No validation data exists to compute F1-score, making the TPE objective undefined
edam_operation: http://edamontology.org/operation_3893
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3307
tools:
- name: Hyperas
  role: Convenience wrapper that lets the Keras model be written with double-curly-bracket
    hyperparameter templates and drives the optimization loop
  repo: https://github.com/maxpumperla/hyperas
- name: Hyperopt
  role: Provides the tree-structured Parzen estimator (tpe.suggest) search algorithm
    and Trials bookkeeping used to maximize validation F1-score
  repo: https://github.com/hyperopt/hyperopt
- name: MU-PseUDeep
  role: The two-branch 1D CNN (sequence and secondary-structure inputs) whose dropout,
    regularizer, optimizer, and learning rate hyperparameters are optimized
  repo: https://github.com/smk5g5/MU-PseUDeep
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:4dee80d92a121d859e51664ce49722b3b57d2ddc09a98689d56fec1ab59033e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maxpumperla/hyperas
  tools:
  - Hyperas
  - Hyperopt
  - MU-PseUDeep
  - PseUI
  - XG-PseU
  - iRNA-PseU
  edam_operation: http://edamontology.org/operation_3893
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# hyperparameter-optimization-deep-learning

## Summary

Optimize Keras deep learning model hyperparameters (dropout rates, L2 kernel regularizer weights, optimizer, learning rate) with Hyperas/Hyperopt using a tree-structured Parzen estimator that maximizes F1-score on validation data. This is the approach used to tune MU-PseUDeep's two-branch 1D CNN for pseudouridine site prediction.

## When to use

Use when training a Keras CNN (e.g., for RNA modification site prediction) and you need to search dropout rates, kernel regularizer weights, optimizer choice, and learning rate to maximize validation F1-score.

## When NOT to use

- The model is a classical ML method (e.g., SVM or XGBoost like iRNA-PseU or XG-PseU baselines) rather than a Keras neural network, since Hyperas only wraps Keras models
- No validation data exists to compute F1-score, making the TPE objective undefined

## Inputs

- Keras model definition with hyperparameter ranges templated for Hyperas
- One-of-K encoded sequence matrices (51 x 4) and secondary-structure matrices (51 x 6) from EDeN sub-shape contexts
- Single-fold stratified train/validation split (80:20, 10-fold design)

## Outputs

- Best hyperparameter configuration (dropout rates, kernel regularizer weights, optimizer, learning rate, e.g., SGD lr=0.0137)
- Trained Keras model checkpoint (best F1 on validation)
- Per-fold accuracy, F1-score, and MCC averaged over bootstrap iterations

## How to apply

Define the Keras model as a template function with hyperparameter ranges in Hyperas double-bracket notation (e.g., {{uniform(0,1)}} for dropout probabilities, {{choice(['rmsprop','adam','sgd'])}} for optimizer). Run optim.minimize with algo=tpe.suggest (tree-structured Parzen estimator) from Hyperopt, minimizing negative F1-score on validation data. In the MU-PseUDeep case, the search covered per-layer dropout rates and L2 kernel regularizer weights, the optimizer, and the SGD learning rate, and was performed on a single fold before applying the best configuration across all 10 folds. The tuned model (SGD learning rate 0.0137, batch size 32, 500 epochs with early-stop patience of 20, glorot_normal initialization, PReLU activations, binary cross-entropy loss) was then evaluated with accuracy, F1, and MCC averaged over bootstrap iterations and folds.

## Related tools

- **Hyperas** (Convenience wrapper that lets the Keras model be written with double-curly-bracket hyperparameter templates and drives the optimization loop) — https://github.com/maxpumperla/hyperas
- **Hyperopt** (Provides the tree-structured Parzen estimator (tpe.suggest) search algorithm and Trials bookkeeping used to maximize validation F1-score) — https://github.com/hyperopt/hyperopt
- **MU-PseUDeep** (The two-branch 1D CNN (sequence and secondary-structure inputs) whose dropout, regularizer, optimizer, and learning rate hyperparameters are optimized) — https://github.com/smk5g5/MU-PseUDeep

## Examples

```
from hyperopt import Trials, STATUS_OK, tpe; best_run, best_model = optim.minimize(model=create_model, data=data, algo=tpe.suggest, max_evals=10, trials=Trials())
```

## Evaluation signals

- Validation F1-score of the tuned configuration should exceed the untuned baseline on the same fold
- The reported best SGD learning rate should be reproducible near 0.0137 with early-stop patience 20 and batch size 32
- Final test metrics should match published values, e.g., human balanced accuracy of 0.726 ± 0.020 and imbalanced accuracy of 0.894 ± 0.011
- Tuned model should outperform iRNA-PseU, PseUI, and XG-PseU baselines on the identical train/test splits

## Limitations

- Hyperparameter optimization is performed for a single fold only, then transferred to all folds; per-fold re-tuning is not done
- Hyperopt minimizes whatever loss metric is supplied, so F1 must be negated ('loss': -f1) in the objective function
- Direct comparison with iPseU-CNN is impossible because no source code or webserver was available

## Evidence

- [methods] Hyperparameter optimization: "A hyperparameter optimization of various hyperparameters was carried out using Hyperas"
- [readme] Hyperas README purpose: [evidence span withheld — anchor status: external_doc]
- [methods] Optimized hyperparameters: [evidence span withheld — anchor status: not_found]
- [methods] Model training configuration: [evidence span withheld — anchor status: not_found]
