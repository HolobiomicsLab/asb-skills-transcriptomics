---
name: single-cell-rna-seq-cluster-significance-testing
description: Use when scRNA-seq data have been log-cpm normalised with a tentative
  high-resolution Leiden clustering producing deliberate over-clustering, and you
  need to determine which cluster pairs are not significantly different so they can
  be merged.
when_to_use_negative:
- Input is raw counts or a feature/marker table rather than an AnnData object log-cpm
  normalised with existing tentative cluster labels — Cytocipher expects an already-processed,
  over-clustered solution to merge
- Clusters are already well-resolved cell types with no over-clustering (README advises
  high Leiden resolution so 'there is alot of over-clustering'); merging would not
  be meaningful
- Goal is clustering from scratch — this skill tests and merges existing clusters,
  it does not perform Leiden clustering itself
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cytocipher
  role: 'Primary method: marker gene identification (cc.tl.get_markers), code scoring
    (cc.tl.code_enrich), and significance-based cluster merging (cc.tl.merge_clusters)'
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: Preprocessing to log-cpm normalised values with Leiden cluster labels following
    the scanpy standard workflow (pbmc3k tutorial)
  repo: https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- name: AnnData
  role: Input data container for the processed single-cell dataset and cluster labels
- name: Jupyter
  role: Notebook environment for inspecting documentation (e.g. ?cc.tl.get_markers)
    and running visual diagnostics
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:28af1887e370d976cc35dbd17d66bd85cc2f72687799cc8b987f396bafca2698
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# single-cell-rna-seq-cluster-significance-testing

## Summary

Statistically tests whether clusters in scRNA-seq data are significantly different from each other using Cytocipher's marker-gene code-scoring and enrichment-based significance testing, then merges non-significant clusters. Enables principled cluster merging (e.g. on deliberately over-clustered Leiden solutions) with quantified test performance such as AUROC against known cell-type labels.

## When to use

Use when scRNA-seq data have been log-cpm normalised with a tentative high-resolution Leiden clustering producing deliberate over-clustering, and you need to determine which cluster pairs are not significantly different so they can be merged.

## When NOT to use

- Input is raw counts or a feature/marker table rather than an AnnData object log-cpm normalised with existing tentative cluster labels — Cytocipher expects an already-processed, over-clustered solution to merge
- Clusters are already well-resolved cell types with no over-clustering (README advises high Leiden resolution so 'there is alot of over-clustering'); merging would not be meaningful
- Goal is clustering from scratch — this skill tests and merges existing clusters, it does not perform Leiden clustering itself

## Inputs

- AnnData object ('data') processed via the scanpy standard workflow to log-cpm normalised values with tentative Leiden cluster labels (high resolution preferred so there is substantial over-clustering)
- Column name holding the cluster labels (e.g. 'leiden')
- Known cell-type labels for AUROC benchmarking (e.g. Tabula Sapiens artificial over-clusters)

## Outputs

- Marker genes per cluster (cc.tl.get_markers)
- Cell-by-cluster enrichment scores (cc.tl.code_enrich)
- Merged cluster labels stored as e.g. 'leiden_merged' after cc.tl.merge_clusters
- Enrichment heatmap and sig_cluster_diagnostics violin plots of enrichment scores with p-values per cluster pair
- AUROC values quantifying discrimination of significant from non-significant artificial over-cluster pairs (e.g. 0.9974 code-scoring / 0.9976 coexpr-scoring on Tabula Sapiens)

## How to apply

