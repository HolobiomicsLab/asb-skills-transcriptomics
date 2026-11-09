---
name: ranked-list-correlation-against-ground-truth
description: Use when simulated or controlled datasets have a known ground-truth set
  of condition-associated key genes and you must quantify how accurately competing
  gene-ranking methods recover them.
when_to_use_negative:
- Real datasets without a known ground-truth set of condition-associated key genes
  — AUPRC/Early Precision cannot be computed against truth
- Single-condition or single-batch datasets where batch and condition effects cannot
  be disentangled (scDisInFact needs multi-batch multi-condition input)
- Tasks other than gene-ranking evaluation, e.g., perturbation prediction or batch
  effect removal quality assessment
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDisInFact
  role: Produces per-gene CKG scores via extract_gene_scores() (l2-norms of first-layer
    unshared-bio encoder weights) for the evaluated ranking
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scINSIGHT
  role: 'Baseline gene ranking: genes ranked by variance of rows of the non-negative
    membership matrix (condition-specific modules)'
  repo: https://github.com/Vivianstats/scINSIGHT
- name: numpy
  role: Array handling for count matrices and score vectors during metric computation
- name: pandas
  role: Assembles per-method metrics into a tidy table (ckg_benchmark_metrics.csv)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:76944f5c8ff401e5a68f58d258e7bae3201d5bfe3632781701c02ce6a73d8508
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/scDisInFact
  tools:
  - numpy
  - pandas
  - scDisInFact
  - scINSIGHT
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ranked-list-correlation-against-ground-truth

## Summary

Benchmark protocol for condition-associated key gene (CKG) detection: rank genes with competing methods (scDisInFact gene scores, scINSIGHT membership variance, Wilcoxon rank-sum) and score each ranked list against known ground-truth CKG sets using AUPRC, Early Precision, and Pearson correlation.

## When to use

Use when simulated or controlled datasets have a known ground-truth set of condition-associated key genes and you must quantify how accurately competing gene-ranking methods recover them.

## When NOT to use

- Real datasets without a known ground-truth set of condition-associated key genes — AUPRC/Early Precision cannot be computed against truth
- Single-condition or single-batch datasets where batch and condition effects cannot be disentangled (scDisInFact needs multi-batch multi-condition input)
- Tasks other than gene-ranking evaluation, e.g., perturbation prediction or batch effect removal quality assessment

## Inputs

- Simulated scRNA-seq count matrices (numpy.array, shape (ncells, ngenes)) with known ground-truth condition-associated key gene sets
- Per-cell metadata (pandas.dataframe) with condition and batch columns (condition_key, batch_key)
- Ranked gene lists or per-gene score vectors from each method (scDisInFact extract_gene_scores, scINSIGHT membership-matrix row variances, Wilcoxon 1 - pval/max(pval) transformed p-values)

## Outputs

- Per-method, per-condition-type benchmark metrics: AUPRC, Early Precision, and Pearson correlation against ground-truth CKGs
- Tidy metrics table written to ckg_benchmark_metrics.csv

## How to apply

For each simulated count matrix, train scDisInFact (create_scdisinfact_dataset with condition_key and batch_key, train_model with nepochs=100) and call model.extract_gene_scores() to obtain per-gene scores (l2-norms of first-layer weights of each unshared-bio encoder) per condition type. Run scINSIGHT on the same data and rank genes by the variance of rows of the non-negative membership matrix; apply a two-sided Wilcoxon rank-sum test per gene and transform p-values as 1 - pval/max(pval) to form the baseline ranking. Compare each ranked gene list to the ground-truth CKG sets by computing AUPRC, Early Precision (early-recovery precision), and Pearson correlation between predicted score ranks and truth, then assemble all metrics into a tidy table (e.g., ckg_benchmark_metrics.csv). The rationale is that AUPRC handles the small fraction of true CKGs, while Early Precision and Pearson correlation confirm that the top of the ranking, not just the aggregate, is correctly ordered.

## Related tools

- **scDisInFact** (Produces per-gene CKG scores via extract_gene_scores() (l2-norms of first-layer unshared-bio encoder weights) for the evaluated ranking) — https://github.com/ZhangLabGT/scDisInFact
- **scINSIGHT** (Baseline gene ranking: genes ranked by variance of rows of the non-negative membership matrix (condition-specific modules)) — https://github.com/Vivianstats/scINSIGHT
- **numpy** (Array handling for count matrices and score vectors during metric computation)
- **pandas** (Assembles per-method metrics into a tidy table (ckg_benchmark_metrics.csv))

## Examples

```
from scDisInFact import scdisinfact, create_scdisinfact_dataset; data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key=["condition 1", "condition 2"], batch_key="batch"); losses = model.train_model(nepochs=100); gene_scores = model.extract_gene_scores()
```

## Evaluation signals

- scDisInFact AUPRC should consistently exceed scINSIGHT and Wilcoxon baseline AUPRC across all values of the perturbation parameter on simulated data
- Early Precision should confirm scDisInFact ranks true CKGs near the top of the list better than baselines
- Pearson correlation between method gene scores and ground-truth CKG signal should be highest for scDisInFact
- Output metrics table (ckg_benchmark_metrics.csv) must contain one row per method × condition type × perturbation-parameter value with non-degenerate (non-NaN) AUPRC values
- Per-gene score vectors must cover all genes of the simulated count matrix (length ngenes) with finite values

## Limitations

- Ground truth exists only in simulations; on real datasets CKG ranking accuracy cannot be directly verified against truth
- scDisInFact assumes condition and batch effects are independent and additive in latent space, which bounds the gene-score quality being benchmarked
- When each batch is measured under only one condition, batch and condition effects cannot be fully disentangled, degrading the scDisInFact ranking being evaluated

## Evidence

- [methods] model.extract_gene_scores(): "gene_scores = model.extract_gene_scores()"
- [abstract] We evaluate scDisInFact on both simulated and real datasets, and compare its performance with baseline methods for each task.: "We evaluate scDisInFact on both simulated and real datasets, and compare its performance with baseline methods for each task."
- [intro] scINSIGHT factorizes matrices into common and condition-specific modules: "scINSIGHT14 factorizes the scRNA-seq matrices into common and condition-specific modules using non-negative matrix factorization"
- [methods] create_scdisinfact_dataset call signature: "data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key = ["condition 1", "condition 2"], batch_key = "batch")"
- [methods] Training with 100 epochs: "losses = model.train_model(nepochs = 100)"
- [readme] README installation of scINSIGHT from CRAN: "You can install `scINSIGHT` from
[CRAN](https://cran.r-project.org/web/packages/scINSIGHT/index.html)
with:"
