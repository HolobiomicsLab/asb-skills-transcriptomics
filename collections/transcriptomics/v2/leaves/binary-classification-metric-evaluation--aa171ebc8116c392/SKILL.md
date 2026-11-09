---
name: binary-classification-metric-evaluation
description: Use when you have a simulated or labelled scRNA-seq AnnData object with
  known per-cluster memberships and need to rank enrichment scoring methods by binary
  classification performance (accuracy, precision, recall, F1) per cluster.
when_to_use_negative:
- The input is raw count data that has not been log-cpm normalised and Leiden-clustered
  — Cytocipher expects an AnnData object processed per the scanpy standard workflow
  with tentative cluster labels.
- The goal is significance testing / cluster merging (cc.tl.merge_clusters) rather
  than benchmarking scoring methods against known labels.
- Clusters in the real dataset of interest are not defined by unique gene combinations
  — the reported method ranking was established on simulated data where clusters are
  defined by gene combinations rather than single marker genes.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: cytocipher
  role: Provides cc.tl.get_markers and cc.tl.code_enrich for deriving per-cluster
    marker sets and the code-scoring enrichment method under benchmark
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: Standard workflow for log-cpm normalisation and Leiden clustering of the AnnData
    input, plus the Scanpy and Scanpy (−) scoring variants being benchmarked
- name: AnnData
  role: Input data container holding the log-cpm normalised simulated dataset and
    Leiden cluster labels
- name: Jupyter
  role: Notebook environment for running the scoring, assignment, and metric computations
    (e.g. ?cc.tl.get_markers for documentation)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:12afa898079db83c1b48c943e2d92ba70d00aff6d448d9a61ebfd4ce44ca51fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Jupyter
  - cytocipher
  - scanpy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# binary-classification-metric-evaluation

## Summary

Benchmarks cell-by-cluster enrichment scoring methods (Cytocipher code scoring, coexpr scoring, Giotto PAGE, Scanpy score_genes, and negative-gene-set-subtraction variants Giotto (−) and Scanpy (−)) by converting enrichment scores into per-cluster binary cell-to-cluster assignments and comparing accuracy, precision, recall, and F1. It reproduces the simulated-data ranking from the Cytocipher paper's Fig. 2D benchmark.

## When to use

Use when you have a simulated or labelled scRNA-seq AnnData object with known per-cluster memberships and need to rank enrichment scoring methods by binary classification performance (accuracy, precision, recall, F1) per cluster.

## When NOT to use

- The input is raw count data that has not been log-cpm normalised and Leiden-clustered — Cytocipher expects an AnnData object processed per the scanpy standard workflow with tentative cluster labels.
- The goal is significance testing / cluster merging (cc.tl.merge_clusters) rather than benchmarking scoring methods against known labels.
- Clusters in the real dataset of interest are not defined by unique gene combinations — the reported method ranking was established on simulated data where clusters are defined by gene combinations rather than single marker genes.

## Inputs

- Simulated 3000-cell scRNA-seq dataset as an AnnData object, log-cpm normalised with high-resolution Leiden cluster labels
- Per-cluster marker gene sets derived via cc.tl.get_markers(data, 'leiden')
- Ground-truth cluster memberships for the binary classification task

## Outputs

- Cell-by-cluster enrichment score matrices for all six scoring methods
- Per-cluster binary cell-to-cluster assignments for each method
- method_benchmark_metrics.csv containing accuracy, precision, recall, and F1 for all six methods

## How to apply

Load the simulated 3000-cell dataset as an AnnData object processed per the scanpy standard workflow to log-cpm normalised values with high-resolution Leiden cluster labels, then derive per-cluster marker gene sets with cc.tl.get_markers(data, 'leiden'). Score every cell against each cluster's marker set using all six methods: cc.tl.code_enrich (Cytocipher code scoring), coexpr scoring, Giotto PAGE, Scanpy score_genes, and the negative-gene-set-subtraction variants Giotto (−) and Scanpy (−), which subtract scores from non-target cluster gene sets. Convert each method's cell-by-cluster enrichment score matrix into per-cluster binary cell-to-cluster assignments, then compute accuracy, precision, recall, and F1 for each method-cluster pair. Assemble the four metrics for all six methods into method_benchmark_metrics.csv and confirm the ranking reproduces the reported pattern: negative-gene-set-subtraction methods and code-scoring outperform the standard variants (except on recall), with Giotto (−) best and coexpr-scoring worst.

## Related tools

- **cytocipher** (Provides cc.tl.get_markers and cc.tl.code_enrich for deriving per-cluster marker sets and the code-scoring enrichment method under benchmark) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (Standard workflow for log-cpm normalisation and Leiden clustering of the AnnData input, plus the Scanpy and Scanpy (−) scoring variants being benchmarked)
- **AnnData** (Input data container holding the log-cpm normalised simulated dataset and Leiden cluster labels)
- **Jupyter** (Notebook environment for running the scoring, assignment, and metric computations (e.g. ?cc.tl.get_markers for documentation))

## Examples

```
cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden')
```

## Evaluation signals

- method_benchmark_metrics.csv contains all six methods (Giotto (−), code-scoring, Scanpy (−), Giotto PAGE, Scanpy-scoring, coexpr-scoring) and all four metrics per cluster
- The method ranking matches the reported pattern: Giotto (−) > code-scoring > Scanpy (−) > Giotto PAGE > Scanpy-scoring > coexpr-scoring, with negative-gene-set-subtraction methods outperforming except on the recall metric
- Every cell in the 3000-cell simulated dataset receives a score against every cluster's marker set for each of the six methods before conversion to binary assignments
- Accuracy, precision, recall, and F1 values are all computable per cluster, i.e. binary assignments are non-degenerate for each method

## Limitations

- The ranking was established on simulated data where clusters are defined by unique gene combinations rather than single marker genes; it may not transfer to datasets with different cluster structure.
- The negative-gene-set-subtraction methods outperform the standard variants except on the recall metric, so method choice depends on which metric matters most.
- No changelog found for the Cytocipher repository, limiting versioned reproducibility checks.

## Evidence

- [results] On the simulated-data binary classification task, methods ranked from greatest to lowest performance as Giotto (−), ‘code-scoring’, Scanpy (−), ‘Giotto PAGE’, ‘Scanpy-scoring’, and then ‘coexpr-scoring’: [evidence span withheld — anchor status: not_found]
- [readme] Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores.: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
- [methods] An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow]: "An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow]"
- [methods] to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering): "to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering)"
