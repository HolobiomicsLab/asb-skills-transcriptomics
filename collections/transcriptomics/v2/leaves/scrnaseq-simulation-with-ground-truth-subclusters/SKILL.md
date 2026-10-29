---
name: scrnaseq-simulation-with-ground-truth-subclusters
description: Use when validating or benchmarking a cluster-merging / cluster-significance
  method (e.g. Cytocipher cluster-merge) on simulated scRNA-seq data where the true
  subcluster structure is known and an AUROC against ground-truth labels is the target
  metric.
when_to_use_negative:
- Input is real experimental scRNA-seq data without any ground-truth subcluster labels
  — ROC/AUROC against the three-way artificial subclusters cannot be computed.
- The Leiden labels are not over-clustered (low resolution) — the README recommends
  high Leiden resolution so there is substantial over-clustering for Cytocipher to
  merge.
- The data is raw counts without log-cpm normalisation and tentative cluster labels
  — the expected input is an AnnData processed via the scanpy standard workflow.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: Cytocipher
  role: Provides get_markers, code_enrich and merge_clusters used to score and merge
    over-clustered groups under six scoring approaches
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: Scanpy-score is one of the enrichment scoring approaches benchmarked; data
    processed via the scanpy standard workflow
- name: AnnData
  role: Container for the simulated log-cpm normalised counts, Leiden labels and ground-truth
    subcluster annotations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:674fe59dbd89436d4798506c6050af8d8cefe5f95f60f2f58627f4585ade5024
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Cytocipher
  - cytocipher
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# scrnaseq-simulation-with-ground-truth-subclusters

## Summary

Benchmark cluster-merge discrimination on simulated scRNA-seq data where groups are defined by unique gene combinations and each group carries a three-way artificial subcluster ground truth, scoring cluster-pair significance via ROC/AUROC across multiple enrichment-scoring approaches.

## When to use

Use when validating or benchmarking a cluster-merging / cluster-significance method (e.g. Cytocipher cluster-merge) on simulated scRNA-seq data where the true subcluster structure is known and an AUROC against ground-truth labels is the target metric.

## When NOT to use

- Input is real experimental scRNA-seq data without any ground-truth subcluster labels — ROC/AUROC against the three-way artificial subclusters cannot be computed.
- The Leiden labels are not over-clustered (low resolution) — the README recommends high Leiden resolution so there is substantial over-clustering for Cytocipher to merge.
- The data is raw counts without log-cpm normalisation and tentative cluster labels — the expected input is an AnnData processed via the scanpy standard workflow.

## Inputs

- Simulated AnnData (.h5ad-style object) with log-cpm normalised counts
- Ground-truth labels of three artificial subclusters per group
- Over-clustered tentative Leiden cluster labels (column 'leiden')

## Outputs

- Per-scoring-method AUROC values written to auroc_results.csv
- ROC curves of true vs false positive rates across score thresholds
- Merged cluster assignments ('leiden_merged') comparable against ground-truth subcluster labels

## How to apply

Load a simulated AnnData object containing log-cpm normalised counts with ground-truth labels of three artificial subclusters per group, plus over-clustered Leiden labels at high resolution (deliberate over-clustering is recommended by the authors). For each scoring approach — code-scoring, coexpr-scoring, Giotto PAGE, Scanpy-score, Giotto (−), Scanpy (−) — run marker gene identification with cc.tl.get_markers(data, 'leiden'), compute cell-by-cluster enrichment scores with cc.tl.code_enrich(data, 'leiden'), then merge non-significantly different clusters with cc.tl.merge_clusters(data, 'leiden'). Compare merged assignments against the known three-way subcluster ground truth to derive true/false positive rates across score thresholds, and compute ROC curves and AUROC per scoring method. Decision point: the skill succeeds when AUROC exceeds 0.99 for all scoring approaches, confirming that significance-based merging robustly discriminates significant vs non-significant cluster pairs regardless of the enrichment scoring method used.

## Related tools

- **Cytocipher** (Provides get_markers, code_enrich and merge_clusters used to score and merge over-clustered groups under six scoring approaches) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (Scanpy-score is one of the enrichment scoring approaches benchmarked; data processed via the scanpy standard workflow)
- **AnnData** (Container for the simulated log-cpm normalised counts, Leiden labels and ground-truth subcluster annotations)

## Examples

```
cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden'); cc.tl.merge_clusters(data, 'leiden')
```

## Evaluation signals

- AUROC exceeds 0.99 for all six scoring approaches (code-scoring, coexpr-scoring, Giotto PAGE, Scanpy-score, Giotto (−), Scanpy (−)).
- Per-method AUROC values are written to auroc_results.csv with one row per scoring method.
- Merged cluster assignments map onto the three-way artificial subcluster ground truth per group, not onto the over-clustered Leiden labels.
- ROC curves are built from true/false positive rates computed across score thresholds against the ground-truth subcluster labels.
- Each cc.tl code path (get_markers → code_enrich → merge_clusters) is run identically per scoring approach so methods are compared on the same pipeline.

## Limitations

- Performance is demonstrated on simulated data with a known three-way artificial subcluster ground truth; it does not by itself validate performance on real data without ground truth.
- The README relies on deliberate over-clustering ('It's better if the Leiden resolution is high, so that there is alot of over-clustering'), so low-resolution starting labels may undermine merge evaluation.
- No repository changelog and no discussion-section content were available, so reproducibility gaps and limitations from the authors' own discussion could not be extracted.

## Evidence

- [results] finding: simulated-data AUROC: [evidence span withheld — anchor status: not_found]
- [results] workflow: scoring approaches: [evidence span withheld — anchor status: not_found]
- [readme] expected input description: [evidence span withheld — anchor status: not_found]
- [readme] over-clustering rationale: "It's better if the Leiden resolution is high, so that there is alot of over-clustering. Cytocipher merges the non-significantly different clusters."
- [methods] merge_clusters usage: "cc.tl.merge_clusters(data, 'leiden')"
