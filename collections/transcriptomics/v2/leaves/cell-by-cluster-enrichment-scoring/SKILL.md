---
name: cell-by-cluster-enrichment-scoring
description: Use when an AnnData object has log-cpm normalised scRNA-seq data with
  tentative (ideally over-clustered) Leiden labels and you need per-cell enrichment
  scores against every cluster's marker signature to support significance-based cluster
  merging.
when_to_use_negative:
- Input is raw un-normalised counts — the AnnData must first be log-cpm normalised
  via the scanpy standard workflow
- No tentative cluster labels exist yet; Leiden clustering must be run before marker
  identification and scoring
- You need per-method differential expression testing rather than cluster-membership
  enrichment scores for merge significance testing
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: cytocipher
  role: Provides cc.tl.get_markers and cc.tl.code_enrich to compute marker-gene-based
    cell-by-cluster enrichment scores
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: Standard workflow used to produce the log-cpm normalised AnnData input with
    Leiden cluster labels
- name: AnnData
  role: Container format for the input data and grouping key carrying cluster labels
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:03d89b47cafa223372bc16d081e72101ec3960a5a4a9adbaf7b51da6032713a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Jupyter
  - cytocipher
  - scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cell-by-cluster-enrichment-scoring

## Summary

Cytocipher's code-scoring step computes cell-by-cluster enrichment scores from marker genes identified for each tentative Leiden cluster, quantifying how strongly each cell expresses each cluster's signature. These scores underpin Cytocipher's statistical cluster-merge significance testing.

## When to use

Use when an AnnData object has log-cpm normalised scRNA-seq data with tentative (ideally over-clustered) Leiden labels and you need per-cell enrichment scores against every cluster's marker signature to support significance-based cluster merging.

## When NOT to use

- Input is raw un-normalised counts — the AnnData must first be log-cpm normalised via the scanpy standard workflow
- No tentative cluster labels exist yet; Leiden clustering must be run before marker identification and scoring
- You need per-method differential expression testing rather than cluster-membership enrichment scores for merge significance testing

## Inputs

- AnnData object with log-cpm normalised scRNA-seq counts (scanpy standard workflow output)
- Tentative cluster labels under a grouping key (e.g. 'leiden'), preferably from high-resolution Leiden clustering

## Outputs

- Cell-by-cluster enrichment score matrix stored in the AnnData under the grouping key
- Enrichment score heatmap (cc.pl.enrich_heatmap) visualising cells vs clusters
- Marker gene sets per cluster from cc.tl.get_markers used as the scoring basis

## How to apply

Start from an AnnData object processed with the scanpy standard workflow to log-cpm normalised data carrying tentative cluster labels (e.g. from Leiden clustering at high resolution so there is deliberate over-clustering). First run cc.tl.get_markers(data, 'leiden') with default parameters to identify marker genes per cluster, then cc.tl.code_enrich(data, 'leiden') to compute the cell-by-cluster enrichment score matrix. Inspect the scores with cc.pl.enrich_heatmap(data, 'leiden') and use ?cc.tl.get_markers in Jupyter to check documentation. These code scores are then consumed downstream by cc.tl.merge_clusters for significance testing; the same scoring mechanism was benchmarked against Giotto PAGE and Scanpy-scoring on the E18.5 hypothalamus data, where code-score reached AUROC 0.9565 and coexpr-score 0.9715.

## Related tools

- **cytocipher** (Provides cc.tl.get_markers and cc.tl.code_enrich to compute marker-gene-based cell-by-cluster enrichment scores) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (Standard workflow used to produce the log-cpm normalised AnnData input with Leiden cluster labels)
- **AnnData** (Container format for the input data and grouping key carrying cluster labels)

## Examples

```
import cytocipher as cc; cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden'); cc.pl.enrich_heatmap(data, 'leiden')
```

## Evaluation signals

- cc.pl.enrich_heatmap(data, 'leiden') renders a cells-by-clusters enrichment heatmap without errors after scoring
- Enrichment scores exist for every cell against every tentative cluster in the grouping key
- Downstream cc.pl.sig_cluster_diagnostics violin plots of score distributions for cluster pairs (e.g. plot_pair=('3','9')) print p-values derived from the scores
- Benchmark comparison: on E18.5 hypothalamus, code-score achieved AUROC 0.9565 and coexpr-score 0.9715, above Giotto (−) at 0.9485
- On PBMC 3K over-clustered at Leiden resolution 2.0, scores from these steps enable cc.tl.merge_clusters to merge CD4+ T cell and CD14+ monocyte over-clusters while keeping CD8+ T cell sub-clusters split

## Limitations

- It is better if the Leiden resolution is high (over-clustering) — scoring assumes tentative clusters are deliberately over-resolved so non-significant ones can be merged
- The README example diagnostics show very small score sets ('3 scores' per cluster), indicating scores-per-cluster counts can be small and p-value comparisons may be noisy for tiny clusters
- Scanpy-scoring was the only approach not exceeding AUROC 0.9 in the benchmark, so score choice affects downstream merge benchmarking

## Evidence

- [readme] Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores.: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
- [readme] Expected input AnnData with log-cpm normalised data and tentative cluster labels: [evidence span withheld — anchor status: not_found]
- [readme] Better to use high Leiden resolution so there is over-clustering: "It's better if the Leiden resolution is high,
 so that there is alot of over-clustering.
 Cytocipher merges the non-significantly different clusters."
- [results] AUROC benchmark of scoring methods from the source card finding: [evidence span withheld — anchor status: not_found]
- [methods] Jupyter documentation access: "In a jupyter notebook, you can see documentation using, for example:"
