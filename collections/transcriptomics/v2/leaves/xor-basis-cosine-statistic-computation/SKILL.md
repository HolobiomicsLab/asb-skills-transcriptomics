---
name: xor-basis-cosine-statistic-computation
description: Use when the goal is to detect subtype-downregulated genes (SDG) from
  batch-corrected gene-by-sample expression data — i.e., genes with near-zero mean
  expression in exactly one subtype and elevated expression in every other subtype.
when_to_use_negative:
- When the target is subtype-enriched marker genes (upregulated in one subtype) —
  use the standard COT ideal-MG reference vector instead of the XOR variant
- When the input matrix is not batch-corrected — COT explicitly requires batch-corrected
  input
- When only a very small number of genes are being tested and p-value estimation is
  required — COT cannot calculate p-values due to limited gene numbers in that case
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: COT
  role: Python module that computes the cosine-based one-sample test; its generate_subtype_means()
    and generate_cos_values() routines implement the subtype-mean aggregation and
    cosine statistic used here
  repo: https://github.com/MintaYLu/COT
- name: NumPy
  role: Vector backend on which COT is built; used to compute the XOR-basis cosine
    similarities
- name: pandas
  role: Holds the batch-corrected df_raw input and the cot.df_cos output table (gene,
    cos, subtype)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbac037
    title: 'COT: an efficient and accurate method for detecting marker genes among
      many subtypes'
  source_content_hash: sha256:d1fbc90b6a51a42b0cd8061b58a2f081e6a1e9e44bf2b4428e54c9e64a27e481
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MintaYLu/COT
  tools:
  - COT
  - NumPy
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# xor-basis-cosine-statistic-computation

## Summary

A modification of the COT cosine-based one-sample test in which the ideal-MG reference vector is replaced by an exclusive disjunction (XOR) with 1, so the reference is 0 in exactly one subtype and 1 in all others. This reorients the statistic to detect subtype-downregulated genes (SDG) — genes silent only in a given subtype but expressed in all others — with the resulting statistic bounded as 1/(K-1) < t_COT(i) < 1.

## When to use

Use when the goal is to detect subtype-downregulated genes (SDG) from batch-corrected gene-by-sample expression data — i.e., genes with near-zero mean expression in exactly one subtype and elevated expression in every other subtype.

## When NOT to use

- When the target is subtype-enriched marker genes (upregulated in one subtype) — use the standard COT ideal-MG reference vector instead of the XOR variant
- When the input matrix is not batch-corrected — COT explicitly requires batch-corrected input
- When only a very small number of genes are being tested and p-value estimation is required — COT cannot calculate p-values due to limited gene numbers in that case

## Inputs

- Batch-corrected gene-by-sample expression matrix (pandas DataFrame, e.g., Example_GSE28490 data; COT input must be batch-corrected)
- Subtype label vector mapping sample columns to subtypes (e.g., subtype_label = ["A", "A", "B", "B"])
- Number of subtypes K (to evaluate the 1/(K-1) lower bound)

## Outputs

- cot.df_cos table with columns gene, cos, subtype (argmax subtype per gene), exportable as sdg_cos_values.csv
- Per-gene XOR-basis cosine statistic t_COT(i) within (1/(K-1), 1)

## How to apply

Load the batch-corrected gene-by-sample expression matrix (e.g., Example_GSE28490) into pandas and instantiate the test as COT(df_raw=df_raw, normalization=False). Aggregate replicate sample columns into subtype-level mean profiles with cot.generate_subtype_means(subtype_label=subtype_label), which maps sample columns to subtypes (e.g., subtype_label=["A","A","B","B"] gives cot.subtypes {"A":["S1","S2"],"B":["S3","S4"]}). For each gene i, compute cos(s_i, e_k XOR 1) for every subtype k — the reference vector e_k XOR 1 is 0 in exactly one subtype and 1 in all others — and take argmax_k; reproduce this through the COT generate_cos_values() routine rather than hand-coding it. Write the resulting cot.df_cos table (columns: gene, cos, subtype) to sdg_cos_values.csv. The rationale is that the XOR reference vector encodes the SDG definition exactly as the original COT vector encodes the ideal marker gene definition, so the cosine similarity remains an exact-match test statistic.

## Related tools

- **COT** (Python module that computes the cosine-based one-sample test; its generate_subtype_means() and generate_cos_values() routines implement the subtype-mean aggregation and cosine statistic used here) — https://github.com/MintaYLu/COT
- **NumPy** (Vector backend on which COT is built; used to compute the XOR-basis cosine similarities)
- **pandas** (Holds the batch-corrected df_raw input and the cot.df_cos output table (gene, cos, subtype))

## Examples

```
from COT.COT import COT; cot = COT(df_raw=df_raw, normalization=False); cot.generate_subtype_means(subtype_label=subtype_label); cot.generate_cos_values()
```

## Evaluation signals

- Every cosine value lies strictly within (1/(K-1), 1) for K subtypes; flag any row outside this range
- For each gene assigned to its argmax subtype k, the subtype mean expression is near zero in subtype k and elevated in all other subtypes; otherwise flag the row
- cot.df_cos has exactly the schema gene, cos, subtype as documented in the COT README output example
- generate_subtype_means correctly maps sample columns to subtypes (e.g., subtype_label ["A","A","B","B"] yields cot.subtypes {"A": ["S1", "S2"], "B": ["S3", "S4"]})
- Results written to sdg_cos_values.csv are reproducible via the COT generate_cos_values() routine

## Limitations

- P-value estimation may fail when the number of genes is too small (e.g., the GSE28490 benchmark example); NaNs should be removed before estimate_p_values()
- The input matrix must be batch-corrected before applying COT
- COT complements rather than replaces existing marker-gene methods

## Evidence

- [other] The COT test statistic can be modified to detect SDG by replacing the ideal-MG reference vector via an exclusive disjunction (XOR) operation; the resulting statistic is bounded as 1/(K-1) < t_COT(i) < 1.: [evidence span withheld — anchor status: not_found]
- [methods] COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic: "COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic"
- [readme] Note that the input of COT should be batch-corrected.: "Note that the input of COT should be batch-corrected."
- [readme] Cannot calculate p-values due to the limited genes numbers, please see the Example_GSE28490 for p-values computation.: "Cannot calculate p-values due to the limited genes numbers, please see the Example_GSE28490 for p-values computation."
