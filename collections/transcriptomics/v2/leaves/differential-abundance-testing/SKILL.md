---
name: differential-abundance-testing
description: Use when testing which cell types credibly change abundance between conditions
  (e.g., COVID-19 BALF severity groups) from per-sample cell-type counts, especially
  with few replicates where univariate tests give false positives.
when_to_use_negative:
- When cell-type clustering/annotation is not yet statistically sound and biologically
  meaningful — scCODA requires pre-specified cell-type definitions and cannot infer
  them
- When the question concerns within-condition response variability or donor heterogeneity
  in treatment response — scCODA does not model response variability within a condition
- When the relationship between covariates and cell abundance is expected to be non-log-linear
  — scCODA assumes a log-linear model that may be mis-specified
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: scCODA
  role: Core Bayesian Dirichlet-Multinomial model with spike-and-slab prior used for
    compositional differential abundance testing and credible-effect calling
  repo: https://github.com/theislab/scCODA
- name: scanpy
  role: Source of annotated single-cell data imported into scCODA's compositional
    dataset via from_scanpy/from_scanpy_list/from_scanpy_dir
- name: TensorFlow
  role: Backend (tensorflow >= 2.4/2.8 with tensorflow-probability) supporting HMC
    inference via sample_hmc()
- name: theislab/scCODA_reproducibility
  role: Code repository reproducing the paper's analyses, including the COVID-19 BALF
    scCODA reanalysis
  repo: https://github.com/theislab/scCODA_reproducibility
- name: pertpy
  role: Maintained Jax-based successor hosting the newer version of the scCODA model
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:9fc9ddb85c6333ba5529a671d2fd351daab60c0a929c955360599c34c84c7f4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scCODA
  tools:
  - Python
  - TensorFlow
  - pandas
  - pertpy
  - scCODA
  - scanpy
  - theislab/scCODA_reproducibility
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# differential-abundance-testing

## Summary

Bayesian compositional differential abundance testing of cell-type proportions from scRNA-seq using scCODA, which models cell counts with a hierarchical Dirichlet-Multinomial and a Logit-normal spike-and-slab prior to control false discoveries that plague univariate proportion tests.

## When to use

Use when testing which cell types credibly change abundance between conditions (e.g., COVID-19 BALF severity groups) from per-sample cell-type counts, especially with few replicates where univariate tests give false positives.

## When NOT to use

- When cell-type clustering/annotation is not yet statistically sound and biologically meaningful — scCODA requires pre-specified cell-type definitions and cannot infer them
- When the question concerns within-condition response variability or donor heterogeneity in treatment response — scCODA does not model response variability within a condition
- When the relationship between covariates and cell abundance is expected to be non-log-linear — scCODA assumes a log-linear model that may be mis-specified

## Inputs

- Cell-type-annotated scRNA-seq data as scanpy AnnData objects (in-memory or on disk) or a pandas DataFrame of per-sample cell-type counts
- Per-sample covariates (e.g., disease condition: healthy/moderate/severe) with donor-level grouping (e.g., 13 BALF donors: 4 healthy, 3 moderate, 6 severe)
- A pre-specified reference cell type (e.g., Plasma cells) and a patsy formula string encoding covariates

## Outputs

- A CAResult object with an effects data frame whose Final Parameter column gives credible-effect calls per cell type and covariate
- Credible-effects table at the chosen FDR threshold (e.g., covid_balf_sccoda_credible_effects.csv) interpretable only relative to the reference cell type

## How to apply

Aggregate annotated single-cell data into per-sample cell-type counts using sccoda.util.cell_composition_data (from_scanpy or from_pandas). Instantiate sccoda.util.comp_ana.CompositionalAnalysis with a patsy formula encoding the condition covariate and specify a reference cell type (in the COVID-19 BALF analysis, Plasma cells), since credible effects are only interpretable relative to this reference. Run inference via sample_hmc() (or sample_nuts()), which fits the Dirichlet-Multinomial model with Logit-normal spike-and-slab prior and log link using TensorFlow/tensorflow-probability. Call summary() and credible_effects() on the CAResult object, applying the false-discovery-rate threshold of 0.2 to read credible effects from the Final Parameter column of the effects data frame. In the COVID-19 BALF reanalysis (4 healthy, 3 moderate, 6 severe donors from zhangzlab/covid_balf), this correctly recovers a credible neutrophil increase in healthy and moderate vs severe cases relative to Plasma, and an NK-cell increase in mild vs healthy, matching Supplementary Data 4.

