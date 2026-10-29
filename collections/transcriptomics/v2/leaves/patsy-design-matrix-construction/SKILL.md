---
name: patsy-design-matrix-construction
description: Use when fitting a scCODA Dirichlet-Multinomial model to cell-type count
  data and a covariate matrix must be constructed from a formula (e.g. 'x_0') with
  a specified or automatically selected reference cell type.
when_to_use_negative:
- The input is raw scRNA-seq expression data without cell-type cluster assignments
  — scCODA requires pre-specified, statistically sound cell-type definitions
- The counts are not compositional cell-type proportions (e.g. a feature/abundance
  table from metagenomics with a different modeling goal)
- Only a single covariate-free dataset with no condition annotation exists, so no
  patsy formula can be specified
tools:
- name: scCODA
  role: Consumes the patsy formula and reference cell type to build the design matrix
    inside ana.CompositionalAnalysis before HMC sampling
  repo: https://github.com/theislab/scCODA
- name: patsy
  role: Parses the R-style formula string into the covariate (design) matrix columns
- name: scanpy
  role: Source of cell-type annotated count data transformed into the compositional
    anndata object via from_scanpy_list/from_scanpy_dir
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:83132feaca7097b5b46c99457774a976cb90770fdca537bcc70585aaa341f70a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scCODA
  tools:
  - Python
  - TensorFlow
  - arviz
  - patsy
  - scCODA
  - scanpy
  - tensorflow-probability
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# patsy-design-matrix-construction

## Summary

Builds the covariate (design) matrix for the scCODA compositional model by parsing a patsy-style formula applied to a compositional anndata object, while fixing the reference cell type whose effects are constrained to zero so all cell-type shifts are interpreted relative to it.

## When to use

Use when fitting a scCODA Dirichlet-Multinomial model to cell-type count data and a covariate matrix must be constructed from a formula (e.g. 'x_0') with a specified or automatically selected reference cell type.

## When NOT to use

- The input is raw scRNA-seq expression data without cell-type cluster assignments — scCODA requires pre-specified, statistically sound cell-type definitions
- The counts are not compositional cell-type proportions (e.g. a feature/abundance table from metagenomics with a different modeling goal)
- Only a single covariate-free dataset with no condition annotation exists, so no patsy formula can be specified

## Inputs

- Compositional analysis anndata object of cell-type counts (from sccoda.util.data_io.from_pandas or from_scanpy/from_scanpy_list/from_scanpy_dir)
- patsy formula string naming covariates (e.g. 'x_0' for a binary condition covariate)
- reference_cell_type specification (a cell-type name or 'Automatic')

## Outputs

- Design/covariate matrix for the Bayesian model, with the reference cell type's effect column constrained to zero
- A configured sccoda ana.CompositionalAnalysis model object ready for sample_hmc() inference

## How to apply

Take the compositional anndata object produced from pandas or scanpy (via from_pandas / from_scanpy_list / from_scanpy_dir) and pass a patsy formula string plus a reference_cell_type argument when generating an instance of sccoda's ana.CompositionalAnalysis. The patsy formula defines the covariate columns (binary or continuous) that enter the hierarchical Dirichlet-Multinomial model with normal-prior intercepts N(0,5) and a Logit-normal spike-and-slab prior on effects. Fixing the reference cell type enforces that its effects are constrained to zero, which makes the model identifiable because credible changes must be interpreted relative to the reference. If no reference is pre-specified, scCODA can automatically select an appropriate cell type as the reference. The resulting design matrix feeds directly into HMC inference via sample_hmc() (or sample_nuts()).

## Related tools

- **scCODA** (Consumes the patsy formula and reference cell type to build the design matrix inside ana.CompositionalAnalysis before HMC sampling) — https://github.com/theislab/scCODA
- **patsy** (Parses the R-style formula string into the covariate (design) matrix columns)
- **scanpy** (Source of cell-type annotated count data transformed into the compositional anndata object via from_scanpy_list/from_scanpy_dir)

## Examples

```
model = sccoda.model.CompositionalAnalysis(cell_counts, formula="x_0", reference_cell_type="Automatic")
```

## Evaluation signals

- The reference cell type's effect parameters are exactly zero in the constructed model (identifiability constraint respected)
- The design matrix columns match the covariates named in the patsy formula, with correct dummy/binary encoding for condition variables
- Downstream summary() on the result object produces an effects data frame with a populated 'Final Parameter' column (0 = not credible, nonzero = credible) interpreted relative to the reference cell type
- The model object supports sample_hmc()/sample_nuts() without errors related to formula parsing or missing covariates

## Limitations

- Credible changes can only be interpreted relative to the chosen reference cell type, so a poor reference choice distorts the interpretation of all effects
- scCODA assumes a log-linear relationship between covariates and cell abundance, which the formula-based design matrix encodes and which may be mis-specified in some cases
- Dependencies among cell compositions beyond compositional effects are not modeled

## Evidence

- [methods] Model setup via CompositionalAnalysis with patsy formula and reference cell type: "Using the scCODA model is easiest by generating an instance of ``ana.CompositionalAnalysis``."
- [intro] Reference-relative interpretation of credible effects: "scCODA can automatically select an appropriate cell type as the reference (Methods—“Automatic reference selection”) or uses a pre-specified reference cell type11"
- [intro] Model likelihood and prior on effects that the design matrix parameterizes: "The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions"
- [methods] Data import sources feeding the compositional anndata object: "You can either import data directly from a pandas DataFrame via ``from_pandas``"
