---
name: cross-method-dimensionality-reduction-comparison
description: Use when you need an empirical ranking of dimensionality reduction methods
  on the same scRNA-seq datasets with ground-truth cell labels, judged by Louvain-cluster
  NMI.
when_to_use_negative:
- The datasets lack ground-truth cell labels, so NMI against a reference clustering
  cannot be computed.
- The goal is qualitative 2D visualization of trajectories rather than quantitative
  cluster-recovery benchmarking.
- Only one method is of interest — a cross-method comparison table is unnecessary
  overhead.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scGAE
  role: Graph-autoencoder embedding under evaluation; builds a KNN cell graph (K=35)
    and multitask-oriented graph autoencoder in Spektral
  repo: https://github.com/ZixiangLuo1161/scGAE
- name: scDeepCluster
  role: Deep-learning competitor embedding method in the NMI benchmark
- name: DCA
  role: Deep-learning competitor embedding method in the NMI benchmark
- name: scVI
  role: Deep-learning competitor embedding method in the NMI benchmark
- name: Ivis
  role: Competitor embedding method in the NMI benchmark
- name: SAUCIE
  role: Deep-learning competitor embedding method in the NMI benchmark
- name: scGNN
  role: Graph-neural-network competitor embedding method in the NMI benchmark
- name: ZINB-WaVE
  role: Competitor embedding method in the NMI benchmark
- name: PCA
  role: Non-deep-learning baseline embedding in the NMI benchmark
- name: PHATE
  role: Non-deep-learning baseline embedding in the NMI benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:247a901fbd9a21bf6fc7e5cbf65f5c55f0aca4b06408888c8296dd86ead2187c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZixiangLuo1161/scGAE
  tools:
  - DCA
  - Ivis
  - PCA
  - PHATE
  - SAUCIE
  - ZINB-WaVE
  - scDeepCluster
  - scGAE
  - scGNN
  - scVI
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cross-method-dimensionality-reduction-comparison

## Summary

Benchmarks multiple dimensionality reduction / embedding methods for scRNA-seq by running each on identical preprocessed inputs, clustering the embeddings with Louvain, and scoring cluster recovery with NMI. In the scGAE paper this compared 10 methods (scGAE, scDeepCluster, DCA, scVI, Ivis, SAUCIE, scGNN, ZINB-WaVE, PCA, PHATE) on two Splatter-simulated scenarios.

## When to use

Use when you need an empirical ranking of dimensionality reduction methods on the same scRNA-seq datasets with ground-truth cell labels, judged by Louvain-cluster NMI.

## When NOT to use

- The datasets lack ground-truth cell labels, so NMI against a reference clustering cannot be computed.
- The goal is qualitative 2D visualization of trajectories rather than quantitative cluster-recovery benchmarking.
- Only one method is of interest — a cross-method comparison table is unnecessary overhead.

## Inputs

- Splatter-simulated scRNA-seq count matrices (scenario3/data3, scenario4/data4)
- Ground-truth cluster labels for each simulated dataset
- Preprocessed (scTransform-normalized) expression matrices
- Installed embedding methods: scGAE, scDeepCluster, DCA scVI, Ivis, SAUCIE, scGNN, ZINB-WaVE, PCA, PHATE

## Outputs

- nmi_comparison.csv with one row per method per scenario (20 rows: 10 methods x 2 scenarios)
- Per-scenario NMI values per method (as visualized in Fig. 3c,d and Supplementary Fig. S2)
- Louvain cluster assignments per method per dataset

## How to apply

Preprocess all count matrices identically (in the scGAE workflow, using scTransform from the Seurat package) and load ground-truth cluster labels for each dataset. Run each embedding method — scGAE builds a KNN cell graph (default K=35) and embeds via a multitask-oriented graph autoencoder implemented in the Spektral package (Run_scGAE.ipynb), while competitors include deep methods (scDeepCluster, DCA, scVI, Ivis, SAUCIE, scGNN, ZINB-WaVE) and non-deep baselines (PCA, PHATE). Apply Louvain clustering to each method's low-dimensional embedding to obtain predicted cell clusters. Compute normalized mutual information (NMI) between predicted and ground-truth clusters per method per dataset. Assemble one row per method per scenario into nmi_comparison.csv (e.g. 10 methods x 2 Splatter scenarios = 20 rows) and check that the expected winner attains maximum NMI within each scenario.

## Related tools

- **scGAE** (Graph-autoencoder embedding under evaluation; builds a KNN cell graph (K=35) and multitask-oriented graph autoencoder in Spektral) — https://github.com/ZixiangLuo1161/scGAE
- **scDeepCluster** (Deep-learning competitor embedding method in the NMI benchmark)
- **DCA** (Deep-learning competitor embedding method in the NMI benchmark)
- **scVI** (Deep-learning competitor embedding method in the NMI benchmark)
- **Ivis** (Competitor embedding method in the NMI benchmark)
- **SAUCIE** (Deep-learning competitor embedding method in the NMI benchmark)
- **scGNN** (Graph-neural-network competitor embedding method in the NMI benchmark)
- **ZINB-WaVE** (Competitor embedding method in the NMI benchmark)
- **PCA** (Non-deep-learning baseline embedding in the NMI benchmark)
- **PHATE** (Non-deep-learning baseline embedding in the NMI benchmark)

## Examples

```
jupyter nbconvert --to notebook --execute Run_scGAE.ipynb  # then apply Louvain to each embedding and compute NMI into nmi_comparison.csv
```

## Evaluation signals

- nmi_comparison.csv contains exactly 20 rows (10 methods x 2 scenarios) with valid NMI values in [0, 1].
- scGAE attains the maximum NMI within each of scenario3 and scenario4, matching the paper's Fig. 3c,d and Supplementary Fig. S2 finding.
- Every method is clustered with the same algorithm (Louvain) on embeddings of identically scTransform-preprocessed data, so the comparison is not confounded by preprocessing differences.
- NMI is computed against the same ground-truth labels for all methods within a scenario.

## Limitations

- The optimal K for KNN graph construction varies among datasets and even across parts of a dataset, so the default K=35 may need adjustment.
- No software/package versions are specified for all tools, and no random seed or stochastic-variation control is reported, so runs may not be exactly reproducible.
- The comparison covers simulated Splatter scenarios 3 and 4; results may not transfer to other simulation designs or purely empirical data without labels.

## Evidence

- [results] On the Splatter-simulated clustering tasks (scenario3 and 4), Louvain clustering on embedded data measured by NMI showed that scGAE achieved the best performance among all evaluated methods (Fig. 3c,d, Supplementary Fig. S2).: [evidence span withheld — anchor status: not_found]
- [discussion] Default KNN graph parameter.: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
- [discussion] Preprocessing choice.: "The scRNA-seq data preprocessing was conducted using scTransform40 in The Seurat package41."
- [discussion] Optimal K unresolved.: "the optimal K varies among different datasets and different parts of a dataset."
- [readme] Spektral implementation and notebook.: [evidence span withheld — anchor status: external_doc]
