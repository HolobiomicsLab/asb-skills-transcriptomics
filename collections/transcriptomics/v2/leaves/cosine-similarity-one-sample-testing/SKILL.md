---
name: cosine-similarity-one-sample-testing
description: Use when a batch-corrected multi-subtype expression or proteomics matrix
  (e.g. FP/FS/NL vascular specimens) must be screened for subtype-specific marker
  proteins/genes and existing test statistics do not satisfy the MG definition.
when_to_use_negative:
- Input data is not batch-corrected — the article explicitly notes 'the input of COT
  should be batch-corrected', so applying COT to uncorrected batches is invalid.
- The dataset has too few genes/molecules to support the empirical-null approximation
  — p-value prediction fails and the marker detection cannot be significance-tested.
- The goal is clustering or subtype discovery rather than marker detection among already-defined
  subtypes — COT requires a pre-specified subtype_label and complements rather than
  replaces existing methods.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: COT
  role: Python module implementing the cosine-based one-sample test; provides COT(),
    generate_subtype_means, generate_cos_values, estimate_p_values, obtain_subtype_markers,
    plot_heatmap, plot_simplex and cos_pipeline
  repo: https://github.com/MintaYLu/COT
- name: NumPy
  role: Core dependency COT is built on top of (>= 1.19.5) for the vector/cosine computations
  repo: https://github.com/MintaYLu/COT
- name: pandas
  role: Core dependency (>= 1.2.0) holding df_raw, df_mean and df_cos tables
  repo: https://github.com/MintaYLu/COT
- name: SciPy
  role: Required dependency (>= 1.3.1) for COT's one-sample test statistics
  repo: https://github.com/MintaYLu/COT
- name: statsmodels
  role: Dependency (>= 0.10.1) supporting FDR q-value estimation in estimate_p_values
  repo: https://github.com/MintaYLu/COT
- name: seaborn
  role: Dependency (>= 0.11.1) for rendering the subtype-marker heatmap via plot_heatmap()
  repo: https://github.com/MintaYLu/COT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac037
    title: 'COT: an efficient and accurate method for detecting marker genes among
      many subtypes'
  source_content_hash: sha256:6cc9234e1021788c9548a410386afb46eeecb60a9374c2119751ffa98505ef2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MintaYLu/COT
  tools:
  - COT
  - NumPy
  - Python
  - SciPy
  - matplotlib
  - pandas
  - scikit-learn
  - seaborn
  - statsmodels
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# cosine-similarity-one-sample-testing

## Summary

Marker-gene (MG) detection using the Cosine-based One-sample Test (COT), which scores each molecule by the cosine similarity between its cross-subtype expression profile and the exact mathematical definition of an ideal marker gene, then assesses significance against an approximated empirical null distribution.

## When to use

Use when a batch-corrected multi-subtype expression or proteomics matrix (e.g. FP/FS/NL vascular specimens) must be screened for subtype-specific marker proteins/genes and existing test statistics do not satisfy the MG definition.

## When NOT to use

- Input data is not batch-corrected — the article explicitly notes 'the input of COT should be batch-corrected', so applying COT to uncorrected batches is invalid.
- The dataset has too few genes/molecules to support the empirical-null approximation — p-value prediction fails and the marker detection cannot be significance-tested.
- The goal is clustering or subtype discovery rather than marker detection among already-defined subtypes — COT requires a pre-specified subtype_label and complements rather than replaces existing methods.

## Inputs

- df_raw: batch-corrected gene/protein x sample expression matrix (pandas DataFrame, genes as rows)
- subtype_label: list assigning each sample to a subtype (e.g. ["A","A","B","B"] or FP/FS/NL labels)

## Outputs

- cot.df_mean: subtype mean expression profiles per gene
- cot.df_cos: per-gene cosine similarity values with assigned subtype, extended with p.value and q.value after estimate_p_values()
- cot.markers: dictionary of subtype-to-marker-gene lists (e.g. {"A": [1], "B": [0]})
- Heatmap (cot.plot_heatmap()) and scatter-space simplex plot (cot.plot_simplex()) of subtype markers

