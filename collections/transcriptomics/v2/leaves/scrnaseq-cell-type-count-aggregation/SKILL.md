---
name: scrnaseq-cell-type-count-aggregation
description: Use when you have cell-type-annotated scRNA-seq data across multiple
  samples/conditions and need to build a cell count matrix (e.g., via sccoda.util.cell_composition_data.from_pandas
  or from_scanpy) as input for compositional differential abundance analysis.
when_to_use_negative:
- Input is already a samples-by-cell-types count table in the correct compositional
  anndata format — re-import is unnecessary
- Data is raw single-cell expression matrices without cell-type annotations or sample
  grouping; clustering/annotation must happen first since scCODA requires pre-specified
  cell-type definitions
- The analysis goal is per-cell gene-level differential expression rather than sample-level
  cell-type composition
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scCODA
  role: Provides from_pandas, from_scanpy, from_scanpy_list and from_scanpy_dir in
    sccoda.util.cell_composition_data to transform annotated single-cell data into
    the compositional analysis dataset
  repo: https://github.com/theislab/scCODA
- name: Scanpy
  role: Source of the annotated single-cell expression data; scCODA's framework is
    fully integrated into the Scanpy ecosystem and can ingest AnnData objects directly
  repo: https://scanpy.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:06cdf04cf9d183eead185c73d46c32e6c766242ba8b4e2965fd74c43357072a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scCODA
  tools:
  - Python (>=3.8)
  - Scanpy
  - scCODA
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# scrnaseq-cell-type-count-aggregation

## Summary

Aggregate cell-type-annotated single-cell RNA-seq data into a samples-by-cell-types count matrix (NxK) with sample covariates, formatted as a scCODA compositional analysis anndata object so differential composition models can be fit. This step condenses raw cell-level annotations into the count-level input required by compositional methods such as scCODA.

## When to use

Use when you have cell-type-annotated scRNA-seq data across multiple samples/conditions and need to build a cell count matrix (e.g., via sccoda.util.cell_composition_data.from_pandas or from_scanpy) as input for compositional differential abundance analysis.

## When NOT to use

- Input is already a samples-by-cell-types count table in the correct compositional anndata format — re-import is unnecessary
- Data is raw single-cell expression matrices without cell-type annotations or sample grouping; clustering/annotation must happen first since scCODA requires pre-specified cell-type definitions
- The analysis goal is per-cell gene-level differential expression rather than sample-level cell-type composition

## Inputs

- Cell-type-annotated scanpy AnnData objects (in-memory or on disk) with one cell per row and sample/covariate metadata
- Pre-aggregated NxK cell count matrix as a pandas DataFrame with sample covariate columns (e.g., age_group for supercentenarian PBMC data from http://gerg.gsc.riken.jp/SC2018/)

## Outputs

- scCODA compositional analysis anndata object with data.X (NxK cell counts) and data.obs (sample covariates)
- Optionally, compositional data visualization plots (boxplots/barplots of per-sample cell-type proportions)

## How to apply

Start from per-cell annotations grouped by sample (e.g., the supercentenarian PBMC dataset with 7 supercentenarians and 5 younger controls). If counts are already tabulated in a pandas DataFrame, build the compositional dataset with sccoda.util.cell_composition_data.from_pandas, passing the NxK cell count matrix into data.X and sample covariates (e.g., an age-group indicator) into data.obs; alternatively use from_scanpy (or from_scanpy_list/from_scanpy_dir) to read cell-type annotations directly from in-memory AnnData objects or files on disk. Verify the resulting object holds integer counts per sample per cell type and that covariate columns needed for the downstream patsy formula (e.g., 'age_group') are present. Inspect the aggregated compositions with sccoda.util.data_visualization before model setup via ana.CompositionalAnalysis. The aggregated count matrix is what the Dirichlet-multinomial model with Logit-normal spike-and-slab prior consumes, so errors in aggregation propagate directly to credible-effect calls.

## Related tools

- **scCODA** (Provides from_pandas, from_scanpy, from_scanpy_list and from_scanpy_dir in sccoda.util.cell_composition_data to transform annotated single-cell data into the compositional analysis dataset) — https://github.com/theislab/scCODA
- **Scanpy** (Source of the annotated single-cell expression data; scCODA's framework is fully integrated into the Scanpy ecosystem and can ingest AnnData objects directly) — https://scanpy.readthedocs.io/en/stable/

## Examples

```
from sccoda.util.cell_composition_data import from_pandas; data_scc = from_pandas(count_df, covariate_columns=["age_group"])
```

## Evaluation signals

- data.X is an integer-valued NxK matrix where N = number of samples (e.g., 12 for 7 supercentenarians + 5 controls) and K = number of cell types
- data.obs contains the covariate column referenced by the downstream patsy formula (e.g., 'age_group') with correct sample-level grouping
- Row sums of data.X match total cell counts per sample in the original annotated dataset; no cells are dropped or double-counted
- Compositional visualization of data proportions shows expected per-sample cell-type distributions before running sample_hmc()
- Re-running from_pandas/from_scanpy on the same input yields an identical count matrix (deterministic aggregation)

## Limitations

- scCODA requires pre-specified cell-type definitions, which hinge on statistically sound and biologically meaningful clustering assignments — poor annotations corrupt the aggregated counts
- scCODA does not model response variability within a condition, so aggregation loses donor-level heterogeneity information beyond the count matrix
- Aggregated proportions retain only compositional information; scCODA does not model dependency structure among cell types beyond that induced by compositionality

## Evidence

- [methods] from_pandas import: "You can either import data directly from a pandas DataFrame via ``from_pandas``"
- [methods] from_scanpy: "``from_scanpy_list`` (for in-memory data) and ``from_scanpy_dir`` (for data stored on disk) can transform the information from these files directly into a compositional analysis dataset"
- [intro] scanpy integration: "the framework offers access to other well-established compositional test statistics and is fully integrated into the Scanpy12 ecosystem"
- [readme] readme scanpy integration: "It also provides a framework for integration of cell-type annotated data directly from [scanpy](https://scanpy.readthedocs.io/en/stable/) and other sources."
- [intro] downstream model: "The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions"
