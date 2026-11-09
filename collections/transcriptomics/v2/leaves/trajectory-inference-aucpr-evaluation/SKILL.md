---
name: trajectory-inference-aucpr-evaluation
description: Use when you have a sample-level distance matrix (e.g., Wasserstein distances
  between cluster distributions from .h5ad single-cell/pathomics cohorts) and need
  to infer a disease trajectory and score it by AUCPR and Spearman correlation against
  known severity labels.
when_to_use_negative:
- The data mixes distinct technologies (single-nucleus vs. single-cell) or distinct
  isolation methods (droplet-based vs. well-based), which the authors state remains
  an open challenge for sample-level analysis
- You need explicit correction of technical artifacts/batch effects — PILOT only provides
  statistical tests to evaluate whether technical factors influence predictions
- Tissue samples were not collected uniformly (similar tissue areas, same technology),
  an assumption the trajectory comparison relies on
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PILOT
  role: Performs diffusion maps followed by EIPLGraph trajectory estimation on the
    Wasserstein distance matrix; distributed as pilotpy
  repo: https://github.com/CostaLab/PILOT
- name: POT library
  role: Provides the emd2 function computing Wasserstein distances between sample
    cluster distributions that feed the trajectory analysis
- name: Monocle 2
  role: Required for the PhEMD competing pipeline (normalization, dimension reduction,
    and clustering of cells) before its trajectory benchmarking
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:5c2ee8fac96f47df8ebd87801144aee6ff05857ba5c328702a7da2ffa3da4488
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - Monocle 2
  - PILOT
  - POT library
  - SCANCell
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# trajectory-inference-aucpr-evaluation

## Summary

Patient-level trajectory estimation from single-cell/pathomics sample distance matrices (diffusion maps + EIPLGraph backbone) and benchmarking of the recovered trajectories by AUCPR of disease-status prediction and Spearman correlation with disease severity, as used to compare PILOT, PhEMD, SCANCell, and Multi-scale PHATE across 12 datasets.

## When to use

Use when you have a sample-level distance matrix (e.g., Wasserstein distances between cluster distributions from .h5ad single-cell/pathomics cohorts) and need to infer a disease trajectory and score it by AUCPR and Spearman correlation against known severity labels.

## When NOT to use

- The data mixes distinct technologies (single-nucleus vs. single-cell) or distinct isolation methods (droplet-based vs. well-based), which the authors state remains an open challenge for sample-level analysis
- You need explicit correction of technical artifacts/batch effects — PILOT only provides statistical tests to evaluate whether technical factors influence predictions
- Tissue samples were not collected uniformly (similar tissue areas, same technology), an assumption the trajectory comparison relies on

## Inputs

- Sample-level cluster distribution matrix or .h5ad AnnData object (e.g., Kidney_IgAN_T.h5ad, GSE174188_CLUES1_adjusted.h5ad.gz)
- Sample-sample distance matrix W (e.g., Wasserstein distances from POT's emd2 between cluster distributions)
- Disease-status and disease-severity labels per patient/sample

## Outputs

- Pseudotime ordering of patients/samples per method per dataset
- Per-method, per-dataset AUCPR and Spearman correlation metrics
- Aggregated metric and ranking table (benchmark_metrics.csv) with Friedman–Nemenyi method rankings

## How to apply

Start from the sample distance matrix W (for PILOT: POT's emd2 Wasserstein distances between cluster distributions with a Dirichlet-multinomial prior adding pseudo cell counts; for competitors: PhEMD EMD distances, SCANCell embeddings, Multi-scale PHATE multi-resolution embeddings). Estimate each trajectory with diffusion maps followed by EIPLGraph backbone inference, providing a root sample chosen to reflect control samples. Then compute, per method and dataset, the AUCPR of disease-status prediction along the inferred trajectory and the Spearman correlation of trajectory position with known disease severity. When benchmarking on the pancreas dataset, exclude the four outlier embryo samples (E12-E15) that appear as trajectory outliers. Aggregate metrics across the 12 datasets and apply the Friedman test with Nemenyi post-hoc correction to produce method rankings (PILOT ranked significantly superior, p < 0.05, Appendix Table S1).

## Related tools

- **PILOT** (Performs diffusion maps followed by EIPLGraph trajectory estimation on the Wasserstein distance matrix; distributed as pilotpy) — https://github.com/CostaLab/PILOT
- **POT library** (Provides the emd2 function computing Wasserstein distances between sample cluster distributions that feed the trajectory analysis)
- **Monocle 2** (Required for the PhEMD competing pipeline (normalization, dimension reduction, and clustering of cells) before its trajectory benchmarking)

## Examples

```
pip install pilotpy; import pilotpy as pl  # trajectory via diffusion maps + EIPLGraph on the Wasserstein distance matrix
```

## Evaluation signals

- Friedman–Nemenyi rankings over AUCPR and Spearman metrics reproduce the reported method ordering in Fig. 2A–D and Appendix Tables S1–S2, with PILOT ranked significantly superior (p < 0.05)
- Every metric (ARI, Silhouette, AUCPR, Spearman) is computed for all four methods (PILOT, PhEMD, SCANCell, Multi-scale PHATE) on all twelve datasets before ranking
- Four pancreas samples (Embryos E12-E15) are excluded as trajectory outliers per the paper's benchmarking protocol
- Each trajectory's root sample is set to the control end of the trajectory, matching the paper's EIPLGraph root-selection convention
- The Silhouette score of the sample distance matrix (via pl.pl.select_best_sil) is checked alongside trajectory metrics as a quality invariant

## Limitations

- The EIPLGraph root sample was visually selected rather than chosen by an objective rule — a reproducibility gap acknowledged by the authors
- PILOT does not explicitly correct for potential technical artefacts; it only provides statistical tests to evaluate their influence (unclear effect on the lung and COVID-19 atlas disease-group comparisons)
- None of the prior competing approaches (Chen 2020, Flores 2023, Kuchroo 2022, Zhang 2022) performed any benchmarking, so cross-paper baseline comparisons are limited

## Evidence

- [discussion] trajectory via diffusion maps and EIPLGraph: "PILOT also performs trajectory analysis by the use of diffusion maps (Coifman et al, 2005) followed by a trajectory estimation with EIPLGraph"
- [discussion] root sample selection: "EIPLGraph also requires a root sample, which was visually selected to reflect parts of the trajectory with control samples."
- [discussion] outlier exclusion in trajectory benchmarking: "We excluded four samples (Embryos E12-E15) due to being outliers by trajectories of methods."
- [readme] unsupervised trajectory discovery at sample level: "This allows us to perform unsupervised analysis at the sample level and uncover trajectories or cellular clusters associated with disease progression."
- [discussion] feature-fitting along trajectory: "it fits regression models with linear, quadratic, and linear-quadratic terms for each feature and uses statistical tests to determine the goodness of fit."
