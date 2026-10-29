---
name: spike-and-slab-credibility-assessment
description: Use when scRNA-seq cell-type counts are available per sample (especially
  with few replicates, e.g. 4 healthy / 3 moderate / 6 severe COVID-19 BALF donors)
  and you need differential abundance calls that control false discoveries under compositionality.
when_to_use_negative:
- Cell-type clustering/annotation is not yet statistically sound — scCODA requires
  pre-specified cell-type definitions and inherits errors from poor clustering
- You need to detect donor-level heterogeneity or response variability within a condition
  — scCODA does not model response variability within a condition
- The relationship between covariates and cell abundance is strongly non-log-linear,
  which may mis-specify scCODA's log-linear model
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: scCODA
  role: Core Bayesian Dirichlet-multinomial model with Logit-normal spike-and-slab
    prior used to infer credible cell-type compositional effects
  repo: https://github.com/theislab/scCODA
- name: scanpy
  role: Source of annotated single-cell count data imported via from_scanpy into the
    compositional dataset
- name: tensorflow-probability (>=0.12)
  role: Backend enabling HMC sampling inference in sample_hmc()
- name: TensorFlow (>= 2.4)
  role: Backend dependency for the Bayesian inference machinery
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:a665d7db56de27e1359d303dbb13d89295c1068e76dea1ef9f97ba2ff7531bd5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scCODA
  tools:
  - Python
  - TensorFlow
  - TensorFlow (>= 2.4)
  - pandas
  - scCODA
  - scanpy
  - tensorflow-probability (>=0.12)
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# spike-and-slab-credibility-assessment

## Summary

Use scCODA's Bayesian Dirichlet-multinomial model with a Logit-normal spike-and-slab prior to call credible cell-type compositional shifts, interpreting all effects relative to a chosen reference cell type and controlling false discoveries via the FDR threshold on the Final Parameter column.

## When to use

Use when scRNA-seq cell-type counts are available per sample (especially with few replicates, e.g. 4 healthy / 3 moderate / 6 severe COVID-19 BALF donors) and you need differential abundance calls that control false discoveries under compositionality.

## When NOT to use

- Cell-type clustering/annotation is not yet statistically sound — scCODA requires pre-specified cell-type definitions and inherits errors from poor clustering
- You need to detect donor-level heterogeneity or response variability within a condition — scCODA does not model response variability within a condition
- The relationship between covariates and cell abundance is strongly non-log-linear, which may mis-specify scCODA's log-linear model

## Inputs

- Cell-type annotated single-cell data as scanpy AnnData objects (e.g. .h5ad) or a pandas DataFrame of per-sample cell-type counts
- Sample-level covariates (e.g. disease condition: healthy / moderate / severe)
- A pre-specified reference cell type (e.g. Plasma) or acceptance of automatic reference selection

## Outputs

- A CAResult object from CompositionalAnalysis with HMC-sampled posterior effects
- An effects data frame whose Final Parameter column indicates credible effects at the chosen FDR threshold (e.g. 0.2), exportable as a results CSV (e.g. covid_balf_sccoda_credible_effects.csv)

## How to apply

Aggregate per-sample cell-type counts into a compositional dataset with sccoda.util.cell_composition_data.from_scanpy or from_pandas, then instantiate sccoda.util.comp_ana.CompositionalAnalysis with a patsy formula encoding the condition covariate and a pre-specified reference cell type (e.g. Plasma cells for the covid_balf data). Run inference via sample_hmc() (or sample_nuts()), which fits the hierarchical Dirichlet-Multinomial likelihood with a Logit-normal spike-and-slab prior and log link. Call summary() and credible_effects() on the CAResult, applying a false-discovery-rate threshold of 0.2, and interpret increases/decreases strictly relative to the reference. For the COVID-19 BALF reanalysis this reproduces the credible neutrophil increase in healthy and moderate vs severe cases reported in Supplementary Data 4, and the NK-cell increase for mild vs healthy.

## Related tools

- **scCODA** (Core Bayesian Dirichlet-multinomial model with Logit-normal spike-and-slab prior used to infer credible cell-type compositional effects) — https://github.com/theislab/scCODA
- **scanpy** (Source of annotated single-cell count data imported via from_scanpy into the compositional dataset)
- **tensorflow-probability (>=0.12)** (Backend enabling HMC sampling inference in sample_hmc())
- **TensorFlow (>= 2.4)** (Backend dependency for the Bayesian inference machinery)

## Examples

```
from sccoda.util import comp_ana, cell_composition_data; data = cell_composition_data.from_scanpy(adata_list, covariate_df); model = comp_ana.CompositionalAnalysis(data, formula="condition", reference_cell_type="Plasma"); res = model.sample_hmc(); res.credible_effects(fdr_threshold=0.2)
```

## Evaluation signals

- The Final Parameter column of the effects data frame reproduces the credible-effect calls in Supplementary Data 4 for the pairwise comparisons
- Neutrophils are called credibly increased in healthy vs severe and moderate vs severe COVID-19 cases at FDR 0.2 using Plasma as reference
- NK cells are called credibly increased in mild vs healthy comparison
- Credible-effect calls change coherently when a different reference cell type is chosen (effects are relative, not absolute)
- HMC sampling completes without divergences and summary() reports non-zero Final Parameter values only for credible cell types

## Limitations

- Credible changes must be interpreted relative to the selected reference cell type; using a different reference changes the calls
- Standard univariate tests (Poisson regression, Beta-Binomial regression, t tests) are inadequate here because they ignore compositionality — comparisons against them are not apples-to-apples
- scCODA does not model the response variability within a condition and cannot detect heterogeneities between samples in response to treatment or donor variability
- Results hinge on statistically sound and biologically meaningful clustering assignments of cell types

## Evidence

- [intro] Model uses Logit-normal spike-and-slab prior and Dirichlet-Multinomial: "The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions"
- [methods] HMC sampling inference step: "Once the model is set up, inference via HMC sampling can be performed via ``sample_hmc()``."
- [methods] Result evaluation via summary(): "To see which effects were found to be significant, call ``summary()`` on the result object."
- [intro] Reference cell type interpretation: "scCODA can automatically select an appropriate cell type as the reference (Methods—“Automatic reference selection”) or uses a pre-specified reference cell type11"
- [discussion] COVID-19 neutrophil finding: "identified additional cell-type shifts that were confirmed by independent studies, including Treg cell enrichment in UC patients and neutrophils increase in severe COVID-19 cases."
- [abstract] False discovery control claim: "scCODA demonstrated excellent detection performance, while reliably controlling for false discoveries, and identified experimentally verified cell type changes"
