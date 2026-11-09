---
name: pancreas-developmental-lineage-marker-interpretation
description: Use when scRNA-seq data are intentionally over-clustered (e.g. Leiden
  resolution 3.5) and the goal is to merge non-significantly different clusters and
  detect intermediate cell states along a developmental lineage from cell-by-cluster
  enrichment scores.
when_to_use_negative:
- The AnnData has not been log-cpm normalized with tentative cluster labels from the
  scanpy standard workflow — Cytocipher expects this specific processed input.
- Clusters are already well-resolved cell types rather than an intentionally over-clustered
  partition; the README states 'It's better if the Leiden resolution is high, so that
  there is alot of over-clustering' for merging to be meaningful.
- The analysis goal is de novo cell-type discovery without a significance test between
  clusters; Cytocipher specifically merges non-significantly different clusters rather
  than defining populations from scratch.
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: cytocipher
  role: Provides the cluster-merge machinery (cc.tl.get_markers, cc.tl.code_enrich,
    cc.tl.merge_clusters) and visualization (cc.pl.enrich_heatmap, cc.pl.sig_cluster_diagnostics)
    used to identify significantly different clusters and intermediate states.
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: Processes the E15.5 pancreas data into a log-cpm normalized AnnData object
    with neighborhood graph and UMAP, and supplies the over-clustered 'leiden' labels
    at resolution 3.5.
  repo: https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- name: AnnData
  role: The data container holding normalized expression, 'leiden' and 'leiden_merged'
    labels, and enrichment scores throughout the workflow.
- name: scvelo
  role: Used with scanpy to embed the cells with UMAP prior to over-clustering.
- name: Jupyter
  role: Notebook environment for running the tutorial workflow and inspecting function
    documentation (e.g. ?cc.tl.get_markers).
  repo: https://github.com/BradBalderson/Cytocipher/tree/main/tutorials/cytocipher_pancreas.ipynb
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:5cfe15ef915339f2f81002cd78cc18479b44943be3379b41b702be7ac95afd58
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# pancreas-developmental-lineage-marker-interpretation

## Summary

Interprets Cytocipher cluster-merge results on the over-clustered mouse E15.5 pancreas scRNA-seq data (Leiden resolution 3.5, adjusted P-value cutoff 0.045) to identify significantly different clusters and annotate developmental lineage intermediate cell states via marker genes such as Isl1, Pax4, and Hhex.

## When to use

Use when scRNA-seq data are intentionally over-clustered (e.g. Leiden resolution 3.5) and the goal is to merge non-significantly different clusters and detect intermediate cell states along a developmental lineage from cell-by-cluster enrichment scores.

## When NOT to use

- The AnnData has not been log-cpm normalized with tentative cluster labels from the scanpy standard workflow — Cytocipher expects this specific processed input.
- Clusters are already well-resolved cell types rather than an intentionally over-clustered partition; the README states 'It's better if the Leiden resolution is high, so that there is alot of over-clustering' for merging to be meaningful.
- The analysis goal is de novo cell-type discovery without a significance test between clusters; Cytocipher specifically merges non-significantly different clusters rather than defining populations from scratch.

## Inputs

- E15.5 mouse pancreas scRNA-seq dataset from the Cytocipher pancreas tutorial repository
- log-cpm normalized AnnData object processed via the scanpy standard workflow
- neighborhood graph and UMAP embedding (scanpy/scvelo)
- 'leiden' cluster labels over-clustered at Leiden resolution 3.5
- marker genes and cell-by-cluster enrichment scores from cc.tl.get_markers and cc.tl.code_enrich

## Outputs

- 'leiden_merged' cluster labels containing 16 significantly different clusters
- UMAP visualization of merged clusters saved as pancreas_merged_umap.png
- heatmap of enrichment scores per cell for each merged cluster (cc.pl.enrich_heatmap)
- annotated intermediate cell states, including the Anxa2-to-Btbd17 endocrine progenitor state (Cluster 6)
- lineage-path marker assignments (Isl1, Pax4, Hhex)

## How to apply