## How to apply

Instantiate the COT class on batch-corrected raw expression data with COT(df_raw=df_raw, normalization=False), since COT requires batch-corrected input. Pass a subtype_label vector to generate_subtype_means() to compute per-subtype mean profiles, then generate_cos_values() to score every molecule's cosine similarity to the ideal MG pattern and assign each gene its best subtype. Call estimate_p_values() to obtain empirical-null p-values and FDR q-values, removing NaN genes beforehand because too few genes prevent p-value prediction; if gene numbers are too limited (as for GSE28490), fall back to the Example_GSE28490 procedure. Finally, obtain_subtype_markers() returns the subtype marker lists, which are rendered with plot_heatmap() (and optionally plot_simplex()); the single-step cos_pipeline(subtype_label=..., top=...) reproduces the same output.

## Related tools

- **COT** (Python module implementing the cosine-based one-sample test; provides COT(), generate_subtype_means, generate_cos_values, estimate_p_values, obtain_subtype_markers, plot_heatmap, plot_simplex and cos_pipeline) — https://github.com/MintaYLu/COT
- **NumPy** (Core dependency COT is built on top of (>= 1.19.5) for the vector/cosine computations) — https://github.com/MintaYLu/COT
- **pandas** (Core dependency (>= 1.2.0) holding df_raw, df_mean and df_cos tables) — https://github.com/MintaYLu/COT
- **SciPy** (Required dependency (>= 1.3.1) for COT's one-sample test statistics) — https://github.com/MintaYLu/COT
- **statsmodels** (Dependency (>= 0.10.1) supporting FDR q-value estimation in estimate_p_values) — https://github.com/MintaYLu/COT
- **seaborn** (Dependency (>= 0.11.1) for rendering the subtype-marker heatmap via plot_heatmap()) — https://github.com/MintaYLu/COT

## Examples

```
from COT.COT import COT; cot = COT(df_raw=df_raw, normalization=False); cot.cos_pipeline(subtype_label=subtype_label, top=2)
```

## Evaluation signals

- Marker counts per subtype match the reference (e.g. 50 FP, 2 FS and 8 NL marker proteins on the pure vascular proteomics cohort, per Table 1 / Figure 3A heatmap).
- Each gene in cot.df_cos has exactly one assigned subtype and a cosine value in [0, 1]; an ideal marker gene yields cosine 1.0 for its subtype.
- After estimate_p_values(), every retained gene carries both a p.value and an FDR q.value with no NaN remaining (NaN genes must be removed before this step).
- cross-cohort consistency: markers detected on the n=10 pure cohort overlap consistently with markers from the independent heterogeneous cohort (n=78, Herrington et al. 2018).
- The cos_pipeline shortcut produces the same cot.markers dictionary as running the individual steps sequentially.

## Limitations

- Too few genes may prevent p-value prediction — 'Cannot calculate p-values due to the limited genes numbers'; NaN genes must be removed before estimate_p_values().
- The empirical-null approximation assumes 'a significant majority of genes are associated with the null hypothesis'; data with many true markers may violate this.
- COT input must already be batch-corrected; COT performs no normalization or batch correction itself (normalization=False).
- COT complements rather than replaces existing marker-detection methods.

## Evidence

- [methods] COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic: "COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic"
- [methods] empirical null: "Under the assumption that a significant majority of genes are associated with the null hypothesis, COT approximates the empirical null distribution for calculating p-values."
- [readme] batch-corrected input: "Note that the input of COT should be batch-corrected."
- [readme] p-value failure mode: "Attention: too few genes may not work for predicting p-values. Please remove NaN before this step."
- [readme] pipeline invocation: "cot = COT(df_raw=df_raw, normalization=False)
    cot.cos_pipeline(subtype_label=subtype_label, top=2)"
