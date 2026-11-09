---
name: argmax-subtype-assignment-validation
description: Use when a batch-corrected expression matrix with per-gene subtype means
  (cot.df_mean) is available and each gene must be scored and assigned to the subtype
  whose ideal-marker-gene definition its cross-subtype pattern most closely matches.
when_to_use_negative:
- The input matrix is not batch-corrected — the COT input must be batch-corrected
  before cosine computation
- The goal is p-value estimation or marker list extraction rather than cosine scoring/assignment
  — use cot.estimate_p_values() or cot.obtain_subtype_markers() instead
- There are too few genes for downstream null-distribution steps (though argmax cosine
  assignment itself still works, p-values cannot be computed — see Example_GSE28490)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: COT
  role: Python module providing the COT class whose generate_cos_values() step computes
    the per-gene cosine similarities and argmax subtype assignments
  repo: https://github.com/MintaYLu/COT
- name: NumPy
  role: Backend used by COT (built on top of NumPy and Pandas) for the vector norm
    and cosine-similarity computations
- name: pandas
  role: Holds the batch-corrected expression matrix df_raw, subtype means cot.df_mean,
    and cosine output cot.df_cos as DataFrames
- name: Python
  role: Runtime for the COT module (requires python >= 3.7.4)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbac037
    title: 'COT: an efficient and accurate method for detecting marker genes among
      many subtypes'
  source_content_hash: sha256:d2ade76a7c7ad7875e8eba36b5fc1c09dd7fe711b95503d37acacab2bb12b8f1
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# argmax-subtype-assignment-validation

## Summary

Assigns each candidate marker gene to its best-matching subtype by taking the argmax over subtypes of the cosine similarity between the gene's cross-subtype expression vector and each subtype's ideal marker-gene indicator, then validates that every statistic lies in (1/K, 1) and matches the recorded argmax subtype.

## When to use

Use when a batch-corrected expression matrix with per-gene subtype means (cot.df_mean) is available and each gene must be scored and assigned to the subtype whose ideal-marker-gene definition its cross-subtype pattern most closely matches.

## When NOT to use

- The input matrix is not batch-corrected — the COT input must be batch-corrected before cosine computation
- The goal is p-value estimation or marker list extraction rather than cosine scoring/assignment — use cot.estimate_p_values() or cot.obtain_subtype_markers() instead
- There are too few genes for downstream null-distribution steps (though argmax cosine assignment itself still works, p-values cannot be computed — see Example_GSE28490)

## Inputs

- Batch-corrected raw expression DataFrame (cot.df_raw) with genes as rows and sample columns (e.g. S1–S4), loaded via COT(df_raw=df_raw, normalization=False)
- Subtype label vector mapping sample columns to subtypes, e.g. subtype_label = ['A', 'A', 'B', 'B']
- Per-gene subtype mean table (cot.df_mean) produced by cot.generate_subtype_means()

## Outputs

- Cosine-statistic table cot.df_cos with columns gene, cos, subtype giving the maximal cosine value and argmax subtype per gene (saved as cot_df_cos.csv)
- Subtype means table cot.df_mean (saved as cot_df_mean.csv)
- Validation check that all cos values fall in (1/K, 1) and recorded subtypes equal the argmax subtypes

## How to apply

After running cot.generate_subtype_means(subtype_label=subtype_label) to average expression within each subtype (e.g. subtype_label = ['A','A','B','B'] giving cot.subtypes {'A': ['S1','S2'], 'B': ['S3','S4']}), call cot.generate_cos_values() on the COT object built with COT(df_raw=df_raw, normalization=False). For each gene i, compute with NumPy the cosine similarity between its cross-subtype mean vector s_i and each subtype's ideal marker-gene indicator vector e_k, then take the argmax over subtypes: t_COT(i) = argmax_k s_ki / ||s_i||. Record the maximal cosine value and its corresponding subtype in cot.df_cos with columns gene, cos, subtype (e.g. gene 1 -> cos 1.0, subtype A; gene 0 -> cos 0.8, subtype B). Validate that every cosine statistic lies in (1/K, 1) for K subtypes and that the recorded subtype equals the argmax subtype; save the table as cot_df_cos.csv (and means as cot_df_mean.csv). This step is the core of COT because its test statistic precisely matches the mathematical definition of an ideal marker gene, unlike existing test statistics.

## Related tools

- **COT** (Python module providing the COT class whose generate_cos_values() step computes the per-gene cosine similarities and argmax subtype assignments) — https://github.com/MintaYLu/COT
- **NumPy** (Backend used by COT (built on top of NumPy and Pandas) for the vector norm and cosine-similarity computations)
- **pandas** (Holds the batch-corrected expression matrix df_raw, subtype means cot.df_mean, and cosine output cot.df_cos as DataFrames)
- **Python** (Runtime for the COT module (requires python >= 3.7.4))

## Examples

```
from COT.COT import COT; cot = COT(df_raw=df_raw, normalization=False); cot.generate_subtype_means(subtype_label=["A","A","B","B"]); cot.generate_cos_values(); cot.df_cos  # columns: gene, cos, subtype
```

## Evaluation signals

- Every cosine statistic in cot.df_cos lies strictly in (1/K, 1) for K subtypes
- The subtype recorded in cot.df_cos for each gene equals the argmax over subtypes of the cosine similarity
- A perfectly subtype-specific gene (e.g. README gene 1: means 1.0 in A, 0.0 in B) yields cos = 1.0 and is assigned to that subtype
- cot.df_cos has the expected schema: columns gene, cos, subtype; and matches the downstream input of cot.estimate_p_values()
- Files cot_df_cos.csv and cot_df_mean.csv are written with contents consistent with cot.df_cos and cot.df_mean

## Limitations

- Existing test statistics, including naive cosine use, must satisfy the exact MG definition — COT's statistic is designed to match it precisely, so this skill should not be substituted with generic correlation or fold-change scoring
- Too few genes may not work for predicting p-values at the following estimate_p_values() step; NaNs must be removed before that step (see Example_GSE28490)
- The input must be batch-corrected; otherwise the cosine statistic is computed on biased cross-subtype profiles
- COT complements, not replaces, existing marker-gene methods

## Evidence

- [methods] COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic: "COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic"
- [abstract] the test statistic in COT precisely matches the mathematical definition of an ideal MG: "Fundamentally different from existing approaches, the test statistic in COT precisely matches the mathematical definition of an ideal MG."
- [readme] cot.generate_cos_values() output df_cos columns gene, cos, subtype: "cot.generate_cos_values()

output:
cot.df_cos

| gene | cos | subtype |"
- [readme] generate_subtype_means maps subtype_label ['A','A','B','B'] to subtypes {'A': ['S1','S2'], 'B': ['S3','S4']}: "subtype_label = ["A", "A", "B", "B"]

output:
cot.subtypes {"A": ["S1", "S2"], "B": ["S3", "S4"]}"
