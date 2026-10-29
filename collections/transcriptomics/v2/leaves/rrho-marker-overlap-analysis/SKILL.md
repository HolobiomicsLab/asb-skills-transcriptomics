---
name: rrho-marker-overlap-analysis
description: Use when two single-cell cohorts of the same biological system must be
  compared at the level of per-cell-type differential marker rankings and downstream
  enrichment results, to verify that a computational surrogate (e.g., scSemiProfiler
  output) reproduces ground-truth analyses.
when_to_use_negative:
- When the goal is quantitative assessment of the semi-profiling pipeline itself (active
  learning selection, VAE-GAN inference) rather than downstream analysis fidelity
  comparison
- When only one dataset is available and no paired semi-profiled/real-profiled comparison
  can be made
- When cell types have not been annotated in both datasets, since marker ranking and
  RRHO require matched per-cell-type gene rankings
edam_operation: http://edamontology.org/operation_2495
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCANPY
  role: Identify per-cell-type top 50 positive/negative markers and top 100 signature
    genes in both semi-profiled and real-profiled cohorts
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: GSEApy
  role: Run GO/Reactome enrichment on the top-100 signature gene sets and compute
    enriched-term overlap
- name: SciPy
  role: Hypergeometric-test p-value computation for RRHO plots and Pearson correlations
    for fidelity statistics
- name: scSemiProfiler
  role: Produces the semi-profiled cohort whose marker/enrichment results are compared
    against the real-profiled ground truth
  repo: https://github.com/mcgilldinglab/scSemiProfiler
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:3dd276c688565a3a1e2ad32038476e6c2c46898a3aaed7d26bc7b519365ac46d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - CellChat
  - GSEApy
  - Monocle3
  - SCANPY
  - SciPy
  - scSemiProfiler
  edam_operation: http://edamontology.org/operation_2495
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# RRHO marker overlap analysis

## Summary

Quantifies agreement between cell-type marker/signature gene sets derived from two single-cell datasets (e.g., semi-profiled vs. real-profiled COVID-19 cohorts) using rank-rank hypergeometric overlap (RRHO) plots and GO/Reactome enrichment overlap statistics.

## When to use

Use when two single-cell cohorts of the same biological system must be compared at the level of per-cell-type differential marker rankings and downstream enrichment results, to verify that a computational surrogate (e.g., scSemiProfiler output) reproduces ground-truth analyses.

## When NOT to use

- When the goal is quantitative assessment of the semi-profiling pipeline itself (active learning selection, VAE-GAN inference) rather than downstream analysis fidelity comparison
- When only one dataset is available and no paired semi-profiled/real-profiled comparison can be made
- When cell types have not been annotated in both datasets, since marker ranking and RRHO require matched per-cell-type gene rankings

## Inputs

- Semi-profiled single-cell cohort produced by scSemiProfiler (e.g., COVID-19 cohort, 124 samples)
- Corresponding real-profiled (ground-truth) single-cell cohort
- Cell type annotations for both datasets

## Outputs

- RRHO plots of hypergeometric-test p-values between ranked marker lists per cell type
- Per-cell-type top 50 positive/negative marker lists from SCANPY
- GO/Reactome enrichment overlap statistics (e.g., 95/100 top-100 signature gene overlap, p = 9.10e-196)
- downstream_fidelity.csv aggregating RRHO, enrichment, cell-cell interaction, and pseudotime agreement statistics

## How to apply

Using SCANPY, identify each cell type's top 50 positive and top 50 negative markers in both the semi-profiled and real-profiled datasets, and construct RRHO plots from the hypergeometric-test p-values computed between the ranked marker lists. Next, identify the top 100 cell-type signature genes per dataset, run GO/Reactome enrichment with GSEApy, and record the overlap of enriched terms; in the COVID-19 benchmark the target agreement is a 95/100 overlap of top-100 CD4 signature genes (hypergeometric p = 9.10e-196). This step sits alongside, and complements, correlation-based fidelity checks such as cell-cell interaction matrix Pearson correlation (target 0.996 via CellChat) and pseudotime grid Pearson correlation (target 0.809 via Monocle3). Input data should already be preprocessed (library size normalization plus log1p transformation) per the scSemiProfiler methods. Assemble RRHO overlap summaries and enrichment overlap statistics into a downstream_fidelity.csv for validation.

## Related tools

- **SCANPY** (Identify per-cell-type top 50 positive/negative markers and top 100 signature genes in both semi-profiled and real-profiled cohorts) — https://github.com/mcgilldinglab/scSemiProfiler
- **GSEApy** (Run GO/Reactome enrichment on the top-100 signature gene sets and compute enriched-term overlap)
- **SciPy** (Hypergeometric-test p-value computation for RRHO plots and Pearson correlations for fidelity statistics)
- **scSemiProfiler** (Produces the semi-profiled cohort whose marker/enrichment results are compared against the real-profiled ground truth) — https://github.com/mcgilldinglab/scSemiProfiler

## Evaluation signals

- Enrichment overlap between real-profiled and semi-profiled top-100 CD4 signature genes reaches 95/100 with hypergeometric p = 9.10e-196
- Enrichment output contains the reported GO terms from the paper
- RRHO plots between ranked marker lists show rank agreement across all cell types
- Companion fidelity statistics fall within tolerance of reported Fig. 3 values: cell-cell interaction Pearson correlation 0.996 and pseudotime Pearson correlation 0.809
- downstream_fidelity.csv is assembled with all RRHO, enrichment, interaction, and pseudotime statistics present

## Limitations

- Agreement metrics are specific to the COVID-19 cohort benchmark; fidelity targets (95/100 overlap, 0.996, 0.809) may differ for other datasets
- Downstream single-cell analyses require true single-cell resolution; bulk deconvolution methods used for comparison are constrained to cell-type-level resolution
- scSemiProfiler inference requires a single-cell reference (from representative samples or a public dataset) and no software versions or dataset accession IDs are stated in the evaluated section

## Evidence

- [methods] Using SCANPY, identify each cell type's top 50 positive and top 50 negative markers in both datasets and construct RRHO plots based on hypergeometric-test p-values: [evidence span withheld — anchor status: not_found]
- [methods] Identify the top 100 cell-type signature genes per dataset, run GO/Reactome enrichment with GSEApy, and record the overlap: [evidence span withheld — anchor status: not_found]
- [discussion] The striking resemblance between semi-profiled and real-profiled data in facets like UMAP visualization and cell type proportions: "The striking resemblance between semi-profiled and real-profiled data in facets like UMAP visualization and cell type proportions ascertains the robustness of scSemiProfiler."
- [readme] UMAP visualizations compare the real-profiled and semi-profiled COVID-19 cohort with 124 samples: [evidence span withheld — anchor status: external_doc]
