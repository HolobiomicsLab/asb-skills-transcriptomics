---
name: single-cell-cluster-count-aggregation
description: 'Use when starting PILOT-style sample-level analysis: you have a cell-by-cluster
  annotated .h5ad (e.g. clusters_col ''cell_subtype'', sample_col ''sampleID'') and
  need per-sample cluster probability distributions for Wasserstein distance computation.'
when_to_use_negative:
- The input is already a per-sample feature/distribution table (e.g. pre-computed
  cluster proportions or pathomics structure counts) — no count aggregation or prior
  smoothing is needed.
- You need distances between individual cells or between clusters rather than between
  whole samples; PILOT's transport step operates on sample-level distributions, not
  single cells.
- Samples come from distinct technologies or isolation methods (single nucleus vs.
  single cell, droplet vs. well based) — the article flags cross-technology analysis
  as an unresolved open challenge for this representation.
tools:
- name: PILOT
  role: Implements the robust Dirichlet-multinomial MAP estimation of per-sample cluster-distribution
    matrices that precede optimal transport
  repo: https://github.com/CostaLab/PILOT
- name: scanpy
  role: Loads the single-cell .h5ad AnnData from which per-sample, per-cluster count
    vectors are aggregated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:3c0ac5b57c0daecb85f07999e99442d73b67af96b9d219f5b2689f4d996afb56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - scanpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# single-cell-cluster-count-aggregation

## Summary

Aggregates per-sample cell-cluster count vectors from a clustered single-cell AnnData and converts them into robust per-sample probability distributions over clusters via a hierarchical Dirichlet-multinomial MAP estimate. These distributions are the input PILOT feeds into its optimal-transport (Wasserstein) comparison of samples.

## When to use

Use when starting PILOT-style sample-level analysis: you have a cell-by-cluster annotated .h5ad (e.g. clusters_col 'cell_subtype', sample_col 'sampleID') and need per-sample cluster probability distributions for Wasserstein distance computation.

## When NOT to use

- The input is already a per-sample feature/distribution table (e.g. pre-computed cluster proportions or pathomics structure counts) — no count aggregation or prior smoothing is needed.
- You need distances between individual cells or between clusters rather than between whole samples; PILOT's transport step operates on sample-level distributions, not single cells.
- Samples come from distinct technologies or isolation methods (single nucleus vs. single cell, droplet vs. well based) — the article flags cross-technology analysis as an unresolved open challenge for this representation.

## Inputs

- Clustered single-cell AnnData .h5ad file (e.g. Datasets/myocardial_infarction.h5ad) with per-cell cluster labels (clusters_col='cell_subtype') and sample identifiers (sample_col='sampleID')

## Outputs

- sample_distributions.csv: rows = samples, columns = clusters, entries = MAP-estimated Dirichlet-multinomial probabilities p̂_lk

## How to apply

Load the single-cell AnnData (e.g. Datasets/myocardial_infarction.h5ad) with scanpy and aggregate cell counts per sample and cluster using clusters_col = 'cell_subtype' and sample_col = 'sampleID', giving count vectors n_lk. Compute global cluster counts N_k and total N across all samples, then build the hierarchical Dirichlet prior alpha_k = N_k/(N*c) with c = 0.1, which adds pseudo cell counts weighted by the cell distribution of all samples. Compute the closed-form MAP estimate p̂_lk = (n_lk + alpha_k)/(N_l + sum_k alpha_k) for every sample l and cluster k. Rationale: the pseudo-count prior guarantees strictly positive probabilities even for clusters absent in a sample, making downstream optimal transport distances well-defined. Save the result as sample_distributions.csv (rows = samples, columns = clusters) for the Wasserstein/emd2 step.

## Related tools

- **PILOT** (Implements the robust Dirichlet-multinomial MAP estimation of per-sample cluster-distribution matrices that precede optimal transport) — https://github.com/CostaLab/PILOT
- **scanpy** (Loads the single-cell .h5ad AnnData from which per-sample, per-cluster count vectors are aggregated)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('Datasets/myocardial_infarction.h5ad'); counts = adata.obs.groupby(['sampleID','cell_subtype']).size().unstack(fill_value=0)
```

## Evaluation signals

- Each row of sample_distributions.csv sums to 1 (valid probability distribution per sample).
- All entries p̂_lk are strictly positive, even for clusters absent in a given sample, because the pseudo-count Dirichlet prior guarantees this.
- Row/column structure matches the sample set (e.g. myocardial infarction samples by sampleID) and the cluster vocabulary from clusters_col='cell_subtype'.
- MAP estimates agree with raw proportions for abundant clusters (prior weight alpha_k = N_k/(N*0.1) is small relative to large n_lk) but lift zero-count clusters off 0.
- Downstream Wasserstein distances (emd2 from POT) can be computed without modification errors, confirming the distributions are in the format PILOT's transport module expects.

## Limitations

- The prior concentration parameter c = 0.1 is fixed; samples with very few cells will have distributions dominated by the global pseudo counts.
- PILOT does not explicitly correct for technical artefacts in these distributions; it only provides statistical tests to evaluate whether technical factors influence predictions.
- The assumption that tissue samples were collected uniformly (similar tissue areas, same technology) underlies the comparability of these per-sample distributions.

## Evidence

- [discussion] Robust estimation of sample probability distributions using a hierarchical Dirichlet-multinomial model with pseudo cell counts: [evidence span withheld — anchor status: not_found]
- [discussion] pseudo counts weighted globally: [evidence span withheld — anchor status: not_found]
- [readme] Wasserstein input format: "PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples."
- [methods] scanpy loading of h5ad: "adata_T=sc.read_h5ad('Datasets/Kidney_IgAN_T.h5ad')"
- [discussion] transport solver basis: "PILOT is based on the emd2 function from POT library (Flamary et al, 2021)"
