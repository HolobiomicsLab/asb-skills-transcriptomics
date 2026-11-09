---
name: single-cell-rna-seq-preprocessing-and-clustering
description: Use when raw single-cell expression files must be filtered, clustered
  into cell-type/subpopulation labels (e.g., microglia vs. DAM), and aggregated into
  per-sample counts feeding a compositional differential-abundance model such as scCODA.
when_to_use_negative:
- Cell counts are already aggregated per sample into a pandas DataFrame — import directly
  via from_pandas instead of re-clustering
- The analysis goal is gene-level differential expression rather than cell-type compositional
  change
- Clustering assignments are exploratory or not biologically validated, since unreliable
  labels propagate into false compositional effects
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scanpy
  role: Preprocessing, filtering, and clustering of scRNA-seq expression data into
    cell subpopulations (e.g., microglia subsets); scCODA data import via from_scanpy
  repo: https://scanpy.readthedocs.io
- name: scCODA
  role: Downstream Bayesian compositional analysis consuming the clustered, per-sample
    cell-count data
  repo: https://github.com/theislab/scCODA
- name: Python (>=3.8)
  role: Runtime environment required to run the preprocessing and clustering pipeline
- name: tensorflow-probability (>=0.12)
  role: Backend supporting HMC/NUTS inference in scCODA after clustering and data
    import
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:851faf33bce7a5e3f450b4f0e4cb1fac153e710e37eb61e7f59318901b5ecb20
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python (>=3.8)
  - scCODA
  - scanpy
  - tensorflow-probability (>=0.12)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# single-cell-rna-seq-preprocessing-and-clustering

## Summary

Preprocess raw scRNA-seq count matrices (e.g., GEO accession GSE98969) with Scanpy, retaining normalization-free counts, and cluster cells into biologically meaningful subpopulations (e.g., microglia subsets including disease-associated microglia, DAM) so that per-sample cell compositions can be modeled by scCODA.

## When to use

Use when raw single-cell expression files must be filtered, clustered into cell-type/subpopulation labels (e.g., microglia vs. DAM), and aggregated into per-sample counts feeding a compositional differential-abundance model such as scCODA.

## When NOT to use

- Cell counts are already aggregated per sample into a pandas DataFrame — import directly via from_pandas instead of re-clustering
- The analysis goal is gene-level differential expression rather than cell-type compositional change
- Clustering assignments are exploratory or not biologically validated, since unreliable labels propagate into false compositional effects

## Inputs

- Raw scRNA-seq expression matrices/files from GEO (e.g., accession GSE98969)
- Per-sample metadata and covariates (brain region: cortex vs. cerebellum; disease condition)
- Scanpy AnnData objects with cell annotations

## Outputs

- Filtered, count-preserving expression matrices with cluster/subpopulation labels (microglia 1, DAM)
- Per-sample cell-count composition anndata from sccoda.util.cell_composition_data.from_scanpy
- credible_effects_summary.csv containing the scCODA summary table

## How to apply

Download the raw scRNA-seq files for the accession of interest (e.g., GSE98969 from GEO) and preprocess the expression matrices, filtering low-quality entries while retaining raw, normalization-free counts per Methods. Cluster cells into subpopulations per the paper's Methods using Scanpy — here, microglia subpopulations including tissue-resident microglia 1 and disease-associated microglia (DAM) — because scCODA 'requires pre-specified cell-type definitions which hinge on statistically sound and biologically meaningful clustering assignments'. Aggregate per-sample cell counts and covariates into the scCODA anndata structure via sccoda.util.cell_composition_data.from_scanpy, choosing the reference cell type (microglia 1) at model setup. Downstream, run HMC inference via sample_hmc() (or sample_nuts()), which takes up to ~5 minutes per dataset, and evaluate with set_fdr(target_fdr_level=0.2), credible_effects(), and summary().

## Related tools

- **scanpy** (Preprocessing, filtering, and clustering of scRNA-seq expression data into cell subpopulations (e.g., microglia subsets); scCODA data import via from_scanpy) — https://scanpy.readthedocs.io
- **scCODA** (Downstream Bayesian compositional analysis consuming the clustered, per-sample cell-count data) — https://github.com/theislab/scCODA
- **Python (>=3.8)** (Runtime environment required to run the preprocessing and clustering pipeline)
- **tensorflow-probability (>=0.12)** (Backend supporting HMC/NUTS inference in scCODA after clustering and data import)

## Examples

```
from sccoda.util.cell_composition_data import from_scanpy; dat = from_scanpy(adata_list)
```

## Evaluation signals

- credible_effects() output at FDR 0.2 matches Supplementary Data 2: credible DAM increase in cortex and no credible change in cerebellum relative to microglia 1
- The 'Final Parameter' column of the scCODA summary shows DAM as credibly increased only in cortex
- Count matrices remain normalization-free raw counts after filtering, as required by the workflow
- Clusters reproduce the Methods-defined microglia subpopulations (microglia 1 and DAM), i.e., clustering is statistically sound and biologically meaningful
- Summary table saved as credible_effects_summary.csv is complete and interpretable via summary()

## Limitations

- scCODA requires pre-specified cell-type definitions, so clustering errors or biologically meaningless assignments directly undermine the compositional results
- scCODA does not model response variability within a condition and cannot detect heterogeneities between samples in response to treatment or donor variability
- The theislab/scCODA implementation is no longer maintained; a Jax-based successor is available in pertpy

## Evidence

- [discussion] Clustering quality gates downstream inference: "the scCODA framework requires pre-specified cell-type definitions which, in turn, hinge on statistically sound and biologically meaningful clustering assignments"
- [methods] from_scanpy converts Scanpy data into the compositional dataset: "``from_scanpy`` generates a compositional analysis dataset from this"
- [methods] Count data source is Scanpy: "get the count data from single-cell expression data used in `scanpy <https://scanpy.readthedocs.io>`_"
- [readme] Environment requirement from README: [evidence span withheld — anchor status: external_doc]
- [readme] Package deprecated in favor of pertpy: "This implementation is no longer maintained. A new version in Jax is available in [pertpy]"
