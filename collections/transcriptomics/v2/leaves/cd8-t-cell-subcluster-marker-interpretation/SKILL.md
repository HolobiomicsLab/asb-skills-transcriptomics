---
name: cd8-t-cell-subcluster-marker-interpretation
description: Use when Cytocipher merging of Leiden resolution-2 over-clusters on PBMC
  3K scRNA-seq leaves two T cell sub-clusters unmerged and you must confirm with cluster-specific
  markers (e.g. GZMK, GZMH) that the split is biologically real.
when_to_use_negative:
- Input data is not PBMC 3K scRNA-seq processed with the scanpy standard workflow
  and log-cpm normalisation, or has no Leiden over-clustering to interpret
- The goal is de novo cell-type annotation of clusters rather than validating which
  Cytocipher merges (or non-merges) are biologically justified
- The Leiden resolution is low enough that no over-clustering exists — Cytocipher
  merging (and hence this interpretation step) expects deliberately high resolution
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: cytocipher
  role: Runs marker identification (cc.tl.get_markers), code enrichment scoring (cc.tl.code_enrich),
    cluster merging (cc.tl.merge_clusters), and diagnostic visualisation (cc.pl.enrich_heatmap,
    cc.pl.sig_cluster_diagnostics) used to interpret the CD8+ T cell sub-cluster split
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: Provides the standard workflow (log-cpm normalisation, Leiden clustering at
    resolution 2.0) that prepares the PBMC 3K AnnData prior to Cytocipher interpretation
  repo: https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- name: AnnData
  role: Container format holding the log-cpm normalised expression matrix and 'leiden'/'leiden_merged'
    cluster labels interpreted by this skill
- name: Jupyter
  role: Notebook environment where Cytocipher functions and their documentation (e.g.
    ?cc.tl.get_markers) are run interactively
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:54b0f7b48068c29ba09ecb5c9c9847a4447015d8f46ea1f40479a6232b163ba4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Jupyter
  - cytocipher
  - scanpy
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cd8-t-cell-subcluster-marker-interpretation

## Summary

Interpret Cytocipher cluster-merge results on over-clustered PBMC 3K data by checking that marker genes such as GZMK and GZMH justify keeping the CD8+ T cell sub-clusters split while CD4+ T cell and CD14+ monocyte over-clusters are merged to ground-truth cell types.

## When to use

Use when Cytocipher merging of Leiden resolution-2 over-clusters on PBMC 3K scRNA-seq leaves two T cell sub-clusters unmerged and you must confirm with cluster-specific markers (e.g. GZMK, GZMH) that the split is biologically real.

## When NOT to use

- Input data is not PBMC 3K scRNA-seq processed with the scanpy standard workflow and log-cpm normalisation, or has no Leiden over-clustering to interpret
- The goal is de novo cell-type annotation of clusters rather than validating which Cytocipher merges (or non-merges) are biologically justified
- The Leiden resolution is low enough that no over-clustering exists — Cytocipher merging (and hence this interpretation step) expects deliberately high resolution

## Inputs

- AnnData object with PBMC 3K data processed via the scanpy standard workflow (log-cpm normalised) with Leiden clusters at resolution 2.0 under key 'leiden'
- Marker gene results from cc.tl.get_markers(data, 'leiden')
- Cell-by-cluster code enrichment scores from cc.tl.code_enrich(data, 'leiden')
- 'leiden_merged' cluster labels from cc.tl.merge_clusters(data, 'leiden')

## Outputs

- Interpretation confirming the CD8+ T cell sub-clusters (Clusters 2 and 8) remain split due to GZMK and GZMH cluster-specific markers
- Merged-cluster enrichment score heatmap from cc.pl.enrich_heatmap(data, 'leiden_merged')
- Pair-level significance diagnostics (violin plots of enrichment scores with printed p-values) from cc.pl.sig_cluster_diagnostics(data, 'leiden', plot_pair=...)
- Resolution sweep (0.5–1.5) result showing CD8 subpopulations are not resolvable without over-clustering other cell types

## How to apply

