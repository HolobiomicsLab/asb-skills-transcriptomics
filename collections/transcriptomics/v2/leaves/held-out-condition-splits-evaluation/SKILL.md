---
name: held-out-condition-splits-evaluation
description: Use when evaluating whether a multi-condition scRNA-seq model predicts
  held-out condition combinations (e.g. <ctrl,severe> cells) correctly, comparing
  against baselines like scGen and scPreGAN.
when_to_use_negative:
- Each batch was measured under only one condition — there is not enough information
  to disentangle batch and condition effects, so held-out cross-batch condition prediction
  is ill-posed
- The task is unsupervised batch correction or key gene detection rather than quantitative
  evaluation of perturbation prediction against gold-standard counts
- Datasets contain highly correlated condition types without redefining them as a
  combined condition type, violating scDisInFact's mutual-independence assumption
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scDisInFact
  role: Primary method under evaluation; predicts counts for target condition/batch
    combinations via predict_counts in in-sample and out-of-sample scenario splits
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scGen
  role: Baseline perturbation-prediction method trained on identical scenario splits
    to produce comparison count matrices
- name: scPreGAN
  role: Baseline perturbation-prediction method trained on identical scenario splits
    to produce comparison count matrices
  repo: https://github.com/XiajieWei/scPreGAN-reproducibility
- name: numpy
  role: Holds count matrices of shape (ncells, ngenes) and computes MSE and correlation
    metrics per cell type
- name: pandas
  role: Stores meta_cells with condition/batch columns and assembles metrics into
    prediction_metrics.csv
- name: PyTorch
  role: Provides the training device (torch.device("cuda:0" if available)) on which
    the scdisinfact model is instantiated and trained
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:390ca0c3f75d6bfa7a02dfc28639873d215815afc97b0bbdf1baea16278ed79d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/scDisInFact
  tools:
  - PyTorch
  - numpy
  - pandas
  - scDisInFact
  - scGen
  - scPreGAN
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# held-out-condition-splits-evaluation

## Summary

Benchmark perturbation-prediction methods by holding out specific condition-effect scenarios (in-sample and out-of-sample, within-batch and cross-batch) and scoring predicted counts against gold standards with cell-type-specific MSE, Pearson correlation, and R2.

## When to use

Use when evaluating whether a multi-condition scRNA-seq model predicts held-out condition combinations (e.g. <ctrl,severe> cells) correctly, comparing against baselines like scGen and scPreGAN.

## When NOT to use

- Each batch was measured under only one condition — there is not enough information to disentangle batch and condition effects, so held-out cross-batch condition prediction is ill-posed
- The task is unsupervised batch correction or key gene detection rather than quantitative evaluation of perturbation prediction against gold-standard counts
- Datasets contain highly correlated condition types without redefining them as a combined condition type, violating scDisInFact's mutual-independence assumption

## Inputs

- Simulated multi-batch multi-condition scRNA-seq count matrix (numpy.array, shape (ncells, ngenes))
- Cell metadata pandas.DataFrame with condition 1, condition 2, and batch columns and cell-type labels
- Gold-standard counts for held-out condition/batch combinations
- Scenario split definitions (in-sample vs out-of-sample; within-batch vs cross-batch for treatment, severity, both)

## Outputs

- Per-scenario, per-method, cell-type-specific MSE, Pearson correlation, and R2 values
- prediction_metrics.csv aggregating metrics across all scenarios and methods (reproducing Fig. 3a,b)

## How to apply

Construct scenario splits spanning within-batch and cross-batch prediction for each condition type (treatment, severity, both) in an in-sample setting, plus an out-of-sample setting where the <ctrl,severe> cell subset is held out entirely from training. Train scDisInFact on the non-held-out data with create_scdisinfact_dataset(condition_key=["condition 1","condition 2"], batch_key="batch"), latent dimensions Ks=[8,2,2], and train_model(nepochs=100), then call model.predict_counts with predict_conds and predict_batch set to the target condition/batch combination. Train scGen and scPreGAN baselines on the same scenario splits and generate predicted count matrices for each scenario. For each method and scenario, compute cell-type-specific MSE, Pearson correlation, and R2 between predicted counts and gold-standard counts, then assemble into prediction_metrics.csv mirroring the article's Fig. 3a,b comparison.

## Related tools

- **scDisInFact** (Primary method under evaluation; predicts counts for target condition/batch combinations via predict_counts in in-sample and out-of-sample scenario splits) — https://github.com/ZhangLabGT/scDisInFact
- **scGen** (Baseline perturbation-prediction method trained on identical scenario splits to produce comparison count matrices)
- **scPreGAN** (Baseline perturbation-prediction method trained on identical scenario splits to produce comparison count matrices) — https://github.com/XiajieWei/scPreGAN-reproducibility
- **numpy** (Holds count matrices of shape (ncells, ngenes) and computes MSE and correlation metrics per cell type)
- **pandas** (Stores meta_cells with condition/batch columns and assembles metrics into prediction_metrics.csv)
- **PyTorch** (Provides the training device (torch.device("cuda:0" if available)) on which the scdisinfact model is instantiated and trained)

## Examples

```
from scDisInFact import scdisinfact, create_scdisinfact_dataset; data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key=["condition 1","condition 2"], batch_key="batch"); model = scdisinfact(data_dict=data_dict, Ks=[8,2,2], device=torch.device("cuda:0")); model.train_model(nepochs=100); counts_predict = model.predict_counts(input_counts=counts_input, meta_cells=meta_input, condition_keys=["condition 1","condition 2"], batch_key="batch", predict_conds=["ctrl","severe"], predict_batch=0)
```

## Evaluation signals

- scDisInFact should achieve the smallest cell-type-specific MSE and the highest Pearson correlation and R2 among the three methods in all in-sample (Fig. 3a) and out-of-sample (Fig. 3b) scenarios
- Metric coverage check: 6 scenarios (within-batch and cross-batch × treatment, severity, both) × 3 methods × 3 metrics are all present in prediction_metrics.csv
- Held-out cells (e.g. <ctrl,severe> subset) must be absent from training data in out-of-sample runs; leakage would invalidate the comparison
- Predicted count matrices must match the gold-standard counts' shape (ncells, ngenes) for each scenario before metric computation
- Demo pipeline runtime sanity: the scDisInFact demo takes ~15 minutes on a server GPU (Nvidia A40) with nepochs=100

## Limitations

- When each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect
- scDisInFact assumes condition types are mutually independent, which does not always hold for real-world datasets
- scDisInFact assumes condition effect is independent of batch effect, batch effects are consistent across conditions, and effects are additive in latent space, limiting modeling of higher-order product conditions or batch effects
- Testing datasets from the manuscript are not included in the repository due to size and are available only upon request

## Evidence

- [other] Out-of-sample <ctrl,severe> held-out subset evaluation: [evidence span withheld — anchor status: not_found]
- [other] Fig 3a,b comparison reproduced via prediction_metrics.csv: [evidence span withheld — anchor status: not_found]
- [readme] predict_counts API for perturbation prediction: "counts_predict = model.predict_counts(input_counts = counts_input, meta_cells = meta_input, condition_keys = ["condition 1", "condition 2"]"
- [abstract] scDisInFact outperforms baselines on perturbation prediction: "Our results demonstrate that scDisInFact outperforms existing methods that focus on individual tasks"
- [intro] Held-out condition matrices define the perturbation prediction task: "predicting unseen data matrices corresponding to certain conditions (the matrices with dashed borders in Fig. 1a (right)), also known as the task of perturbation prediction"
