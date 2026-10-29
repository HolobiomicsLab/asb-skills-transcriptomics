---
name: holdout-batch-prediction-scoring
description: Use when a multi-batch multi-condition scRNA-seq dataset has one matrix
  held out as a test set and you must compare methods that predict the unseen condition/batch
  combination using per-cell-type prediction accuracy.
when_to_use_negative:
- Each batch of cells was measured under only one condition — there is not enough
  information to fully disentangle the batch and condition effect, so held-out batch
  predictions are not well-posed
- The goal is batch effect removal or condition-associated key gene detection rather
  than perturbation prediction accuracy comparison against baselines
- Baselines cannot be trained on identical training matrices, so the accuracy comparison
  would not be fair
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scDisInFact
  role: Batch-aware perturbation-prediction method; creates the scdisinfact dataset,
    trains the conditional VAE, and generates held-out condition/batch predictions
    via predict_counts
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scGen
  role: Perturbation-prediction baseline trained on the same matrices without batch-effect
    modeling
- name: scPreGAN
  role: Perturbation-prediction baseline trained on the same matrices without batch-effect
    modeling
  repo: https://github.com/XiajieWei/scPreGAN-reproducibility
- name: numpy
  role: Holds the input count matrix as a numpy.array of shape (ncells, ngenes)
- name: pandas
  role: Holds meta_cells with condition and batch key columns aligned to the count
    matrix rows
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:227ce373ea0f9f4eadc9ff58d2d78b7aeae2bdb4948ef69006ac420550ac1fd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/scDisInFact
  tools:
  - numpy
  - pandas
  - scDisInFact
  - scGen
  - scPreGAN
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# holdout-batch-prediction-scoring

## Summary

A benchmarking skill for perturbation prediction on multi-batch scRNA-seq data: hold out one count matrix (e.g., GBM matrix PW034-705), train scDisInFact against scGen and scPreGAN baselines, and score held-out condition/batch predictions with cell-type-specific accuracy metrics (MSE, Pearson, R2).

## When to use

Use when a multi-batch multi-condition scRNA-seq dataset has one matrix held out as a test set and you must compare methods that predict the unseen condition/batch combination using per-cell-type prediction accuracy.

## When NOT to use

- Each batch of cells was measured under only one condition — there is not enough information to fully disentangle the batch and condition effect, so held-out batch predictions are not well-posed
- The goal is batch effect removal or condition-associated key gene detection rather than perturbation prediction accuracy comparison against baselines
- Baselines cannot be trained on identical training matrices, so the accuracy comparison would not be fair

## Inputs

- GBM scRNA-seq count matrices (numpy arrays, shape (ncells, ngenes)) with one matrix (PW034-705) held out as test
- Cell metadata pandas dataframe (ncells, xx) with condition-key and batch-key columns matching the count matrices
- Trained scDisInFact model, plus trained scGen and scPreGAN baseline models on the same training matrices

## Outputs

- Held-out predicted count matrices for the unseen condition/batch combination under within-batch and cross-batch scenarios
- prediction_scores_gbm.csv: cell-type-specific (Myeloid, Oligodendrocytes, tumor cells) MSE, Pearson, and R2 scores for scDisInFact, scGen, and scPreGAN

## How to apply

Load the GBM scRNA-seq count matrices and cell metadata, holding out the PW034-705 matrix as the test set while the remaining matrices form the training set. Build the training data with scDisInFact's create_scdisinfact_dataset (count array of shape (ncells, ngenes) plus a pandas meta_cells dataframe with condition and batch key columns), then train with model.train_model(nepochs = 100) on a GPU device (torch.device("cuda:0")). Train scGen and scPreGAN on the identical training matrices without batch-effect modeling so the comparison isolates the value of batch-aware prediction. Generate held-out predictions with model.predict_counts twice: once predicting the condition effect only (within-batch scenario) and once predicting condition plus batch effect (cross-batch scenario, e.g., predict_conds plus a predict_batch argument). Score each method's predictions separately per cell type (Myeloid, Oligodendrocytes, tumor cells) with MSE, Pearson correlation, and R2, and write the results to prediction_scores_gbm.csv; the expected outcome is that scDisInFact outperforms scGen and scPreGAN, with the improvement most pronounced in the cross-batch setting as in Fig 4d.

## Related tools

- **scDisInFact** (Batch-aware perturbation-prediction method; creates the scdisinfact dataset, trains the conditional VAE, and generates held-out condition/batch predictions via predict_counts) — https://github.com/ZhangLabGT/scDisInFact
- **scGen** (Perturbation-prediction baseline trained on the same matrices without batch-effect modeling)
- **scPreGAN** (Perturbation-prediction baseline trained on the same matrices without batch-effect modeling) — https://github.com/XiajieWei/scPreGAN-reproducibility
- **numpy** (Holds the input count matrix as a numpy.array of shape (ncells, ngenes))
- **pandas** (Holds meta_cells with condition and batch key columns aligned to the count matrix rows)

## Examples

```
from scDisInFact import scdisinfact, create_scdisinfact_dataset; data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key=["condition"], batch_key="batch"); model = scdisinfact(data_dict=data_dict, Ks=[8,2], device=torch.device("cuda:0")); losses = model.train_model(nepochs=100)
```

## Evaluation signals

- prediction_scores_gbm.csv contains rows for all three cell types (Myeloid, Oligodendrocytes, tumor cells) under both the within-batch and cross-batch scenarios
- scDisInFact's scores reproduce the relative ranking of Fig 4d: outperforming scGen and scPreGAN in the cross-batch setting
- The improvement over baselines is more pronounced when batch effect exists between the input and held-out count matrices
- Baseline scGen/scPreGAN models were trained on exactly the same training matrices as scDisInFact, ensuring a controlled comparison
- Both prediction scenarios are generated: condition effect only (within-batch) and condition plus batch effect (cross-batch) via predict_counts

## Limitations

- When each batch is measured under only one condition, there is insufficient information to fully disentangle batch and condition effects, weakening held-out batch prediction
- scDisInFact assumes condition types are mutually independent, condition effect is independent of batch effect, batch effects are consistent across conditions, and effects are additive in latent space — violations limit higher-order condition/batch modeling
- Testing datasets from the manuscript (including the GBM data) are not included in the repository due to size and are available only upon request; only demo_data ships with the repo

## Evidence

- [intro] Existing methods for this task, such as scGen17 and scPreGAN18, do not account for batch effects between data matrices: "Existing methods for this task, such as scGen17 and scPreGAN18, do not account for batch effects between data matrices"
- [methods] predict_counts usage: "counts_predict = model.predict_counts(input_counts = counts_input, meta_cells = meta_input, condition_keys = ["condition 1", "condition 2"]"
- [methods] training call: "losses = model.train_model(nepochs = 100)"
- [methods] dataset creation: "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
- [discussion] README limitation on one-condition-per-batch data: "when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect."
- [readme] README input format: "Given the input count matrix `counts` (`numpy.array` of the shape `(ncells, ngenes)`) and meta data `meta_cells` (`pandas.dataframe` of the shape `(ncells, xx)`)"
