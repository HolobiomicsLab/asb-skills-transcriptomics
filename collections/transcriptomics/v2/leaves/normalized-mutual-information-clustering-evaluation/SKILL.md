---
name: normalized-mutual-information-clustering-evaluation
description: Use when comparing how well competing scRNA-seq embedding methods (e.g.
  scGAE, scDeepCluster, DCA, scVI) recover known cell-type labels, and you need a
  single normalized clustering metric per method per dataset.
when_to_use_negative:
- The clustering result of a single method is already known and no cross-method comparison
  against ground truth is needed
- Data have no ground-truth (or externally accepted) cluster labels — NMI against
  'truth' cannot be computed
- The task is trajectory inference or 2D visualization quality rather than discrete
  cell-cluster recovery (the paper evaluates those separately)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGAE
  role: Primary embedding method being benchmarked; builds a KNN cell graph (K=35)
    and produces the multitask-oriented graph-autoencoder embedding whose Louvain
    clusters are scored by NMI
  repo: github.com/ZixiangLuo1161/scGAE
- name: scDeepCluster
  role: Competitor deep-learning embedding method scored on the same NMI benchmark
- name: DCA
  role: Competitor deep-learning embedding method scored on the same NMI benchmark
- name: scVI
  role: Competitor deep-learning embedding method scored on the same NMI benchmark
- name: Ivis
  role: Competitor embedding method scored on the same NMI benchmark
- name: SAUCIE
  role: Competitor deep-learning embedding method scored on the same NMI benchmark
- name: scGNN
  role: Competitor graph-based deep-learning embedding method scored on the same NMI
    benchmark
- name: ZINB-WaVE
  role: Competitor non-deep-learning embedding method scored on the same NMI benchmark
- name: PHATE
  role: Competitor non-deep-learning embedding/baseline scored on the same NMI benchmark
- name: PCA
  role: Non-deep-learning baseline embedding scored on the same NMI benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:2c2a969eb2d673b0c9d0680072583be83cef83162e38399e1568c3984afe10a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DCA
  - Ivis
  - PCA
  - PHATE
  - R
  - SAUCIE
  - SC3
  - SIMLR
  - SSCC
  - ZINB-WaVE
  - scDeepCluster
  - scGAE
  - scGNN
  - scVI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# normalized-mutual-information-clustering-evaluation

## Summary

Benchmark cell-clustering quality of dimensionality-reduction methods on Splatter-simulated scRNA-seq data (scenario3, scenario4) by applying Louvain clustering to each method's embedding and scoring predictions against ground-truth labels with normalized mutual information (NMI).

## When to use

Use when comparing how well competing scRNA-seq embedding methods (e.g. scGAE, scDeepCluster, DCA, scVI) recover known cell-type labels, and you need a single normalized clustering metric per method per dataset.

## When NOT to use

- The clustering result of a single method is already known and no cross-method comparison against ground truth is needed
- Data have no ground-truth (or externally accepted) cluster labels — NMI against 'truth' cannot be computed
- The task is trajectory inference or 2D visualization quality rather than discrete cell-cluster recovery (the paper evaluates those separately)

## Inputs

- Splatter-simulated scRNA-seq count matrices scenario3/data3 and scenario4/data4
- Ground-truth cell cluster labels for each scenario
- Low-dimensional embeddings produced by each evaluated method (scGAE, scDeepCluster, DCA, scVI, Ivis, SAUCIE, scGNN, ZINB-WaVE, PCA, PHATE)
- Louvain-predicted cluster assignments for each embedding

## Outputs

- nmi_comparison.csv with one row per method per scenario (20 rows: 10 methods x 2 scenarios) containing NMI scores
- Per-scenario ranking of methods by NMI (supporting Fig. 3c,d and Supplementary Fig. S2)

## How to apply

