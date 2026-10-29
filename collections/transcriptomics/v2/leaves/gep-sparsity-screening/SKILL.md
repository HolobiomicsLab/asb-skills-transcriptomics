---
name: gep-sparsity-screening
description: Use when you have run kernel-regularized NMF (e.g., par_estimate_nmf
  at rank ~16) on a log-normalized genes x cells matrix and must filter out degenerate
  gene expression programs before downstream use (e.g., biomarker binarization).
when_to_use_negative:
- The weight matrix comes from plain (unregularized) NMF or a method other than the
  q-diffusion kernel-regularized factorization, so the 5%-95% sparsity heuristic from
  this paper does not apply
- The input expression matrix has already undergone feature selection or dimensionality
  reduction — q-diffusion requires the full log-normalized genes x cells matrix
- The goal is a full-transcriptome embedding (qPHATE) rather than discrete gene expression
  programs
tools:
- name: q-Diffusion.jl
  role: Provides par_estimate_nmf for kernel-regularized NMF whose GEP weights are
    then sparsity-screened
  repo: github.com/marmarelis/QDiffusion.jl
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:74f399f4bc82680fa1fc40d8f0439806db5dc0e407062c3ab6d17753f1c6bb69
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gep-sparsity-screening

## Summary

Screens qNMF-estimated gene expression program (GEP) weight matrices for useful sparsity, keeping only programs whose cell weights are between 5% and 95% nonzero — programs that are entirely zero or entirely nonzero carry no discriminative information and signal underdetermined factorization.

## When to use

Use when you have run kernel-regularized NMF (e.g., par_estimate_nmf at rank ~16) on a log-normalized genes x cells matrix and must filter out degenerate gene expression programs before downstream use (e.g., biomarker binarization).

## When NOT to use

- The weight matrix comes from plain (unregularized) NMF or a method other than the q-diffusion kernel-regularized factorization, so the 5%-95% sparsity heuristic from this paper does not apply
- The input expression matrix has already undergone feature selection or dimensionality reduction — q-diffusion requires the full log-normalized genes x cells matrix
- The goal is a full-transcriptome embedding (qPHATE) rather than discrete gene expression programs

## Inputs

- Cells-by-GEP nonnegative weight matrix from par_estimate_nmf (rank 16, fixed seed)
- Log-normalized (genes x cells) scRNAseq matrix via preprocess_cell_counts *.normalized element
- q-diffused affinity matrix (AdaptiveKernel-wrapped DeformedKernel, q=1.2) used as regularization

## Outputs

- Filtered set of GEPs with nonzero-weight fraction between 5% and 95%
- Per-GEP sparsity statistics (fraction of cells with nonzero weight)
- Optionally, binarized (0/1) patient GEP weights forming the qNMF biomarker

## How to apply

After estimating GEPs with par_estimate_nmf (rank=16, regularizations=[0f0, 1f0] so cell GEP weights are regularized toward the q-diffused affinity matrix, seed fixed for reproducibility), inspect each of the 16 GEP columns of the cells-by-GEP weight matrix and compute the fraction of cells with nonzero weight. Retain a GEP only if its sparsity lies between 5% and 95% nonzero; discard programs that are entirely zero (collapsing) or entirely nonzero (uninformative, global programs). Rationale: the q-diffused graph regularization constrains the factorization to lower the risk of underdetermination, and the sparsity screen is the sanity filter confirming each surviving program defines a distinct cell subpopulation. Downstream, retained programs can be binarized per patient (0 for zero, 1 for nonzero weights) to form the qNMF biomarker.

## Related tools

- **q-Diffusion.jl** (Provides par_estimate_nmf for kernel-regularized NMF whose GEP weights are then sparsity-screened) — github.com/marmarelis/QDiffusion.jl

## Examples

```
factors = par_estimate_nmf(kernel, data, bandwidth, rank=16, regularizations=[0f0, 1f0], n_iterations=512, seed=1337)
```

## Evaluation signals

- Every retained GEP has a nonzero-weight fraction strictly between 5% and 95%; zero or all-nonzero programs are excluded
- Top-weighted genes in each qNMF GEP are almost equivalent to those of plain NMF initialized with the same random seed (the paper's reported validation)
- Factorization is reproducible: re-running par_estimate_nmf with the same fixed seed yields identical weights
- Binarized weights of retained GEPs produce non-degenerate (mixed 0/1) patient-level biomarkers

## Limitations

- q-diffusion makes parametric assumptions on the data that might be restrictive in some use cases
- No algorithm for selecting the q-diffusion parameters q and φ is provided, so the 5%-95% screen inherits whatever kernel parameters were chosen
- Nonlinearities of q-diffusion are less directly interpretable in general

## Evidence

- [results] Screen the estimated GEP weights for sparsity between 5% and 95%, excluding entirely zero or entirely nonzero programs.: [evidence span withheld — anchor status: not_found]
- [results] We then binarized the patients' weights for that GEP based on sparsity: 0 for zero and 1 for nonzero, obtaining the qNMF biomarker.: [evidence span withheld — anchor status: not_found]
- [results] it is a q-diffused form of NMF whose factorization is constrained by the q-diffused cell graph to lower the risk of underdetermination.: [evidence span withheld — anchor status: not_found]
- [readme] regularizations=[0f0, 1f0], # how much to regularize by kernel: "regularizations=[0f0, 1f0], # how much to regularize by kernel"
