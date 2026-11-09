---
name: adjusted-rand-index-clustering-evaluation
description: Use when comparing predicted cluster memberships from multiple clustering
  methods or simulation replicates against known/approximated truth labels and you
  need a chance-corrected accuracy score (ARI) per method per dataset.
when_to_use_negative:
- No reference labels exist (marker-based or simulated) — ARI requires an external
  truth labeling to compare against
- You need cluster-number selection rather than label-agreement scoring; the paper
  instead recommends AIC/BIC (returned by BAMMSC) or trying a range of K
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: mclust
  role: Provides adjustedRandIndex used to score every clustering run against truth
    labels
- name: BAMM-SC (BAMMSC R package)
  role: Source of predicted per-individual cluster memberships (mem list) being evaluated
  repo: https://github.com/lichen-lab/BAMMSC
- name: DIMM-SC
  role: Baseline single-individual Dirichlet multinomial clustering whose memberships
    are scored with ARI
  repo: https://github.com/lichen-lab/BAMMSC
- name: SC3
  role: Competing clustering method evaluated via ARI on real and simulated datasets
- name: TSCAN
  role: Competing clustering method evaluated via ARI on real and simulated datasets
- name: SIMLR
  role: Competing clustering method evaluated via ARI in simulation studies
- name: Splatter
  role: Generates simulated multi-individual datasets with known labels for ARI-based
    robustness benchmarks
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_007
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:a6cc22be84f90b9a0065fdfde8ea11ffcaface7c310c17f7a24285f11aeaceda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/BAMMSC
  tools:
  - BAMM-SC
  - BAMM-SC (BAMMSC R package)
  - BAMMSC R package
  - CellTree
  - DIMM-SC
  - MetaNeighbor
  - R
  - Rcpp
  - RcppArmadillo
  - SC3
  - SIMLR
  - Splatter
  - TSCAN
  - dirmult
  - mclust
  - scran
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# adjusted-rand-index-clustering-evaluation

## Summary

Evaluate clustering quality of scRNA-seq cell-type assignments (e.g., BAMM-SC vs. SIMLR, SC3, TSCAN, DIMM-SC) by computing the adjusted Rand index with mclust::adjustedRandIndex against marker-gene-derived or simulated truth labels.

## When to use

Use when comparing predicted cluster memberships from multiple clustering methods or simulation replicates against known/approximated truth labels and you need a chance-corrected accuracy score (ARI) per method per dataset.

## When NOT to use

- No reference labels exist (marker-based or simulated) — ARI requires an external truth labeling to compare against
- You need cluster-number selection rather than label-agreement scoring; the paper instead recommends AIC/BIC (returned by BAMMSC) or trying a range of K

## Inputs

- Predicted cluster membership vectors per individual (e.g., result$mem from BAMMSC or Zpred from BAMMSC.predict)
- Truth label vectors: marker-gene-based approximated labels (real datasets) or simulated true labels (Z)
- Per-method, per-scenario replicate groupings for aggregation

## Outputs

- ARI value per clustering run (numeric, from mclust::adjustedRandIndex)
- Aggregated ARI tables: mean/SD/range per method per dataset (e.g., ari_benchmark_results.csv, simulation_ari_results.csv)
- Training vs. test ARI summaries for supervised runs (bammsc_supervised_ari_results.csv)

## How to apply

For each run, collect the predicted cell memberships — for multi-individual BAMM-SC this is the returned 'mem' list, one vector of cluster labels per individual — and unlist them so they align position-by-position with the union of true labels across individuals. Call mclust::adjustedRandIndex(unlist(predicted), unlist(truth)) to obtain a single ARI per run. For real datasets (human PBMC from 5 donors, mouse lung from 4 mice, human skin from 5 donors), repeat each clustering 10 times and aggregate the ARI replicates into mean, standard deviation, and range (benchmark means: BAMM-SC ~0.487 PBMC, 0.882 mouse lung, 0.762 skin). For simulations, either from the Bayesian hierarchical Dirichlet multinomial mixture or Splatter, score every method against the true simulated labels per replicate and average within each scenario-by-method cell. In supervised BAMMSC.predict runs, compute separate training and test ARIs against the true labels to verify the trained model generalizes to held-out cells.

## Related tools

- **mclust** (Provides adjustedRandIndex used to score every clustering run against truth labels)
- **BAMM-SC (BAMMSC R package)** (Source of predicted per-individual cluster memberships (mem list) being evaluated) — https://github.com/lichen-lab/BAMMSC
- **DIMM-SC** (Baseline single-individual Dirichlet multinomial clustering whose memberships are scored with ARI) — https://github.com/lichen-lab/BAMMSC
- **SC3** (Competing clustering method evaluated via ARI on real and simulated datasets)
- **TSCAN** (Competing clustering method evaluated via ARI on real and simulated datasets)
- **SIMLR** (Competing clustering method evaluated via ARI in simulation studies)
- **Splatter** (Generates simulated multi-individual datasets with known labels for ARI-based robustness benchmarks)

## Examples

```
library(mclust); Zpred=BAMMSC.predict(X=data_BAMMSC,K=4,p=0.5,seed=1); adjustedRandIndex(unlist(Zpred),unlist(Z))
```

## Evaluation signals

- Unlisted predicted and truth vectors must have equal length (all individuals' cells concatenated in matched order) before calling adjustedRandIndex
- Benchmark ARIs should be consistent with published values: BAMM-SC mean ARI ~0.487 (PBMC), 0.882 (mouse lung), 0.762 (human skin); best t-SNE run ARIs ~0.532, 0.910, 0.843
- Simulation result table should reproduce the Fig. 2/Fig. 3 pattern: BAMM-SC ARI increases with number of individuals while competitors stall or decrease, and BAMM-SC is top in nearly all Splatter scenarios
- Supervised runs: test ARI should plateau once the training pool reaches 4000 cells, and the 10%-training run on the 4000-cell dataset completes in roughly 90 seconds
- ARI values must lie within [-1, 1], with values near 0 indicating chance-level agreement

## Limitations

- ARI against marker-gene-based approximated truth labels is only an approximation of true cell type for real datasets
- A 10% cluster-label disagreement simulation scenario exists where BAMM-SC did not achieve the highest ARI — top-ARI in 'most but not all' scenarios should be expected
- ARI does not detect cluster-label permutation issues by design (permutation-invariant), but it cannot assess rare-cluster recovery; the supervised strategy may potentially miss some rare clusters

## Evidence

- [methods] Evaluate clustering with adjustedRandIndex: "adjustedRandIndex(unlist(Zpred),unlist(Z))"
- [full_text] ARI aggregation workflow: [evidence span withheld — anchor status: not_found]
- [results] Benchmark result magnitudes: [evidence span withheld — anchor status: not_found]
- [readme] Supervised classification output: "p: proportion of cells used to train a BAMMSC model and the cluster labels of the rest cells are predicted by the trained label"
