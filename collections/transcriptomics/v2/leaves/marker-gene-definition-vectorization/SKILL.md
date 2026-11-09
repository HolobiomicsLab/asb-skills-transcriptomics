---
name: marker-gene-definition-vectorization
description: Use when you need a per-gene statistic that precisely matches the ideal
  marker-gene definition — a gene high in exactly one subtype and low in all others
  — across many subtypes in a batch-corrected expression matrix.
when_to_use_negative:
- The input matrix is not batch-corrected — the README states 'Note that the input
  of COT should be batch-corrected', so applying the cosine statistic to uncorrected
  data violates the method's premise
- The goal is hypothesis testing with p-values from very few genes — the README warns
  'too few genes may not work for predicting p-values' (e.g. the GSE28490 limited-gene
  case), so the cosine statistic alone cannot yield p-values there
- You need a statistic for a different purpose than marker-gene detection, since COT
  complements rather than replaces existing methods
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: COT
  role: Python module built on NumPy and pandas whose COT class exposes generate_subtype_means
    and generate_cos_values to compute the cosine-similarity test statistic
  repo: https://github.com/MintaYLu/COT
- name: NumPy
  role: Underlying library (>= 1.19.5) used to compute cosine similarity between cross-subtype
    expression vectors and ideal marker-gene vectors
- name: pandas
  role: Underlying library (>= 1.2.0) holding df_raw, cot.df_mean, and cot.df_cos
    tables
- name: SciPy
  role: Dependency (>= 1.3.1) of the COT module used for the one-sample test machinery
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbac037
    title: 'COT: an efficient and accurate method for detecting marker genes among
      many subtypes'
  source_content_hash: sha256:47f0aa56c6f8d8b1fabf76c6ede67ffe7fc8102bb89320313657e201cc3768ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MintaYLu/COT
  tools:
  - COT
  - NumPy
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# marker-gene-definition-vectorization

## Summary

Vectorizes the mathematical definition of an ideal marker gene into subtype indicator vectors and scores each candidate gene by the cosine similarity between its cross-subtype mean expression vector and each subtype's ideal vector, taking the argmax as the COT test statistic t_COT(i) = argmax_k s_ki / ||s_i||.

## When to use

Use when you need a per-gene statistic that precisely matches the ideal marker-gene definition — a gene high in exactly one subtype and low in all others — across many subtypes in a batch-corrected expression matrix.

## When NOT to use

- The input matrix is not batch-corrected — the README states 'Note that the input of COT should be batch-corrected', so applying the cosine statistic to uncorrected data violates the method's premise
- The goal is hypothesis testing with p-values from very few genes — the README warns 'too few genes may not work for predicting p-values' (e.g. the GSE28490 limited-gene case), so the cosine statistic alone cannot yield p-values there
- You need a statistic for a different purpose than marker-gene detection, since COT complements rather than replaces existing methods

## Inputs

- Batch-corrected raw expression matrix df_raw (pandas DataFrame, rows = genes, columns = samples, e.g. S1–S4)
- Per-sample subtype label vector, e.g. subtype_label = ["A", "A", "B", "B"] mapping sample columns to subtypes {"A": ["S1", "S2"], "B": ["S3", "S4"]}

## Outputs

- cot.subtypes: dict mapping subtype labels to sample lists, e.g. {"A": ["S1", "S2"], "B": ["S3", "S4"]}
- cot.df_mean: per-gene subtype mean expression table (pandas DataFrame, columns = subtypes)
- cot.df_cos: table with columns gene, cos, subtype holding the maximal cosine similarity per gene and the argmax subtype
- Saveable CSVs: cot_df_cos.csv (cosine table) and cot_df_mean.csv (means table)

## How to apply

Load the batch-corrected expression matrix df_raw into a COT class instance (COT(df_raw=df_raw, normalization=False)), since COT's input must already be batch-corrected. Call cot.generate_subtype_means(subtype_label=subtype_label) with a per-sample label vector (e.g. subtype_label = ["A", "A", "B", "B"]) to average expression within each subtype and produce cot.df_mean. Then call cot.generate_cos_values() to compute, per gene, the cosine similarity between its cross-subtype mean vector s_i and each subtype's ideal marker-gene indicator vector, and take the argmax over subtypes to record the maximal cosine value and its subtype in cot.df_cos (columns: gene, cos, subtype). The rationale is that existing test statistics do not exactly satisfy the marker-gene definition, whereas the cosine statistic in COT precisely matches the mathematical definition of an ideal MG; the resulting statistic is t_COT(i) = argmax over subtypes k of s_ki divided by the norm of gene i's cross-subtype expression vector s_i.

## Related tools

- **COT** (Python module built on NumPy and pandas whose COT class exposes generate_subtype_means and generate_cos_values to compute the cosine-similarity test statistic) — https://github.com/MintaYLu/COT
- **NumPy** (Underlying library (>= 1.19.5) used to compute cosine similarity between cross-subtype expression vectors and ideal marker-gene vectors)
- **pandas** (Underlying library (>= 1.2.0) holding df_raw, cot.df_mean, and cot.df_cos tables)
- **SciPy** (Dependency (>= 1.3.1) of the COT module used for the one-sample test machinery)

## Examples

```
from COT.COT import COT; cot = COT(df_raw=df_raw, normalization=False); cot.generate_subtype_means(subtype_label=["A","A","B","B"]); cot.generate_cos_values()
```

## Evaluation signals

- Every cosine statistic must lie in the interval (1/K, 1) for K subtypes — values at or below 1/K indicate the gene is no better than the null pattern for any subtype
- The subtype recorded in cot.df_cos must equal the argmax subtype achieving the maximal cosine value for that gene
- cot.df_mean equals the per-subtype average of df_raw columns grouped by subtype_label (e.g. gene 0 means 0.6 for A and 0.8 for B in the README example)
- cot.df_cos has exactly the schema gene, cos, subtype (with p.value/q.value added only after estimate_p_values)
- A perfect one-subtype gene (e.g. gene 1 with means 1.0 and 0.0) must yield cos = 1.0 assigned to that subtype

## Limitations

- Cannot calculate p-values when the gene count is too small (e.g. the GSE28490 example) because the empirical null approximation needs a majority of null-associated genes; NaN rows must be removed before estimate_p_values
- Input must be batch-corrected beforehand; COT itself does not correct batch effects (normalization=False in the example)
- COT complements rather than replaces existing marker-gene methods, so its statistic may need to be combined with other analyses

## Evidence

- [methods] COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic: "COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic"
- [abstract] the test statistic in COT precisely matches the mathematical definition of an ideal MG: "Fundamentally different from existing approaches, the test statistic in COT precisely matches the mathematical definition of an ideal MG."
- [readme] input of COT should be batch-corrected: "Note that the input of COT should be batch-corrected."
- [readme] subtype_label = ["A", "A", "B", "B"] yields cot.subtypes and cot.df_mean: "subtype_label = ["A", "A", "B", "B"]"
- [supplementary] cannot calculate p-values with limited genes: "Cannot calculate p-values due to the limited genes numbers, please see the Example_GSE28490 for p-values computation."
