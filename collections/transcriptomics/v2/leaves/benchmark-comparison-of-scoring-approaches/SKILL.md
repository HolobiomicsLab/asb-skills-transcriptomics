---
name: benchmark-comparison-of-scoring-approaches
description: Use when evaluating whether Cytocipher's cluster-merge can discriminate
  significant vs non-significant cluster pairs on simulated over-clustered scRNA-seq
  data, or when comparing scoring methods via AUROC against known subcluster labels.
when_to_use_negative:
- Input is real experimental data without a known subcluster ground truth — ROC/AUROC
  benchmarking requires artificial labels of three subclusters per group
- Goal is routine cluster annotation or enrichment visualisation rather than method
  benchmarking — use cc.pl.enrich_heatmap or cc.pl.sig_cluster_diagnostics instead
  of the full six-method comparison
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cytocipher
  role: Runs marker identification (cc.tl.get_markers), enrichment scoring (cc.tl.code_enrich),
    and cluster merging (cc.tl.merge_clusters) per scoring approach
  repo: https://github.com/BradBalderson/Cytocipher
- name: scanpy
  role: Scanpy-score is one of the enrichment scoring approaches benchmarked; input
    data processed via the scanpy standard workflow to log-cpm normalised counts with
    Leiden labels
- name: AnnData
  role: Data container for the simulated log-cpm normalised counts with ground-truth
    subcluster labels and 'leiden' column
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:3d921e8ae799e4f9aca7f8d41c79761604a34bdab346d9396469c42c0c92bff9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Cytocipher
  - cytocipher
  - scanpy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# benchmark-comparison-of-scoring-approaches

## Summary

Benchmark task comparing multiple enrichment scoring approaches (code-scoring, coexpr-scoring, Giotto PAGE, Scanpy-score, and their negative variants) for Cytocipher cluster-merge discrimination on simulated scRNA-seq data, judged by ROC/AUROC against a three-way artificial subcluster ground truth.

## When to use

Use when evaluating whether Cytocipher's cluster-merge can discriminate significant vs non-significant cluster pairs on simulated over-clustered scRNA-seq data, or when comparing scoring methods via AUROC against known subcluster labels.

## When NOT to use

- Input is real experimental data without a known subcluster ground truth — ROC/AUROC benchmarking requires artificial labels of three subclusters per group
- Goal is routine cluster annotation or enrichment visualisation rather than method benchmarking — use cc.pl.enrich_heatmap or cc.pl.sig_cluster_diagnostics instead of the full six-method comparison

## Inputs

- Simulated AnnData object with log-cpm normalised counts
- Over-clustered Leiden cluster labels (column 'leiden')
- Ground-truth labels of three artificial subclusters per group
- List of scoring approaches to compare: code-scoring, coexpr-scoring, Giotto PAGE, Scanpy-score, Giotto (−), Scanpy (−)

## Outputs

- ROC curves per scoring method
- Per-method AUROC values written to auroc_results.csv
- Merged cluster assignments ('leiden_merged') per scoring approach

## How to apply

Load the simulated AnnData holding log-cpm normalised counts with ground-truth labels of three artificial subclusters per group, where groups are defined by unique gene combinations. For each of six scoring approaches (code-scoring, coexpr-scoring, Giotto PAGE, Scanpy-score, Giotto (−), Scanpy (−)), run marker identification with cc.tl.get_markers(data, 'leiden') on the over-clustered Leiden labels, compute cell-by-cluster enrichment scores with cc.tl.code_enrich(data, 'leiden'), and merge non-significantly different clusters with cc.tl.merge_clusters(data, 'leiden'). Compare merged assignments against the three-way ground truth to derive true/false positive rates across score thresholds, then compute ROC curves and per-method AUROC. The success criterion from the source task is that AUROC exceeds 0.99 for every scoring approach, demonstrating the result is robust to the choice of enrichment scoring method.

## Related tools

- **Cytocipher** (Runs marker identification (cc.tl.get_markers), enrichment scoring (cc.tl.code_enrich), and cluster merging (cc.tl.merge_clusters) per scoring approach) — https://github.com/BradBalderson/Cytocipher
- **scanpy** (Scanpy-score is one of the enrichment scoring approaches benchmarked; input data processed via the scanpy standard workflow to log-cpm normalised counts with Leiden labels)
- **AnnData** (Data container for the simulated log-cpm normalised counts with ground-truth subcluster labels and 'leiden' column)

## Examples

```
cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden'); cc.tl.merge_clusters(data, 'leiden')
```

## Evaluation signals

- AUROC exceeds 0.99 for all six scoring approaches (code-scoring, coexpr-scoring, Giotto PAGE, Scanpy-score, Giotto (−), Scanpy (−))
- auroc_results.csv is written containing per-method AUROC values
- Merged cluster assignments recover the three-way artificial subcluster ground truth per group
- Marker identification, enrichment scoring, and merging are all run on the same over-clustered 'leiden' labels for every scoring approach, ensuring a fair comparison
- True/false positive rates are derived by sweeping score thresholds against the ground-truth labels, yielding valid ROC curves

## Limitations

- Validation is on simulated data where groups are defined by unique gene combinations; performance on real datasets with subtler structure may differ
- Requires deliberate over-clustering: the README notes 'It's better if the Leiden resolution is high, so that there is alot of over-clustering' — under-clustering leaves nothing for cluster-merge to discriminate
- No changelog available for the repository, and discussion-section limitations are absent, so version-specific caveats for the benchmark are not documented

## Evidence

- [results] task card workflow: [evidence span withheld — anchor status: not_found]
- [results] task card finding: [evidence span withheld — anchor status: not_found]
- [readme] README input expectation: [evidence span withheld — anchor status: not_found]
- [methods] methods pipeline functions: "cc.tl.get_markers(data, 'leiden')
cc.tl.code_enrich(data, 'leiden')
cc.pl.enrich_heatmap(data, 'leiden')"
- [methods] methods merging: "cc.tl.merge_clusters(data, 'leiden')
cc.pl.enrich_heatmap(data, 'leiden_merged')"
