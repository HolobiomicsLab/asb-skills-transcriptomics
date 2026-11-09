---
name: dirichlet-multinomial-map-estimation
description: Use when converting per-sample cell-cluster count vectors from single-cell
  AnnData into probability distributions over clusters as input to optimal transport,
  especially when some samples lack counts for certain clusters and zero probabilities
  would break downstream distance computations.
when_to_use_negative:
- When the goal is cluster-level or cell-level analysis rather than per-sample distributions
  — PILOT's downstream steps (Leiden clustering on W, EIPLGraph trajectories) operate
  on the sample-level distance matrix, not on these distributions directly.
- When samples come from distinct technologies or cell-isolation methods (single nucleus
  vs. single cell, droplet vs. well based), which the authors flag as an open challenge
  and assume uniform collection (similar tissue areas and same technology).
- When the input is already a normalized probability/feature table per sample rather
  than raw cell-cluster counts — re-applying pseudo-count smoothing would distort
  existing proportions.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: PILOT
  role: Implements the hierarchical Dirichlet-multinomial MAP estimation of sample
    cluster-distribution matrices as the preprocessing step before optimal transport
  repo: https://github.com/CostaLab/PILOT
- name: scanpy
  role: Loads and aggregates the single-cell AnnData into per-sample cell-cluster
    count vectors (read_h5ad of Datasets/myocardial_infaration.h5ad)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:e6e66aec4a96f284cda9af585f4ff9e0dcbbb5633112c328e9f6f96e2d19a90c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - scanpy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# dirichlet-multinomial-map-estimation

## Summary

Robustly estimate each sample's probability distribution over cell clusters via a hierarchical Dirichlet-multinomial MAP estimate, where the Dirichlet prior adds 'pseudo cell counts' weighted by the global cell distribution. This produces the strictly positive per-sample cluster-probability vectors that PILOT feeds into its optimal-transport (Wasserstein distance) module.

## When to use

Use when converting per-sample cell-cluster count vectors from single-cell AnnData into probability distributions over clusters as input to optimal transport, especially when some samples lack counts for certain clusters and zero probabilities would break downstream distance computations.

## When NOT to use

- When the goal is cluster-level or cell-level analysis rather than per-sample distributions — PILOT's downstream steps (Leiden clustering on W, EIPLGraph trajectories) operate on the sample-level distance matrix, not on these distributions directly.
- When samples come from distinct technologies or cell-isolation methods (single nucleus vs. single cell, droplet vs. well based), which the authors flag as an open challenge and assume uniform collection (similar tissue areas and same technology).
- When the input is already a normalized probability/feature table per sample rather than raw cell-cluster counts — re-applying pseudo-count smoothing would distort existing proportions.

## Inputs

- Single-cell AnnData file (e.g. Datasets/myocardial_infaration.h5ad) with per-cell cluster labels (clusters_col='cell_subtype') and sample labels (sample_col='sampleID')
- Per-sample cell-cluster count vectors n_lk aggregated from the AnnData

## Outputs

- sample_distributions.csv: per-sample MAP probability distributions over clusters (rows = samples, columns = clusters), each row summing to 1

## How to apply

Aggregate the single-cell AnnData (e.g. Datasets/myocardial_infaration.h5ad loaded with scanpy) into per-sample cell-cluster count vectors n_lk using clusters_col='cell_subtype' and sample_col='sampleID'. Compute global cluster counts N_k and total cell count N across all samples, then construct the hierarchical Dirichlet prior alpha_k = N_k/(N*c) with c = 0.1, which adds pseudo cell counts weighted by the cell distribution of all samples. Compute the closed-form MAP estimate p̂_lk = (n_lk + alpha_k)/(N_l + sum_k alpha_k) for every sample l and cluster k, and save the result as sample_distributions.csv (rows = samples, columns = clusters). The pseudo-count prior guarantees strictly positive probabilities even for clusters absent in a sample, which is required because these distributions p_l are the inputs to PILOT's Wasserstein distance computation (emd2 from the POT library).

## Related tools

- **PILOT** (Implements the hierarchical Dirichlet-multinomial MAP estimation of sample cluster-distribution matrices as the preprocessing step before optimal transport) — https://github.com/CostaLab/PILOT
- **scanpy** (Loads and aggregates the single-cell AnnData into per-sample cell-cluster count vectors (read_h5ad of Datasets/myocardial_infaration.h5ad))

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('Datasets/myocardial_infaration.h5ad'); # aggregate per-sample counts n_lk via clusters_col='cell_subtype', sample_col='sampleID'; alpha_k = N_k/(N*c) with c=0.1; p_hat = (n_lk + alpha_k)/(N_l + alpha_k.sum())
```

## Evaluation signals

- Each row of sample_distributions.csv sums to 1 (probability simplex check).
- All p̂_lk > 0 — the pseudo-count prior guarantees strictly positive probabilities even for clusters absent in a sample.
- The Dirichlet prior concentration parameters equal alpha_k = N_k/(N*c) with c = 0.1, recoverable from global cluster counts N_k and total count N.
- Number of rows in sample_distributions.csv equals the number of samples retained after filtering (e.g. myocardial infarction samples excluding the low-quality sample with < 1000 genes on average per cell).
- Downstream Wasserstein distances (emd2/POT) between pairs of sample distributions are finite and well-defined, since zero-probability clusters would otherwise be undefined.

## Limitations

- The prior concentration c = 0.1 is a fixed hyperparameter; no data-driven selection of c is described in the article.
- PILOT does not explicitly correct for potential technical artefacts; it only provides statistical tests to evaluate their influence, so distributions may reflect batch effects.
- The analysis assumes samples were collected uniformly (similar tissue areas and same technology); comparing distributions across distinct technologies or isolation methods is an open challenge.
- Clusters absent in a sample get a positive but prior-driven (not data-driven) probability, which may slightly distort comparisons for very sparse samples.

## Evidence

- [discussion] Robust estimation of sample probability distributions using a hierarchical Dirichlet-multinomial model with pseudo cell counts: [evidence span withheld — anchor status: not_found]
- [discussion] This prior adds “pseudo cell counts”, which are weighted by the cell distribution of all samples.: "This prior adds “pseudo cell counts”, which are weighted by the cell distribution of all samples."
- [abstract] PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples.: "PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples."
- [discussion] we use optimal transport to find the optimal transport plan ... between two distributions pl and pq describing samples l and q: [evidence span withheld — anchor status: not_found]
- [discussion] We exclude one sample due to low quality (less than 1000 genes (on average) per cell).: "We exclude one sample due to low quality (less than 1000 genes (on average) per cell)."
