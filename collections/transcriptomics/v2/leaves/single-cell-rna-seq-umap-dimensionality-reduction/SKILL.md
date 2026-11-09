---
name: single-cell-rna-seq-umap-dimensionality-reduction
description: Use when scRNA-seq data (e.g. log-cpm normalized AnnData from scanpy)
  needs a low-dimensional UMAP embedding for visualizing over-clustered Leiden labels
  and verifying Cytocipher cluster-merge results.
when_to_use_negative:
- Data is not log-cpm normalized or lacks tentative Leiden cluster labels — Cytocipher
  merging and its UMAP visualization require the scanpy-standard-processed AnnData
  described in the README
- The goal is a definitive cell-type annotation rather than visualizing over-clustering
  and merging — Cytocipher's UMAP overlays show significantly different clusters (including
  intermediate states), not curated annotations
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scanpy
  role: Processes the dataset into a log-cpm normalized AnnData object, computes the
    neighborhood graph, and generates the UMAP embedding
  repo: https://github.com/BradBalderson/Cytocipher/
- name: AnnData
  role: Container format ('data') holding the normalized expression matrix, neighborhood
    graph, UMAP coordinates, and Leiden/leiden_merged label keys
- name: cytocipher
  role: Consumes the over-clustered labels and provides cc.tl.merge_clusters producing
    'leiden_merged' labels that are overlaid on the UMAP
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scvelo
  role: Used together with scanpy to embed the cells on UMAP in the pancreas workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:c296a271ec50c1057574bb72b4bfabb67a4cf206d57e5e11c80d60ffcb1f65b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Jupyter
  - cytocipher
  - scanpy
  - scvelo
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# single-cell-rna-seq-umap-dimensionality-reduction

## Summary

Embedding single-cell RNA-seq data into a 2D UMAP space via a scanpy/scvelo neighborhood graph, used in the Cytocipher workflow to visualize over-clustered (Leiden resolution 3.5) cells and the merged significantly different clusters.

## When to use

Use when scRNA-seq data (e.g. log-cpm normalized AnnData from scanpy) needs a low-dimensional UMAP embedding for visualizing over-clustered Leiden labels and verifying Cytocipher cluster-merge results.

## When NOT to use

- Data is not log-cpm normalized or lacks tentative Leiden cluster labels — Cytocipher merging and its UMAP visualization require the scanpy-standard-processed AnnData described in the README
- The goal is a definitive cell-type annotation rather than visualizing over-clustering and merging — Cytocipher's UMAP overlays show significantly different clusters (including intermediate states), not curated annotations

## Inputs

- scRNA-seq count matrix processed into a log-cpm normalized AnnData object (scanpy standard workflow)
- Leiden cluster labels stored under an AnnData .obs key (e.g. 'leiden' at resolution 3.5)
- Neighborhood graph computed on the AnnData (scanpy/scvelo)

## Outputs

- 2D UMAP embedding of cells stored in the AnnData .obsm slot
- UMAP scatter plot colored by over-clustered 'leiden' labels and by Cytocipher 'leiden_merged' labels (e.g. pancreas_merged_umap.png)

## How to apply

Process the raw count matrix with the scanpy standard workflow to produce a log-cpm normalized AnnData object. Compute the neighborhood graph on this AnnData and embed the cells with scanpy/scvelo UMAP. Over-cluster the cells with Leiden at a deliberately high resolution (e.g. 3.5 in the E15.5 pancreas example), storing labels under the 'leiden' key, since Cytocipher works best when there is substantial over-clustering to merge. After running cc.tl.get_markers, cc.tl.code_enrich, and cc.tl.merge_clusters, project the 'leiden_merged' labels back onto the UMAP and save the visualization (e.g. pancreas_merged_umap.png) to confirm that the merged partition (16 significantly different clusters in the pancreas case) separates intermediate cell states such as the Anxa2-to-Btbd17 endocrine progenitor state.

## Related tools

- **scanpy** (Processes the dataset into a log-cpm normalized AnnData object, computes the neighborhood graph, and generates the UMAP embedding) — https://github.com/BradBalderson/Cytocipher/
- **AnnData** (Container format ('data') holding the normalized expression matrix, neighborhood graph, UMAP coordinates, and Leiden/leiden_merged label keys)
- **cytocipher** (Consumes the over-clustered labels and provides cc.tl.merge_clusters producing 'leiden_merged' labels that are overlaid on the UMAP) — https://github.com/BradBalderson/Cytocipher/
- **scvelo** (Used together with scanpy to embed the cells on UMAP in the pancreas workflow)

## Examples

```
import scanpy as sc; sc.pp.neighbors(data); sc.tl.umap(data); sc.pl.umap(data, color='leiden_merged', save='_merged.png')
```

## Evaluation signals

- The AnnData contains UMAP coordinates in .obsm and a computed neighborhood graph before clustering/merging
- The over-clustered 'leiden' key exists at the deliberately high resolution (e.g. 3.5) as advised ('It's better if the Leiden resolution is high, so that there is alot of over-clustering')
- Overlaying 'leiden_merged' on the UMAP yields the expected merged partition (16 significantly different clusters in the E15.5 pancreas reproduction)
- The saved UMAP figure (e.g. pancreas_merged_umap.png) visually separates intermediate states such as Cluster 6 (Anxa2-to-Btbd17 endocrine progenitor), with lineage-path clusters expressing Isl1, Pax4, and Hhex

## Limitations

- Cytocipher is designed to operate on over-clustered data; a low Leiden resolution gives few clusters to merge and undermines the significant-cluster detection
- The README only demonstrates the workflow via the scanpy PBMC3k-style standard workflow and the pancreas tutorial; no performance guarantees are given for other normalization schemes

## Evidence

- [methods] Compute the neighborhood graph and embed the cells with Scvelo/scanpy UMAP.: [evidence span withheld — anchor status: not_found]
- [methods] Over-cluster the cells with Leiden at resolution 3.5, storing labels under the 'leiden' key.: [evidence span withheld — anchor status: not_found]
- [readme] An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow]: "An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow]"
- [readme] It's better if the Leiden resolution is high, so that there is alot of over-clustering.: "It's better if the Leiden resolution is high, so that there is alot of over-clustering."
- [readme] to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering): "to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering)"
