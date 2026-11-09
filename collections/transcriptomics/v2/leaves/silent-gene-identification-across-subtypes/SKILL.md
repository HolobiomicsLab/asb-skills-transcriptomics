---
name: silent-gene-identification-across-subtypes
description: Use when a batch-corrected gene-by-sample expression matrix has multiple
  labeled subtypes and the goal is to flag genes that are silent specifically in one
  subtype (SDG) rather than subtype-enriched marker genes.
when_to_use_negative:
- The goal is detecting subtype-enriched marker genes (MG) — use the standard COT
  ideal-MG reference vector, not the XOR-based SDG statistic
- The input matrix is not batch-corrected; COT's input should be batch-corrected before
  computing subtype means and cosines
- There are only K=2 subtypes, since the SDG bound 1/(K-1) degenerates to 1 and the
  statistic is uninformative
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: COT
  role: Python module (built on NumPy/pandas) whose COT class, generate_subtype_means()
    and generate_cos_values() routines are adapted with an XOR reference vector to
    compute the SDG cosine statistic
  repo: https://github.com/MintaYLu/COT
- name: pandas
  role: required dependency for loading the batch-corrected df_raw matrix and writing
    cot.df_cos to sdg_cos_values.csv
  repo: https://github.com/MintaYLu/COT
- name: NumPy
  role: required dependency underlying the cosine (scatter-space) computations over
    subtype mean profiles
  repo: https://github.com/MintaYLu/COT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbac037
    title: 'COT: an efficient and accurate method for detecting marker genes among
      many subtypes'
  source_content_hash: sha256:99faab65b54b7cb9f63be4b52aacab127d7799e7fe57c75ce2ee474aebf31f06
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
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# silent-gene-identification-across-subtypes

## Summary

Identifies subtype-downregulated genes (SDG) — genes silent in exactly one subtype but expressed in all others — by replacing COT's ideal-MG reference vector with an XOR-based reference (0 in one subtype, 1 in all others) and computing a bounded cosine statistic (1/(K-1) < t_COT(i) < 1).

## When to use

Use when a batch-corrected gene-by-sample expression matrix has multiple labeled subtypes and the goal is to flag genes that are silent specifically in one subtype (SDG) rather than subtype-enriched marker genes.

## When NOT to use

- The goal is detecting subtype-enriched marker genes (MG) — use the standard COT ideal-MG reference vector, not the XOR-based SDG statistic
- The input matrix is not batch-corrected; COT's input should be batch-corrected before computing subtype means and cosines
- There are only K=2 subtypes, since the SDG bound 1/(K-1) degenerates to 1 and the statistic is uninformative

## Inputs

- batch-corrected gene-by-sample expression matrix (df_raw) as a pandas DataFrame, e.g. from GSE28490
- subtype_label list mapping each sample column to a subtype (e.g. ["A", "A", "B", "B"])

## Outputs

- cot.df_cos table with columns gene, cos, subtype (written to sdg_cos_values.csv), where cos is the XOR-basis SDG cosine statistic t_COT(i) bounded in (1/(K-1), 1)
- validation flags for rows whose cosine falls outside the (1/(K-1), 1) bound or whose argmax subtype does not show near-zero expression

## How to apply

Load the batch-corrected gene-by-sample matrix (e.g., Example_GSE28490) into a COT instance via COT(df_raw=df_raw, normalization=False), then call generate_subtype_means(subtype_label=...) to map sample columns to subtypes. For each gene, compute the cosine similarity between its subtype-mean expression vector and the XOR reference vector e_k XOR 1 — which is 0 in exactly one subtype k and 1 in all others — for every subtype k, taking argmax_k; this reproduces the COT generate_cos_values() routine with the SDG modification. Write the cot.df_cos table (columns: gene, cos, subtype) to sdg_cos_values.csv. Validate that every cosine lies strictly within (1/(K-1), 1) for K subtypes, and that the argmax-assigned subtype has near-zero mean expression while all other subtypes are elevated; flag any row violating these checks. P-values can optionally be approximated afterward with estimate_p_values(), but require enough genes so the empirical null is estimable.

## Related tools

- **COT** (Python module (built on NumPy/pandas) whose COT class, generate_subtype_means() and generate_cos_values() routines are adapted with an XOR reference vector to compute the SDG cosine statistic) — https://github.com/MintaYLu/COT
- **pandas** (required dependency for loading the batch-corrected df_raw matrix and writing cot.df_cos to sdg_cos_values.csv) — https://github.com/MintaYLu/COT
- **NumPy** (required dependency underlying the cosine (scatter-space) computations over subtype mean profiles) — https://github.com/MintaYLu/COT

## Examples

```
from COT.COT import COT; cot = COT(df_raw=df_raw, normalization=False); cot.generate_subtype_means(subtype_label=subtype_label); cot.generate_cos_values(); cot.df_cos.to_csv('sdg_cos_values.csv')
```

## Evaluation signals

- Every computed cosine value lies strictly within the open interval (1/(K-1), 1) for K subtypes
- Each gene assigned to its argmax subtype k has mean expression near zero in subtype k and elevated expression in all other subtypes
- cot.df_cos schema contains exactly the columns gene, cos, subtype before p-value estimation
- cot.subtypes mapping from generate_subtype_means() correctly partitions all sample columns (e.g. {"A": ["S1", "S2"], "B": ["S3", "S4"]})
- NaNs are removed before estimate_p_values(), since too few genes prevents p-value estimation

## Limitations

- p-values cannot be estimated when the number of genes is too small; NaNs must be removed before estimate_p_values()
- COT input must be batch-corrected; uncorrected batch effects distort the cosine statistic
- COT complements rather than replaces existing marker-gene methods, and existing statistics may identify inaccurate marker genes

## Evidence

- [methods] COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic: "COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic"
- [supplementary] The COT test statistic can be modified to detect SDG by replacing the ideal-MG reference vector via an exclusive disjunction (XOR) operation: [evidence span withheld — anchor status: not_found]
- [readme] Note that the input of COT should be batch-corrected.: "Note that the input of COT should be batch-corrected."
- [readme] Under the assumption that a significant majority of genes are associated with the null hypothesis, COT approximates the empirical null distribution for calculating p-values.: "Under the assumption that a significant majority of genes are associated with the null hypothesis, COT approximates the empirical null distribution for calculating p-values."
- [supplementary] Cannot calculate p-values due to the limited genes numbers, please see the Example_GSE28490 for p-values computation.: "Cannot calculate p-values due to the limited genes numbers, please see the Example_GSE28490 for p-values computation."
