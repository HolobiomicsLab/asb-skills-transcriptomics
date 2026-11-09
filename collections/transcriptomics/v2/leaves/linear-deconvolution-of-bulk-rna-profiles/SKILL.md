---
name: linear-deconvolution-of-bulk-rna-profiles
description: Use when you have latent variables (GEP weights or manifold embeddings)
  learned from a scRNAseq reference atlas and need to project them onto bulk RNA profiles
  from patients to build testable clinical biomarkers.
when_to_use_negative:
- When the reference and target are single-cell datasets of comparable resolution
  — direct latent-space alignment or joint embedding is more appropriate than deconvolution
  of bulk mixtures
- When patient bulk profiles require batch-effect correction or alignment to the atlas
  — the authors note no batch-effect correction or alignment of disparate datasets
  was considered in this workflow
- When a GEP's transferred weights are degenerately sparse (nonzero in fewer than
  5% or more than 95% of patients), the program is screened out before biomarker construction
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: q-diffusion
  role: Framework whose q-diffused kernel produces the qNMF gene expression programs
    and qPHATE embeddings that are deconvolved onto patient bulk profiles
- name: q-Diffusion.jl
  role: Julia package used to estimate the qNMF factors (par_estimate_nmf, rank=16,
    kernel-guided regularization, fixed seed) and the qPHATE embedding (leave_it_to_phate)
    whose latent variables are transferred to bulk RNA
  repo: github.com/marmarelis/QDiffusion.jl
- name: PHATE
  role: Embedding method applied with the q-diffused kernel (qPHATE); its coordinates
    are transferred onto patients and thresholded along the principal component for
    a binary biomarker
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:207b07d5296ab30284edea6e97cf96c7dd36d8013c9067063891c2e14acc2d78
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PHATE
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# linear-deconvolution-of-bulk-rna-profiles

## Summary

Transfers atlas-inferred latent variables (qNMF gene expression program weights and qPHATE embedding coordinates) from a scRNAseq reference atlas onto patient bulk RNA profiles via linear deconvolution, enabling single-cell-derived biomarkers to be evaluated against bulk clinical outcomes.

## When to use

Use when you have latent variables (GEP weights or manifold embeddings) learned from a scRNAseq reference atlas and need to project them onto bulk RNA profiles from patients to build testable clinical biomarkers.

## When NOT to use

- When the reference and target are single-cell datasets of comparable resolution — direct latent-space alignment or joint embedding is more appropriate than deconvolution of bulk mixtures
- When patient bulk profiles require batch-effect correction or alignment to the atlas — the authors note no batch-effect correction or alignment of disparate datasets was considered in this workflow
- When a GEP's transferred weights are degenerately sparse (nonzero in fewer than 5% or more than 95% of patients), the program is screened out before biomarker construction

## Inputs

- scRNAseq atlas reference dataset with inferred latent variables (qNMF GEP factorization at rank=16, or qPHATE embedding) — e.g. stage-4 Human Colon Cancer Atlas (c295)
- Patient bulk RNA expression profiles from the CALGB/SWOG 80405 trial with associated outcomes (PFS, OS) and covariates
- Log-normalized (genes x cells) atlas expression matrix and log-normalized patient bulk expression matrix

## Outputs

- Per-patient weights for each atlas-inferred gene expression program (GEP)
- Per-patient transferred qPHATE embedding coordinates
- Binarized patient-level biomarkers (binary GEP usage vectors; qPHATE coordinates thresholded along their principal component)

## How to apply

Infer latent variables from the scRNAseq atlas reference (e.g. the stage-4 Human Colon Cancer Atlas c295, with 16 qNMF GEPs at rank=16 or a q-diffused PHATE embedding), then fit a linear deconvolution that expresses each patient's bulk RNA profile as a mixture of the atlas cell-type/program signatures, yielding per-patient weights for each latent variable. For qNMF GEPs, screen the transferred per-patient weights for sparsity (keeping programs used by between 5% and 95% of patients) and binarize weights as 0 for zero and 1 for nonzero to obtain the qNMF biomarker. For qPHATE, transfer the embedding coordinates onto patient bulk profiles and threshold them along their principal component to create a simple binary biomarker. The transferred latent variables then enter multivariate Cox proportional hazards models for PFS and OS to test differential treatment effects between the two CALGB/SWOG 80405 cohorts.

## Related tools

- **q-diffusion** (Framework whose q-diffused kernel produces the qNMF gene expression programs and qPHATE embeddings that are deconvolved onto patient bulk profiles)
- **q-Diffusion.jl** (Julia package used to estimate the qNMF factors (par_estimate_nmf, rank=16, kernel-guided regularization, fixed seed) and the qPHATE embedding (leave_it_to_phate) whose latent variables are transferred to bulk RNA) — github.com/marmarelis/QDiffusion.jl
- **PHATE** (Embedding method applied with the q-diffused kernel (qPHATE); its coordinates are transferred onto patients and thresholded along the principal component for a binary biomarker)

## Examples

```
factors = par_estimate_nmf(kernel, data, bandwidth, rank=16, regularizations=[0f0, 1f0], n_iterations=512, cuda_devices=[0], seed=1337); embedding = leave_it_to_phate(kernel, data, bandwidth, n_manifold_dims=2, cuda_devices=[0])  # then linearly deconvolve these atlas latent variables onto patient bulk profiles
```

## Evaluation signals

- The final Cox regression recovers FDR-corrected (Benjamini-Hochberg) significant differential treatment-effect differences with opposing signs between cohorts for qNMF latent variables, reproducing the reported bevacizumab differential effect significance
- At least one qNMF gene expression program passes the FDR < 0.05 differential-effect threshold, while the classical-NMF baseline yields no such program (comparatively weaker significance)
- Screened GEPs retain non-degenerate sparsity: nonzero weights in between 5% and 95% of patients before binarization
- Transferred qPHATE coordinates form a coherent distribution along their principal component such that thresholding yields a usable binary biomarker separable across treatment arms
- Bootstrap resampling of patients confirms the strong differential hazard on PFS is robust

## Limitations

- No batch-effect correction or alignment of the disparate atlas and bulk datasets was considered; pre-existing tools would need to be coopted upstream for that purpose
- q-diffusion makes parametric assumptions on the data that might be restrictive in some use cases, and its nonlinearities are less directly interpretable
- Runtime of the q-diffused kernel evaluation is suboptimal (20 min to 3 h per affinity matrix), and no automated algorithm exists for selecting the deformation parameter q and inner bandwidth φ

## Evidence

- [other] Embed with q-diffused PHATE, translate atlas-inferred latent variables onto patient bulk RNA profiles by linear deconvolution: [evidence span withheld — anchor status: not_found]
- [methods] The latent variables were inferred from an scRNAseq "atlas" reference dataset15. The stage-4 portion of the atlas was selected: [evidence span withheld — anchor status: not_found]
- [results] The qPHATE embedding coordinates transferred onto patients were thresholded along their principal component, shown in Fig. 3c, for a simple binary biomarker: "The qPHATE embedding coordinates transferred onto patients were thresholded along their principal component, shown in Fig. 3c, for a simple binary biomarker"
- [results] We then binarized the patients' weights for that GEP based on sparsity: 0 for zero and 1 for nonzero, obtaining the qNMF biomarker.: [evidence span withheld — anchor status: not_found]
- [discussion] Presumably, various pre-existing tools can be coopted upstream of the analysis for this purpose.: "Presumably, various pre-existing tools can be coopted upstream of the analysis for this purpose."
