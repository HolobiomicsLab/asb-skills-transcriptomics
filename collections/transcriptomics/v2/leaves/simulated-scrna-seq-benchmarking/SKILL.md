---
name: simulated-scrna-seq-benchmarking
description: Use when comparing cell-embedding methods (e.g., scGAE vs scDeepCluster,
  DCA, scVI, Ivis, SAUCIE, scGNN, ZINB-WaVE, PCA, PHATE) on simulated scRNA-seq data
  where ground-truth cluster labels are available and clustering quality must be quantified
  by NMI.
when_to_use_negative:
- The data are empirical (real) scRNA-seq datasets without ground-truth cluster labels
  — NMI cannot be computed, and the paper evaluates such data by trajectory/visualization
  instead
- You only need a single method's embedding or 2D visualization rather than a head-to-head
  method comparison across scenarios
- The comparison metric required is something other than clustering-based NMI (e.g.,
  trajectory reconstruction), which this skill does not cover
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scGAE
  role: Method under test; builds KNN cell graph (K=35) and embeds via multitask-oriented
    graph autoencoder in Spektral
  repo: github.com/ZixiangLuo1161/scGAE
- name: scDeepCluster
  role: Competitor embedding method benchmarked on the same simulated datasets
- name: DCA
  role: Competitor embedding method benchmarked on the same simulated datasets
- name: scVI
  role: Competitor embedding method benchmarked on the same simulated datasets
- name: Ivis
  role: Competitor embedding method benchmarked on the same simulated datasets
- name: SAUCIE
  role: Competitor embedding method benchmarked on the same simulated datasets
- name: scGNN
  role: Competitor embedding method benchmarked on the same simulated datasets
- name: ZINB-WaVE
  role: Competitor embedding method benchmarked on the same simulated datasets
- name: PCA
  role: Non-deep-learning baseline embedding method
- name: PHATE
  role: Non-deep-learning baseline embedding method
- name: Spektral
  role: Python package used to implement the scGAE graph autoencoder model
  repo: github.com/ZixiangLuo1161/scGAE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:f5131c7ba0b6674e2e7a8fe190e70fe60bf8fb6d117989a9f37bafa810106b52
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DCA
  - Ivis
  - PCA
  - PHATE
  - SAUCIE
  - Spektral
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
yes_rate: 0.9375
n_claims: 16
---

# simulated-scrna-seq-benchmarking

## Summary

Benchmark dimensionality reduction / embedding methods on Splatter-simulated scRNA-seq data (scenario3 and scenario4) by applying Louvain clustering to each method's embedding and scoring against ground-truth labels with normalized mutual information (NMI).

## When to use

Use when comparing cell-embedding methods (e.g., scGAE vs scDeepCluster, DCA, scVI, Ivis, SAUCIE, scGNN, ZINB-WaVE, PCA, PHATE) on simulated scRNA-seq data where ground-truth cluster labels are available and clustering quality must be quantified by NMI.

## When NOT to use

- The data are empirical (real) scRNA-seq datasets without ground-truth cluster labels — NMI cannot be computed, and the paper evaluates such data by trajectory/visualization instead
- You only need a single method's embedding or 2D visualization rather than a head-to-head method comparison across scenarios
- The comparison metric required is something other than clustering-based NMI (e.g., trajectory reconstruction), which this skill does not cover

## Inputs

- Splatter-simulated scRNA-seq count matrices: scenario3/data3 and scenario4/data4
- Ground-truth cluster labels for the simulated cells
- Preprocessed count matrices (scTransform/Seurat-normalized) as required by the scGAE pipeline

## Outputs

- Per-method, per-scenario NMI values assembled in nmi_comparison.csv (20 rows: 10 methods x 2 scenarios)
- Predicted Louvain cluster assignments for each method's low-dimensional embedding
- Comparison figures showing scGAE vs competitors (cf. Fig. 3c,d and Supplementary Fig. S2)

## How to apply