## Related tools

- **scCODA** (Core Bayesian Dirichlet-Multinomial model with spike-and-slab prior used for compositional differential abundance testing and credible-effect calling) — https://github.com/theislab/scCODA
- **scanpy** (Source of annotated single-cell data imported into scCODA's compositional dataset via from_scanpy/from_scanpy_list/from_scanpy_dir)
- **TensorFlow** (Backend (tensorflow >= 2.4/2.8 with tensorflow-probability) supporting HMC inference via sample_hmc())
- **theislab/scCODA_reproducibility** (Code repository reproducing the paper's analyses, including the COVID-19 BALF scCODA reanalysis) — https://github.com/theislab/scCODA_reproducibility
- **pertpy** (Maintained Jax-based successor hosting the newer version of the scCODA model)

## Examples

```
import sccoda; data = sccoda.util.cell_composition_data.from_scanpy(adata_list, covariate_key='condition'); model = sccoda.util.comp_ana.CompositionalAnalysis(data, formula='condition', reference_cell_type='Plasma'); res = model.sample_hmc(); print(res.summary()); print(res.credible_effects())
```

## Evaluation signals

- The Final Parameter column of the effects data frame must reproduce the credible-effect calls in Supplementary Data 4 for the COVID-19 BALF comparisons (neutrophil increase in healthy vs severe and moderate vs severe; NK-cell increase in mild vs healthy)
- Credible effects are called at FDR level 0.2 and must be interpreted strictly relative to the pre-specified reference cell type (Plasma)
- Inference converges via HMC sampling (sample_hmc) without divergences, and the effects data frame schema (incl. Final Parameter and credible-effects entries) is well-formed
- Reference-based invariance: cell types other than the reference show no credible calls when their proportions change only due to compositional shifts
- The credible-effects table exports reproducibly to covid_balf_sccoda_credible_effects.csv from the theislab/scCODA_reproducibility workflow

## Limitations

- Requires pre-specified cell-type definitions that hinge on statistically sound and biologically meaningful clustering assignments
- Assumes a log-linear relationship between covariates and cell abundance, which may be mis-specified in some cases
- Does not model or infer dependency structure among cell compositions beyond those induced by compositional effects
- Does not model response variability within a condition and cannot detect heterogeneities between samples in response to treatment or donor variability
- The original TensorFlow implementation is no longer maintained; a new Jax version is available in pertpy, and GPU computation with TensorFlow has not been tested with scCODA

## Evidence

- [intro] scCODA models cell-type counts with a hierarchical Dirichlet-Multinomial distribution and uses a Logit-normal spike-and-slab prior: "The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions"
- [intro] scCODA can automatically select an appropriate cell type as the reference or use a pre-specified reference cell type, and credible changes must be interpreted relative to the selected reference: "scCODA can automatically select an appropriate cell type as the reference (Methods—“Automatic reference selection”) or uses a pre-specified reference cell type11"
- [methods] Model setup via CompositionalAnalysis, HMC sampling inference, and result evaluation via summary(): "Once the model is set up, inference via HMC sampling can be performed via ``sample_hmc()``."
- [methods] The Final Parameter column of the effects data frame shows the significances.: "The ``Final Parameter`` column of the effects data frame shows the significances."
- [discussion] scCODA identified additional cell-type shifts confirmed by independent studies, including neutrophils increase in severe COVID-19 cases: "identified additional cell-type shifts that were confirmed by independent studies, including Treg cell enrichment in UC patients and neutrophils increase in severe COVID-19 cases."
- [discussion] scCODA requires pre-specified cell-type definitions: "the scCODA framework requires pre-specified cell-type definitions which, in turn, hinge on statistically sound and biologically meaningful clustering assignments."
- [readme] Reproducibility code available in theislab/scCODA_reproducibility: "Code for reproducing the analysis from the paper is available [here](https://github.com/theislab/scCODA_reproducibility)."
