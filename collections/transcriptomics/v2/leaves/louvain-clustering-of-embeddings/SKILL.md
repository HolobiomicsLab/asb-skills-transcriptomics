---
name: louvain-clustering-of-embeddings
description: Use when a cell-level embedding matrix (e.g., scGAE latent representation
  of Splatter-simulated scenario3/4 scRNA-seq data) is available and discrete cell
  clusters must be recovered and scored by NMI.
when_to_use_negative:
- No low-dimensional embedding exists yet — raw count matrices should first be preprocessed
  (scTransform) and embedded (e.g., via scGAE's KNN graph, K=35, and graph autoencoder)
  before Louvain is applied.
- The goal is trajectory inference or 2D visualization rather than discrete cluster
  recovery — the paper evaluates clustering via NMI separately from trajectory tasks
  where scGAE and scGNN perform similarly.
- A direct 2D layout is the only representation available — the article notes the
  complex data structure is hard to embed directly into two dimensions by a graph
  autoencoder, so clustering should use the intermediate-dimension embedding rather
  than a distorted 2D projection.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGAE
  role: produces the latent embedding and provides get_centers_louvain / clustering_train
    for Louvain-based centroid initialization and clustering training
  repo: github.com/ZixiangLuo1161/scGAE
- name: scDeepCluster
  role: competitor method whose embedding is clustered with the same Louvain procedure
    for NMI benchmarking
- name: scVI
  role: competitor method whose embedding is clustered with the same Louvain procedure
    for NMI benchmarking
- name: scGNN
  role: competitor method whose embedding is clustered with the same Louvain procedure
    for NMI benchmarking
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:31aae752275ba2432c68daeb523f072e90a760bd382213ddf1d5d8f5c910386e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DCA
  - Ivis
  - PHATE
  - SAUCIE
  - ZINB-WaVE
  - scDeepCluster
  - scGAE
  - scGNN
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9565217391304348
n_claims: 23
---

# louvain-clustering-of-embeddings

## Summary

Applies Louvain community detection to low-dimensional single-cell embeddings (e.g., from scGAE's multitask-oriented graph autoencoder) to obtain predicted cell clusters and initial centroids, evaluated by NMI against ground-truth labels.

## When to use

Use when a cell-level embedding matrix (e.g., scGAE latent representation of Splatter-simulated scenario3/4 scRNA-seq data) is available and discrete cell clusters must be recovered and scored by NMI.

## When NOT to use

- No low-dimensional embedding exists yet — raw count matrices should first be preprocessed (scTransform) and embedded (e.g., via scGAE's KNN graph, K=35, and graph autoencoder) before Louvain is applied.
- The goal is trajectory inference or 2D visualization rather than discrete cluster recovery — the paper evaluates clustering via NMI separately from trajectory tasks where scGAE and scGNN perform similarly.
- A direct 2D layout is the only representation available — the article notes the complex data structure is hard to embed directly into two dimensions by a graph autoencoder, so clustering should use the intermediate-dimension embedding rather than a distorted 2D projection.

## Inputs

- low-dimensional cell embedding matrix produced by scGAE (model.embedding()) or a competitor method (scDeepCluster, DCA, scVI, Ivis, SAUCIE, scGNN, ZINB-WaVE, PCA, PHATE)
- scTransform-preprocessed scRNA-seq count matrices (e.g., scenario3/data3 and scenario4/data4 Splatter-simulated datasets)
- ground-truth cluster labels for each cell

## Outputs

- predicted cluster label per cell from Louvain clustering
- initial cluster centroids for scGAE's clustering-training phase
- per-method, per-scenario NMI values (e.g., nmi_comparison.csv with 20 rows: 10 methods x 2 scenarios; or a 4-row table of dropout rate vs NMI)

## How to apply

After the embedding step (scGAE trained via the Spektral package on a KNN cell graph with default K=35), run Louvain clustering on the embedded cells; in the scGAE workflow Louvain is used both to obtain initial centroids for the clustering-training phase (get_centers_louvain, then model.clustering_train()) and to produce the final predicted cluster labels. Compare predicted labels with ground-truth cluster labels using normalized mutual information (NMI); on scenario4 dropout variants (12%, 17%, 23%, 30%) scGAE's Louvain-based NMI values were 0.62, 0.62, 0.65, and 0.61, with reproduced values accepted in a tolerance band such as 0.55-0.70. In benchmark settings, apply the identical Louvain procedure to every competitor's embedding (scDeepCluster, DCA, scVI, Ivis, SAUCIE, scGNN, ZINB-WaVE, PCA, PHATE) on scTransform-preprocessed count matrices so NMI is comparable across methods, then assemble nmi_comparison.csv with one row per method per scenario.

## Related tools

- **scGAE** (produces the latent embedding and provides get_centers_louvain / clustering_train for Louvain-based centroid initialization and clustering training) — github.com/ZixiangLuo1161/scGAE
- **scDeepCluster** (competitor method whose embedding is clustered with the same Louvain procedure for NMI benchmarking)
- **scVI** (competitor method whose embedding is clustered with the same Louvain procedure for NMI benchmarking)
- **scGNN** (competitor method whose embedding is clustered with the same Louvain procedure for NMI benchmarking)

## Examples

```
centers = get_centers_louvain(embedding, n_clusters); model.clustering_train(centers); labels = model.embedding()
```

## Evaluation signals

- Each NMI value falls within the pre-specified tolerance band around the reported number (e.g., 0.55-0.70 for scenario4 dropout variants with expected values 0.62, 0.62, 0.65, 0.61).
- In the cross-method benchmark, scGAE achieves the maximum NMI within each of scenario3 and scenario4, consistent with the paper's finding.
- nmi_comparison.csv contains exactly 20 rows (10 methods x 2 scenarios) with one NMI value per method per scenario.
- Every cell in the embedding receives exactly one predicted cluster label, and the number of predicted clusters is consistent with the ground-truth cluster structure.
- Louvain-derived centroids successfully seed the subsequent clustering-training phase (model.clustering_train()) without error.

## Limitations

- No random seed or stochastic-variation control is reported, so Louvain-based cluster assignments may vary between runs; reproduced NMI values are accepted only within a tolerance band.
- The optimal K for the KNN graph feeding the embedding varies among datasets and different parts of a dataset, and the default K=35 may need adjustment, indirectly affecting clustering quality.
- Louvain operates on the intermediate-dimension embedding plus tSNE visualization rather than a direct 2D graph-autoencoder embedding, since complex data structure is hard to embed directly into two dimensions.

## Evidence

- [discussion] In experiments, we use the pre-trained network to generate the initial embedded data which are clustered to obtain the initial centroids by Louvain37.: "In experiments, we use the pre-trained network to generate the initial embedded data which are clustered to obtain the initial centroids by Louvain37."
- [readme] After getting the clustering centers using get_centers_louvain function, perform the clustering training using model.clustering_train() function.: "After getting the clustering centers using get_centers_louvain function, perform the clustering training using model.clustering_train() function."
- [other] Apply Louvain clustering to each method's low-dimensional embedding to obtain predicted cell clusters.: [evidence span withheld — anchor status: not_found]
- [other] Compute the normalized mutual information (NMI) between predicted clusters and ground truth for each method on each scenario.: [evidence span withheld — anchor status: not_found]
