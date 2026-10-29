---
name: entropy-and-divergence-computation
description: Use when benchmarking label-transfer/annotation accuracy on datasets
  with known cell type label distributions and you need dataset-level characteristics
  (size, type count, entropy, DKL) to correlate with accuracy.
when_to_use_negative:
- When only raw expression matrices without cell type labels for both reference and
  query are available — entropy and DKL require known label distributions
- When the goal is annotation itself rather than explaining/benchmarking annotation
  accuracy across datasets (use TOSICA's train/pre pipeline instead)
- When distributions are single-categorical (degenerate) making DKL undefined or uninformative
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: TOSICA
  role: Source of the benchmark accuracy data and the label-transfer context whose
    accuracy is explained by the entropy/DKL metrics
  repo: https://github.com/JackieHanLab/TOSICA
- name: Python
  role: Analysis environment (pandas DataFrame, scipy.stats.pearsonr) used to compute
    entropy, DKL, and Pearson correlations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:24640971f9490823330718cad04f9b3bf9460df863654f330a1a3316d4dabd23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# entropy-and-divergence-computation

## Summary

Computes label-distribution entropy and Kullback-Leibler divergence (DKL) between reference and query cell type distributions to quantify how uneven cell type composition drives cell type annotation accuracy in scRNA-seq benchmarks.

## When to use

Use when benchmarking label-transfer/annotation accuracy on datasets with known cell type label distributions and you need dataset-level characteristics (size, type count, entropy, DKL) to correlate with accuracy.

## When NOT to use

- When only raw expression matrices without cell type labels for both reference and query are available — entropy and DKL require known label distributions
- When the goal is annotation itself rather than explaining/benchmarking annotation accuracy across datasets (use TOSICA's train/pre pipeline instead)
- When distributions are single-categorical (degenerate) making DKL undefined or uninformative

## Inputs

- Benchmark accuracy table (per-dataset, per-tool cell type annotation accuracy)
- Dataset metadata: number of cells and number of cell types per dataset
- Reference dataset cell type label distribution
- Query dataset cell type label distribution

## Outputs

- Per-dataset metric table (Log size, Types, Entropy, DKL per dataset) written to correlation_results.csv
- Pearson correlation coefficients (PCC) between annotation accuracy and each of Log size, Types, Entropy, DKL

## How to apply

Load the benchmark accuracy table and dataset metadata (dataset sizes, number of cell types, per-tool accuracy) into a pandas DataFrame, and compute Log size (natural-log-transformed dataset size) and the number of cell types per dataset. Compute Entropy per dataset from the cell type label distribution and DKL per dataset from the reference label distribution versus the query. Compute Pearson correlation coefficients (PCC) between annotation accuracy and each metric (Log size, Types, Entropy, DKL) using scipy.stats.pearsonr. Rationale: the TOSICA benchmark found the uneven distribution of cell types between reference and query (DKL) is the dominant negative driver of annotation accuracy, so these divergence metrics diagnose when label transfer is expected to fail.

## Related tools

- **TOSICA** (Source of the benchmark accuracy data and the label-transfer context whose accuracy is explained by the entropy/DKL metrics) — https://github.com/JackieHanLab/TOSICA
- **Python** (Analysis environment (pandas DataFrame, scipy.stats.pearsonr) used to compute entropy, DKL, and Pearson correlations)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(df['accuracy'], df['DKL'])
```

## Evaluation signals

- Reproduced PCC between annotation accuracy and DKL should match the reported value of approximately -0.9 within a small numerical tolerance
- All four metrics (Log size, Types, Entropy, DKL) must be present for every benchmarked dataset in correlation_results.csv
- Entropy values must be computable from the per-dataset cell type label distribution (non-negative, consistent with the number of cell types)
- DKL values must be finite and derived from the reference versus query label distributions on identical cell type label sets

## Limitations

- DKL between reference and query label distributions is the dominant negative factor (PCC ≈ -0.9), so datasets with highly uneven type distributions will skew accuracy conclusions regardless of the annotation method
- Accuracy/DKL conclusions are tied to this TOSICA benchmark's datasets; the correlation magnitude may not transfer to other annotation tasks
- Reference and query gene sets must be consistent for the underlying TOSICA predictions feeding the accuracy table ('the var_names (genes) of the ref_adata and query_adata must be consistent and in the same order')

## Evidence

- [other] Compute Entropy per dataset from the cell type label distribution and compute the DKL (Kullback-Leibler divergence) per dataset from the reference label distribution.: [evidence span withheld — anchor status: not_found]
- [other] Compute Pearson correlation coefficients (PCC) between annotation accuracy and each metric (Log size, Types, Entropy, DKL) using scipy.stats.pearsonr.: [evidence span withheld — anchor status: not_found]
- [other] The reproduced PCC between accuracy and DKL should match the reported value of approximately -0.9 (accept if within a small numerical tolerance of -0.9).: [evidence span withheld — anchor status: not_found]
- [readme] Readme note that reference and query genes must align before predictions feed the accuracy table.: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order"
