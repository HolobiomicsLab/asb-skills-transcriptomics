---
name: qphate-embedding-biomarker-thresholding
description: Use when transferring single-cell atlas latent structure (e.g., the stage-4
  Human Colon Cancer Atlas c295) onto bulk RNA patient profiles to create binary biomarkers
  for survival/differential-treatment-effect testing, and standard PHATE or NMF-derived
  biomarkers lack statistical significance.
when_to_use_negative:
- When the reference and target datasets require batch-effect correction or alignment
  — the study explicitly did not consider batch-effect correction or alignment of
  disparate datasets, so raw transferred coordinates may be confounded.
- When only classical dimensionality reduction suffices — the article reports classical
  NMF and PHATE biomarkers produced no statistically significant differential treatment
  effects, but if no differential effect is biologically expected, this added machinery
  is unnecessary.
- When single-cell data are not available as an atlas reference — the method depends
  on inferring latent variables from an scRNAseq atlas before deconvolution onto bulk
  profiles.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3517
tools:
- name: q-diffusion
  role: Framework whose q-diffused kernel supplies the cell-by-cell geometry that
    makes the full-transcriptome qPHATE embedding possible
  repo: github.com/marmarelis/QDiffusion.jl
- name: q-Diffusion.jl
  role: Julia package providing the deformed/adaptive kernel construction and the
    `leave_it_to_phate` routine used to compute the qPHATE embedding with GPU acceleration
  repo: github.com/marmarelis/QDiffusion.jl
- name: PHATE
  role: Base manifold-learning embedding; run with the q-diffused kernel it yields
    qPHATE, whose transferred coordinates are thresholded into the patient biomarker
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:83a92c6928622b89570dfe39458de39bbfa1aa1a576a90eacf2b9b7b2a325cc1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PHATE
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# qPHATE embedding biomarker thresholding

## Summary

Constructs binary patient biomarkers from q-diffused PHATE (qPHATE) embeddings: atlas-inferred qPHATE coordinates are transferred onto patient bulk RNA by linear deconvolution and thresholded along their principal component, yielding latent-variable biomarkers with sufficient differential treatment-effect signal to reach FDR-corrected significance in the CALGB/SWOG 80405 mCRC trial where classical PHATE embeddings fail.

## When to use

Use when transferring single-cell atlas latent structure (e.g., the stage-4 Human Colon Cancer Atlas c295) onto bulk RNA patient profiles to create binary biomarkers for survival/differential-treatment-effect testing, and standard PHATE or NMF-derived biomarkers lack statistical significance.

## When NOT to use

- When the reference and target datasets require batch-effect correction or alignment — the study explicitly did not consider batch-effect correction or alignment of disparate datasets, so raw transferred coordinates may be confounded.
- When only classical dimensionality reduction suffices — the article reports classical NMF and PHATE biomarkers produced no statistically significant differential treatment effects, but if no differential effect is biologically expected, this added machinery is unnecessary.
- When single-cell data are not available as an atlas reference — the method depends on inferring latent variables from an scRNAseq atlas before deconvolution onto bulk profiles.

## Inputs

- Stage-4 portion of the Human Colon Cancer Atlas (c295) scRNAseq reference dataset
- CALGB/SWOG 80405 trial bulk RNA profiles with patient outcomes and covariates (chemotherapy type, tumor side, sex, age, ECOG score, tumor mutations)
- q-diffused kernel built in QDiffusion.jl (q=1.2, adaptive kNN outer bandwidth, estimated inner bandwidth, in_log=true)
- Atlas-inferred gene expression program weights for sparsity screening (keep 5%-95%)

## Outputs

- qPHATE embedding of atlas cells (n_manifold_dims=2, n_bases=100)
- qPHATE coordinates transferred onto patient bulk RNA profiles by linear deconvolution
- Binary patient qPHATE biomarker from thresholding along the principal component
- Differential treatment-effect statistics: Cox coefficient contrasts per treatment cohort, asymptotic normal z-tests, Benjamini-Hochberg FDR-corrected significance

