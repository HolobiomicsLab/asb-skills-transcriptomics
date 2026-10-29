---
name: random-walk-trajectory-benchmarking
description: Use when evaluating whether a lazy-teleporting random-walk/MCMC stage
  improves pseudotime accuracy on time-series single-cell (e.g., CyTOF) data against
  ablated variants and TI baselines.
when_to_use_negative:
- The input lacks annotated time/day labels, since the validation metric is a Pearson
  correlation of inferred pseudotime against annotated sampling days
- The goal is simulating synthetic toy trajectories for prototyping TI methods — use
  dyntoy/dyngen-style generators instead of this CyTOF benchmark
- The dataset is tree-only scRNA-seq where baselines like Slingshot/STREAM are already
  adequate and the lazy-teleporting MCMC ablation question is not of interest
tools:
- name: VIA (pyVIA)
  role: 'Primary method: computes pseudotime via lazy-teleporting random walks with
    MCMC refinement; ablated via memory=1'
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering stage under VIA (edgepruning_clustering_resolution=0.15) to build
    the cluster-graph
  repo: https://github.com/ShobiStassen/VIA
- name: SlingShot
  role: TI baseline for comparable pseudotime ordering on the same data and KNN settings
- name: Palantir
  role: TI baseline for comparable pseudotime ordering on the same data and KNN settings
- name: STREAM
  role: TI baseline for comparable pseudotime ordering on the same data and KNN settings
- name: pandas
  role: Computes Pearson correlations of pseudotime vs day labels and assembles the
    results table
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:712ff3807f3fcdc7de11b2c718765bccf4686b64b68fd4e67cde776291d591f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - PARC
  - Palantir
  - STREAM
  - SlingShot
  - VIA
  - VIA (pyVIA)
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# random-walk-trajectory-benchmarking

## Summary

Benchmark lazy-teleporting random-walk trajectory inference (VIA) against ablation and TI baselines (Slingshot, Palantir, STREAM, PAGA) by correlating inferred pseudotime with annotated time labels on the mESC CyTOF dataset (~90K cells, 28 antibody markers), demonstrating the MCMC stage's contribution to pseudotime accuracy.

## When to use

Use when evaluating whether a lazy-teleporting random-walk/MCMC stage improves pseudotime accuracy on time-series single-cell (e.g., CyTOF) data against ablated variants and TI baselines.

## When NOT to use

- The input lacks annotated time/day labels, since the validation metric is a Pearson correlation of inferred pseudotime against annotated sampling days
- The goal is simulating synthetic toy trajectories for prototyping TI methods — use dyntoy/dyngen-style generators instead of this CyTOF benchmark
- The dataset is tree-only scRNA-seq where baselines like Slingshot/STREAM are already adequate and the lazy-teleporting MCMC ablation question is not of interest

## Inputs

- mESC CyTOF time-series dataset (~90K cells, 28 antibody markers) with annotated sampling-day labels (mESC_timeseries from the pyVIA Datasets folder)
- AnnData object with PCA stored in adata.obsm['X_pca']
- Root/start cell specification (root_user)

## Outputs

- single_cell_pt_markov pseudotime vector per cell from VIA (memory and memory=1 ablation)
- Pseudotime orderings from Slingshot, Palantir, STREAM, and PAGA baselines
- cytof_pseudotime_correlations.csv — table of Pearson correlations (pseudotime vs annotated days) per method per KNN setting

## How to apply

Load the mESC CyTOF time-series dataset with annotated sampling-day labels from the pyVIA Datasets resources (mESC_timeseries), then run VIA via via.via_wrapper with a sweep of KNN values around the default knn=30 for the HNSWlib KNN graph, using PARC clustering with edgepruning_clustering_resolution=0.15 and pseudotime_threshold_TS=30 to obtain single_cell_pt_markov pseudotime. Re-run VIA in the ablated no-lazy-teleporting mode by setting memory=1 (non-memory Via 1.0 mode) under the same KNN sweep, since the article attributes VIA's CyTOF win specifically to the lazy-teleporting MCMC strategy. Run Slingshot, Palantir, and STREAM (plus PAGA) on the same data and KNN settings to obtain comparable pseudotime orderings. In pandas, compute the Pearson correlation of each method's inferred pseudotime against the annotated day labels for every KNN setting, assemble a single table, and save it as cytof_pseudotime_correlations.csv.

## Related tools

- **VIA (pyVIA)** (Primary method: computes pseudotime via lazy-teleporting random walks with MCMC refinement; ablated via memory=1) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering stage under VIA (edgepruning_clustering_resolution=0.15) to build the cluster-graph) — https://github.com/ShobiStassen/VIA
- **SlingShot** (TI baseline for comparable pseudotime ordering on the same data and KNN settings)
- **Palantir** (TI baseline for comparable pseudotime ordering on the same data and KNN settings)
- **STREAM** (TI baseline for comparable pseudotime ordering on the same data and KNN settings)
- **pandas** (Computes Pearson correlations of pseudotime vs day labels and assembles the results table)

## Examples

```
import via; v = via.via_wrapper(adata, true_label, embedding, knn=30, jac_std_global=0.15, root=[42], dataset='', pseudotime_threshold_TS=30)
```

## Evaluation signals

- VIA's pseudotime-vs-annotated-day Pearson correlation reaches approximately 0.88 on the mESC CyTOF data
- The memory=1 (no-lazy-teleporting) ablation drops VIA's correlation to near PAGA's level, confirming the MCMC component's contribution
- VIA's correlation exceeds all baseline methods (Slingshot, Palantir, STREAM, PAGA) across the KNN sweep
- The results table cytof_pseudotime_correlations.csv contains one row per method per KNN setting with finite correlation values in [-1, 1]
- Running the full KNN sweep (around knn=30) yields stable VIA rankings rather than a single-point result

## Limitations

- Results defer to supplementary figures; no quantitative accuracy metrics are reported in the discussion for the claimed superior topology and lineage prediction
- No software version numbers are stated for VIA or the compared methods, complicating exact reproduction
- Specific parameter ranges and settings used in the stress tests are not fully reported; KNN values must be swept around the default rather than fixed