Start from an AnnData object processed with the scanpy standard workflow to log-cpm normalised values with tentative cluster labels (Leiden at high resolution so over-clustering is abundant). Run cc.tl.get_markers(data, 'leiden') for marker gene identification, then cc.tl.code_enrich(data, 'leiden') to compute cell-by-cluster enrichment scores, inspecting with cc.pl.enrich_heatmap. Run cc.tl.merge_clusters(data, 'leiden') under either code-scoring or coexpr-scoring significance testing; Cytocipher merges the non-significantly different clusters. Validate by computing AUROC of the merged clusters against known cell-type labels — on the 483152-cell Tabula Sapiens artificial over-cluster benchmark this yields ~0.9974 (code-scoring) and ~0.9976 (coexpr-scoring), with Scanpy-based scoring performing worse (AUROC<0.9). Use cc.pl.sig_cluster_diagnostics (optionally with plot_pair=('3','9')) to inspect enrichment-score violins and p-values for significant versus non-significant cluster pairs as a QC diagnostic.

## Related tools

- **Cytocipher** (Primary method: marker gene identification (cc.tl.get_markers), code scoring (cc.tl.code_enrich), and significance-based cluster merging (cc.tl.merge_clusters)) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (Preprocessing to log-cpm normalised values with Leiden cluster labels following the scanpy standard workflow (pbmc3k tutorial)) — https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- **AnnData** (Input data container for the processed single-cell dataset and cluster labels)
- **Jupyter** (Notebook environment for inspecting documentation (e.g. ?cc.tl.get_markers) and running visual diagnostics)

## Examples

```
import cytocipher as cc; cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden'); cc.tl.merge_clusters(data, 'leiden'); cc.pl.sig_cluster_diagnostics(data, 'leiden', plot_pair=('3', '9'))
```

## Evaluation signals

- Merged-cluster AUROC against known cell-type labels matches published benchmarks: 0.9974 (code-scoring) and 0.9976 (coexpr-scoring) on the 483152-cell Tabula Sapiens artificial over-clusters
- Downsampled (7385-cell) AUROCs match reference values: 0.9930 (code-scoring), 0.9937 (coexpr-scoring), 0.9945 (Giotto PAGE), 0.9940 (Giotto (−))
- cc.pl.sig_cluster_diagnostics prints per-pair p-values (e.g. p=0.913 and p=0.812 for pair ('3','9')) and violin plots of enrichment scores for significant vs non-significant bounds
- enrich_heatmap on 'leiden' (pre-merge) and 'leiden_merged' (post-merge) produces interpretable cell-by-cluster enrichment-score heatmaps
- Runtime sanity check on a 500-cell 12-cell-type subset: Sc-SHC <3.5 s vs Cytocipher merging pipeline 178.80 s

## Limitations

- Cytocipher's cluster merging is slow relative to competitors: 178.80 s on a 500-cell 12-cell-type subset vs Sc-SHC <3.5 s
- Default significance bounds should be sanity-checked with cc.pl.sig_cluster_diagnostics before trusting merges; small clusters yield few scores (e.g. 3 cells; 3 scores) making p-value estimates fragile
- The method requires deliberate over-clustering (high Leiden resolution) to be effective; no changelog is available for the repository and discussion-section limitations are not documented

## Evidence

- [readme] An AnnData object, *data*, that has been processed similarly to the scanpy standard workflow to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering).: "An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow]"
- [readme] Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores.: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
- [readme] Cytocipher merges the non-significantly different clusters.: " Cytocipher merges the non-significantly different clusters."
- [abstract] we developed Cytocipher, a bioinformatics method and scverse compatible software package that statistically determines significant clusters: "we developed Cytocipher, a bioinformatics method and scverse compatible software package that statistically determines significant clusters"
- [abstract] Cytocipher also scales to large datasets with high-test performance, as shown by application to the Tabula Sapiens Atlas representing >480 000 cells: "Cytocipher also scales to large datasets with high-test performance, as shown by application to the Tabula Sapiens Atlas representing >480 000 cells"
- [readme] In a jupyter notebook, you can see documentation using, for example: ?cc.tl.get_markers: "In a jupyter notebook, you can see documentation using, for example:"
- [readme] the following visualises the scores as violin plots of the enrichment scores & prints the p-values determined by comparing the scores: "the following visualises the scores as violin plots of the enrichment scores
& prints the p-values determined by comparing the scores"
