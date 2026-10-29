---
name: integration-metric-evaluation-kbet-pcr
description: Use when you have latent embeddings from one or more single-cell integration
  methods and need comparable bio-conservation and batch-correction scores (NMI/ARI/ASW/kBET/PCR)
  to rank methods or justify ablation choices.
when_to_use_negative:
- Input is raw count data that has not yet been integrated — there is no latent embedding
  to score; run an integration method (e.g. scDREAMER with batch size 128 and ADAM)
  first
- No ground-truth cell-type annotations exist for the dataset, making NMI, ARI, cell-type
  ASW and isolated-label metrics undefined
- Atlas-scale datasets with very many cells and batches (e.g. the healthy heart atlas
  with 486,134 cells), where kBET is computationally infeasible — use the lighter
  metrics instead of forcing kBET to zero
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Scanpy
  role: Hosts AnnData objects, builds the kNN graph, and provides Louvain clustering
    used to compute NMI-maximizing clusters for the bio-conservation metrics
- name: Louvain
  role: Clustering algorithm run on the integrated embedding at the resolution that
    maximizes NMI against ground-truth labels
- name: scDREAMER
  role: Source of the integrated embeddings being evaluated (adversarial VAE + batch
    classifier); its ablated variants (discriminator removed, batch classifier removed)
    are scored with this metric suite
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: INSCT
  role: Baseline integration method whose embeddings are scored with the same metric
    suite for benchmarking
  repo: https://github.com/lkmklsmn/insct
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:0cccac5fb4f9e6754dc242e2764af23129e4fcef628aed4f127250e38020e858
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - ADAM optimizer
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
n_claims: 20
---

# integration-metric-evaluation-kbet-pcr

## Summary

Quantitatively scores integrated single-cell embeddings (e.g. from scDREAMER, scVI, Harmony, Scanorama) using bio-conservation metrics (NMI, ARI, scaled cell-type ASW) and batch-correction metrics (scaled batch ASW, PCR batch, graph connectivity, kBET with k=50), then aggregates them into min–max normalized composite scores for benchmarking.

## When to use

Use when you have latent embeddings from one or more single-cell integration methods and need comparable bio-conservation and batch-correction scores (NMI/ARI/ASW/kBET/PCR) to rank methods or justify ablation choices.

## When NOT to use

- Input is raw count data that has not yet been integrated — there is no latent embedding to score; run an integration method (e.g. scDREAMER with batch size 128 and ADAM) first
- No ground-truth cell-type annotations exist for the dataset, making NMI, ARI, cell-type ASW and isolated-label metrics undefined
- Atlas-scale datasets with very many cells and batches (e.g. the healthy heart atlas with 486,134 cells), where kBET is computationally infeasible — use the lighter metrics instead of forcing kBET to zero

## Inputs

- Integrated latent embedding of cells (e.g. scDREAMER output or baseline embeddings from scVI, Scanorama, Harmony, Seurat, BBKNN, INSCT, LIGER, iMAP, scANVI, scGEN, scDML) as a Scanpy AnnData object
- Ground-truth cell-type annotations per cell (for NMI, ARI, cell-type ASW, isolated-label F1/ASW)
- Batch/donor labels per cell (for batch ASW, PCR batch, kBET, iLISI/cLISI)
- kNN graph on the embedding (k=50 for kBET)

## Outputs

- Per-method metric table (CSV such as pancreas_integration_metrics.csv or ablation_metrics.csv) with raw and scaled metrics
- Composite bio-conservation, composite batch-correction, composite isolated-label, and combined composite scores per method after min–max normalization

## How to apply

Load each integrated latent embedding as a Scanpy AnnData object alongside its batch and ground-truth cell-type annotations. Run Louvain clustering on the embedding, sweeping resolution to pick the value that maximizes NMI against ground-truth labels — this avoids penalizing a method for arbitrary cluster granularity. Compute bio-conservation metrics (NMI, ARI, cell-type ASW rescaled as (ASW+1)/2) and batch-correction metrics (batch ASW rescaled as 1−|ASW|, PCR batch, graph connectivity, and kBET with k=50 on the kNN graph), plus graph iLISI/cLISI via Dijkstra graph distances and isolated-label F1/ASW for the rarest-batch cell types when cell types are skewed across batches. Min–max normalize each metric across the compared method set so all metrics lie in [0,1] (1 = perfect mixing for batch metrics), then average into composite bio-conservation, batch-correction, isolated-label, and combined composite scores. Aggregate all methods into a single results table (e.g. pancreas_integration_metrics.csv or ablation_metrics.csv) before ranking.

## Related tools

- **Scanpy** (Hosts AnnData objects, builds the kNN graph, and provides Louvain clustering used to compute NMI-maximizing clusters for the bio-conservation metrics)
- **Louvain** (Clustering algorithm run on the integrated embedding at the resolution that maximizes NMI against ground-truth labels)
- **scDREAMER** (Source of the integrated embeddings being evaluated (adversarial VAE + batch classifier); its ablated variants (discriminator removed, batch classifier removed) are scored with this metric suite) — https://github.com/Zafar-Lab/scDREAMER
- **INSCT** (Baseline integration method whose embeddings are scored with the same metric suite for benchmarking) — https://github.com/lkmklsmn/insct

## Examples

```
sc.pp.neighbors(adata_latent, n_neighbors=50); sc.tl.louvain(adata_latent, resolution=res_max_nmi); nmi = metrics.normalized_mutual_info_score(adata_latent.obs['louvain'], labels)
```

## Evaluation signals

- Louvain resolution used for each embedding is the one that maximizes NMI against the ground-truth cell labels (not a fixed default resolution)
- All scaled metrics are bounded as defined: NMI/ARI in [0,1], cell-type ASW as (ASW+1)/2 in [0,1], batch ASW as 1−|ASW| and scaled kBET in [0,1] with 1 indicating perfect batch mixing
- Composite scores are computable for every compared method (e.g. all twelve methods on the pancreas task), confirming the results table is complete after min–max normalization
- kBET is computed with k=50 on the kNN graph; for atlas-scale data where kBET is infeasible, the entry is recorded as missing rather than zeroed
- Full scDREAMER should achieve the highest combined composite score versus its ablated variants, with the batch classifier driving batch-correction metrics and the discriminator driving bio-conservation metrics

## Limitations

- For the healthy heart atlas dataset, kBET could not be computed for some methods due to computational limitations in handling large numbers of cells and batches
- Bio-conservation metrics require ground-truth cell-type annotations, so this suite cannot score embeddings for completely unlabeled data
- Min–max normalization makes composite scores relative to the specific set of methods being compared; adding or removing a method changes the scores

## Evidence

- [methods] Louvain resolution maximizing NMI: "We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value."
- [methods] kBET computational limitation on large atlases: "for the healthy heart atlas dataset, kBET could not be computed for some of the methods due to computational limitations in handling large number of cells and batches"
- [readme] Embeddings being evaluated come from adversarial VAE plus batch classifier: "employs a novel adversarial variational autoencoder for learning lower-dimensional cellular embeddings and a batch classifier neural network for the removal of batch effects"
- [results] Metrics used to judge ablated scDREAMER variants: "We performed ablation studies to examine the contribution of the two adversarial components of scDREAMER—adversarial discriminator and batch classifier"