Load the Splatter-simulated count matrices (scenario3/data3 and scenario4/data4) together with their ground-truth cluster labels, and preprocess the counts with scTransform as implemented in the Seurat package, matching the scGAE pipeline. Run each of the ten embedding methods on the same preprocessed data: scGAE builds a KNN cell graph (default K=35) and embeds via the multitask-oriented graph autoencoder implemented in the Spektral package (see Run_scGAE.ipynb); the competitors are scDeepCluster, DCA, scVI, Ivis, SAUCIE, scGNN, ZINB-WaVE, PCA, and PHATE. Apply Louvain clustering to each method's low-dimensional embedding to obtain predicted cell clusters, then compute NMI between predicted and ground-truth clusters per method per scenario. Assemble the results into nmi_comparison.csv with one row per method per scenario (20 rows: 10 methods x 2 scenarios). Validate by checking that scGAE attains the maximum NMI within each scenario, consistent with Fig. 3c,d and Supplementary Fig. S2.

## Related tools

- **scGAE** (Method under test; builds KNN cell graph (K=35) and embeds via multitask-oriented graph autoencoder in Spektral) — github.com/ZixiangLuo1161/scGAE
- **scDeepCluster** (Competitor embedding method benchmarked on the same simulated datasets)
- **DCA** (Competitor embedding method benchmarked on the same simulated datasets)
- **scVI** (Competitor embedding method benchmarked on the same simulated datasets)
- **Ivis** (Competitor embedding method benchmarked on the same simulated datasets)
- **SAUCIE** (Competitor embedding method benchmarked on the same simulated datasets)
- **scGNN** (Competitor embedding method benchmarked on the same simulated datasets)
- **ZINB-WaVE** (Competitor embedding method benchmarked on the same simulated datasets)
- **PCA** (Non-deep-learning baseline embedding method)
- **PHATE** (Non-deep-learning baseline embedding method)
- **Spektral** (Python package used to implement the scGAE graph autoencoder model) — github.com/ZixiangLuo1161/scGAE

## Examples

```
adj = get_adj(data); model = SCGAE(adj, data); model.train(); centers = get_centers_louvain(model.embedding(), K=35); model.clustering_train(centers); emb = model.embedding()  # then Louvain on emb, NMI vs ground truth
```

## Evaluation signals

- nmi_comparison.csv contains exactly 20 rows (10 methods x 2 scenarios) with valid NMI values in [0, 1] for each method-scenario pair
- scGAE achieves the maximum NMI within both scenario3 and scenario4, reproducing Fig. 3c,d / Supplementary Fig. S2
- All ten methods are run on the identically scTransform-preprocessed count matrices, ensuring a fair comparison
- Louvain clustering is applied to each method's low-dimensional embedding before NMI computation (not to raw counts)
- KNN graph construction for scGAE uses the default K=35 unless adjusted for the dataset

## Limitations

- The optimal K for KNN graph construction varies among different datasets and different parts of a dataset, so the default K=35 may need adjustment
- No random seed or stochastic-variation control is reported, so NMI values may vary across runs
- No software/package versions were specified in the paper (README lists specific versions: tensorflow 2.3.0, spektral 0.6.1, scanpy 1.8.0, etc.), which may affect reproducibility
- Simulated (Splatter) benchmarks may not fully transfer to empirical data, where scGAE is assessed by trajectory and visualization instead

## Evidence

- [results] On the Splatter-simulated clustering tasks (scenario3 and 4), Louvain clustering on embedded data measured by NMI showed that scGAE achieved the best performance among all evaluated methods: [evidence span withheld — anchor status: not_found]
- [discussion] Construct cell graph with KNN K=35: [evidence span withheld — anchor status: not_found]
- [discussion] Preprocessing with scTransform in Seurat: "The scRNA-seq data preprocessing was conducted using scTransform40 in The Seurat package41."
- [discussion] Spektral used to implement the model: "We used the Python package Spektral32 to implement our model."
- [readme] README quick-start: embedding generation step: "5.Generate embedding
Get the latent embedding of the data by model.embedding()"
