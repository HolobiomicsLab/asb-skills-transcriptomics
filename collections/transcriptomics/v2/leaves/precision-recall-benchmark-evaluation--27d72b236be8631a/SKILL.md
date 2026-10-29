---
name: precision-recall-benchmark-evaluation
description: Use when ranked per-gene scores from competing methods must be compared
  against a known ground-truth gene set (simulated or curated) to quantify ranking
  accuracy via AUPRC and early-precision metrics.
when_to_use_negative:
- No ground-truth CKG labels exist (e.g. purely real datasets without simulated or
  curated key-gene annotations), since AUPRC/Early Precision require true positives
- Methods output clusterings or latent factors only, with no per-gene score that can
  be ranked
- Evaluating perturbation prediction accuracy (count-level tasks) rather than gene-ranking
  accuracy
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDisInFact
  role: Source of per-gene CKG scores (l2-norms of first-layer weights of unshared-bio
    encoders) being benchmarked
  repo: https://github.com/ZhangLabGT/scDisInFact
- name: scINSIGHT
  role: Baseline method; genes ranked by variance of rows of the non-negative membership
    matrix
  repo: https://github.com/Vivianstats/scINSIGHT
- name: numpy
  role: Array handling for count matrices and score vectors during metric computation
- name: pandas
  role: Assembling per-method, per-condition metrics into a tidy table written to
    ckg_benchmark_metrics.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:b20c25d4c1366279cf825981c815ba819fbad632a10888f5b42454e299b26bd0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/scDisInFact
  tools:
  - numpy
  - pandas
  - scDisInFact
  - scINSIGHT
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# precision-recall-benchmark-evaluation

## Summary

Benchmarks ranked gene lists against known ground-truth condition-associated key gene (CKG) sets by computing AUPRC, Early Precision, and Pearson correlation, as done to show scDisInFact outperforms scINSIGHT and the Wilcoxon rank-sum test on simulated datasets.

## When to use

Use when ranked per-gene scores from competing methods must be compared against a known ground-truth gene set (simulated or curated) to quantify ranking accuracy via AUPRC and early-precision metrics.

## When NOT to use

- No ground-truth CKG labels exist (e.g. purely real datasets without simulated or curated key-gene annotations), since AUPRC/Early Precision require true positives
- Methods output clusterings or latent factors only, with no per-gene score that can be ranked
- Evaluating perturbation prediction accuracy (count-level tasks) rather than gene-ranking accuracy

## Inputs

- Simulated count matrices (numpy.array of shape (ncells, ngenes)) with known ground-truth condition-associated key gene sets
- Ranked per-gene score lists from scDisInFact extract_gene_scores() for each condition type
- scINSIGHT non-negative membership matrix rows (variance used as condition-specific score)
- Per-gene two-sided Wilcoxon rank-sum p-values between condition groups

## Outputs

- Tidy metrics table (e.g. ckg_benchmark_metrics.csv) with AUPRC, Early Precision, and Pearson correlation per method, condition type, and perturbation-parameter value

## How to apply

For each method, produce a ranked gene list: scDisInFact per-gene CKG scores come from model.extract_gene_scores() (l2-norms of first-layer weights of each unshared-bio encoder per condition type); scINSIGHT genes are ranked by the variance of rows of the non-negative membership matrix; Wilcoxon p-values are transformed as 1 - pval/max(pval) to form the baseline ranking. Compare each ranked list to the known ground-truth CKG sets of the simulations, computing AUPRC, Early Precision (early retrieval accuracy), and Pearson correlation between predicted and true score rankings. Sweep across all values of the simulation perturbation parameter to test robustness. Assemble metrics for all methods and condition types into a tidy table (e.g. ckg_benchmark_metrics.csv) with pandas. Correct application means the ranked-list metrics reproduce the expected ordering (scDisInFact > scINSIGHT, Wilcoxon) across the full perturbation-parameter range.

## Related tools

- **scDisInFact** (Source of per-gene CKG scores (l2-norms of first-layer weights of unshared-bio encoders) being benchmarked) — https://github.com/ZhangLabGT/scDisInFact
- **scINSIGHT** (Baseline method; genes ranked by variance of rows of the non-negative membership matrix) — https://github.com/Vivianstats/scINSIGHT
- **numpy** (Array handling for count matrices and score vectors during metric computation)
- **pandas** (Assembling per-method, per-condition metrics into a tidy table written to ckg_benchmark_metrics.csv)

## Examples

```
from scDisInFact import scdisinfact, create_scdisinfact_dataset; data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key=["condition 1", "condition 2"], batch_key="batch"); model.train_model(nepochs=100); gene_scores = model.extract_gene_scores()
```

## Evaluation signals

- scDisInFact AUPRC exceeds both scINSIGHT and Wilcoxon baselines across all simulated perturbation-parameter values
- Early Precision confirms scDisInFact retrieves true CKGs earlier in the ranked list than baselines
- Pearson correlation between predicted gene scores and ground-truth ranking supports the same method ordering
- Output CSV contains one row per (method, condition type, perturbation value) with non-degenerate AUPRC values in (0, 1]
- Wilcoxon p-value transform yields scores in [0, 1] with the best gene (smallest p) mapped to score 1

## Limitations

- Applicable only where ground-truth key genes are known; real-dataset evaluation lacks this certainty
- Early Precision depends on an arbitrary 'early' cutoff choice, and the Wilcoxon baseline ranking uses the ad hoc transform 1 - pval/max(pval)
- scDisInFact gene scores assume condition and batch effects are separable; when each batch is measured under only one condition there is not enough information to fully disentangle the batch and condition effect

## Evidence

- [abstract] scDisInFact learns latent factors that disentangle condition effect from batch effect, enabling batch effect removal, condition-associated key gene detection, and perturbation prediction simultaneously.: "scDisInFact learns latent factors that disentangle condition effect from batch effect, enabling it to simultaneously perform three tasks: batch effect removal, condition-associated key gene"
- [methods] Key gene detection via extract_gene_scores: "gene_scores = model.extract_gene_scores()"
- [readme] README usage: scores available per condition type: "scores for condition type 1
genes_scores[0]
# scores for condition type 2"
- [abstract] Evaluation design against baselines: "We evaluate scDisInFact on both simulated and real datasets, and compare its performance with baseline methods for each task."
