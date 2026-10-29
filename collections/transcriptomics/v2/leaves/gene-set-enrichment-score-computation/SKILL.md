---
name: gene-set-enrichment-score-computation
description: Use when an AnnData object processed via the scanpy standard workflow
  (log-cpm normalised, high-resolution Leiden labels) needs per-cluster enrichment
  scores to assign cells to marker-defined clusters, or when comparing which scoring
  method discriminates clusters defined by unique gene combinations.
when_to_use_negative:
- Input is raw unnormalised counts that have not been log-cpm normalised and clustered
  — the enrichment functions expect a scanpy-standard-processed AnnData with existing
  Leiden labels
- You need statistically validated cluster merging rather than raw enrichment scores
  alone — Cytocipher's cc.tl.merge_clusters / sig_cluster_diagnostics workflow is
  the relevant skill instead
- Clusters are already biologically validated and you only need visualisation, not
  scoring (cc.pl.enrich_heatmap alone would not add classification value)
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: cytocipher
  role: Provides cc.tl.get_markers for marker gene identification and cc.tl.code_enrich
    for code scoring (cell by cluster enrichment scores), the paper's own scoring
    method under evaluation
  repo: https://github.com/BradBalderson/Cytocipher
- name: scanpy
  role: Standard workflow (log-cpm normalisation, Leiden clustering) producing the
    input AnnData; scanpy's score_genes is also one of the six benchmarked scorers
    (Scanpy (−) variant)
  repo: https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- name: AnnData
  role: Container format holding the processed expression matrix and cluster labels
    the scoring functions consume
- name: Jupyter
  role: Notebook environment for running the scoring functions and inspecting documentation
    (e.g. ?cc.tl.get_markers)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:05b1d7048ca88e67bd99aa169b1ae578a3b579f0d1c0b5545f85a4b60a223184
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Jupyter
  - cytocipher
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# gene-set-enrichment-score-computation

## Summary

Computes cell-by-cluster gene-set enrichment scores from a log-cpm normalised, Leiden-clustered scRNA-seq AnnData object, using Cytocipher's code scoring alongside alternative scorers (Scanpy score_genes, Giotto PAGE, coexpr scoring, and negative-gene-set-subtraction variants). The paper's simulated-data benchmark found negative-set-subtraction methods (Giotto (−), Scanpy (−)) and Cytocipher's code-scoring were the top performers on per-cluster binary classification.

## When to use

Use when an AnnData object processed via the scanpy standard workflow (log-cpm normalised, high-resolution Leiden labels) needs per-cluster enrichment scores to assign cells to marker-defined clusters, or when comparing which scoring method best discriminates clusters defined by unique gene combinations.

## When NOT to use

- Input is raw unnormalised counts that have not been log-cpm normalised and clustered — the enrichment functions expect a scanpy-standard-processed AnnData with existing Leiden labels
- You need statistically validated cluster merging rather than raw enrichment scores alone — Cytocipher's cc.tl.merge_clusters / sig_cluster_diagnostics workflow is the relevant skill instead
- Clusters are already biologically validated and you only need visualisation, not scoring (cc.pl.enrich_heatmap alone would not add classification value)

## Inputs

- AnnData object processed per the scanpy standard workflow (log-cpm normalised) with tentative high-resolution Leiden cluster labels (e.g. in data.obs['leiden'])
- Per-cluster marker gene sets derived via cc.tl.get_markers(data, 'leiden')

## Outputs

- Cell-by-cluster enrichment score matrix stored in the AnnData object (inspectable via cc.pl.enrich_heatmap)
- Per-cluster binary cell-to-cluster assignments and classification metrics (accuracy, precision, recall, F1) per scoring method, e.g. assembled into method_benchmark_metrics.csv

## How to apply

Start from an AnnData object, *data*, produced by the scanpy standard workflow so it holds log-cpm normalised values and tentative cluster labels; use high Leiden resolution so there is a lot of over-clustering, which improves downstream significance assessment. Derive per-cluster marker sets with cc.tl.get_markers(data, 'leiden'), then score every cell against each cluster's marker set with cc.tl.code_enrich(data, 'leiden') to fill a cell-by-cluster enrichment score matrix; visualise with cc.pl.enrich_heatmap(data, 'leiden'). For benchmarking, run the same marker sets through all six methods — Cytocipher code scoring, coexpr-scoring, Giotto PAGE, Scanpy score_genes, and the negative-gene-set-subtraction variants Giotto (−) and Scanpy (−) — then convert scores to per-cluster binary cell-to-cluster assignments and compute accuracy, precision, recall and F1 per method. Expect the negative-subtraction approaches and code-scoring to rank highest (Giotto (−) > code-scoring > Scanpy (−) > Giotto PAGE > Scanpy-scoring > coexpr-scoring), with the subtraction methods outperforming except on recall.

## Related tools

- **cytocipher** (Provides cc.tl.get_markers for marker gene identification and cc.tl.code_enrich for code scoring (cell by cluster enrichment scores), the paper's own scoring method under evaluation) — https://github.com/BradBalderson/Cytocipher
- **scanpy** (Standard workflow (log-cpm normalisation, Leiden clustering) producing the input AnnData; scanpy's score_genes is also one of the six benchmarked scorers (Scanpy (−) variant)) — https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- **AnnData** (Container format holding the processed expression matrix and cluster labels the scoring functions consume)
- **Jupyter** (Notebook environment for running the scoring functions and inspecting documentation (e.g. ?cc.tl.get_markers))

## Examples

```
import cytocipher as cc; cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden'); cc.pl.enrich_heatmap(data, 'leiden')
```

## Evaluation signals

- The output benchmark table contains all six methods (code-scoring, coexpr-scoring, Giotto PAGE, Giotto (−), Scanpy-scoring, Scanpy (−)) with all four metrics (accuracy, precision, recall, F1) per cluster
- The method ranking matches the reported pattern: Giotto (−) > code-scoring > Scanpy (−) > Giotto PAGE > Scanpy-scoring > coexpr-scoring, with negative-gene-set-subtraction methods outperforming the standard variants except on recall
- The enrichment score matrix is cell-by-cluster shaped (n_cells × n_leiden_clusters) and can be rendered by cc.pl.enrich_heatmap without errors
- Marker sets were regenerated with cc.tl.get_markers for the exact same cluster label column ('leiden') used by cc.tl.code_enrich, so scores correspond to the same cluster identity

## Limitations

- On the simulated-data binary classification benchmark, Cytocipher's code-scoring ranked second behind Giotto (−); the negative-gene-set-subtraction variants outperformed on all metrics except recall, so method choice matters for recall-sensitive tasks
- coexpr-scoring was the lowest performer on the simulated data, making it a poor default choice
- Cytocipher scoring assumes heavy over-clustering from high-resolution Leiden labels — low-resolution labels reduce its intended benefit

## Evidence

- [methods] Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores.: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
- [readme] cc.tl.get_markers(data, 'leiden') / cc.tl.code_enrich(data, 'leiden') / cc.pl.enrich_heatmap(data, 'leiden'): "cc.tl.get_markers(data, 'leiden')
cc.tl.code_enrich(data, 'leiden')
cc.pl.enrich_heatmap(data, 'leiden')"
- [readme] to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering): "to produce log-cpm normalised data with tentative cluster labels 
(e.g. from Leiden clustering)"
- [readme] It's better if the Leiden resolution is high, so that there is alot of over-clustering.: "It's better if the Leiden resolution is high,
 so that there is alot of over-clustering."
