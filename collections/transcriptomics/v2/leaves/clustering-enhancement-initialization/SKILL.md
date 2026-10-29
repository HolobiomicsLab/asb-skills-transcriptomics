---
name: clustering-enhancement-initialization
description: Use when you already have cluster assignments from another scRNA-seq
  clustering method (e.g. scDeepCluster, DESC, SCCAF, ADClust) on a preprocessed AnnData
  object and want scAce to update and enhance those assignments.
when_to_use_negative:
- No prior clustering results exist and a de novo initialization is wanted — scAce
  then uses Leiden to obtain initial cluster labels instead of the clustering-enhancement
  path
- The number of clusters must be known in advance or the goal is purely to reproduce
  another method's clusters without improvement
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scAce
  role: 'Provides the clustering-enhancement option: applies a cluster splitting approach
    to existing clustering results to obtain initial cluster labels, then enhances
    them via VAE pre-training and adaptive cluster merging (installed via pip install
    scace)'
  repo: https://github.com/sldyns/scAce
- name: scanpy
  role: Builds the AnnData input and runs the preprocessing chain (filter_genes min_cells=3,
    filter_cells min_genes=200, normalize_per_cell, log1p, scale) before enhancement
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:01183cc2d363c2d4b373d2f29a392c8f35611be5d6629050eacc519b1917ae41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Python 3.8
  - scAce
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# clustering-enhancement-initialization

## Summary

Initializes scAce clustering by applying a cluster splitting approach to pre-existing clustering results from other scRNA-seq methods, producing initial cluster labels that scAce's VAE and adaptive merging then enhance.

## When to use

Use when you already have cluster assignments from another scRNA-seq clustering method (e.g. scDeepCluster, DESC, SCCAF, ADClust) on a preprocessed AnnData object and want scAce to update and enhance those assignments.

## When NOT to use

- No prior clustering results exist and a de novo initialization is wanted — scAce then uses Leiden to obtain initial cluster labels instead of the clustering-enhancement path
- The number of clusters must be known in advance or the goal is purely to reproduce another method's clusters without improvement

## Inputs

- Single-cell gene expression data loaded into a scanpy AnnData object (adata = sc.AnnData(data))
- Pre-existing cluster labels from one or more scRNA-seq clustering methods (e.g. scDeepCluster, DESC, SCCAF, ADClust results)

## Outputs

- AnnData with enhanced cluster labels in adata.obs['scace_cluster']
- Cell embeddings in adata.obsm['scace_emb'] for downstream analyses

## How to apply

Preprocess the AnnData object per scAce's recipe: filter_genes(min_cells=3), filter_cells(min_genes=200), store adata.raw, normalize_per_cell, save the scale factor (n_counts / median n_counts) in adata.obs['scale_factor'], then log1p and scale. Take initial cluster labels from a set of existing clustering results (e.g. outputs of scDeepCluster, DESC, SCCAF, or ADClust) and obtain scAce's initialization by applying a cluster splitting approach to those existing labels. Pass these initial labels to run_scace so the pre-trained VAE network parameters, cell embeddings, cluster labels and centroids are iteratively updated via alternating network update and cluster merging steps. Rationale: the ablation (Fig. S9A) shows that even with scAce's initialization and merging, competing methods (avg ARI 0.72 for scDeepCluster, 0.61 for ADClust) trail scAce (avg ARI 0.82), so the enhancement is most effective within scAce's full iterative framework.

## Related tools

- **scAce** (Provides the clustering-enhancement option: applies a cluster splitting approach to existing clustering results to obtain initial cluster labels, then enhances them via VAE pre-training and adaptive cluster merging (installed via pip install scace)) — https://github.com/sldyns/scAce
- **scanpy** (Builds the AnnData input and runs the preprocessing chain (filter_genes min_cells=3, filter_cells min_genes=200, normalize_per_cell, log1p, scale) before enhancement)

## Examples

```
from scace import run_scace; adata = run_scace(adata, init_cluster=existing_labels)  # labels from prior clustering; output in adata.obs['scace_cluster']
```

## Evaluation signals

- Final labels land in adata.obs['scace_cluster'] and embeddings in adata.obsm['scace_emb'] after run_scace completes
- Adjusted Rand index (ARI) of the enhanced labels against reference labels should meet or exceed the original method's ARI on the same dataset
- Averaged over the seven real datasets, the ARI should approach scAce's reported average of 0.82 (vs 0.72 for scDeepCluster and 0.61 for ADClust with the same initialization/merging)
- The scale factor (adata.obs.n_counts / median n_counts) was stored in adata.obs['scale_factor'] before scaling, per the README preprocessing recipe
- The iterative network-update/cluster-merging loop terminates with stable cluster labels and centroids

## Limitations

- Clustering enhancement requires prior clustering results from other methods; without them scAce falls back to de novo Leiden initialization
- The ablation shows initialization and merging alone are insufficient — competing methods given the same components (avg ARI 0.72, 0.61) still trail scAce (0.82), so enhancement should run within scAce's full iterative network-update framework
- Quality depends on the initial cluster splitting of existing results; degraded existing labels may propagate into initialization

## Evidence

- [readme] with clustering enhancement, initial cluster labels are obtained by applying a cluster splitting approach to a set of existing clustering results: "with clustering enhancement, initial cluster labels are obtained by applying a cluster splitting approach to a set of existing clustering results"
- [abstract] scAce provides an option to perform clustering enhancement, which can update and enhance cluster assignments based on previous clustering results from other methods: "scAce provides an option to perform clustering enhancement, which can update and enhance cluster assignments based on previous clustering results from other methods"
- [methods] by alternately performing network update and cluster merging steps: "by alternately performing network update and cluster merging steps"
- [results] Even when given scAce's initialization and merging, scDeepCluster (average ARI = 0.72) and ADClust (average ARI = 0.61) trail scAce (average ARI = 0.82): [evidence span withheld — anchor status: not_found]
