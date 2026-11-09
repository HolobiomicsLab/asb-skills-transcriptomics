---
name: concordance-correlation-metric-computation
description: Use when comparing estimated cell-type proportions from deconvolution
  or semi-profiling against real-profiled ground-truth proportions across pseudobulk
  samples, and you need agreement metrics (CCC, Pearson, RMSE) plus method-level significance
  testing.
when_to_use_negative:
- The input is raw bulk or single-cell expression data rather than paired estimated-vs-ground-truth
  proportion vectors — deconvolution must be run first before metrics can be computed.
- The analysis goal is single-cell-resolution downstream work (UMAP, pathway activation,
  pseudotime) rather than aggregate cell-type proportion accuracy.
- The cohort has no matched real single-cell profiles, so no ground-truth proportions
  exist to compute CCC/Pearson/RMSE against.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: SciPy
  role: Computes the Pearson correlation coefficient and the one-sided Wilcoxon test
    used for pairwise method significance
- name: Scikit-learn
  role: Trains the MLP classifier that annotates generated cells so cell-type proportions
    can be computed before metric evaluation
- name: scSemiProfiler
  role: Produces the estimated single-cell data and cell-type proportions (28- and
    4-representative reference variants) evaluated by the metrics
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: CIBERSORTx
  role: Competing deconvolution method whose estimated proportions are scored with
    CCC, Pearson, and RMSE
- name: MuSiC
  role: Competing deconvolution method whose package also supplies the NNLS baseline
    evaluated with the same metrics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:d20ece84c7bf626b433e3228bf3496c55d90e23e34ea6ecb1ad9c950477f6139
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - Bisque
  - CIBERSORTx
  - EPIC
  - MuSiC
  - NNLS
  - Scaden
  - SciPy
  - Scikit-learn
  - TAPE
  - scSemiProfiler
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# concordance-correlation-metric-computation

## Summary

Compute Lin's concordance correlation coefficient (CCC), Pearson correlation, and RMSE to quantify agreement between estimated and ground-truth cell-type proportions, as done in the scSemiProfiler deconvolution benchmark on the 124-sample COVID-19 pseudobulk cohort.

## When to use

Use when comparing estimated cell-type proportions from deconvolution or semi-profiling against real-profiled ground-truth proportions across pseudobulk samples, and you need agreement metrics (CCC, Pearson, RMSE) plus method-level significance testing.

## When NOT to use

- The input is raw bulk or single-cell expression data rather than paired estimated-vs-ground-truth proportion vectors — deconvolution must be run first before metrics can be computed.
- The analysis goal is single-cell-resolution downstream work (UMAP, pathway activation, pseudotime) rather than aggregate cell-type proportion accuracy.
- The cohort has no matched real single-cell profiles, so no ground-truth proportions exist to compute CCC/Pearson/RMSE against.

## Inputs

- Per-sample estimated cell-type proportion matrix from each deconvolution method (124 COVID-19 pseudobulk samples x cell types)
- Ground-truth cell-type proportions from real-profiled single-cell data of the same 124 pseudobulk samples
- Bulk pseudobulk expression profiles for the cohort used to run the deconvolution methods

## Outputs

- deconvolution_benchmark.csv with one row per method-condition (9 rows: scSemiProfiler-28, scSemiProfiler-4, CIBERSORTx, Bisque, Scaden, TAPE, EPIC, NNLS, MuSiC) and CCC, Pearson, RMSE, and Wilcoxon p-value columns

## How to apply

Annotate the inferred (semi-profiled) single cells — in the COVID-19 cohort this is done with a Scikit-learn MLP classifier trained on the representatives' annotated data — and aggregate per-sample cell type proportions over all 124 pseudobulk samples. Run each deconvolution method under matched capacity conditions: scSemiProfiler with the full 28-representative reference and a fair-comparison 4-representative variant, CIBERSORTx via its web portal, Bisque/TAPE/MuSiC/EPIC from their official GitHub repositories, NNLS from the MuSiC package, and Scaden via the PyTorch version in TAPE's repository (trained on only 5000 sampled cells). For each method, compute Lin's CCC, the Pearson correlation coefficient (SciPy), and RMSE of estimated versus ground-truth proportions across samples. Assess pairwise significance between methods with a one-sided Wilcoxon test implemented in SciPy. Write the results as one row per method-condition (9 rows: scSemiProfiler-28, scSemiProfiler-4, and the 7 competing methods) with CCC, Pearson, RMSE, and Wilcoxon p-value columns in deconvolution_benchmark.csv.

## Related tools

- **SciPy** (Computes the Pearson correlation coefficient and the one-sided Wilcoxon test used for pairwise method significance)
- **Scikit-learn** (Trains the MLP classifier that annotates generated cells so cell-type proportions can be computed before metric evaluation)
- **scSemiProfiler** (Produces the estimated single-cell data and cell-type proportions (28- and 4-representative reference variants) evaluated by the metrics) — https://github.com/mcgilldinglab/scSemiProfiler
- **CIBERSORTx** (Competing deconvolution method whose estimated proportions are scored with CCC, Pearson, and RMSE)
- **MuSiC** (Competing deconvolution method whose package also supplies the NNLS baseline evaluated with the same metrics)

## Examples

```
from scipy.stats import pearsonr, wilcoxon; r, _ = pearsonr(pred_props, true_props); p = wilcoxon(err_methodA, err_methodB, alternative='less').pvalue
```

## Evaluation signals

- deconvolution_benchmark.csv exists with exactly 9 method-condition rows (scSemiProfiler-28, scSemiProfiler-4, and 7 competing methods).
- CCC values fall within [-1, 1] and RMSE values are non-negative; Pearson r and CCC should agree in sign for each method.
- Per-sample estimated proportion vectors sum to 1 across cell types before metrics are computed.
- Metric values match the article's Source Data for Fig. 2j-l of doi 10.1038/s41467-024-50150-1.
- Wilcoxon p-values are one-sided and correspond to pairwise method comparisons, not per-method self-tests.

## Limitations

- Ground-truth proportions require real single-cell profiling of the same cohort; without matched real-profiled samples the metrics cannot be computed.
- Fair comparison requires equalizing single-cell reference capacity — scSemiProfiler must also be run with its reference confined to the first 4 representatives to match the capacity of the competing methods.
- Scaden's PyTorch implementation samples only 5000 cells for training, and CIBERSORTx is run through a web portal, so competitor implementations carry practical constraints that affect comparability.
- The tool requires a single-cell reference to perform single-cell level deconvolution, limiting application to settings where such a reference exists.

## Evidence

- [methods] Compute Lin's CCC, Pearson correlation coefficient (SciPy), and RMSE for each method's estimated proportions versus the ground-truth proportions across the cohort's pseudobulk samples.: [evidence span withheld — anchor status: not_found]
- [methods] Assess pairwise statistical significance between methods with a one-sided Wilcoxon test implemented in SciPy.: [evidence span withheld — anchor status: not_found]
- [discussion] The tool’s deep generative model enables “single-cell level deconvolution” across various studies, provided there is a single-cell reference.: "The tool’s deep generative model enables “single-cell level deconvolution” across various studies, provided there is a single-cell reference."
- [readme] Once the pipeline is completed, single-cell data for the target sample is generated and can be used for cell type annotation.: [evidence span withheld — anchor status: external_doc]
