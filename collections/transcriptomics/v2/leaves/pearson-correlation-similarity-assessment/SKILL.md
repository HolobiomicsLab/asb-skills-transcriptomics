---
name: pearson-correlation-similarity-assessment
description: Use when validating that computationally inferred (semi-profiled) single-cell
  data reproduces the ground-truth cohort structure, by correlating per-condition
  cell-type proportion vectors and pathway activation scores between two cohorts.
when_to_use_negative:
- When the goal is to capture variability within cell types — aggregate proportion
  correlations operate at cell-type-level resolution and cannot attest to single-cell-level
  agreement
- When the two cohorts cannot be merged on a shared condition axis (e.g., mismatched
  severity labels), since correlations are computed per condition
- When data have not passed semi-profiling QC (library size normalization, log1p,
  highly variable gene selection), as raw counts will distort proportion and pathway-score
  estimates
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scSemiProfiler
  role: Produces the semi-profiled cohort and provides the cell-type-proportion utility
    used to compute per-sample/per-condition proportions
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: SciPy
  role: scipy.stats.pearsonr computes the Pearson correlation coefficients between
    semi-profiled and real-profiled proportion and pathway-score vectors
- name: SCANPY
  role: tl.score_genes computes IFN pathway activation scores per cell type and severity
    for both cohorts
- name: scikit-learn
  role: MLP classifier assigns cell types to semi-profiled cells before proportion
    calculation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:da39457dd3b01785e8ff33b7308b88ce19ab3777abe039101d9b8545b8419133
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - SCANPY
  - SciPy
  - scSemiProfiler
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# pearson-correlation-similarity-assessment

## Summary

Quantifies similarity between semi-profiled and real-profiled single-cell cohorts by computing per-condition Pearson correlations of cell-type proportions (via scipy.stats.pearsonr) and of IFN pathway activation scores, writing the metrics to similarity_metrics.csv.

## When to use

Use when validating that computationally inferred (semi-profiled) single-cell data reproduces the ground-truth cohort structure, by correlating per-condition cell-type proportion vectors and pathway activation scores between two cohorts.

## When NOT to use

- When the goal is to capture variability within cell types — aggregate proportion correlations operate at cell-type-level resolution and cannot attest to single-cell-level agreement
- When the two cohorts cannot be merged on a shared condition axis (e.g., mismatched severity labels), since correlations are computed per condition
- When data have not passed semi-profiling QC (library size normalization, log1p, highly variable gene selection), as raw counts will distort proportion and pathway-score estimates

## Inputs

- Semi-profiled COVID-19 cohort (inferred single-cell data from the upstream scSemiProfiler inference step)
- Real-profiled ground-truth single-cell dataset (COVID-19 dataset, preprocessed as detailed by ref. 45)
- Bulk data for the cohort and annotated representative single-cell data for training the MLP cell-type classifier
- Interferon pathway gene set (e.g., curated gene sets from MSigDB)

## Outputs

- similarity_metrics.csv with exactly 6 condition rows (Healthy, Asymptomatic, Mild, Moderate, Severe, Critical) and columns for the cell-type proportion Pearson and the IFN pathway Pearson
- Per-condition cell-type proportion tables for semi-profiled and real-profiled cohorts
- IFN pathway activation scores (fold change relative to healthy) per cell type and severity for both cohorts

## How to apply

Merge the semi-profiled cohort (produced by the scSemiProfiler VAE-GAN inference step) with the real-profiled ground-truth dataset on severity condition (Healthy, Asymptomatic, Mild, Moderate, Severe, Critical). Annotate semi-profiled cells with a scikit-learn MLP classifier trained on the annotated representative single-cell data, then compute per-sample and per-condition cell-type proportions with the scSemiProfiler cell-type-proportion utility. For each of the 6 conditions, calculate the Pearson correlation between semi-profiled and real-profiled proportion vectors using scipy.stats.pearsonr. In parallel, compute interferon pathway activation scores for both cohorts with SCANPY's tl.score_genes following the COVID-19 source-study procedure, averaging gene-set expression per cell type and severity and taking fold changes relative to the healthy condition, then correlate those scores between cohorts. Write all metrics to similarity_metrics.csv with exactly 6 condition rows and columns for the cell-type proportion Pearson and the IFN pathway Pearson.

## Related tools

- **scSemiProfiler** (Produces the semi-profiled cohort and provides the cell-type-proportion utility used to compute per-sample/per-condition proportions) — https://github.com/mcgilldinglab/scSemiProfiler
- **SciPy** (scipy.stats.pearsonr computes the Pearson correlation coefficients between semi-profiled and real-profiled proportion and pathway-score vectors)
- **SCANPY** (tl.score_genes computes IFN pathway activation scores per cell type and severity for both cohorts)
- **scikit-learn** (MLP classifier assigns cell types to semi-profiled cells before proportion calculation)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(semiprofiled_proportions, realprofiled_proportions)
```

## Evaluation signals

- similarity_metrics.csv contains exactly 6 rows (one per severity condition) and the required cell-type-proportion and IFN-pathway Pearson columns
- All 6 cell-type proportion Pearson values exceed 0.95 (published values: 0.987 Healthy, 0.970 Asymptomatic, 0.996 Mild, 0.992 Moderate, 0.978 Severe, 0.989 Critical)
- The IFN pathway Pearson correlation falls within 0.75–0.85
- Conditions merge cleanly on the six severity labels (Healthy, Asymptomatic, Mild, Moderate, Severe, Critical) with no unmatched samples
- Stacked bar plots of proportions per disease condition show semi-profiled and real-profiled cohorts nearly identical, matching the ground-truth visual check

## Limitations

- Cell-type proportion and pathway-score correlations are cohort-level summaries; they do not demonstrate true single-cell resolution agreement within cell types
- The deep generative model requires a single-cell reference to perform single-cell level deconvolution, so this validation is only possible once representative profiling is done
- No software versions for scSemiProfiler or dependencies are stated, and no accession IDs for the evaluated cohorts are given in the discussion, affecting reproducibility of the comparison

## Evidence

- [readme] Stacked bar plots indicate that the semi-profiled cohort has nearly identical cell type proportions across disease conditions compared to the real-profiled ground truth.: [evidence span withheld — anchor status: external_doc]
- [discussion] The striking resemblance between semi-profiled and real-profiled data in facets like UMAP visualization and cell type proportions ascertains the robustness of scSemiProfiler.: "The striking resemblance between semi-profiled and real-profiled data in facets like UMAP visualization and cell type proportions ascertains the robustness of scSemiProfiler."
- [abstract] Extensive validation across heterogeneous datasets verifies the precision of the semi-profiling approach, aligning closely with true single-cell profiling data.: [evidence span withheld — anchor status: not_found]
- [intro] conventional methods are constrained to cell type level resolution and are incapable of achieving the true single-cell resolution required to capture the significant variability within cell types: "conventional methods are constrained to cell type level resolution and are incapable of achieving the true single-cell resolution required to capture the significant variability within cell types"
- [methods] gene set scores are calculated using all curated gene sets from MSigDB59: "gene set scores are calculated using all curated gene sets from MSigDB59 and Ernst et al.91"
