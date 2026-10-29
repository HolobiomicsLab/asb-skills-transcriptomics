---
name: single-cell-rnaseq-enrichment-scoring
description: Use when an AnnData object with log-cpm normalized counts and over-clustered
  (high-resolution) Leiden labels needs statistically grounded merging of non-significantly
  different cell populations.
when_to_use_negative:
- Input AnnData has not been log-cpm normalized and clustered with Leiden labels —
  the code-score requires normalized expression and tentative cluster groupings
- Goal is standard additive enrichment scoring (e.g. per-gene-set sum/mean) rather
  than joint-probability-based co-expression code scoring across cluster marker sets
- Only a single, well-supported clustering exists with no over-clustering — Cytocipher
  is designed to merge non-significantly different clusters from high-resolution over-clustering
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: cytocipher
  role: Provides cc.tl.get_markers, cc.tl.code_enrich, cc.tl.merge_clusters, and cc.pl.sig_cluster_diagnostics
    used to derive markers, score enrichment, merge clusters, and diagnose significance
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: Standard workflow preprocessing producing log-cpm normalized data with tentative
    Leiden cluster labels
  repo: https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- name: AnnData
  role: Container format for the input data and the resulting enrichment score matrices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:75e3c241bc0e8d7f6ac02a4ba2eca71420b92d310c8d8c44413897cdfc52a0fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - cytocipher
  - scanpy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-rnaseq-enrichment-scoring

## Summary

Cytocipher's code-scoring quantitatively scores gene set co-expression per cell as the log of the inverse joint probability of expressing each marker gene at >= the level observed in that cell, producing a cell-by-cluster enrichment score matrix used to test which Leiden clusters are significantly different.

## When to use

Use when an AnnData object with log-cpm normalized counts and over-clustered (high-resolution) Leiden labels needs statistically grounded merging of non-significantly different cell populations.

## When NOT to use

- Input AnnData has not been log-cpm normalized and clustered with Leiden labels — the code-score requires normalized expression and tentative cluster groupings
- Goal is standard additive enrichment scoring (e.g. per-gene-set sum/mean) rather than joint-probability-based co-expression code scoring across cluster marker sets
- Only a single, well-supported clustering exists with no over-clustering — Cytocipher is designed to merge non-significantly different clusters from high-resolution over-clustering

## Inputs

- AnnData object (.h5ad) with log-cpm normalized counts from the scanpy standard workflow
- Tentative Leiden cluster labels (high resolution / over-clustered)
- Cluster grouping column name, e.g. 'leiden'

## Outputs

- Cell-by-cluster enrichment score matrix stored in the AnnData object (exportable as enrichment_scores.csv)
- Per-cluster marker and negative gene sets from cc.tl.get_markers
- Merged cluster labels ('leiden_merged') after cc.tl.merge_clusters
- p-values comparing enrichment score distributions for cluster pairs (e.g. p=0.913, p=0.813 for a non-significant pair)

## How to apply

Start from an AnnData object processed per the scanpy standard workflow (log-cpm normalized) with tentative Leiden cluster labels, preferably at high resolution so clusters are over-clustered. Run cc.tl.get_markers(data, 'leiden') to derive per-cluster marker and negative gene sets with the winsorization function f bounding expression values. Run cc.tl.code_enrich(data, 'leiden') to apply the code-score (Eq. 3) and coexpr-score (Eq. 7), assigning each cell an enrichment score against every cluster as the log of the inverse joint probability of expressing each marker gene at >= the observed level. Extract the cell-by-cluster enrichment score matrix (e.g. to enrichment_scores.csv), then use cc.tl.merge_clusters to merge non-significantly different clusters and cc.pl.sig_cluster_diagnostics to inspect score distributions and p-values for cluster pairs.

## Related tools

- **cytocipher** (Provides cc.tl.get_markers, cc.tl.code_enrich, cc.tl.merge_clusters, and cc.pl.sig_cluster_diagnostics used to derive markers, score enrichment, merge clusters, and diagnose significance) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (Standard workflow preprocessing producing log-cpm normalized data with tentative Leiden cluster labels) — https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- **AnnData** (Container format for the input data and the resulting enrichment score matrices)

## Examples

```
import cytocipher as cc; cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden'); cc.pl.enrich_heatmap(data, 'leiden')
```

## Evaluation signals

- cc.pl.sig_cluster_diagnostics on a non-significant cluster pair yields p-values near 1 (documented example: p=0.9132771265170103 and p=0.8128313109661132 for pair ('3','9'))
- cc.pl.enrich_heatmap on the cell-by-cluster enrichment scores renders without error and shows enrichment structure per cluster
- The enrichment score matrix extracted to enrichment_scores.csv has rows equal to the number of cells and columns equal to the number of clusters
- After cc.tl.merge_clusters, a 'leiden_merged' label column exists and cc.pl.enrich_heatmap(data, 'leiden_merged') visualizes scores for the new merged clusters
- CC PL diagnostics of default bounds plot violins for upper- and lower-bounds of significant versus non-significant cluster pairs, giving a sanity range

## Limitations

- Requires substantial over-clustering; Cytocipher is intended to merge the non-significantly different clusters, so low-resolution labels reduce its utility
- Only a single example p-value pair is documented; significance thresholds and bounds should be checked per dataset using cc.pl.sig_cluster_diagnostics
- No changelog found for the repository, so version-specific behavior may be uncertain

## Evidence

- [readme] Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores.: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
- [readme] An AnnData object, *data*, that has been processed similarly to the scanpy standard workflow to produce log-cpm normalised data with tentative cluster labels: "An AnnData object, *data*, that has been processed similarly to the 
[scanpy standard workflow](https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html)
to produce log-cpm normalised data with"
- [readme] It's better if the Leiden resolution is high, so that there is alot of over-clustering. Cytocipher merges the non-significantly different clusters.: "It's better if the Leiden resolution is high,
 so that there is alot of over-clustering. 
 Cytocipher merges the non-significantly different clusters."
- [results] quantitatively scores gene set co-expression per cell as the log of the inverse joint probability of expressing each gene in the marker gene set: [evidence span withheld — anchor status: not_found]
- [readme] the following visualises the scores as violin plots of the enrichment scores & prints the p-values determined by comparing the scores: "the following visualises the scores as violin plots of the enrichment scores
& prints the p-values determined by comparing the scores"
