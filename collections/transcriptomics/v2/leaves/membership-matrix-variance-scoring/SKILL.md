---
name: membership-matrix-variance-scoring
description: Use when benchmarking condition-associated key gene (CKG) detection on
  multi-condition scRNA-seq data and a baseline ranking from an NMF-based method with
  known condition-specific modules is needed, or when ground-truth CKG labels allow
  AUPRC/early-precision comparison.
when_to_use_negative:
- When batch effects are confounded with condition effects and no batch-aware disentanglement
  is possible — scINSIGHT factorizes expression matrices without explicit batch-effect
  modeling
- When no ground-truth or validation set exists to compare the resulting gene ranking
  against an alternative method's scores
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scINSIGHT
  role: Provides the non-negative matrix factorization whose membership matrix row
    variances form the baseline condition-specific gene scores
  repo: https://github.com/Vivianstats/scINSIGHT
- name: scDisInFact
  role: Primary method being benchmarked; its per-gene CKG scores (l2-norms of first-layer
    weights of unshared-bio encoders) are compared against this baseline ranking
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: Wilcoxon rank-sum test
  role: Second baseline; per-gene two-sided rank-sum p-values transformed as 1 - pval/max(pval)
    form the comparison gene ranking
- name: pandas
  role: Assembles AUPRC, Early Precision, and Pearson correlation metrics for all
    methods and condition types into a tidy CSV table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:26045d6b92779a2c0104dc3ec74a07af796cbb7ebb31e51dd94b8cff25f5a5f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scINSIGHT
  tools:
  - Wilcoxon rank-sum test
  - numpy
  - pandas
  - scDisInFact
  - scINSIGHT
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# membership-matrix-variance-scoring

## Summary

A baseline gene-ranking method that uses scINSIGHT's non-negative matrix factorization to identify condition-specific gene modules, then scores each gene by the variance of its membership (loadings) across those modules. It serves as the baseline against which scDisInFact's per-gene CKG scores are benchmarked by AUPRC on simulated data with known ground-truth condition-associated key genes.

## When to use

Use when benchmarking condition-associated key gene (CKG) detection on multi-condition scRNA-seq data and a baseline ranking from an NMF-based method with known condition-specific modules is needed, or when ground-truth CKG labels allow AUPRC/early-precision comparison.

## When NOT to use

- When batch effects are confounded with condition effects and no batch-aware disentanglement is possible — scINSIGHT factorizes expression matrices without explicit batch-effect modeling
- When no ground-truth or validation set exists to compare the resulting gene ranking against an alternative method's scores

## Inputs

- Multi-condition single-cell RNA-seq gene expression matrices (e.g., simulated count matrices with known ground-truth condition-associated key genes)
- Condition group labels defining the biological condition of each sample/cell matrix

## Outputs

- A per-gene condition-specific score vector derived from row variances of the scINSIGHT non-negative membership matrix
- A ranked gene list evaluated against ground-truth CKGs via AUPRC, Early Precision, and Pearson correlation (assembled into a tidy metrics table such as ckg_benchmark_metrics.csv)

## How to apply

Run scINSIGHT (installed from CRAN with install.packages('scINSIGHT')) on the same multi-condition single-cell gene expression samples used by the primary method. scINSIGHT factorizes the scRNA-seq matrices into common and condition-specific modules via non-negative matrix factorization, yielding a non-negative membership matrix whose entries quantify each gene's loading in each module. For each gene, compute the variance across its row of the membership matrix restricted to the condition-specific modules; this variance is the condition-specific gene score. Rank genes in decreasing score order and compare against ground-truth CKG sets (e.g., simulated condition-associated genes) using AUPRC, Early Precision, and Pearson correlation, alongside scDisInFact's model.extract_gene_scores() output and a two-sided Wilcoxon rank-sum baseline whose p-values are transformed as 1 - pval/max(pval).

## Related tools

- **scINSIGHT** (Provides the non-negative matrix factorization whose membership matrix row variances form the baseline condition-specific gene scores) — https://github.com/Vivianstats/scINSIGHT
- **scDisInFact** (Primary method being benchmarked; its per-gene CKG scores (l2-norms of first-layer weights of unshared-bio encoders) are compared against this baseline ranking) — https://github.com/ZhangLabGT/scDisInFact
- **Wilcoxon rank-sum test** (Second baseline; per-gene two-sided rank-sum p-values transformed as 1 - pval/max(pval) form the comparison gene ranking)
- **pandas** (Assembles AUPRC, Early Precision, and Pearson correlation metrics for all methods and condition types into a tidy CSV table)

## Examples

```
install.packages("scINSIGHT")  # then factorize samples, score genes by row-wise variance of the non-negative membership matrix
```

## Evaluation signals

- The scINSIGHT run returns a non-negative membership matrix with rows equal to the number of genes (check row dimension and non-negativity of entries)
- The resulting gene ranking should be reproducible across repeated factorization runs on the same simulated dataset
- Benchmarking against known ground-truth CKGs should yield computable AUPRC and Early Precision values in [0, 1] for every method and condition type
- On the simulated datasets in the paper, scDisInFact's extract_gene_scores() ranking should outperform this membership-variance baseline across all values of the perturbation parameter (as the paper reports)
- The Wilcoxon baseline p-values must be transformed as 1 - pval/max(pval) before ranking to keep score direction consistent

## Limitations

- scINSIGHT factorizes expression matrices into common and condition-specific modules but does not explicitly model or remove batch effects, unlike scDisInFact's disentangled latent factors
- As a baseline, it was consistently outperformed by scDisInFact in CKG detection (AUPRC, Early Precision, Pearson correlation) across all simulated perturbation settings in the article

## Evidence

- [readme] scINSIGHT uses a novel matrix factorization model to jointly analyze multiple single-cell gene expression samples from biologically heterogeneous sources: [evidence span withheld — anchor status: external_doc]
- [intro] scINSIGHT factorizes matrices into common and condition-specific modules via NMF: "scINSIGHT14 factorizes the scRNA-seq matrices into common and condition-specific modules using non-negative matrix factorization"
- [readme] Gene modules are sparse non-negative linear combinations; cells have common and condition-specific module expression: [evidence span withheld — anchor status: external_doc]
- [methods] scDisInFact's extract_gene_scores supplies the competing per-gene CKG scores: "gene_scores = model.extract_gene_scores()"
