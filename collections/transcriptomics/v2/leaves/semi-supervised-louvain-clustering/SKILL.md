---
name: semi-supervised-louvain-clustering
description: Use when clustering a perturbation scRNA-seq AnnData where control cells
  already have trusted cluster labels (e.g. from Louvain on controls) and treated
  cells must be assigned without letting them distort the control-defined states.
when_to_use_negative:
- No trusted control labels exist to anchor the semi-supervised step — plain unsupervised
  Louvain on the full dataset is the appropriate choice
- The goal is batch integration or imputation of perturbation effects rather than
  resolving cell states across treatments (ICAT is explicitly a clustering tool, not
  an integration workflow)
- Input is not a gene-by-cell count/expression matrix in AnnData format (e.g. already-clustered
  feature tables or bulk RNA-seq)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ICAT
  role: Provides the models.icat class implementing semi-supervised Louvain clustering
    with control labels fixed
  repo: https://github.com/BradhamLab/icat
- name: icat-sc
  role: pip-installable package supplying icat.models, icat.optimize, and icat.simulate
    used to run the semi-supervised clustering
  repo: https://github.com/BradhamLab/icat
- name: scanpy
  role: Provides AnnData handling, log1p normalization, neighbors, Louvain, and UMAP
    used around the semi-supervised step
- name: AnnData
  role: Data structure carrying obs['treatment'], obs['sslouvain'], and obsm['X_icat']
    through the clustering workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:19c1b5c58ae22872b9856c5b5fb8c1c24a6b159d99f309c76311589cea16b64d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - AnnData
  - ICAT
  - icat-sc
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# semi-supervised-louvain-clustering

## Summary

ICAT's final clustering step: after NCFS gene weighting transforms the data, cells are clustered with semi-supervised Louvain so control-defined labels stay fixed while perturbed cells (e.g. MK-886 or chlorate-treated sea urchin PMCs) are free to join any cluster, resolving treatment-affected states.

## When to use

Use when clustering a perturbation scRNA-seq AnnData where control cells already have trusted cluster labels (e.g. from Louvain on controls) and treated cells must be assigned without letting them distort the control-defined states.

## When NOT to use

- No trusted control labels exist to anchor the semi-supervised step — plain unsupervised Louvain on the full dataset is the appropriate choice
- The goal is batch integration or imputation of perturbation effects rather than resolving cell states across treatments (ICAT is explicitly a clustering tool, not an integration workflow)
- Input is not a gene-by-cell count/expression matrix in AnnData format (e.g. already-clustered feature tables or bulk RNA-seq)

## Inputs

- log1p-normalized AnnData combining control and perturbed cells (e.g. SMART-Seq2 counts for GSE164240, 435 PMC cells) with a treatment label column
- control cell cluster labels (e.g. 'louvain' or 'control_clusters' from control-only Louvain clustering)
- NCFS sigma and reg parameters (optimized via icat.optimize.optimize_ncfs, e.g. sigma in [0.5–3], reg in [0.25–3], 5-NN MCC over 3-fold CV)

## Outputs

- AnnData with final cell-state assignments in obs['sslouvain']
- Neighbors computed in the NCFS-weighted space (adata.obsm['X_icat']) enabling UMAP visualization
- Cluster assignments for perturbed cells that can be tested for treatment enrichment (G-test / Fisher exact, Fig 5A-style table)

## How to apply

Fit an icat models.icat model (e.g. ncfs_kws reg=1, sigma=3; neighbor_kws n_neighbors=15; cluster_kws resolution=0.75) on a log1p-normalized AnnData combining control and perturbed cells annotated with a 'treatment' column. Run model.cluster(adata, adata.obs['treatment']) with ctrl_value='control', so control labels remain constant and experimental cells join any cluster regardless of treatment; collect final labels from the 'sslouvain' column of the returned AnnData. By default ICAT reuses the n_neighbors and resolution optimized for the control Louvain step (via icat.optimize.optimize_louvain, e.g. Calinski-Harabasz over n_neighbors 3–50, resolution 0.3–1.2); optionally optimize sslouvain parameters separately with icat.optimize.optimize_sslouvain over the same grids with max_cells=750. Because ICAT defines neighbors in the NCFS-weighted space, adata.obsm['X_icat'] already has neighbors computed, so UMAP can be drawn directly with sc.tl.umap for visual verification. For the GSE164240 PMC data this workflow reproduces five clusters, with cluster 3 (sm50+/pks2−) and cluster 4 (sm50−/pks2+) capturing perturbation-specific phenotypes.

## Related tools

- **ICAT** (Provides the models.icat class implementing semi-supervised Louvain clustering with control labels fixed) — https://github.com/BradhamLab/icat
- **icat-sc** (pip-installable package supplying icat.models, icat.optimize, and icat.simulate used to run the semi-supervised clustering) — https://github.com/BradhamLab/icat
- **scanpy** (Provides AnnData handling, log1p normalization, neighbors, Louvain, and UMAP used around the semi-supervised step)
- **AnnData** (Data structure carrying obs['treatment'], obs['sslouvain'], and obsm['X_icat'] through the clustering workflow)

## Examples

```
model = models.icat(ctrl_value='control', ncfs_kws={'reg': 1, 'sigma': 3}, neighbor_kws={'n_neighbors': 15}, cluster_kws={'resolution': 0.75}); out = model.cluster(adata, adata.obs['treatment']); print(out.obs['sslouvain'].unique())
```

## Evaluation signals

- The returned AnnData contains an obs['sslouvain'] column and out.obs['sslouvain'].unique() lists the expected number of states (exactly five clusters for the 435-cell PMC dataset)
- Control cells retain their originally identified Louvain labels — the semi-supervised constraint must not change control assignments
- Cluster phenotype check: cluster 3 is sm50+/pks2− and cluster 4 is sm50−/pks2+ (inverse marker expression)
- Treatment enrichment: G-test / Fisher exact per cluster reproduces Fig 5A associations, with cluster 3 and cluster 4 each enriched in a different perturbation (MK-886 vs chlorate)
- Neighbors exist in the weighted space such that sc.tl.umap(out) runs directly on obsm['X_icat'] without recomputing neighbors

## Limitations

- Optimization functions and clustering require Linux installation via pip; the ICAT_manuscript pipeline 'is not guaranteed to run on your system out-of-the-box' (run via snakemake in the icat conda environment)
- By default the same n_neighbors and resolution are reused for semi-supervised clustering as for control clustering; separate optimization (optimize_sslouvain) is optional and was capped at max_cells=750 in the README example
- ICAT does not automatically compute UMAP/tSNE; reduced-dimension visualization must be done afterwards on the X_icat matrix

## Evidence

- [readme] perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster with any other cells regardless of treatment status.: "perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster with any other cells regardless of"
- [readme] The final clustering is stored in the `sslouvain` column of the returned `AnnData` object.: "The 
final clustering is stored in the `sslouvain` column of the returned `AnnData`
object."
- [readme] By default, ICAT uses the same `n_neighbors` and `resolution` parameters during semi-supervised clustering as it does during control clustering.: "By default, ICAT uses the same `n_neighbors` and `resolution` parameters during
semi-supervised clustering as it does during control clustering."
- [methods] out = model.cluster(adata, adata.obs['treatment'])
    print(out.obs['sslouvain'].unique()): "out = model.cluster(adata, adata.obs['treatment'])
    print(out.obs['sslouvain'].unique())"
- [readme] it is possible to pass the upweighted count matrix (found in `adata.obsm["X_icat"]`) to these algorithms.: "it is possible to pass the upweighted count matrix
(found in `adata.obsm["X_icat"]`) to these algorithms."
