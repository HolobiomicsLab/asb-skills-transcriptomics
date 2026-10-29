---
name: unshared-encoder-weight-norm-extraction
description: Use when a trained scDisInFact model exists on multi-batch multi-condition
  scRNA-seq counts and per-gene rankings of condition-associated key genes are needed
  for one or more condition types.
when_to_use_negative:
- Single-batch, single-condition datasets where no condition effect can be disentangled
  from batch effect (each batch measured under only one condition)
- When the goal is batch effect removal or perturbation prediction rather than condition-associated
  key gene detection
- When condition types are strongly correlated — combine them into a single condition
  type before training, per the authors' discussion
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDisInFact
  role: Provides the trained conditional VAE whose unshared-bio encoder first-layer
    weights are L2-normalized to produce per-gene CKG scores via extract_gene_scores()
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scINSIGHT
  role: 'Baseline comparator: genes ranked by variance of non-negative membership
    matrix rows as condition-specific scores'
  repo: https://github.com/Vivianstats/scINSIGHT
- name: numpy
  role: Array handling for count matrices and score vectors
- name: pandas
  role: meta_cells dataframe with condition and batch columns for dataset construction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:2904fdd3bc7671ebc496734692a4adee5f100225afbcb743ed279987977117a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/scDisInFact
  tools:
  - numpy
  - pandas
  - scDisInFact
  - scINSIGHT
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# unshared-encoder-weight-norm-extraction

## Summary

Extracts per-gene condition-associated key gene (CKG) scores from a trained scDisInFact model by taking the L2-norms of the first-layer weights of each unshared-bio (condition-specific) encoder, one score vector per condition type.

## When to use

Use when a trained scDisInFact model exists on multi-batch multi-condition scRNA-seq counts and per-gene rankings of condition-associated key genes are needed for one or more condition types.

## When NOT to use

- Single-batch, single-condition datasets where no condition effect can be disentangled from batch effect (each batch measured under only one condition)
- When the goal is batch effect removal or perturbation prediction rather than condition-associated key gene detection
- When condition types are strongly correlated — combine them into a single condition type before training, per the authors' discussion

## Inputs

- Trained scDisInFact model (scdisinfact object with unshared-bio encoders)
- Count matrix (numpy.array of shape (ncells, ngenes)) used for training, to define gene order
- meta_cells pandas.dataframe with condition and batch columns

## Outputs

- gene_scores: per-condition-type vectors of per-gene CKG scores (L2-norms of unshared-bio encoder first-layer weights)
- Ranked gene lists per condition type for downstream benchmarking (AUPRC, Eprec, Pearson correlation)

## How to apply

First build the scdisinfact dataset with create_scdisinfact_dataset (condition_key for each condition type, batch_key for batch) and train the conditional variational autoencoder with model.train_model(nepochs=100), declaring Ks latent dimensions for shared-bio and each unshared-bio factor. Then call model.extract_gene_scores(), which computes the L2-norm of the first-layer weights of each unshared-bio encoder, yielding one gene-score vector per condition type (genes_scores[0], genes_scores[1], ...). Rank genes by these scores to prioritize candidate CKGs; in benchmarks the ranking is evaluated against ground-truth CKG sets with AUPRC, Early Precision, and Pearson correlation. Correct application means each returned vector aligns with the gene order of the input count matrix and with the corresponding condition type.

## Related tools

- **scDisInFact** (Provides the trained conditional VAE whose unshared-bio encoder first-layer weights are L2-normalized to produce per-gene CKG scores via extract_gene_scores()) — https://github.com/ZhangLabGT/scDisInFact
- **scINSIGHT** (Baseline comparator: genes ranked by variance of non-negative membership matrix rows as condition-specific scores) — https://github.com/Vivianstats/scINSIGHT
- **numpy** (Array handling for count matrices and score vectors)
- **pandas** (meta_cells dataframe with condition and batch columns for dataset construction)

## Examples

```
from scDisInFact import scdisinfact, create_scdisinfact_dataset; data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key=["condition 1", "condition 2"], batch_key="batch"); model.train_model(nepochs=100); gene_scores = model.extract_gene_scores()
```

## Evaluation signals

- gene_scores contains one score vector per condition type (index 0 for condition type 1, index 1 for condition type 2)
- Score vector length equals the number of genes in the training count matrix
- Against simulated data with known ground-truth CKGs, scDisInFact scores should achieve higher AUPRC than scINSIGHT membership-variance and Wilcoxon (1 - pval/max(pval)) rankings across all perturbation parameter values
- Model training loss converges after train_model(nepochs=100) before score extraction

## Limitations

- When each batch is measured under only one condition, there is not enough information to fully disentangle batch and condition effect, undermining score validity
- scDisInFact assumes condition types are mutually independent, which does not always hold in real data
- Condition effect is assumed independent of batch effect and batch effects consistent across conditions; additivity in latent space limits modeling of higher-order effects

## Evidence

- [methods] Call model.extract_gene_scores() to obtain scDisInFact per-gene scores (l2-norms of the first-layer weights of each unshared-bio encoder) for each condition type.: [evidence span withheld — anchor status: not_found]
- [readme] Key gene detection via extract_gene_scores in the README usage.: "gene_scores = model.extract_gene_scores()"
- [methods] Losses = model.train_model(nepochs = 100) training step.: "losses = model.train_model(nepochs = 100)"
- [methods] Training data construction with condition and batch keys.: "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
- [discussion] Discussion limitation on disentanglement when batches and conditions are confounded.: "when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect."
