---
name: pathway-activation-score-computation-single-cell
description: Use when single-cell or semi-profiled cohort data (annotated cells with
  condition labels, e.g., Healthy/Asymptomatic/Mild/Moderate/Severe/Critical) must
  be compared for pathway-level fidelity, or when pathway activation patterns are
  a downstream deliverable.
when_to_use_negative:
- Input is bulk sequencing data without a single-cell reference — bulk deconvolution
  approaches (CIBERSORTx, Bisque, DWLS, MuSiC, NNLS) are constrained to cell-type-level
  resolution and cannot produce single-cell-level pathway scores
- Data are from other modalities such as proteomics — the article states such extensions
  are future work, not yet implemented
- The cohort lacks a healthy baseline condition, since fold changes are defined relative
  to the healthy condition
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: SCANPY
  role: Provides tl.score_genes to compute per-cell pathway/gene-set activation scores,
    per the COVID-19 source-study procedure
- name: SciPy
  role: scipy.stats.pearsonr is used to correlate IFN pathway activation scores between
    semi-profiled and real-profiled cohorts
- name: scSemiProfiler
  role: Generates the semi-profiled cohort whose inferred single cells are annotated
    (MLP classifier) and scored for pathway activation against the real-profiled ground
    truth
  repo: https://github.com/mcgilldinglab/scSemiProfiler
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:80677eb7d877efdc01147b4ccb23bf083394306cae1530f50b3122a70c4c10a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - SCANPY
  - SciPy
  - scSemiProfiler
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pathway-activation-score-computation-single-cell

## Summary

Computes per-cell pathway/gene-set activation scores (e.g., interferon) from single-cell data with SCANPY's tl.score_genes, aggregates them per cell type and disease condition as fold changes versus healthy, and uses the resulting score matrices to quantify agreement between semi-profiled and real-profiled cohorts.

## When to use

Use when single-cell or semi-profiled cohort data (annotated cells with condition labels, e.g., Healthy/Asymptomatic/Mild/Moderate/Severe/Critical) must be compared for pathway-level fidelity, or when pathway activation patterns are a downstream deliverable.

## When NOT to use

- Input is bulk sequencing data without a single-cell reference — bulk deconvolution approaches (CIBERSORTx, Bisque, DWLS, MuSiC, NNLS) are constrained to cell-type-level resolution and cannot produce single-cell-level pathway scores
- Data are from other modalities such as proteomics — the article states such extensions are future work, not yet implemented
- The cohort lacks a healthy baseline condition, since fold changes are defined relative to the healthy condition

## Inputs

- Annotated single-cell expression matrices (.h5ad) for both the semi-profiled and real-profiled COVID-19 cohorts
- Interferon (IFN) pathway gene list from MSigDB curated gene sets (plus Ernst et al. sets used by the paper)
- Cell-type annotations (e.g., from the MLP classifier trained on representative single-cell data) and severity condition labels (Healthy, Asymptomatic, Mild, Moderate, Severe, Critical)

## Outputs

- Per-cell pathway activation scores (e.g., IFN score per cell from sc.tl.score_genes)
- Averaged pathway activation scores per cell type and severity condition, as fold changes relative to the Healthy condition
- Pearson correlation of IFN pathway activation between semi-profiled and real-profiled cohorts, recorded in similarity_metrics.csv (6 condition rows with cell-type proportion and IFN pathway Pearson columns)

## How to apply

Compute gene-set activation scores per cell using SCANPY's tl.score_genes, drawing gene sets from the curated MSigDB collections (and Ernst et al. marker sets as in the scSemiProfiler methods) — for the COVID-19 cohort use the interferon (IFN) pathway gene set, following the source study's procedure. Average the per-cell scores per cell type and per severity condition for both the semi-profiled cohort (cells annotated via the MLP classifier trained on representative single-cell data) and the real-profiled ground truth. Express each condition/cell-type score as a fold change relative to the Healthy condition to remove baseline effects. Quantify cohort agreement by Pearson correlation (scipy.stats.pearsonr) of the IFN activation scores between semi-profiled and real-profiled data, writing results to similarity_metrics.csv alongside the six per-condition cell-type proportion correlations.

## Related tools

- **SCANPY** (Provides tl.score_genes to compute per-cell pathway/gene-set activation scores, per the COVID-19 source-study procedure)
- **SciPy** (scipy.stats.pearsonr is used to correlate IFN pathway activation scores between semi-profiled and real-profiled cohorts)
- **scSemiProfiler** (Generates the semi-profiled cohort whose inferred single cells are annotated (MLP classifier) and scored for pathway activation against the real-profiled ground truth) — https://github.com/mcgilldinglab/scSemiProfiler

## Examples

```
import scanpy as sc; from scipy.stats import pearsonr; sc.tl.score_genes(adata, gene_list=ifn_genes, score_name='IFN_score'); r = pearsonr(semi_ifn, real_ifn)
```

## Evaluation signals

- The IFN pathway Pearson correlation between semi-profiled and real-profiled cohorts falls within 0.75–0.85
- similarity_metrics.csv contains exactly 6 condition rows (Healthy, Asymptomatic, Mild, Moderate, Severe, Critical) with both cell-type proportion and IFN pathway Pearson columns
- All six per-condition cell-type proportion Pearson correlations exceed 0.95 (e.g., 0.987 Healthy, 0.996 Mild, 0.989 Critical)
- Fold changes are computed relative to the Healthy condition, so Healthy-baseline scores normalize to zero
- Scores are averaged per cell type and severity condition before correlation, not compared at raw per-cell level

## Limitations

- Gene-set scoring depends on the curated sets used (all curated MSigDB gene sets plus Ernst et al.); pathway results follow the COVID-19 source study's specific procedure
- scSemiProfiler inference requires a single-cell reference; pathway-level fidelity inherits the VAE-GAN inference quality
- No software versions are stated for scSemiProfiler or its dependencies, complicating exact reproduction

## Evidence

- [methods] gene set scores are calculated using all curated gene sets from MSigDB59 and Ernst et al.91: "gene set scores are calculated using all curated gene sets from MSigDB59 and Ernst et al.91"
- [intro] such as UMAP27, pathway activation pattern analysis, biomarker discovery, gene functional enrichment: "such as UMAP27, pathway activation pattern analysis, biomarker discovery, gene functional enrichment"
- [readme] scSemiProfiler offers a cost-effective AI-generated alternative to real-profiled single-cell data with high fidelity.: [evidence span withheld — anchor status: external_doc]
