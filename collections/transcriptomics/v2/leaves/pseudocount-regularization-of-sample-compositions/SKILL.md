---
name: pseudocount-regularization-of-sample-compositions
description: Use when converting per-sample single-cell cluster count vectors into
  probability distributions over clusters as input for optimal transport (Wasserstein/EMD)
  sample-level analysis, especially when some clusters are absent in individual samples.
when_to_use_negative:
- Input is already a normalized sample x cluster probability/composition table that
  feeds directly into the Wasserstein distance computation — re-applying pseudo-counts
  would double-regularize.
- The goal is per-cell or per-cluster analysis rather than sample-level distributions;
  this skill only regularizes sample-level cluster compositions.
- Data are pathomics tissue structures with a different clustering representation
  where the count-multinomial assumption does not hold (check whether the hierarchical
  Dirichlet-multinomial model is appropriate for the structure counts).
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: PILOT
  role: implements robust estimation of sample probability distributions via a hierarchical
    Dirichlet-multinomial model with pseudo cell counts, before Wasserstein distance
    computation
  repo: https://github.com/CostaLab/PILOT
- name: scanpy
  role: loads the single-cell AnnData (.h5ad) used to aggregate per-sample cell-cluster
    count vectors
- name: POT library
  role: downstream consumer of the estimated distributions; its emd2 function computes
    Wasserstein distances between them
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:76d05a730c9268b8b9242bc058765a9d6f82999290d2691f696048575449c46d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - POT library
  - scanpy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# pseudocount-regularization-of-sample-compositions

## Summary

Robustly estimates each sample's cell-cluster probability distribution via a hierarchical Dirichlet-multinomial MAP estimate, where a Dirichlet prior adds pseudo cell counts weighted by the global cell distribution of all samples. This yields strictly positive per-sample cluster probabilities p_lk that feed PILOT's Wasserstein-distance computation.

## When to use

Use when converting per-sample single-cell cluster count vectors into probability distributions over clusters as input for optimal transport (Wasserstein/EMD) sample-level analysis, especially when some clusters are absent in individual samples.

## When NOT to use

- Input is already a normalized sample x cluster probability/composition table that feeds directly into the Wasserstein distance computation — re-applying pseudo-counts would double-regularize.
- The goal is per-cell or per-cluster analysis rather than sample-level distributions; this skill only regularizes sample-level cluster compositions.
- Data are pathomics tissue structures with a different clustering representation where the count-multinomial assumption does not hold (check whether the hierarchical Dirichlet-multinomial model is appropriate for the structure counts).

## Inputs

- single-cell AnnData object (.h5ad, e.g. Datasets/myocardial_infarction.h5ad) with per-cell cluster labels (clusters_col='cell_subtype') and sample identifiers (sample_col='sampleID')

## Outputs

- sample_distributions.csv: per-sample MAP probability distributions over clusters (rows = samples, columns = clusters), each row summing to 1

## How to apply

Aggregate the single-cell AnnData (e.g. Datasets/myocardial_infarction.h5ad loaded with scanpy) into per-sample cell-cluster count vectors n_lk using clusters_col='cell_subtype' and sample_col='sampleID'. Compute global cluster counts N_k and total N across all samples, then construct the Dirichlet prior alpha_k = N_k/(N*c) with c = 0.1, which adds pseudo cell counts weighted by the cell distribution of all samples. Compute the closed-form MAP estimate p̂_lk = (n_lk + alpha_k)/(N_l + sum_k alpha_k) for every sample l and cluster k. The pseudo-count prior guarantees strictly positive probabilities even for clusters absent in a sample, which is required before PILOT feeds these distributions into its emd2-based Wasserstein distance computation.

## Related tools

- **PILOT** (implements robust estimation of sample probability distributions via a hierarchical Dirichlet-multinomial model with pseudo cell counts, before Wasserstein distance computation) — https://github.com/CostaLab/PILOT
- **scanpy** (loads the single-cell AnnData (.h5ad) used to aggregate per-sample cell-cluster count vectors)
- **POT library** (downstream consumer of the estimated distributions; its emd2 function computes Wasserstein distances between them)

## Examples

```
import pilotpy as pl, scanpy as sc; adata = sc.read_h5ad('Datasets/myocardial_infarction.h5ad'); pl.tl.load_data(adata, clusters_col='cell_subtype', sample_col='sampleID')
```

## Evaluation signals

- Every row of sample_distributions.csv sums to 1.
- All p̂_lk > 0: the pseudo-count prior guarantees strictly positive probabilities even for clusters absent in a sample.
- Output has one row per valid sample (e.g. after excluding low-quality samples such as those with fewer than 1000 genes on average per cell) and one column per cluster in clusters_col.
- Estimated p̂_lk are mean-reverting toward the global cell distribution N_k/N for small-count samples (shrinkage toward alpha_k = N_k/(N*c)).

## Limitations

- PILOT assumes tissue samples were collected uniformly (similar tissue areas and same technology); analysis across distinct technologies (single nucleus vs. single cell) or isolation methods is an open challenge.
- PILOT does not explicitly correct for potential technical artefacts; it only provides statistical tests to evaluate their influence, so compositional estimates may absorb batch effects.
- The prior constant c = 0.1 is fixed in the reported setup; alternative values are not benchmarked in the paper.

## Evidence

- [discussion] Robust estimation of sample probability distributions using a hierarchical Dirichlet-multinomial model with pseudo cell counts: [evidence span withheld — anchor status: not_found]
- [discussion] prior adds pseudo cell counts weighted by global distribution: [evidence span withheld — anchor status: not_found]
- [abstract] PILOT Wasserstein between samples: "PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples."
- [discussion] POT emd2 solver: "PILOT is based on the emd2 function from POT library (Flamary et al, 2021)"
- [readme] install pilotpy via pip: "The easiest way to install PILOT and the required packages is using the following way:

pip install pilotpy"
