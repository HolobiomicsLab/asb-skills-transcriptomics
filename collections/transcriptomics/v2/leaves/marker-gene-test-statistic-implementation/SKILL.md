---
name: marker-gene-test-statistic-implementation
description: Use when you have a batch-corrected gene-by-sample expression matrix
  with subtype labels and need to score genes by how closely their cross-subtype pattern
  matches an ideal marker gene (or an XOR-defined subtype-downregulated gene) profile.
when_to_use_negative:
- Input data is not batch-corrected — COT expects batch-corrected input and the statistic
  will be distorted otherwise
- The goal is differential expression between exactly two conditions rather than marker
  detection among many subtypes
- Too few genes are provided to approximate the empirical null distribution — p-values
  cannot be computed reliably (see the Example_GSE28490 vignette)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: COT
  role: Python module implementing the cosine-based one-sample test statistic, subtype-mean
    aggregation, p-value estimation, and marker extraction
  repo: https://github.com/MintaYLu/COT
- name: NumPy
  role: Underlying numerical library for computing cosine similarities between expression
    profiles and reference vectors
- name: pandas
  role: DataFrame handling for df_raw, df_mean, and df_cos tables
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbac037
    title: 'COT: an efficient and accurate method for detecting marker genes among
      many subtypes'
  source_content_hash: sha256:2b8b6c177d2daee16d49ed628b1f6e754076a53f4981f46e65214863ae3047f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MintaYLu/COT
  tools:
  - COT
  - NumPy
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# marker-gene-test-statistic-implementation

## Summary

Implement the COT Cosine-based One-sample Test statistic: the cosine similarity between a molecule's cross-subtype expression profile and the exact mathematical definition of an ideal marker gene (and its XOR-based extension for subtype-downregulated genes), used to detect marker genes among many subtypes.

## When to use

Use when you have a batch-corrected gene-by-sample expression matrix with subtype labels and need to score genes by how closely their cross-subtype pattern matches an ideal marker gene (or an XOR-defined subtype-downregulated gene) profile.

## When NOT to use

- Input data is not batch-corrected — COT expects batch-corrected input and the statistic will be distorted otherwise
- The goal is differential expression between exactly two conditions rather than marker detection among many subtypes
- Too few genes are provided to approximate the empirical null distribution — p-values cannot be computed reliably (see the Example_GSE28490 vignette)

## Inputs

- Batch-corrected gene-by-sample expression DataFrame (df_raw) with genes as rows and sample columns as features
- Subtype label list mapping each sample column to a subtype (e.g. subtype_label = ["A", "A", "B", "B"])

## Outputs

- cot.df_mean — subtype-level mean expression table per gene
- cot.df_cos — table with columns gene, cos, subtype (plus p.value and q.value after estimate_p_values())
- cot.markers — dict mapping each subtype to its detected marker genes

## How to apply

Instantiate the COT class on a batch-corrected raw expression DataFrame with COT(df_raw=df_raw, normalization=False), then call generate_subtype_means(subtype_label=subtype_label) to aggregate replicate samples (e.g. subtype_label=["A","A","B","B"]) into subtype-level mean profiles. Compute the cosine similarity between each gene's cross-subtype expression vector and the ideal marker-gene reference vector via generate_cos_values(), which for each gene takes the argmax over subtypes of the cosine and records it in cot.df_cos (columns: gene, cos, subtype). For subtype-downregulated genes (SDG), replace the reference with an exclusive disjunction (XOR) vector that is 0 in exactly one subtype and 1 in all others — cos(s_i, e_k XOR 1) for every subtype k — yielding a statistic bounded as 1/(K-1) < t_COT(i) < 1 for K subtypes. Optionally estimate_p_values() under the null assumption that a majority of genes follow the null hypothesis, then obtain_subtype_markers() to assign top genes per subtype.

## Related tools

- **COT** (Python module implementing the cosine-based one-sample test statistic, subtype-mean aggregation, p-value estimation, and marker extraction) — https://github.com/MintaYLu/COT
- **NumPy** (Underlying numerical library for computing cosine similarities between expression profiles and reference vectors)
- **pandas** (DataFrame handling for df_raw, df_mean, and df_cos tables)

## Examples

```
from COT.COT import COT; cot = COT(df_raw=df_raw, normalization=False); cot.generate_subtype_means(subtype_label=subtype_label); cot.generate_cos_values(); cot.estimate_p_values(); cot.obtain_subtype_markers()
```

## Evaluation signals

- Every cosine value in cot.df_cos lies strictly within (1/(K-1), 1) for K subtypes when the XOR/SDG variant is used
- Each gene assigned to the argmax subtype has mean expression near zero in that subtype and elevated expression in all others; flag rows that violate this
- cot.df_cos contains exactly the columns gene, cos, subtype (with p.value and q.value appended after estimate_p_values())
- cot.markers maps each subtype label to the genes with the highest cosine statistic for that subtype (e.g. {"A": [1], "B": [0]} in the README toy example)
- A perfectly subtype-specific gene (e.g. row with 1.0 in subtype A and 0.0 elsewhere) attains cos = 1.0 for that subtype

## Limitations

- p-values cannot be calculated when the number of genes is too small; remove NaNs before estimate_p_values() and see the Example_GSE28490 vignette for p-value computation
- The input to COT must be batch-corrected; uncorrected data will bias the cosine statistic
- COT complements rather than replaces existing marker-gene methods

## Evidence

- [methods] COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic: "COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic"
- [abstract] Under the assumption that a significant majority of genes are associated with the null hypothesis, COT approximates the empirical null distribution for calculating p-values.: "Under the assumption that a significant majority of genes are associated with the null hypothesis, COT approximates the empirical null distribution for calculating p-values."
- [readme] Note that the input of COT should be batch-corrected.: "Note that the input of COT should be batch-corrected."
- [readme] cot = COT(df_raw=df_raw, normalization=False): "cot = COT(df_raw=df_raw, normalization=False)"
- [readme] Cannot calculate p-values due to the limited genes numbers, please see the Example_GSE28490 for p-values computation.: "Cannot calculate p-values due to the limited genes numbers, please see the Example_GSE28490 for p-values computation."
- [abstract] the test statistic in COT precisely matches the mathematical definition of an ideal MG: "Fundamentally different from existing approaches, the test statistic in COT precisely matches the mathematical definition of an ideal MG."
