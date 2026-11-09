---
name: knn-parameter-sensitivity-analysis
description: Use when validating that a graph-based trajectory-inference result is
  robust to the KNN hyperparameter on time-labeled single-cell data (e.g., ~90K-cell
  CyTOF panels with day annotations) by correlating inferred pseudotime against known
  sampling days across a KNN sweep.
when_to_use_negative:
- The dataset has no annotated time-point/sampling-day labels, so no pseudotime-vs-day
  Pearson correlation can serve as the accuracy metric
- The goal is lineage/fate discovery on a new dataset rather than validating robustness
  of an existing TI run to the KNN hyperparameter
- The candidate method requires dimensionality reduction (UMAP/t-SNE) before trajectory
  parsing, which the paper shows makes results susceptible to user-defined input parameters
  and confounds a KNN-sensitivity comparison
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: VIA (pyVIA)
  role: 'Primary trajectory inference: computes pseudotime (single_cell_pt_markov)
    on the HNSWlib KNN graph across the KNN sweep; memory=1 disables lazy-teleporting
    MCMC for ablation'
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering algorithm underlying VIA's cluster-graph construction, run with
    edgepruning_clustering_resolution=0.15
  repo: https://github.com/ShobiStassen/VIA
- name: SlingShot
  role: Baseline TI method run on the same data and KNN settings to obtain comparable
    pseudotime orderings
- name: Palantir
  role: Baseline TI method run on the same data and KNN settings to obtain comparable
    pseudotime orderings
- name: STREAM
  role: Baseline TI method run on the same data and KNN settings to obtain comparable
    pseudotime orderings
- name: pandas
  role: Computes Pearson correlations of each method's pseudotime against annotated
    day labels per KNN setting and assembles cytof_pseudotime_correlations.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:b13d17eb6c427b9400d839c519a46bdfcb654dec290fc6a38b4c3827f39170fe
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
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# knn-parameter-sensitivity-analysis

## Summary

Sweep K-nearest-neighbor (KNN) settings around the VIA default (knn=30, HNSWlib graph) on the mESC CyTOF time-series data to quantify how pseudotime accuracy (Pearson correlation vs. annotated sampling days) changes with KNN, comparing full VIA, the no-lazy-teleporting ablation (memory=1), and baselines (Slingshot, Palantir, STREAM, PAGA).

## When to use

Use when validating that a graph-based trajectory-inference result is robust to the KNN hyperparameter on time-labeled single-cell data (e.g., ~90K-cell CyTOF panels with day annotations) by correlating inferred pseudotime against known sampling days across a KNN sweep.

## When NOT to use

- The dataset has no annotated time-point/sampling-day labels, so no pseudotime-vs-day Pearson correlation can serve as the accuracy metric
- The goal is lineage/fate discovery on a new dataset rather than validating robustness of an existing TI run to the KNN hyperparameter
- The candidate method requires dimensionality reduction (UMAP/t-SNE) before trajectory parsing, which the paper shows makes results susceptible to user-defined input parameters and confounds a KNN-sensitivity comparison

## Inputs

- mESC CyTOF time-series dataset (~90,000 cells, 28 antibody markers) with annotated sampling-day labels (mESC_timeseries from pyVIA Datasets)
- AnnData object with features (markers) and day labels per cell
- KNN settings sweep (values around the VIA default knn=30)

## Outputs

- single_cell_pt_markov pseudotime vectors per KNN setting and method
- Pearson correlation table of pseudotime vs. annotated day per method (VIA, no-LT ablation, Slingshot, Palantir, STREAM, PAGA) per KNN value
- cytof_pseudotime_correlations.csv

## How to apply

Load the mESC CyTOF time-series dataset (~90K cells, 28 antibody markers) with annotated sampling-day labels from the pyVIA Datasets resources. Run VIA via via.via_wrapper using the HNSWlib KNN graph with PARC clustering (edgepruning_clustering_resolution=0.15, pseudotime_threshold_TS=30), sweeping KNN around the default knn=30, and extract single_cell_pt_markov pseudotime. Re-run VIA with memory=1 (the non-memory Via 1.0 mode) to ablate the lazy-teleporting MCMC stage under the same KNN sweep. Compute each method's Pearson correlation of inferred pseudotime against the annotated day labels per KNN setting and assemble the results into a single table (cytof_pseudotime_correlations.csv). Accept the result if full VIA reaches ~0.88 correlation and stays above the no-LT ablation (which falls to near PAGA's level) and the baseline methods.

## Related tools

- **VIA (pyVIA)** (Primary trajectory inference: computes pseudotime (single_cell_pt_markov) on the HNSWlib KNN graph across the KNN sweep; memory=1 disables lazy-teleporting MCMC for ablation) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering algorithm underlying VIA's cluster-graph construction, run with edgepruning_clustering_resolution=0.15) — https://github.com/ShobiStassen/VIA
- **SlingShot** (Baseline TI method run on the same data and KNN settings to obtain comparable pseudotime orderings)
- **Palantir** (Baseline TI method run on the same data and KNN settings to obtain comparable pseudotime orderings)
- **STREAM** (Baseline TI method run on the same data and KNN settings to obtain comparable pseudotime orderings)
- **pandas** (Computes Pearson correlations of each method's pseudotime against annotated day labels per KNN setting and assembles cytof_pseudotime_correlations.csv)

## Examples

```
via.via_wrapper(adata, true_label, embedding, knn=30, ncomps=20, jac_std_global=0.15, edgepruning_clustering_resolution=0.15, pseudotime_threshold_TS=30, root=[42])
```

## Evaluation signals

- Full VIA attains a pseudotime-vs-day Pearson correlation of approximately 0.88 on the mESC CyTOF data
- Ablating the lazy-teleporting MCMC stage (memory=1) drops the correlation to near PAGA's level, confirming the component's contribution
- VIA's correlation exceeds Slingshot, Palantir, STREAM, and PAGA at every KNN setting in the sweep
- All correlations are assembled into a single table and saved as cytof_pseudotime_correlations.csv
- PARC clustering uses edgepruning_clustering_resolution=0.15 and terminal-state detection uses pseudotime_threshold_TS=30 across all sweep points

## Limitations

- No software version numbers are stated for VIA or the compared methods, so exact reproducibility of the correlation values may vary across releases
- Specific parameter ranges for the paper's sensitivity/stress tests are not fully reported; the KNN sweep is defined only as 'around the default knn=30'
- No quantitative accuracy metrics are given in the discussion for topology/lineage claims; results defer to supplementary figures, so the ~0.88 correlation benchmark is the primary validation signal

## Evidence

- [methods] HNSWlib KNN graph default sweep: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='',"
- [discussion] CyTOF lazy-teleporting MCMC superiority: "We first applied VIA to the mESC CyTOF dataset and showed that the lazy-teleporting MCMCs strategy in VIA enables it to outperform other methods"
- [methods] pseudotime threshold parameter: [evidence span withheld — anchor status: not_found]
- [discussion] parameter sensitivity claim: "which show that VIA behaves more predictably (allowing controllable degrees of analytical granularity) and accurately than other methods with regards to topology and lineage prediction"
- [discussion] UMAP parameter susceptibility: "We showed that methods which require UMAP (or t-SNE) before parsing MOCA are highly susceptible to user defined input parameters"
