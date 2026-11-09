---
name: ablation-variant-comparison-lazy-teleporting
description: Use when you must isolate the performance contribution of a specific
  algorithmic component (e.g., lazy-teleporting MCMC in VIA) by comparing a full-method
  run to an ablated variant and baselines on time-series single-cell data with day
  labels.
when_to_use_negative:
- The dataset lacks time-point or sampling-day labels, so pseudotime cannot be validated
  against a ground-truth temporal ordering
- The goal is topology/lineage evaluation (terminal states, fate probabilities) rather
  than pseudotime accuracy against known time labels
- The data is scRNA-seq rather than low-dimensional proteomic/mass-cytometry feature
  space, where the ablation effect on CyTOF was specifically demonstrated
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: VIA (pyVIA)
  role: Runs the full lazy-teleporting MCMC trajectory and the ablated memory=1 (Via
    1.0 non-memory) variant via via.via_wrapper over a KNN sweep
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering algorithm producing the cluster-graph nodes underlying VIA's trajectory
    computations
  repo: https://github.com/ShobiStassen/VIA
- name: SlingShot
  role: Trajectory-inference baseline used for comparison pseudotime ordering
- name: Palantir
  role: Trajectory-inference baseline used for comparison pseudotime ordering
- name: STREAM
  role: Trajectory-inference baseline used for comparison pseudotime ordering
- name: pandas
  role: Computes Pearson correlations of each method's pseudotime against annotated
    day labels and assembles the CSV table
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:275789b7e277105832956951ed9437f43e93d1564fa9d6453b5b5871ea1c493a
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# ablation-variant-comparison-lazy-teleporting

## Summary

An ablation-comparison protocol that quantifies the contribution of VIA's lazy-teleporting MCMC stage by re-running VIA in its non-memory mode (memory=1) and against TI baselines (Slingshot, Palantir, STREAM, PAGA), scoring each method's pseudotime by Pearson correlation to annotated sampling-day labels on the mESC CyTOF dataset (~90K cells, 28 antibody markers).

## When to use

Use when you must isolate the performance contribution of a specific algorithmic component (e.g., lazy-teleporting MCMC in VIA) by comparing a full-method run to an ablated variant and baselines on time-series single-cell data with day labels.

## When NOT to use

- The dataset lacks time-point or sampling-day labels, so pseudotime cannot be validated against a ground-truth temporal ordering
- The goal is topology/lineage evaluation (terminal states, fate probabilities) rather than pseudotime accuracy against known time labels
- The data is scRNA-seq rather than low-dimensional proteomic/mass-cytometry feature space, where the ablation effect on CyTOF was specifically demonstrated

## Inputs

- mESC CyTOF time-series dataset (~90K cells, 28 antibody markers) with annotated sampling-day labels (mESC_timeseries from pyVIA Datasets folder)
- AnnData object with PCA stored in adata.obsm['X_pca']
- KNN sweep values around default knn=30

## Outputs

- single_cell_pt_markov pseudotime per cell from full VIA and memory=1 ablated VIA
- Pseudotime orderings from Slingshot, Palantir, STREAM, and PAGA baselines
- cytof_pseudotime_correlations.csv: table of Pearson correlations (pseudotime vs annotated days) per method per KNN setting

## How to apply

Load the mESC CyTOF time-series dataset (mESC_timeseries from the pyVIA Datasets folder) with annotated sampling-day labels. Run VIA via via.via_wrapper with a KNN sweep around the default knn=30 for the HNSWlib graph, using PARC clustering with edgepruning_clustering_resolution=0.15 and pseudotime_threshold_TS=30 to obtain single_cell_pt_markov pseudotime. Re-run the ablated variant by setting memory=1 (the non-memory Via 1.0 mode, lacking the lazy-teleporting higher-order walk) under identical KNN settings, and run Slingshot, Palantir, STREAM, and PAGA baselines on the same data. In pandas, compute the Pearson correlation of each method's inferred pseudotime against the annotated day labels per KNN setting, then assemble all correlations into cytof_pseudotime_correlations.csv. The rationale is that ablation to memory=1 collapses VIA's correlation to near PAGA's level, demonstrating the lazy-teleporting MCMC component's critical contribution (~88% Pearson correlation for full VIA).

## Related tools

- **VIA (pyVIA)** (Runs the full lazy-teleporting MCMC trajectory and the ablated memory=1 (Via 1.0 non-memory) variant via via.via_wrapper over a KNN sweep) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering algorithm producing the cluster-graph nodes underlying VIA's trajectory computations) — https://github.com/ShobiStassen/VIA
- **SlingShot** (Trajectory-inference baseline used for comparison pseudotime ordering)
- **Palantir** (Trajectory-inference baseline used for comparison pseudotime ordering)
- **STREAM** (Trajectory-inference baseline used for comparison pseudotime ordering)
- **pandas** (Computes Pearson correlations of each method's pseudotime against annotated day labels and assembles the CSV table)

## Examples

```
v = via.via_wrapper(adata, true_label, embedding=adata.obsm['X_pca'][:,0:2], knn=30, jac_std_global=0.15, pseudotime_threshold_TS=30, memory=1, root=[42])
```

## Evaluation signals

- Full VIA attains a pseudotime-vs-day Pearson correlation of approximately 0.88 on the mESC CyTOF data
- The memory=1 ablated run drops the correlation to near PAGA's level, confirming the lazy-teleporting MCMC stage is the differentiating component
- VIA's correlation exceeds that of the no-LT ablation and the Slingshot/Palantir/STREAM/PAGA baselines across the KNN sweep
- cytof_pseudotime_correlations.csv contains one correlation row per method per KNN setting
- PARC settings are consistent across runs: edgepruning_clustering_resolution=0.15 and pseudotime_threshold_TS=30

## Limitations

- The ablation result is demonstrated on the mESC CyTOF proteomic modality; transfer to other modalities is not established in this source
- No software version numbers are stated for VIA or the compared methods, complicating exact reproduction
- Specific parameter ranges for the KNN sweep beyond 'around the default knn=30' are not reported in the discussion

## Evidence

- [discussion] MCMC ablation drops correlation: "We first applied VIA to the mESC CyTOF dataset and showed that the lazy-teleporting MCMCs strategy in VIA enables it to outperform other methods"
- [intro] objective of ablation: "reconstruct cell lineages based on lazy-teleporting random walks integrated with Markov chain Monte"
- [readme] non-memory mode: "StaVia extends the lazy-teleporting walks to higher order random walks with **memory** to allow better lineage detection, pathway recovery and preservation of global features"
- [methods] wrapper invocation: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='',"
- [methods] parameter meaning: [evidence span withheld — anchor status: not_found]