Load the E15.5 pancreas dataset from the Cytocipher pancreas tutorial repository and process it with scanpy into a log-cpm normalized AnnData object, then compute the neighborhood graph and embed cells with a UMAP. Over-cluster at Leiden resolution 3.5 under the 'leiden' key — high resolution is deliberately preferred so there is ample over-clustering for Cytocipher to merge. Run cc.tl.get_markers(data, 'leiden') followed by cc.tl.code_enrich(data, 'leiden') to obtain marker genes and cell-by-cluster enrichment scores, then run cc.tl.merge_clusters(data, 'leiden') with default parameters at adjusted P-value cutoff 0.045 to produce 'leiden_merged' labels. Visualize with cc.pl.enrich_heatmap(data, 'leiden_merged') and the merged clusters on UMAP, and use cc.pl.sig_cluster_diagnostics(data, 'leiden', plot_pair=('3','9')) to inspect violin plots of enrichment scores and p-values for specific cluster pairs. Judge success by verifying the merged partition contains 16 distinctly different clusters, that Cluster 6 corresponds to the Anxa2-to-Btbd17 endocrine progenitor intermediate state, and that lineage-path clusters express Isl1, Pax4, and Hhex.

## Related tools

- **cytocipher** (Provides the cluster-merge machinery (cc.tl.get_markers, cc.tl.code_enrich, cc.tl.merge_clusters) and visualization (cc.pl.enrich_heatmap, cc.pl.sig_cluster_diagnostics) used to identify significantly different clusters and intermediate states.) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (Processes the E15.5 pancreas data into a log-cpm normalized AnnData object with neighborhood graph and UMAP, and supplies the over-clustered 'leiden' labels at resolution 3.5.) — https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- **AnnData** (The data container holding normalized expression, 'leiden' and 'leiden_merged' labels, and enrichment scores throughout the workflow.)
- **scvelo** (Used with scanpy to embed the cells with UMAP prior to over-clustering.)
- **Jupyter** (Notebook environment for running the tutorial workflow and inspecting function documentation (e.g. ?cc.tl.get_markers).) — https://github.com/BradBalderson/Cytocipher/tree/main/tutorials/cytocipher_pancreas.ipynb

## Examples

```
import cytocipher as cc; cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden'); cc.tl.merge_clusters(data, 'leiden'); cc.pl.enrich_heatmap(data, 'leiden_merged')
```

## Evaluation signals

- The merged 'leiden_merged' partition contains exactly 16 distinctly different clusters.
- Cluster 6 corresponds to the Anxa2-to-Btbd17 endocrine progenitor intermediate state on the UMAP.
- Lineage-path clusters express Isl1, Pax4, and Hhex.
- cc.pl.enrich_heatmap(data, 'leiden_merged') renders the heatmap of enrichment scores per cell for the new merged clusters.
- cc.pl.sig_cluster_diagnostics prints p-values for the chosen cluster pair (e.g. p≈0.913 and p≈0.813 for pair ('3','9')) and violin plots of enrichment scores against the upper/lower significance bounds.

## Limitations

- The significance call depends on the adjusted P-value cutoff (0.045 in the pancreas workflow); changing this cutoff changes which clusters merge and the resulting count of significant clusters.
- Merging quality depends on deliberately over-clustered input — the README notes it is 'better if the Leiden resolution is high, so that there is alot of over-clustering'; under-clustered input undermines the approach.
- Small cluster pairs give unstable diagnostics (e.g. 3 cells vs 3 cells in the README example), so p-values for very small clusters should be checked with sig_cluster_diagnostics before trusting merges.
- No discussion-section content or repository changelog was available, so limitations and reproducibility gaps documented by the authors are not fully extractable.

## Evidence

- [readme] Cytocipher merges the non-significantly different clusters.: "Cytocipher merges the non-significantly different clusters."
- [readme] It's better if the Leiden resolution is high, so that there is alot of over-clustering.: "It's better if the Leiden resolution is high,
 so that there is alot of over-clustering."
- [readme] Below runs the cluster merging and visualises the heatmap of enrichment scores per cell for each of the new merged clusters.: "Below runs the cluster merging and visualises the heatmap of enrichment 
scores per cell for each of the new merged clusters."
- [methods] Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores.: "Functions below run the marker gene identification, code scoring, & 
subsequent visualisation of the resulting cell by cluster enrichment scores."
- [results] resulted in 16 distinctly different clusters, including several intermediate cell states not identified with the original annotations.: [evidence span withheld — anchor status: not_found]
- [results] Cluster 6 corresponds to the Anxa2-to-Btbd17 endocrine progenitor intermediate state, and that lineage-path clusters express Isl1, Pax4, and Hhex.: [evidence span withheld — anchor status: not_found]
- [abstract] cell lineage intermediate states during mouse pancreas development: "cell lineage intermediate states during mouse pancreas development"
