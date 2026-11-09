---
name: enrichment-score-significance-testing
description: Use when an AnnData object with log-cpm normalized counts and high-resolution
  (over-clustered) Leiden labels needs statistical determination of which clusters
  represent genuinely distinct cell populations versus clusters to be merged.
when_to_use_negative:
- The input is raw counts without log-cpm normalization or lacks tentative cluster
  labels — Cytocipher expects scanpy-standard-workflow output
- Clusters are already known to be biologically distinct and no significance-based
  merging is needed
- Only a single cluster (no cluster pairs to compare) is present
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cytocipher
  role: Provides cc.tl.get_markers, cc.tl.code_enrich, cc.tl.merge_clusters, and cc.pl.sig_cluster_diagnostics
    for marker identification, code/coexpr scoring, significance testing, and cluster
    merging
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: Standard workflow used upstream to produce the log-cpm normalized AnnData
    input with tentative cluster labels
- name: AnnData
  role: Container format holding normalized counts, cluster labels, and the cell-by-cluster
    enrichment score matrix
- name: Jupyter
  role: Environment in which function documentation (e.g. ?cc.tl.get_markers) and
    diagnostics plots are inspected
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:a61d147d791ff0bdeb483e5ee25c1dc12519f830529d1e116b6ed62081c7c85b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Cytocipher
  - Jupyter
  - cytocipher
  - scanpy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# enrichment-score-significance-testing

## Summary

Statistical testing of per-cell enrichment scores to determine which clusters in an over-clustered scRNA-seq dataset are significantly different, as implemented in Cytocipher's code-scoring/coexpr-scoring and cluster-merging pipeline.

## When to use

Use when an AnnData object with log-cpm normalized counts and high-resolution (over-clustered) Leiden labels needs statistical determination of which clusters represent genuinely distinct cell populations versus clusters to be merged.

## When NOT to use

- The input is raw counts without log-cpm normalization or lacks tentative cluster labels — Cytocipher expects scanpy-standard-workflow output
- Clusters are already known to be biologically distinct and no significance-based merging is needed
- Only a single cluster (no cluster pairs to compare) is present

## Inputs

- AnnData object with log-cpm normalized scRNA-seq counts and tentative high-resolution Leiden cluster labels (e.g. column 'leiden')
- Cluster label column name to test (e.g. 'leiden')

## Outputs

- Per-cluster marker and negative gene sets from cc.tl.get_markers
- Cell-by-cluster enrichment score matrix stored in the AnnData object (exportable as enrichment_scores.csv)
- Merged cluster labels (e.g. 'leiden_merged') after cc.tl.merge_clusters
- Enrichment heatmaps and violin plots of enrichment-score distributions with printed pairwise p-values

## How to apply

Start from an AnnData object preprocessed per the scanpy standard workflow (log-cpm normalized, tentative Leiden labels at high resolution so there is substantial over-clustering). Run cc.tl.get_markers(data, 'leiden') to derive per-cluster marker and negative gene sets with winsorization bounding expression values, then cc.tl.code_enrich(data, 'leiden') which applies the code-score (Eq. 3, log of the inverse joint probability of expressing each marker gene at >= the observed level per cell) and coexpr-score (Eq. 7) to assign each cell an enrichment score against every cluster. Assess significance by comparing enrichment-score distributions between cluster pairs with cc.pl.sig_cluster_diagnostics(data, 'leiden', plot_pair=('3','9')), which prints the p-values for both directional comparisons; a non-significant pair yields p-values near 1 (e.g. p=0.913 and p=0.813 in the documented example). Merge non-significantly different clusters with cc.tl.merge_clusters(data, 'leiden') and visualize the cell-by-cluster enrichment scores with cc.pl.enrich_heatmap(data, 'leiden_merged').

## Related tools

- **Cytocipher** (Provides cc.tl.get_markers, cc.tl.code_enrich, cc.tl.merge_clusters, and cc.pl.sig_cluster_diagnostics for marker identification, code/coexpr scoring, significance testing, and cluster merging) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (Standard workflow used upstream to produce the log-cpm normalized AnnData input with tentative cluster labels)
- **AnnData** (Container format holding normalized counts, cluster labels, and the cell-by-cluster enrichment score matrix)
- **Jupyter** (Environment in which function documentation (e.g. ?cc.tl.get_markers) and diagnostics plots are inspected)

## Examples

```
import cytocipher as cc; cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden'); cc.tl.merge_clusters(data, 'leiden')
```

## Evaluation signals

- cc.tl.code_enrich populates a cell-by-cluster enrichment score matrix in the AnnData object that can be extracted to enrichment_scores.csv
- For a non-significant cluster pair, cc.pl.sig_cluster_diagnostics prints p-values near 1 (documented example: p=0.913 for pair ('3','9') in one direction and p=0.813 in the other)
- cc.pl.enrich_heatmap on 'leiden_merged' shows fewer, more distinct score blocks than the original over-clustered 'leiden' labels
- Input AnnData must contain log-cpm normalized values (scanpy standard workflow) and a high-resolution cluster-label column
- Diagnostics without plot_pair plot the upper- and lower-bounds for significant versus non-significant cluster pairs, providing a reference range for score separation

## Limitations

- Significance testing depends on substantial over-clustering: the README notes 'It's better if the Leiden resolution is high, so that there is alot of over-clustering'
- The README example shows very small groups being compared ('(3 cells; 3 scores) vs (9 cells; 3 scores)'), so p-values on tiny cluster pairs may be unstable
- The paper's discussion content was not available in the provided context, so limitations and reproducibility gaps could not be extracted from that section

## Evidence

- [readme] Expected input requires scanpy-standard processing: [evidence span withheld — anchor status: not_found]
- [readme] High-resolution over-clustering recommendation: "It's better if the Leiden resolution is high, so that there is alot of over-clustering. Cytocipher merges the non-significantly different clusters."
- [methods] Core three-step scoring workflow: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
- [readme] Diagnostics call with documented p-values: "cc.pl.sig_cluster_diagnostics(data, 'leiden', plot_pair=('3', '9'))"
- [readme] Merging step and merged-label visualization: "Below runs the cluster merging and visualises the heatmap of enrichment scores per cell for each of the new merged clusters."
- [abstract] Abstract statement of tool purpose: "we developed Cytocipher, a bioinformatics method and scverse compatible software package that statistically determines significant clusters"
