---
name: rmse-correlation-performance-evaluation
description: Use when comparing competing marker gene sets (e.g., COT-derived via
  obtain_subtype_markers(), one-vs-rest t-test, a priori lists) for deconvolving bulk
  expression mixtures where true cell-type proportions are known, and a head-to-head
  accuracy table is needed.
when_to_use_negative:
- Ground-truth cell-type proportions are unknown (e.g., real bulk tissue samples),
  so RMSE against true proportions cannot be computed
- You only need to detect marker genes themselves (that is COT's obtain_subtype_markers()
  task), not to evaluate how well they support deconvolution
- Input is a single bulk sample rather than a panel of mixtures with known composition
  — sample-wise correlation over 100 mixtures is the design's core signal
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: COT
  role: supplies the COT-derived marker gene set via cot.obtain_subtype_markers();
    its performance is the hypothesis under test
  repo: https://github.com/MintaYLu/COT
- name: NumPy
  role: draws random mixture proportions, mixes subtype profiles, and runs least-squares
    deconvolution and RMSE computation
- name: pandas
  role: loads the GSE28490 subtype profiles and assembles/writes the Supplementary
    Table S2 summary CSV
- name: scikit-learn
  role: marker-gene-restricted linear regression (least squares) to estimate cell-type
    proportions from each mixture
- name: SciPy
  role: statistical support, e.g., sample-wise correlation computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac037
    title: 'COT: an efficient and accurate method for detecting marker genes among
      many subtypes'
  source_content_hash: sha256:dbe1c8bee99e943df146761c3b8f5591128c52a3454e64aa911245ed2b4f5f8b
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
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# rmse-correlation-performance-evaluation

## Summary

Benchmark skill for quantifying marker-gene-guided deconvolution accuracy: cell-type proportions estimated by marker-gene-restricted linear regression on simulated bulk mixtures are scored by RMSE against true proportions and by sample-wise Pearson correlation, as done for COT-MG versus OVR t-test and a priori marker sets (Supplementary Table S2 on GSE28490).

## When to use

Use when comparing competing marker gene sets (e.g., COT-derived via obtain_subtype_markers(), one-vs-rest t-test, a priori lists) for deconvolving bulk expression mixtures where true cell-type proportions are known, and a head-to-head accuracy table is needed.

## When NOT to use

- Ground-truth cell-type proportions are unknown (e.g., real bulk tissue samples), so RMSE against true proportions cannot be computed
- You only need to detect marker genes themselves (that is COT's obtain_subtype_markers() task), not to evaluate how well they support deconvolution
- Input is a single bulk sample rather than a panel of mixtures with known composition — sample-wise correlation over 100 mixtures is the design's core signal

## Inputs

- Subtype expression profiles of the GSE28490 five benchmark cell types (gene x subtype matrix, batch-corrected)
- Marker gene sets to compare: COT-derived markers (cot.obtain_subtype_markers()), one-vs-rest t-test markers, and the a priori marker list
- 100 randomly simulated bulk mixture expression profiles with known ground-truth cell-type proportions

## Outputs

- Per-marker-gene-set RMSE between true and estimated cell-type proportions
- Per-marker-gene-set summary of sample-wise correlations between true and estimated proportions
- Supplementary Table S2-style comparison table (e.g., supplementary_table_s2.csv)

## How to apply

Download the benchmark subtype expression profiles (GSE28490, five immune cell types) and load them with pandas. Simulate ~100 random bulk mixtures by drawing random cell-type proportions and combining the five subtype profiles with NumPy, keeping the true proportions as ground truth. For each candidate marker gene set, deconvolve each mixture by marker-gene-restricted least squares (NumPy or scikit-learn LinearRegression) restricted to that set's genes, yielding estimated proportions per cell type per mixture. For each marker gene set, compute the RMSE between true and estimated proportion vectors across all mixtures, and compute the sample-wise correlation (true vs estimated proportion vector within each mixture), then summarize correlations per marker gene set. Assemble RMSE and correlation summaries into a table (reproducing Supplementary Table S2) and write it out, e.g. supplementary_table_s2.csv; the winning set is the one with the lowest RMSE and highest sample-wise correlation.

## Related tools

- **COT** (supplies the COT-derived marker gene set via cot.obtain_subtype_markers(); its performance is the hypothesis under test) — https://github.com/MintaYLu/COT
- **NumPy** (draws random mixture proportions, mixes subtype profiles, and runs least-squares deconvolution and RMSE computation)
- **pandas** (loads the GSE28490 subtype profiles and assembles/writes the Supplementary Table S2 summary CSV)
- **scikit-learn** (marker-gene-restricted linear regression (least squares) to estimate cell-type proportions from each mixture)
- **SciPy** (statistical support, e.g., sample-wise correlation computation)

## Examples

```
cot = COT(df_raw=df_raw, normalization=False); cot.cos_pipeline(subtype_label=subtype_label, top=2); markers = cot.obtain_subtype_markers()
```

## Evaluation signals

- The COT-derived marker set yields the lowest RMSE across the 100 mixtures compared with OVR t-test and a priori sets (expected reproduction of Supplementary Table S2)
- COT-MG also achieves the highest sample-wise correlation between true and estimated proportions
- Deconvolution estimates are non-negative proportion-like values and, per mixture, true and estimated proportion vectors are over the same five GSE28490 cell types
- The output CSV has one RMSE entry and one correlation summary per marker gene set (three sets), matching Supplementary Table S2 structure
- Each deconvolution uses only the genes in the given marker set (marker-gene-restricted regression), not all genes

## Limitations

- COT cannot calculate p-values when gene numbers are too limited; NaNs must be removed before estimate_p_values(), and the GSE28490 example shows how to handle this
- COT input must be batch-corrected; uncorrected raw profiles can distort subtype means and cosine values feeding the marker sets
- Evaluation rests on randomly simulated mixtures from a single five-type benchmark; the article notes COT complements rather than replaces existing methods

## Evidence

- [results] task_005 finding: [evidence span withheld — anchor status: not_found]
- [supplementary] supplementary dataset: "Benchmark dataset used in this paper can be downloaded from Gene Expression Omnibus (https://www.ncbi.nlm.nih.gov/geo/) under the accession number: GSE28490."
- [methods] COT statistic: "COT uses the cosine similarity between a molecule's cross-subtype expression pattern and the exact mathematical definition of MG as the test statistic"
- [methods] p-value caveat: "Cannot calculate p-values due to the limited genes numbers, please see the Example_GSE28490 for p-values computation."
- [readme] batch correction: "Note that the input of COT should be batch-corrected."
