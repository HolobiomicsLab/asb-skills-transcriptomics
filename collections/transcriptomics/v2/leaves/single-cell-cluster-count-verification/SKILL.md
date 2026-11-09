---
name: single-cell-cluster-count-verification
description: Use when scRNA-seq data has been deliberately over-clustered with Leiden
  at high resolution (e.g. 3.5) and you need to verify how many significantly different
  clusters remain after Cytocipher merging and whether intermediate cell states were
  recovered.
when_to_use_negative:
- Input AnnData lacks tentative cluster labels or has been clustered at low Leiden
  resolution — Cytocipher expects high-resolution over-clustering so that non-significant
  clusters can be merged
- Data is not log-cpm normalised via the scanpy standard workflow (e.g. raw counts
  or a feature table)
- The goal is subpopulation discovery in a different biological context (e.g. CD8+
  T cell subtypes in PBMC or prostate luminal epithelial subpopulations) rather than
  verification of cluster counts and intermediate states
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: cytocipher
  role: Provides cc.tl.get_markers, cc.tl.code_enrich, and cc.tl.merge_clusters to
    statistically determine significantly different clusters and produce merged labels
  repo: https://github.com/BradBalderson/Cytocipher
- name: scanpy
  role: Standard workflow used to log-cpm normalise the data, compute the neighborhood
    graph, and run Leiden over-clustering before merging
- name: AnnData
  role: Container format holding the normalised expression matrix, UMAP embedding,
    and 'leiden'/'leiden_merged' cluster labels
- name: Jupyter
  role: Notebook environment for running the tutorial workflow and inspecting function
    documentation (e.g. ?cc.tl.get_markers)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:0da84453094688f582b9c0047c8addce6af67b8059bf39f856f40e27ae800fc5
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# single-cell-cluster-count-verification

## Summary

Verifies the number of significantly different cell clusters produced by Cytocipher's cluster-merge on over-clustered scRNA-seq data, confirming that non-significant Leiden clusters are merged and that biologically meaningful intermediate cell states are retained.

## When to use

Use when scRNA-seq data has been deliberately over-clustered with Leiden at high resolution (e.g. 3.5) and you need to verify how many significantly different clusters remain after Cytocipher merging and whether intermediate cell states were recovered.

## When NOT to use

- Input AnnData lacks tentative cluster labels or has been clustered at low Leiden resolution — Cytocipher expects high-resolution over-clustering so that non-significant clusters can be merged
- Data is not log-cpm normalised via the scanpy standard workflow (e.g. raw counts or a feature table)
- The goal is subpopulation discovery in a different biological context (e.g. CD8+ T cell subtypes in PBMC or prostate luminal epithelial subpopulations) rather than verification of cluster counts and intermediate states

## Inputs

- AnnData object log-cpm normalised per the scanpy standard workflow, with tentative Leiden cluster labels stored under a 'leiden' key at high over-clustering resolution
- Cytocipher pancreas tutorial scRNA-seq dataset (E15.5 mouse pancreas development)

## Outputs

- 'leiden_merged' cluster labels in the AnnData object (merged significantly-different clusters)
- Cell-by-cluster enrichment score matrix (from cc.tl.code_enrich)
- Marker genes per cluster (from cc.tl.get_markers)
- Verified cluster count (e.g. 16 merged clusters) and identified intermediate cell states, e.g. UMAP visualisation such as pancreas_merged_umap.png

## How to apply

Process the AnnData object with the scanpy standard workflow to produce log-cpm normalised data, compute a neighborhood graph, and over-cluster with Leiden at a deliberately high resolution (e.g. 3.5 for the E15.5 mouse pancreas data) so there is substantial over-clustering. Run cc.tl.get_markers(data, 'leiden') for marker gene identification, then cc.tl.code_enrich(data, 'leiden') to compute cell-by-cluster enrichment (code) scores. Run cc.tl.merge_clusters(data, 'leiden') with default parameters at an adjusted P-value cutoff of 0.045 to produce 'leiden_merged' labels; Cytocipher merges clusters that are not significantly different. Verify the merged partition's cluster count against the expected result (16 distinctly different clusters for the E15.5 pancreas tutorial), and check that intermediate states such as the Anxa2-to-Btbd17 endocrine progenitor state and lineage-path clusters expressing Isl1, Pax4, and Hhex are present. Use cc.pl.sig_cluster_diagnostics to inspect the enrichment-score violins and p-values for specific cluster pairs to confirm the significance decisions.

## Related tools

- **cytocipher** (Provides cc.tl.get_markers, cc.tl.code_enrich, and cc.tl.merge_clusters to statistically determine significantly different clusters and produce merged labels) — https://github.com/BradBalderson/Cytocipher
- **scanpy** (Standard workflow used to log-cpm normalise the data, compute the neighborhood graph, and run Leiden over-clustering before merging)
- **AnnData** (Container format holding the normalised expression matrix, UMAP embedding, and 'leiden'/'leiden_merged' cluster labels)
- **Jupyter** (Notebook environment for running the tutorial workflow and inspecting function documentation (e.g. ?cc.tl.get_markers))

## Examples

```
cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden'); cc.tl.merge_clusters(data, 'leiden')
```

## Evaluation signals

- Merged partition ('leiden_merged') contains the expected number of significantly different clusters — 16 for the E15.5 pancreas data at adjusted P-value cutoff 0.045
- Cluster 6 in the merged partition corresponds to the Anxa2-to-Btbd17 endocrine progenitor intermediate state
- Lineage-path clusters in the merged partition express Isl1, Pax4, and Hhex
- cc.pl.sig_cluster_diagnostics violins and printed p-values confirm merged cluster pairs are non-significant while retained pairs are significant
- Merged labels visualised on the scanpy/scvelo UMAP match the new merged cluster assignments (heatmap of enrichment scores per merged cluster)

## Limitations

- Leiden resolution must be deliberately high (over-clustering) for Cytocipher to work as intended; clustering at default resolution gives no non-significant clusters to merge
- Cytocipher was released as a manuscript version on Zenodo (10.5281/zenodo.8089546) with no changelog in the repository, so reproducibility gaps and limitations could not be fully documented
- Significance p-values depend on the number of cells and scores per cluster (small clusters, e.g. 3 cells; 3 scores, yield weakly powered comparisons)

## Evidence

- [readme] Cytocipher merges the non-significantly different clusters.: "It's better if the Leiden resolution is high,
 so that there is alot of over-clustering. 
 Cytocipher merges the non-significantly different clusters."
- [readme] cc.tl.merge_clusters(data, 'leiden'): "Below runs the cluster merging and visualises the heatmap of enrichment 
scores per cell for each of the new merged clusters."
- [methods] Functions run marker gene identification, code scoring, and visualisation of cell by cluster enrichment scores.: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
- [abstract] cell lineage intermediate states during mouse pancreas development: "cell lineage intermediate states during mouse pancreas development"
- [methods] log-cpm normalised data with tentative Leiden cluster labels as expected input: "An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow]"
