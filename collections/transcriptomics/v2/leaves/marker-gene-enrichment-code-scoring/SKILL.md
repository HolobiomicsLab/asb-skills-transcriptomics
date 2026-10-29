---
name: marker-gene-enrichment-code-scoring
description: Use when an AnnData with log-cpm normalised counts and over-clustered
  Leiden labels needs per-cluster marker genes and cell-by-cluster enrichment scores,
  e.g. to judge which cluster pairs are significantly different before merging.
when_to_use_negative:
- The data is already a feature table or aggregated scores rather than a raw/normalised
  scRNA-seq count AnnData requiring marker-gene-based enrichment scoring
- Clusters were produced at low Leiden resolution with no over-clustering, so there
  are few non-significantly different pairs for the enrichment scores to discriminate
- The goal is direct cell-level clustering rather than testing whether existing cluster
  pairs are significantly different
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: cytocipher
  role: Provides cc.tl.get_markers, cc.tl.code_enrich, cc.tl.merge_clusters and cc.pl.*
    functions that implement marker identification, enrichment scoring and significance
    testing
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: Standard workflow used upstream to produce the log-cpm normalised AnnData
    with Leiden cluster labels that Cytocipher consumes
  repo: https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- name: AnnData
  role: Input container format expected by all Cytocipher functions
  repo: https://github.com/BradBalderson/Cytocipher/
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:00f7df747222ba3de450e00cd5f5c30c24532dcd3908d4f1bc7026be7c8d8988
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# marker-gene-enrichment-code-scoring

## Summary

Cytocipher's marker-gene identification and cell-by-cluster enrichment scoring ('code scoring') pipeline, which computes per-cluster enrichment scores on log-cpm normalised scRNA-seq AnnData and feeds the significance testing behind cluster-merge (AUROC > 0.99 across scoring approaches).

## When to use

Use when an AnnData with log-cpm normalised counts and over-clustered Leiden labels needs per-cluster marker genes and cell-by-cluster enrichment scores, e.g. to judge which cluster pairs are significantly different before merging.

## When NOT to use

- The data is already a feature table or aggregated scores rather than a raw/normalised scRNA-seq count AnnData requiring marker-gene-based enrichment scoring
- Clusters were produced at low Leiden resolution with no over-clustering, so there are few non-significantly different pairs for the enrichment scores to discriminate
- The goal is direct cell-level clustering rather than testing whether existing cluster pairs are significantly different

## Inputs

- AnnData object processed with the scanpy standard workflow to log-cpm normalised counts
- Tentative over-clustered Leiden labels stored under a cluster key (e.g. 'leiden', resolution 3.5 for the pancreas example)

## Outputs

- Per-cluster marker genes (from cc.tl.get_markers)
- Cell-by-cluster enrichment score matrix (from cc.tl.code_enrich)
- Enrichment-score heatmap (cc.pl.enrich_heatmap)
- 'leiden_merged' cluster assignments after significance testing in cc.tl.merge_clusters
- AUROC values per scoring method (e.g. written to auroc_results.csv)

## How to apply

Start from an AnnData processed with the scanpy standard workflow to log-cpm normalised values with tentative Leiden labels, deliberately over-clustered at high Leiden resolution (resolution 3.5 in the E15.5 pancreas example) so there are many non-significantly different clusters to merge. Run cc.tl.get_markers(data, 'leiden') on the over-clustered labels to identify marker genes per cluster, then cc.tl.code_enrich(data, 'leiden') to compute the cell-by-cluster enrichment scores; cc.pl.enrich_heatmap(data, 'leiden') visualises the resulting score matrix. Six scoring variants are supported for benchmarking — code-scoring, coexpr-scoring, Giotto PAGE, Scanpy-score, Giotto (−), Scanpy (−) — and the scores feed cc.tl.merge_clusters(data, 'leiden') which merges non-significantly different clusters at the adjusted P-value cutoff (0.045 in the pancreas run). Validate by comparing merged assignments against ground truth (three-way artificial subclusters on simulated data, or Tabula Sapiens cell-type labels) and computing AUROC, and inspect score distributions for significant versus non-significant pairs with cc.pl.sig_cluster_diagnostics(data, 'leiden', plot_pair=('3', '9')).

## Related tools

- **cytocipher** (Provides cc.tl.get_markers, cc.tl.code_enrich, cc.tl.merge_clusters and cc.pl.* functions that implement marker identification, enrichment scoring and significance testing) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (Standard workflow used upstream to produce the log-cpm normalised AnnData with Leiden cluster labels that Cytocipher consumes) — https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- **AnnData** (Input container format expected by all Cytocipher functions) — https://github.com/BradBalderson/Cytocipher/

## Examples

```
import cytocipher as cc; cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden'); cc.pl.enrich_heatmap(data, 'leiden')
```

## Evaluation signals

- On simulated data with three artificial subclusters per group, AUROC exceeds 0.99 for all six scoring approaches
- On the 483152-cell Tabula Sapiens over-cluster benchmark, AUROC is 0.9974 (code-scoring) and 0.9976 (coexpr-scoring)
- Downsampled to 7385 cells, AUROCs match 0.9930, 0.9937, 0.9945 and 0.9940 for Cytocipher (code/coexpr) and Giotto PAGE / Giotto (−) respectively
- Applying merge_clusters to the E15.5 pancreas over-clustered data (adjusted P-value cutoff 0.045) yields 16 distinctly different merged clusters
- cc.pl.sig_cluster_diagnostics violin plots show score distributions and printed p-values for a given cluster pair, e.g. ('3', '9')

## Limitations

- Scanpy-based scoring variants performed worse on the large Tabula Sapiens benchmark (AUROC < 0.9) than code-scoring and coexpr-scoring
- Cytocipher's merging pipeline is slow compared with Sc-SHC: 178.80 s versus <3.5 s on a 500-cell, 12-cell-type subset
- Over-clustering is expected — the README notes 'It's better if the Leiden resolution is high, so that there is alot of over-clustering' — so low-resolution inputs give few merge decisions
- No changelog was found for the repository, limiting traceability of parameter or API changes

## Evidence

- [readme] workflow steps / readme: "An AnnData object, *data*, that has been processed similarly to the 
[scanpy standard workflow]"
- [readme] leiden resolution: "It's better if the Leiden resolution is high,
 so that there is alot of over-clustering. 
 Cytocipher merges the non-significantly different clusters."
- [readme] core functions: "Functions below run the marker gene identification, code scoring, & 
subsequent visualisation of the resulting cell by cluster enrichment scores."
- [abstract] abstract claim: "we developed Cytocipher, a bioinformatics method and scverse compatible software package that statistically determines significant clusters"
- [results] simulated benchmark finding: [evidence span withheld — anchor status: not_found]
