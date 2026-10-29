---
name: pseudotime-correlation-assessment
description: Use when benchmarking a trajectory inference method on dyntoy synthetic
  datasets where gold-standard pseudotime and cell-fate labels are known and you must
  quantify agreement of inferred pseudotime.
when_to_use_negative:
- The dataset has no gold-standard pseudotime or milestone labels (e.g., real biological
  data), since correlation against ground truth is undefined
- You only need topology or terminal-fate detection quality, not pseudotime ordering
  — F1-branch/GED or F1 cell fate alone suffice
- The data are not a trajectory-shaped cell ordering (e.g., static clustering or already
  a feature table)
tools:
- name: VIA
  role: Method under evaluation; its inferred pseudotime is correlated against the
    dyntoy gold standard
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering step that builds the cluster-graph VIA uses to compute pseudotime
  repo: https://github.com/ShobiStassen/VIA
- name: dynverse/dyntoy
  role: Generates the synthetic trajectory datasets with gold-standard pseudotime
    used as the reference in the correlation
  repo: https://github.com/dynverse/dyntoy
- name: Palantir
  role: Baseline trajectory method scored alongside VIA on the same nine datasets
- name: Slingshot
  role: Baseline trajectory method scored alongside VIA on the same nine datasets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:d483d03be0be2f3e399217dee515982735369d943ee5b104255439d8de3cbc18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - Monocle3
  - PARC
  - Palantir
  - STREAM
  - SlingShot
  - Slingshot
  - VIA
  - dynverse/dyntoy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# pseudotime-correlation-assessment

## Summary

Scores an inferred single-cell trajectory's pseudotime against a synthetic gold standard (dyntoy) using the dynverse pseudotime-correlation metric, as part of the five-metric composite benchmark used to rank VIA against Palantir, PAGA, Slingshot, STREAM, and Monocle3.

## When to use

Use when benchmarking a trajectory inference method on dyntoy synthetic datasets where gold-standard pseudotime and cell-fate labels are known and you must quantify agreement of inferred pseudotime.

## When NOT to use

- The dataset has no gold-standard pseudotime or milestone labels (e.g., real biological data), since correlation against ground truth is undefined
- You only need topology or terminal-fate detection quality, not pseudotime ordering — F1-branch/GED or F1 cell fate alone suffice
- The data are not a trajectory-shaped cell ordering (e.g., static clustering or already a feature table)

## Inputs

- dyntoy synthetic dataset (1000 cells) with gold-standard milestone_network, pseudotime, and cell-fate labels
- Inferred pseudotime per cell from VIA (pyVIA output on first 10 PCs)
- Inferred pseudotime from baseline methods (Monocle3, Slingshot, STREAM, PAGA, Palantir)
- dynverse metric definitions (GED, F1-branch, Ipsen-Mikhailov, pseudotime correlation, F1 cell fate)

## Outputs

- Per-dataset pseudotime correlation scores for each method against the dyntoy gold standard
- Composite accuracy table aggregating the five dynverse metric means per dataset (dyntoy_ti_benchmark_metrics.csv)

## How to apply

Generate or load a dyntoy toy dataset (e.g., the 4-leaf multifurcation, 1000 cells) which retains gold-standard topology, pseudotime, and cell-fate labels. Reduce to the first 10 PCs and run VIA (knn=20, fixed random_seed) via via.via_wrapper for connected topologies, or via.via_wrapper_disconnected with root=[1,1] and preserve_disconnected=True for Disconn1; run baselines with benchmark-prescribed settings. Score each inferred trajectory against the dyntoy gold standard using the five dynverse metrics — graph edit distance (GED), F1-branch, Ipsen-Mikhailov distance, pseudotime correlation, and F1 cell fate — and aggregate per-dataset means into a composite accuracy table. Rationale: pseudotime correlation directly measures whether the ordering of cells along the inferred trajectory matches the simulated ground truth, complementing topology-level (GED, Ipsen-Mikhailov) and fate-level (F1-branch, F1 cell fate) metrics.

## Related tools

- **VIA** (Method under evaluation; its inferred pseudotime is correlated against the dyntoy gold standard) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering step that builds the cluster-graph VIA uses to compute pseudotime) — https://github.com/ShobiStassen/VIA
- **dynverse/dyntoy** (Generates the synthetic trajectory datasets with gold-standard pseudotime used as the reference in the correlation) — https://github.com/dynverse/dyntoy
- **Palantir** (Baseline trajectory method scored alongside VIA on the same nine datasets)
- **Slingshot** (Baseline trajectory method scored alongside VIA on the same nine datasets)

## Examples

```
via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='')
```

## Evaluation signals

- VIA's pseudotime correlation against the dyntoy gold standard should exceed that of Palantir, PAGA, Slingshot, STREAM, and Monocle3
- VIA should attain the best composite accuracy across all five dynverse metrics (GED, F1-branch, Ipsen-Mikhailov, pseudotime correlation, F1 cell fate)
- On the four-leaf multifurcation dataset, VIA must detect the elusive M2 terminal state that all baselines merge with the M8 lineage
- VIA should correctly recover the cyclic and disconnected topologies that tree-based baselines miss
- Per-dataset metric means should be reproducible with a fixed random_seed (knn=20) before saving dyntoy_ti_benchmark_metrics.csv

## Limitations

- dyntoy data are simplistic and 'far from realistic'; the README recommends dyngen for more realistic synthetic benchmarks
- The discussion reports no quantitative accuracy metrics in the main text — topology and lineage claims defer to supplementary figures
- No software version numbers are stated for VIA or the compared methods, complicating exact reproduction
- Methods requiring a UMAP/t-SNE embedding first (e.g., at MOCA scale) are highly susceptible to user-defined input parameters, which can distort comparisons

## Evidence

- [readme] dyntoy simulates single-cell expression data in which a single-cell trajectory is present.: [evidence span withheld — anchor status: external_doc]
- [methods] Run VIA via wrapper function with marker genes: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='',"
- [discussion] VIA behaves more predictably and accurately than other methods: "which show that VIA behaves more predictably (allowing controllable degrees of analytical granularity) and accurately than other methods with regards to topology and lineage prediction"
- [methods] Run VIA for disconnected trajectories with multiple roots: "via.via_wrapper_disconnected(adata_counts, true_label, embedding=adata_counts.obsm['X_pca'][:, 0:2], root=[1,1], preserve_disconnected=True"
