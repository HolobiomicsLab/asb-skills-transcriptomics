---
name: dataset-metadata-feature-engineering
description: Use when benchmarking cell type annotation tools (e.g., TOSICA) across
  multiple reference-query dataset pairs and you need to explain accuracy variation
  via dataset characteristics such as size, cell type count, label entropy, and reference-query
  label divergence (DKL).
when_to_use_negative:
- You need to improve annotation accuracy itself (e.g., train or run TOSICA) rather
  than explain accuracy variation across datasets — this skill only correlates metadata
  features with accuracy.
- Only one dataset is available — Pearson correlation across dataset-level features
  requires multiple benchmarked datasets to be meaningful.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0091
tools:
- name: Python 3.8.11
  role: Implementation language for the pandas/scipy correlation workflow
- name: TOSICA
  role: Cell type annotation tool whose benchmark accuracy across datasets is the
    target variable being correlated with metadata features
  repo: https://github.com/JackieHanLab/TOSICA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:63e2351e9ff2480fd7ceebe561767975156d8a461c6493bf3388adf12c226acd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - Python 3.8.11
  - TOSICA
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# dataset-metadata-feature-engineering

## Summary

Engineer per-dataset metadata features (log-transformed dataset size, number of cell types, entropy of the cell type label distribution, and KL divergence between reference and query label distributions) and correlate them with cell type annotation accuracy to identify which dataset characteristics drive annotation performance.

## When to use

Use when benchmarking cell type annotation tools (e.g., TOSICA) across multiple reference-query dataset pairs and you need to explain accuracy variation via dataset characteristics such as size, cell type count, label entropy, and reference-query label divergence (DKL).

## When NOT to use

- You need to improve annotation accuracy itself (e.g., train or run TOSICA) rather than explain accuracy variation across datasets — this skill only correlates metadata features with accuracy.
- Only one dataset is available — Pearson correlation across dataset-level features requires multiple benchmarked datasets to be meaningful.

## Inputs

- Benchmark accuracy table (per-dataset, per-tool annotation accuracy)
- Dataset metadata: number of cells per dataset
- Cell type label distributions for reference and query datasets
- Number of cell types per dataset

## Outputs

- correlation_results.csv containing the per-dataset metric table (Log size, Types, Entropy, DKL) and Pearson correlation coefficients
- Pearson correlation coefficient between annotation accuracy and DKL (~-0.9 expected)

## How to apply

Load the benchmark accuracy table and dataset metadata (dataset sizes, cell type counts, per-tool accuracy) into a pandas DataFrame. Compute Log size as the natural-log-transformed dataset size, record the number of cell Types per dataset, compute Entropy from each dataset's cell type label distribution, and compute DKL (Kullback-Leibler divergence) from the reference label distribution relative to the query. Then compute Pearson correlation coefficients (PCC) between annotation accuracy and each metric using scipy.stats.pearsonr, and write the per-dataset metric table and correlation coefficients to correlation_results.csv. The key decision point is validating the reproduction: the PCC between accuracy and DKL should match the reported value of approximately -0.9.

## Related tools

- **Python 3.8.11** (Implementation language for the pandas/scipy correlation workflow)
- **TOSICA** (Cell type annotation tool whose benchmark accuracy across datasets is the target variable being correlated with metadata features) — https://github.com/JackieHanLab/TOSICA

## Examples

```
import pandas as pd; from scipy.stats import pearsonr; df = pd.read_csv('benchmark_metadata.csv'); df['Log size'] = np.log(df['size']); df['DKL'] = compute_dkl(df); df['Entropy'] = compute_entropy(df); pcc, p = pearsonr(df['accuracy'], df['DKL']); df.to_csv('correlation_results.csv')
```

## Evaluation signals

- Reproduced Pearson correlation coefficient between accuracy and DKL matches approximately -0.9 within a small numerical tolerance
- All four metrics (Log size, Types, Entropy, DKL) are present for every benchmarked dataset in correlation_results.csv
- Log size values equal natural-log transforms of the raw dataset sizes (verifiable spot-check)
- Entropy and DKL values are non-negative and consistent with the cell type label distributions used as inputs
- correlation_results.csv contains both the per-dataset metric table and the four correlation coefficients

## Limitations

- Uneven distribution of cell types between reference and query (high DKL) is the dominant negative factor for annotation accuracy, so correlations reflect this dataset characteristic rather than tool-specific behavior.
- The workflow was validated against a single reported value (PCC ≈ -0.9 for accuracy vs DKL); other correlations are reported but have no pre-registered validation targets.

## Evidence

- [results] The correlation analysis shows that uneven distribution of cell types between reference and query (DKL) has the most negative impact on annotation accuracy, with a Pearson correlation coefficient between accuracy and DKL of −0.9.: [evidence span withheld — anchor status: not_found]
- [methods] Load the benchmark accuracy table and dataset metadata into a pandas DataFrame; compute Log size, Types, Entropy, DKL; use scipy.stats.pearsonr; write correlation_results.csv.: [evidence span withheld — anchor status: not_found]
- [discussion] Python version 3.8.11 and R version 4.0.5 were used for downstream analysis: "Python version 3.8.11 and R version 4.0.5 were used for downstream analysis"
- [intro] transferring cell type annotation from a reference dataset to newly generated query datasets with consistency and reproducibility is increasingly important and necessary: "transferring cell type annotation from a reference dataset to newly generated query datasets with consistency and reproducibility is increasingly important and necessary"
