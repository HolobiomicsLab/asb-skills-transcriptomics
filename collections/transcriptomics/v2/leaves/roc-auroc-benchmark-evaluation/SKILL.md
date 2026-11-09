---
name: roc-auroc-benchmark-evaluation
description: Use when you must quantitatively benchmark cell-cluster scoring or cluster-merge
  significance methods as binary classifiers, by computing ROC curves and AUROC against
  known ground-truth subcluster labels on scRNA-seq data.
when_to_use_negative:
- Input AnnData has raw counts without log-cpm normalisation or tentative Leiden cluster
  labels, so no enrichment scores or merges can be benchmarked.
- The goal is de novo cluster discovery rather than evaluating/benchmarking a scoring
  or merge-significance method against a known ground truth.
- Only one scoring method is of interest with no comparison set (code-scoring, coexpr-scoring,
  Giotto PAGE, Giotto (−), Scanpy-score, Scanpy (−)) and no reference labels to compute
  ROC against.
edam_operation: http://edamontology.org/operation_2990
tools:
- name: cytocipher
  role: Provides the benchmarked pipeline functions cc.tl.get_markers, cc.tl.code_enrich
    and cc.tl.merge_clusters that generate the enrichment scores and cluster merges
    evaluated by ROC/AUROC
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: Used to load and preprocess the AnnData (scanpy standard workflow producing
    log-cpm normalised data with Leiden labels) prior to benchmarking
- name: AnnData
  role: Container format for the simulated and hypothalamus datasets carrying counts,
    normalised values and cluster labels used in the ROC benchmark
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:0b33c4e3616b9a5ae4a47bda033e49e32d00e4a676fe21f3ea443130ed2f2af4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Jupyter
  - cytocipher
  - scanpy
  edam_operation: http://edamontology.org/operation_2990
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# roc-auroc-benchmark-evaluation

## Summary

Benchmark of Cytocipher cluster-merge significance testing via binary ROC/AUROC evaluation, comparing enrichment scoring methods (code-scoring, coexpr-scoring, Giotto PAGE, Giotto (−), Scanpy-score, Scanpy (−)) against ground-truth cluster labels on simulated scRNA-seq and the mouse E18.5 hypothalamus dataset.

## When to use

Use when you must quantitatively benchmark cell-cluster scoring or cluster-merge significance methods as binary classifiers, by computing ROC curves and AUROC against known ground-truth subcluster labels on scRNA-seq data.

## When NOT to use

- Input AnnData has raw counts without log-cpm normalisation or tentative Leiden cluster labels, so no enrichment scores or merges can be benchmarked.
- The goal is de novo cluster discovery rather than evaluating/benchmarking a scoring or merge-significance method against a known ground truth.
- Only one scoring method is of interest with no comparison set (code-scoring, coexpr-scoring, Giotto PAGE, Giotto (−), Scanpy-score, Scanpy (−)) and no reference labels to compute ROC against.

## Inputs

- Log-cpm normalised AnnData object with tentative high-resolution Leiden cluster labels (processed via the scanpy standard workflow)
- Simulated scRNA-seq AnnData with ground-truth labels of three artificial subclusters per group (defined by unique gene combinations)
- E18.5 mouse hypothalamus neuropeptidergic-dopaminergic neuron AnnData (Yaghmaeian Salmani et al. 2022)
- Per-method cell-by-cluster enrichment scores from cc.tl.code_enrich for the compared scoring approaches

## Outputs

- ROC curves per scoring method (true/false positive rates across score thresholds)
- Per-method AUROC values (e.g. auroc_results.csv, auroc_benchmark.csv)
- Cluster-merge ROC plot
- Merged cluster assignments from cc.tl.merge_clusters for each scoring approach

## How to apply

Start from log-cpm normalised AnnData with tentative over-clustered Leiden labels (high resolution is preferred so Cytocipher can merge the non-significantly different clusters). For each scoring approach, run cc.tl.get_markers(data, 'leiden') for marker gene identification, cc.tl.code_enrich(data, 'leiden') for cell-by-cluster enrichment scores, and cc.tl.merge_clusters(data, 'leiden') to obtain merged clusters. Compare merged cluster assignments against the ground truth (e.g. three artificial subclusters per group in the simulation) to obtain true/false positive rates across score thresholds, then compute ROC curves and area under the curve (AUROC) per method. On simulated data AUROC should exceed 0.99 for all six scoring approaches; on the E18.5 hypothalamus neuropeptidergic-dopaminergic neuron data, expected ordering is coexpr-score (0.9715) > Giotto PAGE (0.9605) > code-score (0.9565) > Giotto (−) (0.9485), with Scanpy-scoring the only approach not exceeding 0.9. Write per-method AUROC values to auroc_results.csv / auroc_benchmark.csv and a cluster-merge ROC plot.

## Related tools

- **cytocipher** (Provides the benchmarked pipeline functions cc.tl.get_markers, cc.tl.code_enrich and cc.tl.merge_clusters that generate the enrichment scores and cluster merges evaluated by ROC/AUROC) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (Used to load and preprocess the AnnData (scanpy standard workflow producing log-cpm normalised data with Leiden labels) prior to benchmarking)
- **AnnData** (Container format for the simulated and hypothalamus datasets carrying counts, normalised values and cluster labels used in the ROC benchmark)

## Examples

```
cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden'); cc.tl.merge_clusters(data, 'leiden')
```

## Evaluation signals

- On simulated data with a three-way subcluster ground truth, AUROC must exceed 0.99 for all six scoring approaches (code-scoring, coexpr-scoring, Giotto PAGE, Scanpy-score, Giotto (−), Scanpy (−)).
- On the E18.5 hypothalamus data, verify AUROC ordering: coexpr-score 0.9715 > Giotto PAGE 0.9605 > code-score 0.9565 > Giotto (−) 0.9485, and that Scanpy-scoring does not exceed 0.9.
- ROC curves are computed from true/false positive rates obtained by thresholding scores and comparing merged assignments against the known subcluster labels.
- Per-method AUROC values are written to auroc_results.csv (simulated) and auroc_benchmark.csv (hypothalamus), plus a cluster-merge ROC plot is produced.

## Limitations

- Benchmark performance depends on the enrichment scoring method: on the E18.5 hypothalamus data, Scanpy-scoring was the only approach not exceeding an AUROC of 0.9, so results are not scoring-method invariant at the lower end.
- The cluster-merge AUROC depends on over-clustered Leiden labels; the README recommends high Leiden resolution so that Cytocipher merges the non-significantly different clusters.

## Evidence

- [other] On the E18.5 hypothalamus data, 'Cytocipher cluster-merge' achieved high performance regardless of scoring method: [evidence span withheld — anchor status: not_found]
- [other] coexpr-score had the highest AUROC (0.9715): [evidence span withheld — anchor status: not_found]
- [readme] Cytocipher merges the non-significantly different clusters.: "Cytocipher merges the non-significantly different clusters."
- [readme] Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores.: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
- [readme] log-cpm normalised data with tentative cluster labels: "to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering)"