Load the Splatter-simulated datasets scenario3/data3 and scenario4/data4 with ground-truth cluster labels, preprocessing each count matrix with scTransform as in the scGAE pipeline. For each method under test (scGAE, scDeepCluster, DCA, scVI, Ivis, SAUCIE, scGNN, ZINB-WaVE, PCA, PHATE), generate the low-dimensional embedding; for scGAE specifically, build the cell graph with KNN (default K=35) and embed via the multitask-oriented graph autoencoder in the Spektral package. Apply Louvain clustering to each embedding to obtain predicted cell clusters, then compute NMI between predicted clusters and ground truth for each method on each scenario. Assemble results into nmi_comparison.csv with one row per method per scenario (20 rows: 10 methods x 2 scenarios), and validate that the maximum NMI within each scenario matches the expected best method (scGAE in the paper, Fig. 3c,d).

## Related tools

- **scGAE** (Primary embedding method being benchmarked; builds a KNN cell graph (K=35) and produces the multitask-oriented graph-autoencoder embedding whose Louvain clusters are scored by NMI) — github.com/ZixiangLuo1161/scGAE
- **scDeepCluster** (Competitor deep-learning embedding method scored on the same NMI benchmark)
- **DCA** (Competitor deep-learning embedding method scored on the same NMI benchmark)
- **scVI** (Competitor deep-learning embedding method scored on the same NMI benchmark)
- **Ivis** (Competitor embedding method scored on the same NMI benchmark)
- **SAUCIE** (Competitor deep-learning embedding method scored on the same NMI benchmark)
- **scGNN** (Competitor graph-based deep-learning embedding method scored on the same NMI benchmark)
- **ZINB-WaVE** (Competitor non-deep-learning embedding method scored on the same NMI benchmark)
- **PHATE** (Competitor non-deep-learning embedding/baseline scored on the same NMI benchmark)
- **PCA** (Non-deep-learning baseline embedding scored on the same NMI benchmark)

## Examples

```
from sklearn.metrics import normalized_mutual_info_score; nmi = normalized_mutual_info_score(labels_true, louvain_labels_pred)  # per method, per scenario
```

## Evaluation signals

- nmi_comparison.csv contains exactly 20 rows (10 methods x 2 scenarios) with an NMI value for every method on both scenario3 and scenario4
- All NMI values fall within the valid [0, 1] range
- The best-performing method achieves the maximum NMI within each scenario (scGAE in the paper, consistent with Fig. 3c,d and Supplementary Fig. S2)
- Every NMI score was computed after applying the same clustering algorithm (Louvain) to each method's embedding on scTransform-preprocessed data
- scGAE's embedding derives from the documented pipeline: KNN graph with default K=35, Spektral-implemented multitask-oriented graph autoencoder, followed by Louvain-initialized clustering training

## Limitations

- The optimal K for KNN graph construction varies among different datasets and different parts of a dataset, so the default K=35 may not be optimal for new data
- No random seed or stochastic-variation control is reported, so NMI scores may vary between runs
- No software/package versions are specified in the paper; only the README pins versions (e.g. tensorflow 2.3.0, spektral 0.6.1)
- Results are demonstrated on Splatter-simulated scenarios 3 and 4; generalization to other simulation settings or empirical datasets is not established by this benchmark

## Evidence

- [results] scGAE achieved the best performance among all evaluated methods (Fig. 3c,d, Supplementary Fig. S2): [evidence span withheld — anchor status: not_found]
- [discussion] Louvain initial centroids for clustering training: "we use the pre-trained network to generate the initial embedded data which are clustered to obtain the initial centroids by Louvain37"
- [discussion] KNN graph default K=35: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
- [discussion] scTransform preprocessing in the scGAE pipeline: "The scRNA-seq data preprocessing was conducted using scTransform40 in The Seurat package41."
- [readme] Louvain-based clustering training step in the scGAE workflow: "After getting the clustering centers using get_centers_louvain function, perform the clustering training using model.clustering_train() function."
