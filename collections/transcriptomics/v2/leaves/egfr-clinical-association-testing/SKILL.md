---
name: egfr-clinical-association-testing
description: Use when a sample-level trajectory or progression score (e.g., PILOT
  EIPLGraph ranking of pathomics samples) must be shown to predict a clinical outcome
  such as kidney failure or eGFR category, and benchmarked against a multivariate
  baseline.
when_to_use_negative:
- The dataset lacks per-sample clinical annotations (eGFR values or kidney-failure
  status) — the LRT comparison cannot be performed
- The analysis goal is batch-effect evaluation or patient sub-group clustering rather
  than validating a trajectory score against an outcome
- Only raw pathology images or unclustered structures are available — the PILOT progression
  score must first be derived via EMD computation, diffusion maps, and EIPLGraph fitting
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: PILOT
  role: Produces the sample-level progression score (EIPLGraph principal graph ranking)
    and morphometric feature fits that are tested against clinical endpoints
  repo: https://github.com/CostaLab/PILOT
- name: scanpy
  role: Loads the tubuli and glomeruli pathomics AnnData objects (Kidney_IgAN_T.h5ad,
    Kidney_IgAN_G.h5ad) used to derive the score
  repo: https://github.com/CostaLab/PILOT
- name: POT library
  role: Underlies the emd2-based Wasserstein distances (pl.tl.wasserstein_distance)
    whose combined EMD matrix yields the trajectory being validated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:c059d284f8cc6b6de5e56c4ea730c727ac73ca48ba00db6f35a2658e5fd24f92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - POT library
  - scanpy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# egfr-clinical-association-testing

## Summary

Validates a PILOT-derived disease progression score against clinical kidney-function endpoints by associating it with eGFR groups (<30 Low, 30–60 Reduced, >60 normal) and comparing it to a multivariate morphometric model for kidney failure via likelihood-ratio testing.

## When to use

Use when a sample-level trajectory or progression score (e.g., PILOT EIPLGraph ranking of pathomics samples) must be shown to predict a clinical outcome such as kidney failure or eGFR category, and benchmarked against a multivariate baseline.

## When NOT to use

- The dataset lacks per-sample clinical annotations (eGFR values or kidney-failure status) — the LRT comparison cannot be performed
- The analysis goal is batch-effect evaluation or patient sub-group clustering rather than validating a trajectory score against an outcome
- Only raw pathology images or unclustered structures are available — the PILOT progression score must first be derived via EMD computation, diffusion maps, and EIPLGraph fitting

## Inputs

- PILOT progression score (ranked sample ordering from pl.pl.fit_pricipla_graph on combined Kidney_IgAN_T.h5ad + Kidney_IgAN_G.h5ad EMD matrix)
- Clinical eGFR group annotations per sample (<30 Low, 30–60 Reduced, >60 normal)
- Kidney-failure outcome status per patient
- Log-transformed morphometric feature table (pl.tl.norm_morphological_features output: glom_sizes, glom_tuft_sizes, glom_bowman_sizes, tubule_diameters, etc.)

## Outputs

- Likelihood-ratio test p-values comparing PILOT progression score vs. multivariate morphometric model (expected ~2.4e−11 vs ~7e−05 in the IgAN cohort)
- Association of trajectory position with eGFR groups (normal-eGFR samples at trajectory start, case samples toward the end)
- Set of significant morphological features (p<0.05) from the competing model, e.g., 14 glomerular and 3 tubular features

## How to apply

After fitting the EIPLGraph principal graph on the combined tubuli+glomeruli EMD distance matrix (sum of adata_G.uns['EMD'] and adata_T.uns['EMD'], root at source_node=2, a normal-eGFR sample), treat the ranked sample order as the PILOT progression score. Cross-tabulate score position against the clinical eGFR groups (<30 Low, 30–60 Reduced, >60 normal) to confirm trajectory ordering. Fit a univariate model with the PILOT progression score and a multivariate model combining all log-transformed morphometric variables (e.g., glom_sizes, glom_diameters, tubule_diameters from pl.tl.extract_cells_from_pathomics normalized via pl.tl.norm_morphological_features), then compare the two nested models with a likelihood-ratio test. In the IgAN VALIGA cohort the PILOT score achieved LRT p=2.4e−11 versus p=7e−05 for the multivariate morphometric model, demonstrating superior association with kidney failure.

## Related tools

- **PILOT** (Produces the sample-level progression score (EIPLGraph principal graph ranking) and morphometric feature fits that are tested against clinical endpoints) — https://github.com/CostaLab/PILOT
- **scanpy** (Loads the tubuli and glomeruli pathomics AnnData objects (Kidney_IgAN_T.h5ad, Kidney_IgAN_G.h5ad) used to derive the score) — https://github.com/CostaLab/PILOT
- **POT library** (Underlies the emd2-based Wasserstein distances (pl.tl.wasserstein_distance) whose combined EMD matrix yields the trajectory being validated)

## Examples

```
import numpy as np; from scipy.stats import chi2; lr = 2*(ll_pilot - ll_morpho); p = chi2.sf(lr, df_pilot - df_morpho)  # LRT: PILOT score vs multivariate morphometric model for kidney failure
```

## Evaluation signals

- PILOT progression score yields a smaller likelihood-ratio p-value than the multivariate morphometric model (2.4e−11 vs 7e−05 in the reference IgAN analysis)
- Trajectory validation: normal-eGFR samples order at the start and case samples toward the end of the EIPLGraph ranking rooted at source_node=2
- Morphometric baseline reproduces published significance counts: 14 significant glomerular and 3 tubular features with p<0.05 in linear/quadratic robust fits
- eGFR groups (<30, 30–60, >60) show monotonic separation along the progression score

## Limitations

- The EIPLGraph root sample was visually selected to reflect control samples rather than chosen by an objective rule, introducing a reproducibility gap in the score the LRT depends on
- PILOT does not explicitly correct for technical artefacts; it only provides statistical tests to evaluate whether technical factors influence predictions
- Analysis assumes tissue samples were collected uniformly (similar tissue areas and technology); combining distinct technologies or isolation methods remains an open challenge

## Evidence

- [results] LRT comparison result: [evidence span withheld — anchor status: not_found]
- [methods] eGFR grouping and LRT workflow: [evidence span withheld — anchor status: not_found]
- [discussion] Statistical machinery for trajectory association: "it fits regression models with linear, quadratic, and linear-quadratic terms for each feature and uses statistical tests to determine the goodness of fit."
- [readme] Purpose of trajectory-linked statistics: "PILOT provides a statistical approach to find changes in cell populations, gene expression, and tissue structures related to the trajectories or clusters supporting interpretation of predictions."
