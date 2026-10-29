---
name: semi-profiling-accuracy-evaluation
description: Use when a semi-profiled (AI-inferred) single-cell cohort has been generated
  from bulk data plus representative samples and must be validated against a real-profiled
  ground-truth cohort before downstream use.
when_to_use_negative:
- Input lacks a real-profiled ground-truth cohort to compare against — the skill is
  a semi-profiled vs real-profiled comparison, not a standalone QC
- The cohort has not passed through the upstream scSemiProfiler inference step, so
  no semi-profiled cells exist to annotate and compare
- Goal is bulk-level cell-type deconvolution evaluation (CIBERSORTx, Bisque, DWLS,
  MuSiC, NNLS) rather than single-cell-resolution semi-profiling fidelity
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSemiProfiler
  role: Produces the semi-profiled cohort under evaluation and provides the cell-type-proportion
    utility used to compute per-sample/per-condition proportions
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: SciPy
  role: scipy.stats.pearsonr computes the per-condition Pearson correlation between
    semi-profiled and real-profiled proportion vectors
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:40decbe4cfc3b1bbe511d879866d268f26689a3e2fe6c471e4721e1961aa12cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - SciPy
  - scSemiProfiler
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# semi-profiling-accuracy-evaluation

## Summary

Quantifies how faithfully scSemiProfiler-inferred (semi-profiled) single-cell cohorts reproduce their real-profiled ground truth by comparing per-condition cell-type proportions and pathway activation scores via Pearson correlation. Anchored on the COVID-19 cohort benchmark where all six condition-level proportion correlations exceeded 0.95.

## When to use

Use when a semi-profiled (AI-inferred) single-cell cohort has been generated from bulk data plus representative samples and must be validated against a real-profiled ground-truth cohort before downstream use.

## When NOT to use

- Input lacks a real-profiled ground-truth cohort to compare against — the skill is a semi-profiled vs real-profiled comparison, not a standalone QC
- The cohort has not passed through the upstream scSemiProfiler inference step, so no semi-profiled cells exist to annotate and compare
- Goal is bulk-level cell-type deconvolution evaluation (CIBERSORTx, Bisque, DWLS, MuSiC, NNLS) rather than single-cell-resolution semi-profiling fidelity

## Inputs

- Semi-profiled COVID-19 single-cell cohort produced by the upstream scSemiProfiler inference step (single-cell expression data with severity condition labels)
- Real-profiled ground-truth single-cell dataset (annotated single-cell expression data across the same six severity conditions)
- Annotated representative single-cell data for training the MLP cell-type classifier
- Interferon pathway gene set per the COVID-19 dataset source study

## Outputs

- similarity_metrics.csv with exactly 6 condition rows and columns for cell-type proportion Pearson and IFN pathway Pearson
- Per-condition cell-type proportion Pearson correlation coefficients (6 values)
- IFN pathway activation Pearson correlation between semi-profiled and real-profiled cohorts

## How to apply

Load the semi-profiled cohort from the upstream scSemiProfiler inference step and the real-profiled ground-truth single-cell dataset, merging on disease-severity condition (Healthy, Asymptomatic, Mild, Moderate, Severe, Critical). Annotate semi-profiled cells by training a scikit-learn MLP classifier on the annotated representative single-cell data, then compute cell-type proportions per sample and per condition with the scSemiProfiler cell-type-proportion utility. For each condition, compute the Pearson correlation between semi-profiled and real-profiled proportion vectors using scipy.stats.pearsonr; in the COVID-19 benchmark these were 0.987 (Healthy), 0.970 (Asymptomatic), 0.996 (Mild), 0.992 (Moderate), 0.978 (Severe), and 0.989 (Critical). Additionally, compute interferon pathway activation scores for both cohorts per the source study's procedure — averaging gene-set expression per cell type and severity with SCANPY's tl.score_genes and taking fold changes relative to the healthy condition — then correlate IFN scores across cohorts. Write all metrics to similarity_metrics.csv with exactly 6 condition rows and columns for both Pearson metrics; validate the row count, that all proportion correlations exceed 0.95, and that the IFN pathway Pearson falls within 0.75–0.85.

## Related tools

- **scSemiProfiler** (Produces the semi-profiled cohort under evaluation and provides the cell-type-proportion utility used to compute per-sample/per-condition proportions) — https://github.com/mcgilldinglab/scSemiProfiler
- **SciPy** (scipy.stats.pearsonr computes the per-condition Pearson correlation between semi-profiled and real-profiled proportion vectors)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(semi_props, real_props)
```

## Evaluation signals

- similarity_metrics.csv has exactly 6 rows (one per severity condition: Healthy, Asymptomatic, Mild, Moderate, Severe, Critical) and the required metric columns
- All 6 cell-type proportion Pearson correlations exceed 0.95 (observed range 0.970–0.996 in the COVID-19 benchmark)
- IFN pathway Pearson correlation falls within 0.75–0.85
- Semi-profiled cell-type proportions per disease condition track the real-profiled ground truth in stacked bar comparison
- MLP classifier applied to semi-profiled cells yields cell-type assignments consistent with the annotated representative reference data

## Limitations

- Fidelity depends on having a single-cell reference; the tool requires a single-cell reference to perform single-cell level deconvolution
- Validation thresholds are benchmark-specific to the COVID-19 cohort (124 samples); other cohorts may show different correlation ranges
- Method evaluation covered three distinct cohorts in the article, but no named datasets or accession IDs are given in the discussion section for the evaluated cohorts
- Supports scRNA-seq only; extension to proteomics and other modalities is future work

## Evidence

- [results] Per-condition cell-type proportion Pearson correlations: [evidence span withheld — anchor status: not_found]
- [methods] For each of the 6 severity conditions: [evidence span withheld — anchor status: not_found]
- [discussion] Discussion claim of resemblance: "The striking resemblance between semi-profiled and real-profiled data in facets like UMAP visualization and cell type proportions ascertains the robustness of scSemiProfiler."
- [readme] README stacked bar observation: [evidence span withheld — anchor status: external_doc]
- [methods] Gene set score computation basis: "gene set scores are calculated using all curated gene sets from MSigDB59 and Ernst et al.91"
