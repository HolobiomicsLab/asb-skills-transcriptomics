---
name: leiden-high-resolution-over-clustering
description: Use when scRNA-seq AnnData is log-cpm normalized with a neighborhood
  graph and you need tentative fine-grained Leiden labels that over-cluster cells,
  as the required starting point for Cytocipher cluster-merge testing.
when_to_use_negative:
- Data is not log-cpm normalised AnnData or lacks the scanpy-standard preprocessing
  assumed by Cytocipher
- You already have biologically validated cluster annotations and do not need statistical
  testing of cluster differences
- The goal is a final partition at natural resolution rather than an over-clustered
  input for merge testing
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scanpy
  role: scanpy standard workflow to produce log-cpm normalised data, neighborhood
    graph and UMAP before Leiden over-clustering
  repo: https://github.com/BradBalderson/Cytocipher/
- name: AnnData
  role: container holding the log-cpm normalised data and the 'leiden' cluster labels
    consumed by Cytocipher
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scvelo
  role: used with scanpy UMAP to embed cells for visualising the over-clustered labels
- name: cytocipher
  role: downstream consumer of the over-clustered 'leiden' labels; merges non-significantly
    different clusters via cc.tl.merge_clusters
  repo: https://github.com/BradBalderson/Cytocipher
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:b9daf72797cf383121c6c9fe5bdb45146b3f65d768e7544d7b495ffc26a25e7a
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# leiden-high-resolution-over-clustering

## Summary

Deliberately over-cluster log-cpm normalized single-cell RNA-seq data with Leiden at high resolution (e.g. resolution 3.5 in the E15.5 pancreas example) so that Cytocipher can statistically merge non-significantly different clusters and expose intermediate cell states.

## When to use

Use when scRNA-seq AnnData is log-cpm normalized with a neighborhood graph and you need tentative fine-grained Leiden labels that over-cluster cells, as the required starting point for Cytocipher cluster-merge testing.

## When NOT to use

- Data is not log-cpm normalised AnnData or lacks the scanpy-standard preprocessing assumed by Cytocipher
- You already have biologically validated cluster annotations and do not need statistical testing of cluster differences
- The goal is a final partition at natural resolution rather than an over-clustered input for merge testing

## Inputs

- AnnData object processed with the scanpy standard workflow (log-cpm normalised expression)
- Neighborhood graph computed with scanpy (scvelo/scanpy UMAP embedding for visualisation)

## Outputs

- Tentative over-clustered cell labels stored in AnnData under the 'leiden' key (e.g. Leiden resolution 3.5)
- UMAP embedding colored by over-clustered labels as input for subsequent cc.tl.merge_clusters

## How to apply

First process the data with the scanpy standard workflow to produce log-cpm normalised data, compute the neighborhood graph, and embed cells with UMAP. Then run Leiden clustering at a high resolution — the Cytocipher pancreas tutorial over-clusters E15.5 pancreas cells at Leiden resolution 3.5, storing labels under the 'leiden' key in the AnnData object. The rationale is deliberate over-clustering: the README states 'It's better if the Leiden resolution is high, so that there is alot of over-clustering', because Cytocipher's merge step tests enrichment-code score differences (adjusted P-value cutoff 0.045 in the pancreas example) and merges clusters that are not significantly different. Verify by visualising labels on UMAP and counting the resulting clusters — the E15.5 pancreas run yields 16 distinctly different merged clusters including the Anxa2-to-Btbd17 endocrine progenitor intermediate state.

## Related tools

- **scanpy** (scanpy standard workflow to produce log-cpm normalised data, neighborhood graph and UMAP before Leiden over-clustering) — https://github.com/BradBalderson/Cytocipher/
- **AnnData** (container holding the log-cpm normalised data and the 'leiden' cluster labels consumed by Cytocipher) — https://github.com/BradBalderson/Cytocipher/
- **scvelo** (used with scanpy UMAP to embed cells for visualising the over-clustered labels)
- **cytocipher** (downstream consumer of the over-clustered 'leiden' labels; merges non-significantly different clusters via cc.tl.merge_clusters) — https://github.com/BradBalderson/Cytocipher

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('e155_pancreas.h5ad'); sc.pp.normalize_total(adata, target_sum=1e4); sc.pp.log1p(adata); sc.pp.neighbors(adata); sc.tl.umap(adata); sc.tl.leiden(adata, resolution=3.5, key_added='leiden')
```

## Evaluation signals

- Labels are stored in AnnData under the 'leiden' key, matching the key string passed to cc.tl.get_markers / cc.tl.code_enrich / cc.tl.merge_clusters
- Leiden resolution is high enough that the partition is over-clustered relative to expected biology (resolution 3.5 in the E15.5 pancreas tutorial)
- Merged partition from cc.tl.merge_clusters contains 16 distinctly different clusters for the E15.5 pancreas data at adjusted P-value cutoff 0.045
- Merged clusters recover intermediate cell states absent from original annotations, e.g. Cluster 6 as the Anxa2-to-Btbd17 endocrine progenitor intermediate state with lineage-path expression of Isl1, Pax4, and Hhex
- Over-clustered labels render coherently on the scanpy/scvelo UMAP embedding

## Limitations

- High-resolution Leiden labels are deliberately tentative and biologically over-split; they are only meaningful as input to a statistical merge step, not as final annotations
- Cytocipher merging requires enrichment-code scores from marker genes (cc.tl.get_markers, cc.tl.code_enrich) computed on the over-clustered labels before significance testing
- Significance of cluster differences depends on the adjusted P-value cutoff (0.045 in the pancreas example) and diagnostic violins should be checked via cc.pl.sig_cluster_diagnostics

## Evidence

- [readme] log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering). It's better if the Leiden resolution is high: [evidence span withheld — anchor status: not_found]
- [readme] Cytocipher merges the non-significantly different clusters: "so that there is alot of over-clustering. Cytocipher merges the non-significantly different clusters."
- [other] Over-cluster the cells with Leiden at resolution 3.5, storing labels under the 'leiden' key: [evidence span withheld — anchor status: not_found]
- [results] 16 distinctly different clusters, including several intermediate cell states not identified with the original annotations: [evidence span withheld — anchor status: not_found]
- [methods] tentative cluster labels (e.g. from Leiden clustering): "tentative cluster labels (e.g. from Leiden clustering). It's better if the Leiden resolution is high"
