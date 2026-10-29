---
name: marker-gene-detection-cosine-similarity
description: Use when a batch-corrected gene/protein expression matrix with per-sample
  subtype labels is available and you need to identify subtype-specific marker genes
  across many subtypes, where one-vs-rest statistics (OVR t-test, Limma/EdgeR) yield
  inaccurate MG.
when_to_use_negative:
- The expression matrix has not been batch-corrected — COT explicitly expects batch-corrected
  input and offers no internal batch correction
- The dataset contains very few genes, so the empirical null distribution cannot be
  estimated and p-values/q-values cannot be computed (see the GSE28490 example for
  the workaround)
- The goal is differential expression between exactly two conditions, where standard
  one-vs-rest methods suffice and COT's many-subtype formulation adds no benefit (COT
  'complements not replaces' existing methods)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: COT
  role: Core Python module implementing the cosine-based one-sample test for marker
    gene detection
  repo: https://github.com/MintaYLu/COT
- name: Python
  role: Runtime environment (>= 3.7.4) hosting the COT module and pipeline calls
  repo: https://github.com/MintaYLu/COT
- name: NumPy
  role: Numerical backend COT is built on top of for the scatter-space cosine computation
- name: pandas
  role: DataFrame structures for df_raw, df_mean, and df_cos tables flowing through
    the pipeline
- name: SciPy
  role: Statistical dependency (>= 1.3.1) of COT used for p-value estimation machinery
- name: statsmodels
  role: Dependency (>= 0.10.1) supporting q-value / multiple-testing adjustment in
    estimate_p_values
- name: scikit-learn
  role: Dependency (>= 0.21.3) of the COT module
- name: R
  role: Used to run peer marker-detection methods (Limma, EdgeR) for benchmarking
    COT's marker lists
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac037
    title: 'COT: an efficient and accurate method for detecting marker genes among
      many subtypes'
  source_content_hash: sha256:9b685ea6436113b4d6f9777fb7833f9f0651de2754a126c51b19cc8ac7bd7e10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MintaYLu/COT
  tools:
  - COT
  - NumPy
  - Python
  - R
  - SciPy
  - matplotlib
  - pandas
  - scikit-learn
  - seaborn
  - statsmodels
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# marker-gene-detection-cosine-similarity

## Summary

Detects subtype-enriched marker genes (MG) among many subtypes by formulating detection as a one-sample Cosine-based One-sample Test (COT): the cosine similarity between a gene's cross-subtype expression pattern and the ideal MG definition is the test statistic, with p-values drawn from an approximated empirical null distribution.

## When to use

Use when a batch-corrected gene/protein expression matrix with per-sample subtype labels is available and you need to identify subtype-specific marker genes across many subtypes, where one-vs-rest statistics (OVR t-test, Limma/EdgeR) yield inaccurate MG.

## When NOT to use

- The expression matrix has not been batch-corrected — COT explicitly expects batch-corrected input and offers no internal batch correction
- The dataset contains very few genes, so the empirical null distribution cannot be estimated and p-values/q-values cannot be computed (see the GSE28490 example for the workaround)
- The goal is differential expression between exactly two conditions, where standard one-vs-rest methods suffice and COT's many-subtype formulation adds no benefit (COT 'complements not replaces' existing methods)

## Inputs

- Batch-corrected raw expression matrix (df_raw: pandas DataFrame, rows = genes, columns = samples; normalization handled by caller)
- Per-sample subtype label vector (subtype_label), e.g. ['A','A','B','B'], aligned to df_raw columns

## Outputs

- cot.subtypes: mapping of subtype label to member sample names
- cot.df_mean: per-subtype mean expression per gene
- cot.df_cos: per-gene cosine statistic, best subtype, p.value and q.value columns
- cot.markers: dict of subtype to detected marker genes (from obtain_subtype_markers)
- Simplex and heatmap plots (plot_simplex, plot_heatmap)

## How to apply

Install COT (pip install git+https://github.com/MintaYLu/COT.git; requires python >= 3.7.4, numpy >= 1.19.5, scipy >= 1.3.1, pandas >= 1.2.0, statsmodels, scikit-learn). Input a batch-corrected raw expression matrix (df_raw, genes x samples) plus a subtype_label vector; set normalization=False since COT expects pre-normalized, batch-corrected input. Run the stepwise pipeline — generate_subtype_means, generate_cos_values, estimate_p_values, obtain_subtype_markers — or the single call cos_pipeline(subtype_label, top=N); subtype means are computed per label group, the cosine statistic is evaluated per gene-subtype pair, and q-value thresholds (or top-k per subtype) select the final marker lists. Remove NaN rows before estimate_p_values, because too few genes prevents fitting the empirical null; the GSE28490 example illustrates p-value computation on limited gene counts. Validate by benchmarking the resulting marker lists against a priori MG subsets or peer methods using the quantitative performance index P1 (COT reached P1 = 0.019 vs OVR t-test 0.069 vs a priori subset 0.118 on GSE28490).

## Related tools

- **COT** (Core Python module implementing the cosine-based one-sample test for marker gene detection) — https://github.com/MintaYLu/COT
- **Python** (Runtime environment (>= 3.7.4) hosting the COT module and pipeline calls) — https://github.com/MintaYLu/COT
- **NumPy** (Numerical backend COT is built on top of for the scatter-space cosine computation)
- **pandas** (DataFrame structures for df_raw, df_mean, and df_cos tables flowing through the pipeline)
- **SciPy** (Statistical dependency (>= 1.3.1) of COT used for p-value estimation machinery)
- **statsmodels** (Dependency (>= 0.10.1) supporting q-value / multiple-testing adjustment in estimate_p_values)
- **scikit-learn** (Dependency (>= 0.21.3) of the COT module)
- **R** (Used to run peer marker-detection methods (Limma, EdgeR) for benchmarking COT's marker lists)

## Examples

```
from COT.COT import COT; cot = COT(df_raw=df_raw, normalization=False); cot.cos_pipeline(subtype_label=subtype_label, top=2)
```

## Evaluation signals

- cot.markers is populated for every subtype in subtype_label and each marker's df_cos row carries a significant q.value at the chosen threshold
- df_mean contains one column per unique subtype label and df_cos contains one best-subtype assignment per gene
- No NaN rows remain before estimate_p_values; NaN removal was done explicitly since 'too few genes may not work for predicting p-values'
- On GSE28490, the P1 performance index of COT markers (~0.019) is substantially lower than OVR t-test (~0.069) and the a priori MG subset (~0.118)
- plot_simplex/plot_heatmap render without error, confirming df_cos and cot.markers are well-formed

## Limitations

- Input must be batch-corrected beforehand; COT does not correct batch effects itself
- P-values cannot be calculated when the number of genes is too limited (the empirical null is ill-estimated); the Example_GSE28490 vignette covers this case
- COT complements rather than replaces existing marker detection methods, per the authors

## Evidence

- [methods] COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic: "COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic"
- [readme] Under the assumption that a significant majority of genes are associated with the null hypothesis, COT approximates the empirical null distribution for calculating p-values.: "Under the assumption that a significant majority of genes are associated with the null hypothesis, COT approximates the empirical null distribution for calculating p-values."
- [readme] Note that the input of COT should be batch-corrected.: "Note that the input of COT should be batch-corrected."
- [readme] Attention: too few genes may not work for predicting p-values. Please remove NaN before this step.: "Attention: too few genes may not work for predicting p-values. Please remove NaN before this step."
- [abstract] Nevertheless, COT complements not replaces existing methods.: "Nevertheless, COT complements not replaces existing methods."
