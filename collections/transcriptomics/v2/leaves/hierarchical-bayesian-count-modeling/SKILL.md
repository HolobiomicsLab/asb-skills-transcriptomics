---
name: hierarchical-bayesian-count-modeling
description: Use when raw per-sample cell-cluster count vectors n_lk (from an AnnData
  of single-cell samples) must be converted into sample-level probability distributions
  over clusters before optimal-transport or distance computations.
when_to_use_negative:
- Input is already a normalized per-sample feature/probability matrix — no count aggregation
  or prior smoothing is needed
- Data mixes distinct technologies or isolation methods (e.g., single nucleus vs.
  single cell, droplet vs. well based) — the article states cross-technology analysis
  'is still an open challenge', so pooled-prior weighting may be inappropriate
- The downstream method requires raw cell counts rather than smoothed probabilities
  (the Dirichlet prior deliberately perturbs raw proportions)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PILOT
  role: Defines the hierarchical Dirichlet-multinomial model whose MAP estimate yields
    per-sample cluster distributions consumed by its Wasserstein module
  repo: https://github.com/CostaLab/PILOT
- name: scanpy
  role: Loads the .h5ad single-cell data used to aggregate per-sample cell-cluster
    counts n_lk
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:602647ec2b6a1d091eec128013eaecd640d9fda560f22e3093ea4077f11077be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - scanpy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# hierarchical-bayesian-count-modeling

## Summary

Robust estimation of per-sample probability distributions over cell clusters via a hierarchical Dirichlet-multinomial MAP model, where a global prior adds 'pseudo cell counts' weighted by the cell distribution of all samples. This produces strictly positive per-sample cluster distributions p_lk that serve as inputs to PILOT's Wasserstein-distance computation.

## When to use

Use when raw per-sample cell-cluster count vectors n_lk (from an AnnData of single-cell samples) must be converted into sample-level probability distributions over clusters before optimal-transport or distance computations.

## When NOT to use

- Input is already a normalized per-sample feature/probability matrix — no count aggregation or prior smoothing is needed
- Data mixes distinct technologies or isolation methods (e.g., single nucleus vs. single cell, droplet vs. well based) — the article states cross-technology analysis 'is still an open challenge', so pooled-prior weighting may be inappropriate
- The downstream method requires raw cell counts rather than smoothed probabilities (the Dirichlet prior deliberately perturbs raw proportions)

## Inputs

- Single-cell AnnData .h5ad file with per-cell cluster annotations (e.g., 'cell_subtype') and sample identifiers (e.g., 'sampleID'), such as Datasets/myocardial_infarction.h5ad
- Cluster label column and sample label column names

## Outputs

- sample_distributions.csv: per-sample MAP probability distributions over cell clusters (rows = samples l, columns = clusters k), each row summing to 1

## How to apply

Load the single-cell AnnData (e.g., Datasets/myocardial_infarction.h5ad) with scanpy and aggregate per-sample cell-cluster count vectors n_lk using clusters_col='cell_subtype' and sample_col='sampleID'. Compute global cluster counts N_k and total cell count N across all samples, then construct the Dirichlet prior alpha_k = N_k/(N*c) with c = 0.1, which adds pseudo cell counts weighted by the cell distribution of all samples. Compute the closed-form MAP estimate p̂_lk = (n_lk + alpha_k)/(N_l + sum_k alpha_k) for every sample l and cluster k. The prior guarantees strictly positive probabilities even for clusters absent in a sample, which is required because PILOT feeds these distributions directly into its emd2/POT Wasserstein computation. Save the result as sample_distributions.csv (rows = samples, columns = clusters).

## Related tools

- **PILOT** (Defines the hierarchical Dirichlet-multinomial model whose MAP estimate yields per-sample cluster distributions consumed by its Wasserstein module) — https://github.com/CostaLab/PILOT
- **scanpy** (Loads the .h5ad single-cell data used to aggregate per-sample cell-cluster counts n_lk)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('Datasets/myocardial_infarction.h5ad')  # then aggregate n_lk by clusters_col='cell_subtype', sample_col='sampleID' and apply the MAP formula
```

## Evaluation signals

- Each row of sample_distributions.csv sums to 1 within numerical tolerance
- All p̂_lk > 0 — the pseudo-count prior alpha_k = N_k/(N*c) with c = 0.1 guarantees strictly positive probabilities even for clusters absent in a sample
- Recomputing MAP estimates from the closed form (n_lk + alpha_k)/(N_l + sum_k alpha_k) reproduces the saved values
- Row dimension equals the number of unique sampleIDs in the AnnData; column dimension equals the number of unique cell subtypes
- Samples with few cells (small N_l) are pulled more strongly toward the global all-sample cell distribution, as expected from the prior

## Limitations

- The prior is global and weighted by the cell distribution of all samples, which implicitly assumes samples were collected uniformly (similar tissue areas, same technology); analysis across distinct technologies or isolation methods is an open challenge
- Choice of the prior scale c = 0.1 controls shrinkage; samples absent for rare clusters receive prior-driven probabilities rather than zeros
- PILOT does not explicitly correct for potential technical artefacts; it only provides statistical tests to evaluate their influence

## Evidence

- [discussion] Pseudo-count prior weighted by global distribution: [evidence span withheld — anchor status: not_found]
- [readme] Distributions feed optimal transport: "PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples. This allows us to perform unsupervised analysis at the sample level"
- [discussion] Optimal transport plan between sample distributions p_l and p_q: [evidence span withheld — anchor status: not_found]
- [methods] AnnData loading with scanpy: "adata_T=sc.read_h5ad('Datasets/Kidney_IgAN_T.h5ad')"