## How to apply

First compute a q-diffused kernel (e.g., q=1.2, adaptive 16-NN outer bandwidth, estimated inner bandwidth, in_log=true) and embed the scRNAseq atlas with the q-diffused PHATE routine `leave_it_to_phate(kernel, data, bandwidth, ...)` from QDiffusion.jl. Translate the atlas latent coordinates onto patient bulk RNA profiles via linear deconvolution, screening gene expression program weights for sparsity between 5% and 95% so only informative latent variables are kept. Threshold the transferred qPHATE coordinates along their principal component to obtain a simple binary biomarker (as in Fig. 3c). Use the binary qPHATE biomarker — alongside qNMF biomarkers binarized by weight sparsity — as a covariate in multivariate Cox proportional hazards models for PFS and OS, contrasting coefficients between treatment cohorts and testing differences via the asymptotic normal approximation with Benjamini-Hochberg FDR correction.

## Related tools

- **q-diffusion** (Framework whose q-diffused kernel supplies the cell-by-cell geometry that makes the full-transcriptome qPHATE embedding possible) — github.com/marmarelis/QDiffusion.jl
- **q-Diffusion.jl** (Julia package providing the deformed/adaptive kernel construction and the `leave_it_to_phate` routine used to compute the qPHATE embedding with GPU acceleration) — github.com/marmarelis/QDiffusion.jl
- **PHATE** (Base manifold-learning embedding; run with the q-diffused kernel it yields qPHATE, whose transferred coordinates are thresholded into the patient biomarker)

## Examples

```
embedding = leave_it_to_phate(kernel, data, bandwidth, n_landmarks=-1, n_bases=100, n_manifold_dims=2, cuda_devices=[0])
```

## Evaluation signals

- The binary biomarker thresholds transferred qPHATE coordinates along their principal component, and the resulting Cox coefficient contrast between the two treatment cohorts shows opposing signs (as required by the validation criterion).
- The qPHATE-derived biomarker recovers a differential bevacizumab effect with FDR-corrected significance (FDR < 0.05), reproducing the reported statistical significance, while classical PHATE and standard-NMF biomarkers show no significant differential effects.
- Gene expression program weights transferred to patients fall within the 5%-95% sparsity screen; latent variables outside this range are excluded from biomarker construction.
- Bootstrap resampling of patients confirms the strong differential hazard on PFS is robust.

## Limitations

- No algorithm is provided for selecting the q-diffusion parameters q and φ (deformation parameter and inner bandwidth); the article notes a careful selection algorithm would be needed.
- Runtime of evaluating the q-diffused kernel across all pairs of cells is suboptimal (20 min to 3 h for one affinity matrix).
- q-diffusion makes parametric assumptions on the data that might be restrictive in some use cases, and its nonlinearities are less directly interpretable.
- No predictive marker for bevacizumab efficacy is currently available for clinical patient selection, so the biomarker is not yet clinically actionable.

## Evidence

- [results] Threshold qPHATE embedding coordinates along their principal component to create a binary biomarker: "The qPHATE embedding coordinates transferred onto patients were thresholded along their principal component, shown in Fig. 3c, for a simple binary biomarker"
- [results] PHATE with a q-diffused kernel yielded qPHATE: "PHATE with a q-diffused kernel yielded a new kind of full-transcriptome embedding that we term qPHATE"
- [results] Differential effects identified only for qNMF in Cox regression: [evidence span withheld — anchor status: not_found]
- [results] Binarize patient GEP weights by sparsity: [evidence span withheld — anchor status: not_found]
- [readme] leave_it_to_phate parameters from README: "embedding = leave_it_to_phate(kernel, data, bandwidth,
  n_landmarks=-1, # a positive integer invokes the landmark approx
  n_bases=100"
