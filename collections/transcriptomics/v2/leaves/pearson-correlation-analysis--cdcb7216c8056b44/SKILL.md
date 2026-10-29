---
name: pearson-correlation-analysis
description: Use when you have a benchmark accuracy table plus per-dataset metadata
  (sizes, cell type counts, label distributions) and need to identify which dataset
  characteristics drive annotation accuracy.
when_to_use_negative:
- Input lacks per-dataset cell type label distributions for both reference and query,
  so Entropy and DKL cannot be computed
- The analysis goal is causal modeling or non-monotonic association rather than linear
  correlation between metrics and accuracy
- Accuracy values come from a single dataset only, leaving too few benchmark points
  for a meaningful PCC
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: Python 3.8.11
  role: 'Implementation language: pandas DataFrame assembly, metric computation, and
    scipy.stats.pearsonr correlation'
- name: TOSICA
  role: Source of the benchmark annotation accuracy values being correlated against
    dataset characteristics
  repo: https://github.com/JackieHanLab/TOSICA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:ae0e76779197186c14b7740ec24ab157fe7a038988b0d9fc05ebe662eacb39a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - Python 3.8.11
  - TOSICA
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# pearson-correlation-analysis

## Summary

Quantifies linear relationships between benchmark dataset characteristics (dataset size, cell type count, entropy, DKL between reference and query) and cell type annotation accuracy using Pearson correlation coefficients (PCC) computed with scipy.stats.pearsonr in Python.

## When to use

Use when you have a benchmark accuracy table plus per-dataset metadata (sizes, cell type counts, label distributions) and need to identify which dataset characteristics drive annotation accuracy.

## When NOT to use

- Input lacks per-dataset cell type label distributions for both reference and query, so Entropy and DKL cannot be computed
- The analysis goal is causal modeling or non-monotonic association rather than linear correlation between metrics and accuracy
- Accuracy values come from a single dataset only, leaving too few benchmark points for a meaningful PCC

## Inputs

- Benchmark accuracy table with per-dataset, per-tool annotation accuracy
- Dataset metadata: number of cells, number of cell types, cell type label distributions for reference and query
- pandas DataFrame environment (Python 3.8 with scipy)

## Outputs

- correlation_results.csv containing the per-dataset metric table (Log size, Types, Entropy, DKL) and Pearson correlation coefficients
- Ranking of dataset characteristics by magnitude of correlation with annotation accuracy

## How to apply

Load the benchmark accuracy table and dataset metadata (dataset sizes, cell type counts, per-tool accuracy) into a pandas DataFrame. Compute natural-log-transformed dataset size (Log size) and record the number of cell Types per dataset. Compute Entropy from the per-dataset cell type label distribution and DKL (Kullback-Leibler divergence) from the reference label distribution versus the query. Compute Pearson correlation coefficients between annotation accuracy and each metric (Log size, Types, Entropy, DKL) using scipy.stats.pearsonr, then write the per-dataset metric table and correlation coefficients to correlation_results.csv. The rationale is to isolate which dataset property most affects label transfer performance; in the TOSICA benchmark, uneven cell type distribution between reference and query (DKL) dominated, with accuracy-vs-DKL PCC ≈ −0.9.

## Related tools

- **Python 3.8.11** (Implementation language: pandas DataFrame assembly, metric computation, and scipy.stats.pearsonr correlation)
- **TOSICA** (Source of the benchmark annotation accuracy values being correlated against dataset characteristics) — https://github.com/JackieHanLab/TOSICA

## Examples

```
import pandas as pd; from scipy.stats import pearsonr; df = pd.read_csv('benchmark_accuracy.csv'); r, p = pearsonr(df['accuracy'], df['DKL']); df.to_csv('correlation_results.csv')
```

## Evaluation signals

- Reproduced Pearson correlation coefficient between annotation accuracy and DKL should match the reported value of approximately −0.9 (accept within a small numerical tolerance)
- All four metrics (Log size, Types, Entropy, DKL) must be present for every benchmarked dataset in correlation_results.csv
- Entropy values must be non-negative and consistent with the cell type label distribution of each dataset
- correlation_results.csv must contain both the per-dataset metric table and the four correlation coefficients
- scipy.stats.pearsonr output should include a p-value; extremely strong coefficients (|r| > 0.9) should be checked against the small number of benchmarked datasets

## Limitations

- Pearson correlation captures only linear association; the reported −0.9 for DKL is observational and does not establish causation
- The result depends on the specific benchmark datasets used (their sizes, cell type counts, and reference-query distribution mismatch) and may not generalize to other benchmarks
- Metrics must be computed over identical dataset sets; missing cell type distributions for any dataset breaks the DKL/Entropy calculation

## Evidence

- [results] The correlation analysis shows that uneven distribution of cell types between reference and query (DKL) has the most negative impact on annotation accuracy: [evidence span withheld — anchor status: not_found]
- [methods] Workflow: correlation of accuracy with dataset metrics: [evidence span withheld — anchor status: not_found]
- [methods] Validation: reproduced PCC must match reported value: [evidence span withheld — anchor status: not_found]
- [discussion] Python used for downstream analysis: "Python version 3.8.11 and R version 4.0.5 were used for downstream analysis"
