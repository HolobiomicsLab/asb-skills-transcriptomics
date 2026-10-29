---
name: single-cell-benchmark-result-reproduction
description: Use when a benchmark table of per-dataset annotation accuracies is available
  with dataset metadata (cell counts, cell type counts, label distributions) and a
  reported correlation (e.g., accuracy vs. DKL) must be independently reproduced.
when_to_use_negative:
- The input is raw scRNA-seq data (e.g., raw counts in .h5ad) rather than a precomputed
  benchmark accuracy table with dataset metadata
- The analysis goal is running the annotation method itself (e.g., TOSICA train/predict)
  rather than statistically auditing its reported benchmark results
- Label distributions for both reference and query are unavailable, making DKL computation
  impossible
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: TOSICA
  role: Source of the benchmark accuracy data being reproduced; its reported performance
    (fast, accurate, batch-insensitive annotation) is the target of the correlation
    analysis
  repo: https://github.com/JackieHanLab/TOSICA
- name: Python
  role: Analysis environment (Python 3.8.11 reported) used with pandas and scipy.stats.pearsonr
    to compute per-dataset metrics and correlation coefficients
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:3cd0be8bfc34dc7c1bc17e98c029d032a6a2c0a6f9f23b893606cb7ec46043cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# single-cell-benchmark-result-reproduction

## Summary

Reproduce benchmark statistics from single-cell method papers by recomputing per-dataset metrics (dataset size, number of cell types, entropy, DKL) and Pearson correlation coefficients against reported accuracies, validating the reproduction against reported values such as the TOSICA accuracy–DKL PCC of ~−0.9.

## When to use

Use when a benchmark table of per-dataset annotation accuracies is available with dataset metadata (cell counts, cell type counts, label distributions) and a reported correlation (e.g., accuracy vs. DKL) must be independently reproduced.

## When NOT to use

- The input is raw scRNA-seq data (e.g., raw counts in .h5ad) rather than a precomputed benchmark accuracy table with dataset metadata
- The analysis goal is running the annotation method itself (e.g., TOSICA train/predict) rather than statistically auditing its reported benchmark results
- Label distributions for both reference and query are unavailable, making DKL computation impossible

## Inputs

- Benchmark accuracy table with per-dataset, per-tool cell type annotation accuracy
- Dataset metadata: number of cells, number of cell types, per-dataset cell type label distributions for reference and query
- Python environment with pandas and scipy

## Outputs

- correlation_results.csv containing the per-dataset metric table (Log size, Types, Entropy, DKL) and the four Pearson correlation coefficients
- Reproduced PCC value for accuracy vs. DKL for validation against the reported ~−0.9

## How to apply

Load the benchmark accuracy table and dataset metadata (dataset sizes, cell type counts, per-tool accuracy) into a pandas DataFrame. Compute per-dataset metrics: log-transformed dataset size (natural log), number of cell types, Shannon entropy of the cell type label distribution, and the Kullback-Leibler divergence (DKL) between reference and query label distributions. Compute Pearson correlation coefficients (PCC) between annotation accuracy and each of the four metrics using scipy.stats.pearsonr, mirroring the TOSICA benchmark analysis that identified uneven reference-query cell type distributions (DKL) as the dominant negative factor. Write the per-dataset metric table and correlation coefficients to an output CSV. Validate by checking the reproduced PCC for accuracy vs. DKL matches the reported −0.9 within a small numerical tolerance.

## Related tools

- **TOSICA** (Source of the benchmark accuracy data being reproduced; its reported performance (fast, accurate, batch-insensitive annotation) is the target of the correlation analysis) — https://github.com/JackieHanLab/TOSICA
- **Python** (Analysis environment (Python 3.8.11 reported) used with pandas and scipy.stats.pearsonr to compute per-dataset metrics and correlation coefficients)

## Examples

```
import pandas as pd, scipy.stats; df = pd.read_csv('benchmark_accuracy.csv'); pcc = scipy.stats.pearsonr(df['accuracy'], df['DKL'])
```

## Evaluation signals

- Reproduced Pearson correlation coefficient between annotation accuracy and DKL is within a small numerical tolerance of the reported −0.9
- All four metrics (Log size, Types, Entropy, DKL) are present for every benchmarked dataset in correlation_results.csv
- Entropy values computed from cell type label distributions are non-negative and DKL values are non-negative (information-theoretic invariant)
- Per-dataset accuracy values in the output match the source benchmark table exactly (no transcription drift)
- Pearson coefficients for all four metrics fall within [−1, 1] and the DKL coefficient is the most negative, consistent with the reported finding

## Limitations

- Correlation analysis requires complete label distributions for both reference and query datasets; missing distributions prevent DKL computation for that dataset
- The reproduction validates a single reported statistic (accuracy–DKL PCC of −0.9); tolerance for numerical agreement is small, so minor pipeline differences may cause false validation failures
- Correlation does not establish causation — the analysis identifies which dataset characteristics (DKL, Log size, Types, Entropy) associate with accuracy, not a predictive model

## Evidence

- [other] The correlation analysis shows that uneven distribution of cell types between reference and query (DKL) has the most negative impact on annotation accuracy: [evidence span withheld — anchor status: not_found]
- [results] Validation target for the reproduced PCC: [evidence span withheld — anchor status: not_found]
- [discussion] TOSICA benchmark context for the reproduced results: "The high accuracy and batch-insensitivity of TOSICA can be mainly attributed to the attention layers and tokens masked by high-level biologically relevant pathways or regulons"
- [readme] Installation of the tool whose benchmarks are reproduced: "pip install git+https://github.com/JackieHanLab/TOSICA.git"