Over-cluster the scanpy-standard-workflow-processed PBMC 3K AnnData (log-cpm normalised) with Leiden at resolution 2.0 to get 19 tentative clusters under the key 'leiden'. Run cc.tl.get_markers(data, 'leiden'), then cc.tl.code_enrich(data, 'leiden') to compute cell-by-cluster code enrichment scores, and cc.tl.merge_clusters(data, 'leiden') with default parameters to produce 'leiden_merged' labels. Interpretation step: expect all over-clusters to merge after one iteration to the ground-truth cell types EXCEPT the two CD8+ T cell sub-clusters (Clusters 2 and 8), whose separation is supported by the cluster-specific markers GZMK and GZMH. Visualise the merged-cluster enrichment heatmap with cc.pl.enrich_heatmap(data, 'leiden_merged') and inspect pair-level significance with cc.pl.sig_cluster_diagnostics(data, 'leiden', plot_pair=...) violin plots, which print p-values for enrichment-score comparisons. Sweep Leiden resolutions 0.5–1.5 to confirm no resolution resolves the CD8 subpopulations without over-clustering other cell types, validating that the CD8 split is robust.

## Related tools

- **cytocipher** (Runs marker identification (cc.tl.get_markers), code enrichment scoring (cc.tl.code_enrich), cluster merging (cc.tl.merge_clusters), and diagnostic visualisation (cc.pl.enrich_heatmap, cc.pl.sig_cluster_diagnostics) used to interpret the CD8+ T cell sub-cluster split) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (Provides the standard workflow (log-cpm normalisation, Leiden clustering at resolution 2.0) that prepares the PBMC 3K AnnData prior to Cytocipher interpretation) — https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- **AnnData** (Container format holding the log-cpm normalised expression matrix and 'leiden'/'leiden_merged' cluster labels interpreted by this skill)
- **Jupyter** (Notebook environment where Cytocipher functions and their documentation (e.g. ?cc.tl.get_markers) are run interactively)

## Examples

```
import cytocipher as cc; cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden'); cc.tl.merge_clusters(data, 'leiden'); cc.pl.enrich_heatmap(data, 'leiden_merged')
```

## Evaluation signals

- After one merge iteration, 'leiden_merged' labels match ground-truth cell types for all over-clusters except the CD8+ T cell sub-clusters (Clusters 2 and 8), which remain split
- Cluster-specific markers GZMK and GZMH are identified for the two CD8+ T cell sub-clusters, justifying the non-merge
- The cc.pl.enrich_heatmap(data, 'leiden_merged') heatmap file is generated and shows distinct enrichment profiles for the retained CD8 sub-clusters
- cc.pl.sig_cluster_diagnostics violin plots print p-values for cluster-pair enrichment-score comparisons consistent with significant vs non-significant pair decisions
- Leiden resolution sweep 0.5–1.5 shows no resolution separates the CD8 subpopulations without over-clustering other cell types

## Limitations

- The CD8+ T cell subpopulations cannot be resolved at any Leiden resolution between 0.5 and 1.5 without over-clustering other cell types, so the split is only detectable via the over-clustered resolution-2.0 run plus merge
- Cytocipher merging relies on deliberately high Leiden resolution ('It's better if the Leiden resolution is high, so that there is alot of over-clustering'); at inappropriate resolutions the interpretation baseline changes
- The manuscript discussion/reproducibility section was unavailable (no discussion text or changelog found), so caveats beyond those stated in methods/README cannot be verified

## Evidence

- [abstract] such as CD8+ T cell subtypes in human peripheral blood mononuclear cells: "such as CD8+ T cell subtypes in human peripheral blood mononuclear cells"
- [readme] Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores.: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
- [methods] An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow](https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html): "An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow](https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html)"
- [readme] tentative cluster labels (e.g. from Leiden clustering). It's better if the Leiden resolution is high: "tentative cluster labels (e.g. from Leiden clustering). It's better if the Leiden resolution is high"
- [readme] To visualise the scores being compared for a given pair of clusters, the following visualises the scores as violin plots of the enrichment scores: "To visualise the scores being compared for a given pair of clusters, the following visualises the scores as violin plots of the enrichment scores"
