---
name: louvain-clustering-bio-conservation-scoring
description: Use when comparing the output embeddings of multiple scRNA-seq integration
  methods (e.g., scDREAMER, scVI, Harmony) and you need standardized, composite bio-conservation
  and batch-correction scores to rank them.
when_to_use_negative:
- Evaluating a single integration output with no competing methods — min-max normalization
  across methods is required for the composite scores and is undefined for a lone
  method
- Clustering raw counts or non-integrated embeddings to measure biological structure
  alone — this skill is specifically for scoring batch-corrected integrated embeddings
  against cell-type ground truth
- Datasets lacking cell-type annotations, since NMI, ARI, and cell-type ASW all require
  ground-truth labels
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Scanpy
  role: Provides AnnData handling, the kNN graph, and the Louvain clustering used
    to compute conservation metrics
- name: Louvain
  role: Cluster the integrated embedding at the resolution maximizing NMI for downstream
    NMI/ARI scoring
- name: scDREAMER
  role: One of the integrated embeddings being scored; best-performing unsupervised
    method on the heart atlas by combined composite score
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: scDREAMER-Sup
  role: Supervised variant embedding scored against supervised competitors
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: INSCT
  role: Competing batch-aware triplet-network integration method whose embedding is
    scored in the benchmark
  repo: https://github.com/lkmklsmn/insct
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:a0043305c916acd5b1a85ef39ebb0a029b71224c548bc021ef37596b59fb4568
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - ADAM
  - BBKNN
  - Harmony
  - INSCT
  - LIGER
  - Louvain
  - Scanorama
  - Scanpy
  - Seurat
  - iMAP
  - scANVI
  - scDML
  - scDREAMER
  - scDREAMER-Sup
  - scGEN
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 24
---

# Louvain clustering bio-conservation scoring

## Summary

A benchmarking skill for evaluating single-cell data integration methods: it Louvain-clusters each integrated embedding at the resolution that maximizes NMI, computes biological conservation metrics (NMI, ARI, cell-type ASW) alongside batch-correction metrics (ASW batch, PCR batch, graph connectivity, kBET), and aggregates min-max normalized values into composite scores.

## When to use

Use when comparing the output embeddings of multiple scRNA-seq integration methods (e.g., scDREAMER, scVI, Harmony) and you need standardized, composite bio-conservation and batch-correction scores to rank them.

## When NOT to use

- Evaluating a single integration output with no competing methods — min-max normalization across methods is required for the composite scores and is undefined for a lone method
- Clustering raw counts or non-integrated embeddings to measure biological structure alone — this skill is specifically for scoring batch-corrected integrated embeddings against cell-type ground truth
- Datasets lacking cell-type annotations, since NMI, ARI, and cell-type ASW all require ground-truth labels

## Inputs

- Integrated low-dimensional embedding (e.g., from scDREAMER, scVI, Scanorama, Harmony, Seurat v3, BBKNN, INSCT, LIGER, iMAP, scANVI, scGEN, or scDML) stored in a Scanpy AnnData object
- Per-cell batch label vector
- Per-cell cell-type annotation vector (ground truth)

## Outputs

- Per-method table of NMI, ARI, cell-type ASW, ASW batch, PCR batch, graph connectivity, and kBET values (e.g., heart_atlas_integration_metrics.csv)
- Composite bio-conservation, composite batch-correction, and combined composite scores per method
- Best-resolution Louvain cluster assignments per integrated embedding

## How to apply

For each integrated embedding, run Louvain clustering (Scanpy) and sweep the resolution parameter, selecting the resolution that maximizes the NMI between clusters and cell-type labels. Compute biological conservation metrics — NMI, ARI, and cell-type average silhouette width scaled as (ASW+1)/2 — and batch-correction metrics — ASW batch scaled as 1−abs(ASW), PCR batch, graph connectivity, and kBET with k=50 computed on the kNN graph. Min-max normalize each metric across all competing methods, then average into composite bio-conservation, composite batch-correction, and combined composite scores. When a metric is infeasible (e.g., kBET on the 147-batch, 486,134-cell healthy heart atlas), explicitly mark it as excluded for that method rather than imputing a value, as the paper did under computational limitations.

## Related tools

- **Scanpy** (Provides AnnData handling, the kNN graph, and the Louvain clustering used to compute conservation metrics)
- **Louvain** (Cluster the integrated embedding at the resolution maximizing NMI for downstream NMI/ARI scoring)
- **scDREAMER** (One of the integrated embeddings being scored; best-performing unsupervised method on the heart atlas by combined composite score) — https://github.com/Zafar-Lab/scDREAMER
- **scDREAMER-Sup** (Supervised variant embedding scored against supervised competitors) — https://github.com/Zafar-Lab/scDREAMER
- **INSCT** (Competing batch-aware triplet-network integration method whose embedding is scored in the benchmark) — https://github.com/lkmklsmn/insct

## Examples

```
import scanpy as sc; sc.pp.neighbors(adata, use_rep='latent'); sc.tl.louvain(adata, resolution=1.0, key_added='louvain'); from sklearn.metrics import normalized_mutual_info_score as nmi; print(nmi(adata.obs['louvain'], adata.obs['celltype']))
```

## Evaluation signals

- The metrics table contains one row per competing method (twelve methods on the heart atlas) with all metric columns populated or explicitly marked as excluded (kBET on the heart atlas)
- ASW cell-type values are scaled to (ASW+1)/2 and ASW batch values to 1−abs(ASW), so all metrics fall in [0,1] before min-max normalization
- Each metric's min-max normalization is computed across all competing methods before averaging into composite scores
- On the heart atlas benchmark, scDREAMER attains the highest combined composite score, with ~25% improvement over the second-best method driven by batch-correction metrics
- The selected Louvain resolution is the one that maximizes NMI for each method's embedding, not a fixed default

## Limitations

- kBET could not be computed for some methods on the healthy heart atlas dataset (486,134 cells, 147 batches) due to computational limitations, requiring metric exclusion rather than a fair head-to-head comparison
- Louvain resolution must be re-tuned per embedding to maximize NMI, adding a tuning step that couples the clustering result to the metric being optimized
- Min-max normalization means scores are relative to the specific pool of competing methods; adding or removing a method changes all composite scores

## Evidence

- [methods] Louvain clustering at NMI-maximizing resolution: "We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value."
- [other] Metric definitions and composites: [evidence span withheld — anchor status: not_found]
- [methods] kBET infeasibility on heart atlas: "for the healthy heart atlas dataset, kBET could not be computed for some of the methods due to computational limitations in handling large number of cells and batches"
- [methods] Highly variable gene input preparation: [evidence span withheld — anchor status: not_found]
- [abstract] Scalability constraint: "scDREAMER is scalable and can perform atlas-level cross-species (e.g., human and mouse) integration while being faster than other deep-learning-based methods"
