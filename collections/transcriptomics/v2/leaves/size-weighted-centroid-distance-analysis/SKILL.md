---
name: size-weighted-centroid-distance-analysis
description: Use when clustering single-cell RNA-seq embeddings and you need a data-adaptive
  criterion to decide which cluster pairs to merge and when merging stops, without
  pre-specifying the cluster count.
when_to_use_negative:
- The number of clusters is already known or fixed externally by the experimental
  design — the adaptive merging would alter a validated clustering
- Input is raw gene expression without scAce embeddings and initial cluster labels;
  the criterion is defined on scAce's learned embedding space and centroids, not on
  raw counts
- You need a hard-splitting of clusters rather than merging — scAce's enhancement
  mode applies a cluster splitting approach to existing results, not this merging
  rule
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scAce
  role: Source of the adaptive cluster merging implementation, pre-trained VAE embeddings,
    and initial cluster labels the merging criterion operates on
  repo: https://github.com/sldyns/scAce
- name: scanpy
  role: Provides the AnnData container holding scace_emb embeddings and scace_cluster
    labels used as inputs to the merging analysis
- name: Python 3.8
  role: Implementation language of the scAce package and the merging reconstruction
    script
- name: Leiden
  role: Produces initial cluster labels during de novo initialization that the adaptive
    merging step then refines
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:ea2e41e8b0acb34a9bda6def9a14f7cafa61df3a3fb2e852d53e33fbfd1271c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Leiden
  - Python
  - Python 3.8
  - scAce
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# size-weighted-centroid-distance-analysis

## Summary

Implements scAce's adaptive cluster merging rule: compute size-weighted inter-cluster centroid distances (Eqs. 7-10) between every cluster pair and iteratively merge the closest pair while the minimum weighted distance stays below d_bar/2, so the number of clusters is determined from the data rather than estimated in advance.

## When to use

Use when clustering single-cell RNA-seq embeddings and you need a data-adaptive criterion to decide which cluster pairs to merge and when merging stops, without pre-specifying the cluster count.

## When NOT to use

- The number of clusters is already known or fixed externally by the experimental design — the adaptive merging would alter a validated clustering
- Input is raw gene expression without scAce embeddings and initial cluster labels; the criterion is defined on scAce's learned embedding space and centroids, not on raw counts
- You need a hard-splitting of clusters rather than merging — scAce's enhancement mode applies a cluster splitting approach to existing results, not this merging rule

## Inputs

- scanpy AnnData object with cell embeddings in adata.obsm['scace_emb'] and initial cluster labels in adata.obs['scace_cluster']
- Cluster centroids derived from the scACE VAE embeddings and initial labels

## Outputs

- Merge decision sequence (which cluster pairs are merged at each iteration)
- Final cluster count (positive integer) after the d_bar/2 stopping criterion is met
- Updated cluster labels per cell (e.g. cluster_merging_results.csv with per-iteration records)

## How to apply

Load cell embeddings (adata.obsm['scace_emb']) and initial cluster labels (adata.obs['scace_cluster']) from the scAce run on a preprocessed AnnData object (filtered with min_cells=3/min_genes=200, then normalized, log1p and scaled). Compute intra-cluster distances and size-weighted inter-cluster centroid distances for every cluster pair as defined by Eqs. 7-10 of Methods 2.3. Iteratively merge the pair with the smallest size-weighted distance (w_ij * d_ij_inter) as long as it is below the adaptive threshold d_bar/2, recomputing cluster labels and centroids after each merge; stop when no pair satisfies w_ij * d_ij_inter < d_bar/2. Record the merge sequence and the cluster count at each iteration to show the merging was driven by the criterion rather than an externally chosen K.

## Related tools

- **scAce** (Source of the adaptive cluster merging implementation, pre-trained VAE embeddings, and initial cluster labels the merging criterion operates on) — https://github.com/sldyns/scAce
- **scanpy** (Provides the AnnData container holding scace_emb embeddings and scace_cluster labels used as inputs to the merging analysis)
- **Python 3.8** (Implementation language of the scAce package and the merging reconstruction script)
- **Leiden** (Produces initial cluster labels during de novo initialization that the adaptive merging step then refines)

## Examples

```
from scace import run_scace; adata = run_scace(adata); emb = adata.obsm['scace_emb']; labels = adata.obs['scace_cluster']
```

## Evaluation signals

- cluster_merging_results.csv exists and contains fields for each merge iteration plus a final cluster count
- Final cluster count is a positive integer consistent with the recorded merge sequence (initial count minus number of merges)
- At the stopping point, the minimum size-weighted inter-cluster distance exceeds d_bar/2, and every merged pair satisfied w_ij * d_ij_inter < d_bar/2
- Each iteration merges exactly the pair with the smallest size-weighted distance and cluster labels/centroids are recomputed after each merge
- Output labels align with adata.obs['scace_cluster'] dimensionality (one label per cell in the AnnData)

## Limitations

- The criterion is adaptive and defined relative to d_bar/2; mis-estimating intra-cluster distances or centroids propagates into incorrect merge decisions
- Merging operates on scAce's VAE-learned embeddings, so results depend on the pre-training and network update steps being run correctly (alternating network update and cluster merging)
- Merging only reduces cluster count — initial labels from Leiden (de novo) or from splitting existing results must be reasonable for merging to improve clustering

## Evidence

- [methods] Merging rule and stopping criterion: [evidence span withheld — anchor status: not_found]
- [abstract] Adaptive merging rationale: "we develop an adaptive cluster merging approach which achieves improved clustering results without the need to estimate the number of clusters in advance"
- [readme] Merging within iterative update loop: "the network parameters, cell embeddings, cluster labels and centroids are iteratively updated by alternately performing network update and cluster merging steps"
- [readme] Input objects for merging analysis: "The output adata contains cluster labels in `adata.obs['scace_cluster']` and the cell embeddings in `adata.obsm['scace_emb']`."
