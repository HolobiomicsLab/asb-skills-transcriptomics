---
name: compositional-data-analysis
description: Use when per-sample cell-type counts from scRNA-seq (e.g., BALF donors
  split into healthy/moderate/severe groups) must be tested for differential abundance
  across conditions with few replicates, where univariate tests would be inadequate.
when_to_use_negative:
- Standard univariate tests (Poisson regression, Beta-Binomial regression, t tests,
  Wilcoxon rank-sum) are the goal and the compositional nature of the data is being
  ignored — these are inadequate because they do not account for compositionality.
- The analysis requires modeling response variability within a condition or detecting
  heterogeneities between samples in treatment response or donor variability — scCODA
  does not model this.
- Cell-type definitions come from clustering that is not statistically sound or biologically
  meaningful — scCODA requires pre-specified, reliable cell-type annotations.
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: scCODA
  role: Bayesian Dirichlet-Multinomial model with Logit-normal spike-and-slab prior
    used for compositional differential abundance testing
  repo: https://github.com/theislab/scCODA
- name: scanpy
  role: source of annotated single-cell data imported into scCODA via from_scanpy
  repo: https://scanpy.readthedocs.io
- name: TensorFlow
  role: backend (tensorflow >=2.8 with tensorflow-probability >=0.16) for HMC-based
    inference in scCODA
- name: pandas
  role: DataFrame input for cell count data via from_pandas
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:eb888c354d7494d104217b2fba78e6f4a6b1f68c1a08f276e197bae289e78179
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scCODA
  tools:
  - Python
  - TensorFlow
  - pandas
  - scCODA
  - scanpy
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# compositional-data-analysis

## Summary

Bayesian differential composition analysis of single-cell cell-type counts using scCODA, which models counts with a hierarchical Dirichlet-Multinomial distribution and a Logit-normal spike-and-slab prior to detect credible cell-type proportion shifts while controlling false discoveries under compositionality.

## When to use

Use when per-sample cell-type counts from scRNA-seq (e.g., BALF donors split into healthy/moderate/severe groups) must be tested for differential abundance across conditions with few replicates, where univariate tests would be inadequate.

## When NOT to use

- Standard univariate tests (Poisson regression, Beta-Binomial regression, t tests, Wilcoxon rank-sum) are the goal and the compositional nature of the data is being ignored — these are inadequate because they do not account for compositionality.
- The analysis requires modeling response variability within a condition or detecting heterogeneities between samples in treatment response or donor variability — scCODA does not model this.
- Cell-type definitions come from clustering that is not statistically sound or biologically meaningful — scCODA requires pre-specified, reliable cell-type annotations.

## Inputs

- cell-type annotated scRNA-seq data (scanpy AnnData objects, in-memory list or on-disk directory)
- per-sample cell-type count table as a pandas DataFrame
- sample-level covariates (e.g., disease condition: healthy/moderate/severe)
- pre-specified reference cell type (e.g., Plasma cells) or automatic reference selection

## Outputs

- CAResult object with an effects data frame whose 'Final Parameter' column indicates credible effects
- credible-effects table per covariate vs reference cell type at a given FDR threshold (e.g., covid_balf_sccoda_credible_effects.csv)
- compositional dataset (scCODA ComData) and optional visualizations via util.data_visualization

## How to apply

Aggregate annotated single-cell data into per-sample cell-type counts using sccoda.util.cell_composition_data.from_scanpy or from_pandas, then instantiate an scCODA model via sccoda.util.comp_ana.CompositionalAnalysis with a patsy formula encoding the condition covariate and either a pre-specified reference cell type (e.g., Plasma cells in the COVID-19 BALF analysis) or automatic reference selection. Run Bayesian inference with sample_hmc() (or sample_nuts()), then call summary() and credible_effects() on the CAResult object to identify credible effects at the chosen FDR threshold (e.g., 0.2). Interpretation of credible changes must always be relative to the selected reference. In the COVID-19 BALF reanalysis this correctly calls a neutrophil increase in healthy and moderate vs severe cases and an NK-cell increase in mild vs healthy.

## Related tools

- **scCODA** (Bayesian Dirichlet-Multinomial model with Logit-normal spike-and-slab prior used for compositional differential abundance testing) — https://github.com/theislab/scCODA
- **scanpy** (source of annotated single-cell data imported into scCODA via from_scanpy) — https://scanpy.readthedocs.io
- **TensorFlow** (backend (tensorflow >=2.8 with tensorflow-probability >=0.16) for HMC-based inference in scCODA)
- **pandas** (DataFrame input for cell count data via from_pandas)

## Examples

```
import sccoda; data = sccoda.util.cell_composition_data.from_scanpy([adata_h, adata_m, adata_s]); model = sccoda.util.comp_ana.CompositionalAnalysis(data, formula='condition', reference_cell_type='Plasma'); res = model.sample_hmc(); res.credible_effects()
```

## Evaluation signals

- The 'Final Parameter' column of the effects data frame must reproduce the credible-effect calls reported in Supplementary Data 4 for the compared conditions.
- At FDR 0.2 with Plasma as reference, the COVID-19 BALF analysis must call neutrophils credibly increased in healthy and moderate vs severe cases.
- The NK-cell increase (mild vs healthy) and neutrophil increases (healthy vs severe, moderate vs severe) must both be called credible.
- Model diagnostics from summary() should be inspected after sample_hmc() inference to confirm convergence of the spike-and-slab posterior.
- Benchmarks (reproducible via theislab/scCODA_reproducibility) should show scCODA reliably controlling the false discovery rate while retaining detection power.

## Limitations

- scCODA requires pre-specified cell-type definitions, which hinge on statistically sound and biologically meaningful clustering assignments.
- scCODA assumes a log-linear relationship between covariates and cell abundance, which may be mis-specified in some cases.
- scCODA does not model or infer any dependency structure among cell compositions beyond those induced by compositional effects.
- scCODA does not model response variability within a condition and cannot detect heterogeneities between samples in response to treatment or donor variability.

## Evidence

- [intro] scCODA models cell-type counts with a hierarchical Dirichlet-Multinomial distribution and uses a Logit-normal spike-and-slab prior: [evidence span withheld — anchor status: not_found]
- [methods] CompositionalAnalysis setup with patsy formula and reference: "Using the scCODA model is easiest by generating an instance of ``ana.CompositionalAnalysis``."
- [methods] HMC inference step: "Once the model is set up, inference via HMC sampling can be performed via ``sample_hmc()``."
- [intro] Interpretation relative to reference cell type: "scCODA can automatically select an appropriate cell type as the reference (Methods—“Automatic reference selection”) or uses a pre-specified reference cell type"
- [discussion] Inadequacy of univariate tests: "standard univariate tests, such as Poisson regression models, Beta-Binomial regression, or t tests are inadequate for cell-type analysis, since they do not account for the compositional nature of the"
- [discussion] COVID-19 neutrophil finding: "identified additional cell-type shifts that were confirmed by independent studies, including Treg cell enrichment in UC patients and neutrophils increase in severe COVID-19 cases."
- [readme] Reproducibility code availability: "Code for reproducing the analysis from the paper is available [here](https://github.com/theislab/scCODA_reproducibility)."
- [other] covid_balf data source: "zhangzlab/covid_balf"
