---
name: glomeruli-tubuli-morphometric-feature-regression
description: Use when pathomics AnnData objects (e.g., Kidney_IgAN_G.h5ad, Kidney_IgAN_T.h5ad)
  already carry a PILOT/EIPLGraph trajectory and you need to test which glomerular
  or tubular morphometric features change along the disease progression score.
when_to_use_negative:
- Input is single-cell transcriptomic data rather than pathomics structure annotations
  — morphometric feature extraction requires tissue-structure measurements (sizes,
  diameters, distances)
- No PILOT trajectory/pseudotime has been fit yet — the regressions map features onto
  the EIPLGraph-derived progression score, which must exist first (note the root sample
  was visually selected in the paper)
- Samples come from mixed technologies or tissue-isolation methods — the article flags
  cross-technology analysis as an open challenge and PILOT does not correct technical
  artefacts explicitly
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PILOT
  role: Provides pl.tl.extract_cells_from_pathomics, pl.tl.norm_morphological_features
    and pl.tl.morphological_features_importance used to extract, normalize and regress
    morphometric features against the trajectory
  repo: https://github.com/CostaLab/PILOT
- name: scanpy
  role: Loads the tubuli and glomeruli .h5ad pathomics AnnData objects before feature
    extraction
- name: POT library
  role: Its emd2 solver underlies the Wasserstein/EMD sample-distance computation
    that defines the trajectory on which the morphometric regressions are run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:e0b7292610f5a3f91f580163c2657b409c0316d15792acd5707c3bfbfa1afde5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - POT library
  - scanpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# glomeruli-tubuli-morphometric-feature-regression

## Summary

Extracts glomerular and tubular morphometric features (sizes, diameters, distances to closest instance) mapped to PILOT pseudotime, log-transforms them, and fits robust linear/quadratic Huber regressions to identify which tissue-structure changes track the IgAN disease progression trajectory.

## When to use

Use when pathomics AnnData objects (e.g., Kidney_IgAN_G.h5ad, Kidney_IgAN_T.h5ad) already carry a PILOT/EIPLGraph trajectory and you need to test which glomerular or tubular morphometric features change along the disease progression score.

## When NOT to use

- Input is single-cell transcriptomic data rather than pathomics structure annotations — morphometric feature extraction requires tissue-structure measurements (sizes, diameters, distances)
- No PILOT trajectory/pseudotime has been fit yet — the regressions map features onto the EIPLGraph-derived progression score, which must exist first (note the root sample was visually selected in the paper)
- Samples come from mixed technologies or tissue-isolation methods — the article flags cross-technology analysis as an open challenge and PILOT does not correct technical artefacts explicitly

## Inputs

- Pathomics AnnData objects Kidney_IgAN_G.h5ad (glomeruli) and Kidney_IgAN_T.h5ad (tubuli) loaded with scanpy
- PILOT trajectory output: EIPLGraph-ranked samples / disease progression score (source_node=2, a normal-eGFR sample)
- Combined EMD distance matrix (adata_G.uns['EMD'] + adata_T.uns['EMD'])
- eGFR group labels per sample (Low <30, Reduced 30-60, normal >60)

## Outputs

- Per-feature regression results: best-fit model class (linear, quadratic, linear-quadratic) with p-values for each morphometric feature
- Ranked list of significant glomerular and tubular features along pseudotime (e.g., 14 significant glomerular and 3 tubular features at p<0.05)
- Likelihood-ratio test comparison of PILOT progression score vs. multivariate morphometric model for kidney failure

## How to apply

Run pl.tl.extract_cells_from_pathomics to pull structure-level features mapped to pseudotime: for glomeruli use glom_sizes, glom_diameters, glom_tuft_sizes, glom_bowman_sizes and glom_distance_to_closest_glom; for tubuli use tubule_diameters, tubule_sizes and tubule_distance_to_closest_instance. Log-transform the features with pl.tl.norm_morphological_features so skewed size distributions meet regression assumptions. Fit robust (Huber) regression models with linear, quadratic, and linear-quadratic terms per feature via pl.tl.morphological_features_importance and use the statistical tests to pick the best goodness-of-fit model per feature. Finally, associate the fitted PILOT progression score with eGFR groups (<30 Low, 30-60 Reduced, >60 normal) and compare it to a multivariate model of all morphometric variables by likelihood-ratio test, which established the score's superiority (p=2.4e-11 vs 7e-05).

## Related tools

- **PILOT** (Provides pl.tl.extract_cells_from_pathomics, pl.tl.norm_morphological_features and pl.tl.morphological_features_importance used to extract, normalize and regress morphometric features against the trajectory) — https://github.com/CostaLab/PILOT
- **scanpy** (Loads the tubuli and glomeruli .h5ad pathomics AnnData objects before feature extraction)
- **POT library** (Its emd2 solver underlies the Wasserstein/EMD sample-distance computation that defines the trajectory on which the morphometric regressions are run)

## Examples

```
import pilotpy as pl; feats = pl.tl.extract_cells_from_pathomics(adata_G); feats = pl.tl.norm_morphological_features(feats); pl.tl.morphological_features_importance(feats)
```

## Evaluation signals

- Significant-feature recovery: expect 14 significant glomerular features and 3 tubular features with p<0.05 in the robust fits
- Trajectory ordering invariant: normal-eGFR samples should be placed at the trajectory start (root source_node=2) and case samples toward the end before fitting regressions
- Likelihood-ratio check: PILOT progression score should reproduce stronger kidney-failure association (p=2.4e-11) than the multivariate morphometric model (p=7e-05)
- Normalization check: features must be log-transformed via pl.tl.norm_morphological_features before model fitting
- Model-selection sanity: each feature's best fit among linear/quadratic/linear-quadratic terms should be reported by the goodness-of-fit statistical tests, not assumed a priori

## Limitations

- EIPLGraph requires a root sample, which was visually selected (source_node=2) rather than chosen by an objective rule — a reproducibility gap
- PILOT does not explicitly correct for potential technical artefacts; it only provides statistical tests to evaluate their influence
- PCA dimensionality is compartment-specific — 10 main components kept for glomeruli and 2 for tubules — and must be respected before distance/trajectory computation feeding this skill
- Analysis assumes uniformly collected tissue samples (similar areas, same technology); across distinct technologies is an open challenge

## Evidence

- [discussion] Step-wise Huber regression (linear, quadratic, linear-quadratic) to identify features associated with disease progression: "it fits regression models with linear, quadratic, and linear-quadratic terms for each feature and uses statistical tests to determine the goodness of fit."
- [discussion] Compartment-specific PCA dimensionality: "kept the 10 and 2 main components for glomeruli and tubules, respectively."
- [abstract] Statistical framing for trajectory-associated feature changes: "PILOT provides a statistical approach to find changes in cell populations, gene expression, and tissue structures related to the trajectories or clusters"
- [methods] Data loading pattern for the pathomics inputs: "adata_T=sc.read_h5ad('Datasets/Kidney_IgAN_T.h5ad')"
