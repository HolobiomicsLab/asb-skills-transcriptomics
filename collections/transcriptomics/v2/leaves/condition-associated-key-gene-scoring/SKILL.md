---
name: condition-associated-key-gene-scoring
description: Use when you have multi-batch, multi-condition scRNA-seq count matrices
  and need per-gene scores ranking condition-associated key genes, especially with
  known ground-truth CKGs to validate via AUPRC/Eprec.
when_to_use_negative:
- When each batch of cells is measured under only one condition — the paper states
  there is not enough information to fully disentangle the batch and condition effect
- When the goal is perturbation prediction or batch effect removal rather than CKG
  detection (use model.predict_counts or shared-bio factor extraction instead)
- When condition types are strongly correlated without a combined condition type,
  since scDisInFact assumes condition types are mutually independent
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scDisInFact
  role: 'Primary method: create_scdisinfact_dataset and train_model build the disentangled
    model; extract_gene_scores produces per-gene CKG scores per condition type'
  repo: github.com/ZhangLabGT/scDisInFact
- name: scINSIGHT
  role: 'Baseline: rank genes by the variance of the non-negative membership matrix
    rows as the condition-specific score'
  repo: https://github.com/Vivianstats/scINSIGHT
- name: numpy
  role: Holds the input count matrix in the (ncells, ngenes) array format required
    by create_scdisinfact_dataset
- name: pandas
  role: Holds meta_cells with condition and batch columns used to build the scdisinfact
    dataset and assemble tidy benchmark metric tables
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:af7f87aa438f18ecf22dc40a0df7df3e50accf0f179b7d1231099d77f9b223ef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - numpy
  - pandas
  - scDisInFact
  - scINSIGHT
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# condition-associated-key-gene-scoring

## Summary

Ranks genes by their association with biological conditions in multi-batch multi-condition scRNA-seq data by extracting per-gene condition-associated key gene (CKG) scores from scDisInFact's unshared-bio encoders, and benchmarks the rankings against scINSIGHT and Wilcoxon rank-sum baselines using AUPRC, Early Precision, and Pearson correlation.

## When to use

Use when you have multi-batch, multi-condition scRNA-seq count matrices and need per-gene scores ranking condition-associated key genes, especially with known ground-truth CKGs to validate via AUPRC/Eprec.

## When NOT to use

- When each batch of cells is measured under only one condition — the paper states there is not enough information to fully disentangle the batch and condition effect
- When the goal is perturbation prediction or batch effect removal rather than CKG detection (use model.predict_counts or shared-bio factor extraction instead)
- When condition types are strongly correlated without a combined condition type, since scDisInFact assumes condition types are mutually independent

## Inputs

- Simulated or real scRNA-seq count matrix (numpy.array, shape (ncells, ngenes))
- meta_cells pandas.dataframe (shape (ncells, xx)) with condition columns for each condition type and a batch ID column
- Ground-truth condition-associated key gene sets (for simulated benchmarking)

## Outputs

- Per-gene CKG score vectors from model.extract_gene_scores(), one per condition type (l2-norms of unshared-bio encoder first-layer weights)
- scINSIGHT gene rankings from variance of the non-negative membership matrix rows
- Wilcoxon baseline gene rankings (1 - pval/max(pval))
- Tidy benchmark metrics table (AUPRC, Early Precision, Pearson correlation) written to ckg_benchmark_metrics.csv

## How to apply

Build a scdisinfact dataset from the count matrix (numpy.array, shape (ncells, ngenes)) and meta_cells dataframe with create_scdisinfact_dataset, passing condition_key for each condition type and batch_key for batch, then train with train_model(nepochs=100) on an available GPU/CPU device. Call model.extract_gene_scores(), which returns per-gene scores computed as l2-norms of the first-layer weights of each unshared-bio encoder, one score vector per condition type. For benchmarking, run scINSIGHT on the same matrices and rank genes by the variance of rows of its non-negative membership matrix, and run two-sided Wilcoxon rank-sum tests per gene transforming p-values as 1 - pval/max(pval). Compare each ranked list to the ground-truth CKG set by computing AUPRC, Early Precision, and Pearson correlation, and assemble all methods and condition types into a tidy table (e.g. ckg_benchmark_metrics.csv).

## Related tools

- **scDisInFact** (Primary method: create_scdisinfact_dataset and train_model build the disentangled model; extract_gene_scores produces per-gene CKG scores per condition type) — github.com/ZhangLabGT/scDisInFact
- **scINSIGHT** (Baseline: rank genes by the variance of the non-negative membership matrix rows as the condition-specific score) — https://github.com/Vivianstats/scINSIGHT
- **numpy** (Holds the input count matrix in the (ncells, ngenes) array format required by create_scdisinfact_dataset)
- **pandas** (Holds meta_cells with condition and batch columns used to build the scdisinfact dataset and assemble tidy benchmark metric tables)

## Examples

```
gene_scores = model.extract_gene_scores(); scores_cond1 = gene_scores[0]
```

## Evaluation signals

- scDisInFact AUPRC against the ground-truth CKG set should exceed scINSIGHT and Wilcoxon baseline AUPRC across all values of the perturbation parameter
- Early Precision of the scDisInFact ranking should confirm superior placement of true CKGs at the top of the ranked list
- Pearson correlation between predicted scores and ground-truth per-gene effect sizes should be highest for scDisInFact
- extract_gene_scores must return one score vector per condition type (e.g. gene_scores[0] and gene_scores[1] for two condition types)
- All method × condition-type metrics assemble without NaNs into the tidy ckg_benchmark_metrics.csv table

## Limitations

- When each batch is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect
- scDisInFact assumes condition types are mutually independent, which does not always hold for real-world datasets
- The method assumes condition effect is independent of batch effect, batch effects are consistent across conditions, and effects are additive in latent space, limiting modeling of higher-order effects
- Testing datasets used in the manuscript are not included in the repository due to size and are available only upon request

## Evidence

- [methods] extract_gene_scores returns l2-norms of unshared-bio encoder weights: [evidence span withheld — anchor status: not_found]
- [readme] create_scdisinfact_dataset invocation with condition and batch keys: "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
- [methods] Training configuration for the scdisinfact model: "losses = model.train_model(nepochs = 100)"
- [abstract] Disentanglement rationale for CKG detection: "scDisInFact learns latent factors that disentangle condition effect from batch effect, enabling it to simultaneously perform three tasks: batch effect removal, condition-associated key gene"
- [discussion] Limitation on disentangling when batches confound conditions: "when each batch of cells is measured under only one condition, there is not enough information to fully disentangle the batch and condition effect."
