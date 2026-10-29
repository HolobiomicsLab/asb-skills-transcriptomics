---
name: cosine-similarity-range-validation
description: Use when cot.df_cos has been produced by COT's generate_cos_values()
  (or the XOR-basis SDG variant) and the cosine values must be checked against theoretical
  bounds before ranking markers or writing sdg_cos_values.csv.
when_to_use_negative:
- cot.df_cos does not yet exist — the cosine statistics must first be computed with
  generate_cos_values() before ranges can be validated
- The gene-by-sample matrix is not batch-corrected, since COT's input is explicitly
  required to be batch-corrected and uncorrected values can distort cosine scores
- Data are replicate-level samples rather than aggregated subtype means and generate_subtype_means()
  has not yet been run with the subtype_label mapping
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: COT
  role: Provides generate_cos_values() and the df_cos output whose cosine similarities
    are validated against theoretical bounds; installed via pip from the project repo
  repo: https://github.com/MintaYLu/COT
- name: NumPy
  role: Numerical backend used by the COT module for computing cosine similarities
    and the XOR reference vector
- name: pandas
  role: Holds df_raw, df_mean and df_cos tables used in and produced by the validation
    workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbac037
    title: 'COT: an efficient and accurate method for detecting marker genes among
      many subtypes'
  source_content_hash: sha256:c8d5d9514cf397ede53703f18e017f625cb84f8ae90533ab1d067aa1825e0f76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MintaYLu/COT
  tools:
  - COT
  - NumPy
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# cosine-similarity-range-validation

## Summary

Validates that cosine similarity statistics computed by COT's generate_cos_values() lie within their theoretical bounds: cos ≤ 1 with 1 attained by ideal marker genes (MG), and, for the XOR-modified subtype-downregulated gene (SDG) statistic of Eq 7, strictly within (1/(K-1), 1) for K subtypes.

## When to use

Use when cot.df_cos has been produced by COT's generate_cos_values() (or the XOR-basis SDG variant) and the cosine values must be checked against theoretical bounds before ranking markers or writing sdg_cos_values.csv.

## When NOT to use

- cot.df_cos does not yet exist — the cosine statistics must first be computed with generate_cos_values() before ranges can be validated
- The gene-by-sample matrix is not batch-corrected, since COT's input is explicitly required to be batch-corrected and uncorrected values can distort cosine scores
- Data are replicate-level samples rather than aggregated subtype means and generate_subtype_means() has not yet been run with the subtype_label mapping

## Inputs

- cot.df_cos table (columns: gene, cos, subtype) from cot.generate_cos_values()
- cot.df_mean subtype-mean expression table from cot.generate_subtype_means()
- Number of subtypes K (derived from subtype_label mapping, e.g. {'A': ['S1','S2'], 'B': ['S3','S4']})

## Outputs

- Per-gene validation flags (rows with cos outside (0,1] for MG or outside (1/(K-1),1) for the SDG statistic)
- sdg_cos_values.csv (gene, cos, subtype) for validated XOR-basis SDG cosine values

## How to apply

Load the batch-corrected gene-by-sample matrix into a COT instance (COT(df_raw=df_raw, normalization=False)), aggregate replicate samples into subtype-level means via cot.generate_subtype_means(subtype_label=subtype_label), and run cot.generate_cos_values() to obtain the df_cos table (columns: gene, cos, subtype). Then verify, for each gene, that cos lies in (0, 1] for the standard MG statistic; for the SDG/XOR statistic (reference vector e_k XOR 1, i.e. 0 in exactly one subtype and 1 in all others), verify 1/(K-1) < t_COT(i) < 1 strictly. Cross-check the argmax subtype: an ideal MG hits cos = 1 (e.g. gene 1 in the README toy data with pattern 1,1,0,0 → cos 1.0, subtype A), while a non-marker (gene 0, expressed in all subtypes, cos 0.8, subtype B) must score below 1. Flag any row violating the bound or whose argmax subtype does not show near-zero mean expression in that subtype (MG case) / elevated expression in all others (SDG case).

## Related tools

- **COT** (Provides generate_cos_values() and the df_cos output whose cosine similarities are validated against theoretical bounds; installed via pip from the project repo) — https://github.com/MintaYLu/COT
- **NumPy** (Numerical backend used by the COT module for computing cosine similarities and the XOR reference vector)
- **pandas** (Holds df_raw, df_mean and df_cos tables used in and produced by the validation workflow)

## Examples

```
from COT.COT import COT; cot = COT(df_raw=df_raw, normalization=False); cot.generate_subtype_means(subtype_label=subtype_label); cot.generate_cos_values()
```

## Evaluation signals

- Every cos value in cot.df_cos satisfies 0 < cos ≤ 1 for the standard MG statistic
- For the XOR-based SDG statistic with K subtypes, every t_COT(i) lies strictly within (1/(K-1), 1)
- Genes with an ideal marker pattern (e.g. README gene 1: 1,1,0,0 across subtypes) attain cos = 1.0 exactly
- Non-marker genes expressed in all subtypes (e.g. README gene 0: 0.5,0.7,0.7,0.9) score cos < 1 (0.8 in the README example)
- Each gene assigned to its argmax subtype has mean expression near zero in that subtype (MG) or elevated in all other subtypes (SDG); rows failing this are flagged

## Limitations

- With too few genes, downstream p-value estimation cannot be performed ('Cannot calculate p-values due to the limited genes numbers'); NaNs must be removed before estimate_p_values()
- Input must be batch-corrected; uncorrected expression values can shift cosine scores and produce spurious failures or passes
- The COT instance is created with normalization=False, so raw input is used as-is — the analyst must ensure prior normalization outside COT

## Evidence

- [methods] COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic: "COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic"
- [readme] README toy example df_cos: "| gene | cos | subtype |
| ---- | --- | ------- |
| 0    | 0.8 | B       |
| 1    | 1.0 | A       |"
- [methods] p-value limitation: "Cannot calculate p-values due to the limited genes numbers, please see the Example_GSE28490 for p-values computation."
- [readme] batch correction requirement: "Note that the input of COT should be batch-corrected."
