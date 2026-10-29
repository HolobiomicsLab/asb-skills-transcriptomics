---
name: cluster-marker-gene-identification
description: Use when an AnnData object has been processed via the scanpy standard
  workflow (log-cpm normalised) with tentative cluster labels (e.g. 'leiden') and
  per-cluster marker genes are needed for downstream code scoring or cluster merging.
when_to_use_negative:
- Input is raw counts that have not been log-cpm normalised via the scanpy standard
  workflow — Cytocipher expects normalised data with tentative cluster labels already
  present
- The goal is final cell-type annotation on a well-resolved, non-over-clustered labelling
  — high-resolution Leiden over-clustering is deliberately preferred as Cytocipher
  input
- Data is not scRNA-seq with cluster labels (e.g. bulk RNA-seq or a pre-computed feature
  table), where per-cell code scoring is not applicable
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: cytocipher
  role: Provides cc.tl.get_markers for per-cluster marker gene identification and
    the downstream code_enrich/merge_clusters steps that consume the marker sets
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: Standard-workflow preprocessing (log-cpm normalisation, Leiden clustering)
    that produces the AnnData input with tentative cluster labels
  repo: https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- name: AnnData
  role: Container object ('data') carrying the log-cpm normalised expression matrix
    and the 'leiden' grouping key passed to get_markers
- name: Jupyter
  role: Notebook environment in which the Cytocipher API is called and documented
    via ?cc.tl.get_markers
provenance:
  source_task_ids:
  - task_006
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:16a94a3523d9b61c0cb8cfc8f5d4e9b53563526b50819ecb9dea18540ee8c6d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Jupyter
  - cytocipher
  - scanpy
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cluster-marker-gene-identification

## Summary

Identifies per-cluster marker gene sets from log-cpm normalised single-cell RNA-seq data with tentative Leiden cluster labels, as the first step of the Cytocipher pipeline for detecting significantly different cell populations.

## When to use

Use when an AnnData object has been processed via the scanpy standard workflow (log-cpm normalised) with tentative cluster labels (e.g. 'leiden') and per-cluster marker genes are needed for downstream code scoring or cluster merging.

## When NOT to use

- Input is raw counts that have not been log-cpm normalised via the scanpy standard workflow — Cytocipher expects normalised data with tentative cluster labels already present
- The goal is final cell-type annotation on a well-resolved, non-over-clustered labelling — high-resolution Leiden over-clustering is deliberately preferred as Cytocipher input
- Data is not scRNA-seq with cluster labels (e.g. bulk RNA-seq or a pre-computed feature table), where per-cell code scoring is not applicable

## Inputs

- AnnData object processed per the scanpy standard workflow (log-cpm normalised)
- Tentative cluster labels in a grouping key such as 'leiden' (high Leiden resolution recommended for over-clustering)

## Outputs

- Per-cluster marker gene sets (data.varm-based marker tables in the AnnData object)
- Input for cc.tl.code_enrich cell-by-cluster enrichment scores

## How to apply

Start from an AnnData object that has been processed similarly to the scanpy standard workflow (PBMC 3K tutorial-style) to produce log-cpm normalised data with tentative cluster labels, ideally from Leiden clustering at high resolution so there is deliberate over-clustering. Run cc.tl.get_markers(data, 'leiden') with default parameters to derive the marker gene set for each cluster; this is the prerequisite for cc.tl.code_enrich, which converts the marker sets into cell-by-cluster enrichment scores. The same marker sets feed the benchmarking of alternative scoring approaches (Cytocipher code scoring, coexpr scoring, Giotto PAGE, Scanpy score_genes) used for cluster-merge significance testing. Verify the call by inspecting the returned marker tables per cluster and by visualising the downstream enrichment heatmap with cc.pl.enrich_heatmap(data, 'leiden').

## Related tools

- **cytocipher** (Provides cc.tl.get_markers for per-cluster marker gene identification and the downstream code_enrich/merge_clusters steps that consume the marker sets) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (Standard-workflow preprocessing (log-cpm normalisation, Leiden clustering) that produces the AnnData input with tentative cluster labels) — https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- **AnnData** (Container object ('data') carrying the log-cpm normalised expression matrix and the 'leiden' grouping key passed to get_markers)
- **Jupyter** (Notebook environment in which the Cytocipher API is called and documented via ?cc.tl.get_markers)

## Examples

```
cc.tl.get_markers(data, 'leiden')
```

## Evaluation signals

- The AnnData object must carry log-cpm normalised values and a non-empty grouping key (e.g. 'leiden') before cc.tl.get_markers is called
- Marker tables are populated for every cluster in the grouping key (e.g. all 19 clusters of PBMC 3K Leiden resolution 2.0)
- Downstream cc.tl.code_enrich produces a cell-by-cluster enrichment score matrix that renders without error in cc.pl.enrich_heatmap(data, 'leiden')
- On the PBMC 3K Fig. 4 task, cluster-specific markers such as GZMK and GZMH emerge for the CD8+ T cell sub-clusters 2 and 8 that remain unmerged
- Marker sets must be derived before code scoring and cluster merging — out-of-order execution (merge_clusters before get_markers) is invalid

## Limitations

- Cluster quality depends on the tentative labels: the README advises high Leiden resolution so there is 'alot of over-clustering', and PBMC 3K Fig. 4 shows CD8+ T cell sub-clusters (2 and 8) are never correctly resolvable by sweeping resolution 0.5–1.5 without over-clustering other cell types
- No changelog is available for the repository, so version-specific behaviour of get_markers cannot be traced from documentation
- Marker identification is only the first stage; its output is meaningful for population detection only when followed by code_enrich and merge_clusters significance testing

## Evidence

- [readme] Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores.: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
- [readme] An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow](https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html): "An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow](https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html)"
- [methods] tentative cluster labels (e.g. from Leiden clustering). It's better if the Leiden resolution is high: "tentative cluster labels (e.g. from Leiden clustering). It's better if the Leiden resolution is high"
- [methods] to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering): "to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering)"
