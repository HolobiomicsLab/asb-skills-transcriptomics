---
name: simulated-scrnaseq-benchmark-reproduction
description: Use when a simulated scRNA-seq AnnData with high-resolution Leiden labels
  must be used to benchmark enrichment-scoring methods on per-cluster binary classification,
  or to verify that negative-gene-set-subtraction variants outperform standard scoring.
when_to_use_negative:
- The dataset is real experimental scRNA-seq without ground-truth cluster labels,
  so per-cluster binary classification accuracy cannot be computed against known assignments.
- The goal is Cytocipher's primary use case of merging non-significantly different
  clusters (cc.tl.merge_clusters) rather than benchmarking scoring methods.
- Clusters in the input are defined by single marker genes rather than unique gene
  combinations, which is the regime the simulated benchmark was designed to test.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: cytocipher
  role: Provides cc.tl.get_markers and cc.tl.code_enrich for marker identification
    and code scoring in the benchmark
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: Standard workflow preprocessing (log-cpm normalisation, Leiden clustering)
    and the score_genes baseline scorer
- name: AnnData
  role: Container format for the processed simulated dataset passed to the scoring
    functions
- name: Jupyter
  role: Notebook environment for running the benchmark and inspecting function documentation
    (e.g. ?cc.tl.get_markers)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:a948d478db260154a43ff0931bff686e3ba77ccbaede39cb7b6e75d237bf6c65
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# simulated-scrnaseq-benchmark-reproduction

## Summary

Reproduces the Cytocipher paper's Fig. 2D benchmark: scoring 3000 simulated scRNA-seq cells (clusters defined by unique gene combinations) against their Leiden marker sets with six enrichment methods and comparing per-cluster binary classification accuracy, precision, recall, and F1.

## When to use

Use when a simulated scRNA-seq AnnData with high-resolution Leiden labels must be used to benchmark enrichment-scoring methods on per-cluster binary classification, or to verify that negative-gene-set-subtraction variants outperform standard scoring.

## When NOT to use

- The dataset is real experimental scRNA-seq without ground-truth cluster labels, so per-cluster binary classification accuracy cannot be computed against known assignments.
- The goal is Cytocipher's primary use case of merging non-significantly different clusters (cc.tl.merge_clusters) rather than benchmarking scoring methods.
- Clusters in the input are defined by single marker genes rather than unique gene combinations, which is the regime the simulated benchmark was designed to test.

## Inputs

- Simulated 3000-cell scRNA-seq dataset as an AnnData object (log-cpm normalised, scanpy standard workflow)
- High-resolution Leiden cluster labels column (e.g. 'leiden')
- Per-cluster marker gene sets from cc.tl.get_markers(data, 'leiden')

## Outputs

- Cell-by-cluster enrichment score matrices from six scoring methods (code-scoring, coexpr-scoring, Giotto PAGE, Scanpy-scoring, Giotto (−), Scanpy (−))
- Per-cluster binary cell-to-cluster assignments per method
- method_benchmark_metrics.csv containing accuracy, precision, recall, and F1 for all six methods

## How to apply

Load the simulated 3000-cell dataset as an AnnData object processed per the scanpy standard workflow (log-cpm normalised, high-resolution Leiden cluster labels, since Cytocipher assumes over-clustering). Derive per-cluster marker sets with cc.tl.get_markers(data, 'leiden'), then score every cell against each cluster's markers using all six methods: cc.tl.code_enrich (Cytocipher code scoring), coexpr-scoring, Giotto PAGE, Scanpy score_genes, and the negative-gene-set-subtraction variants Giotto (−) and Scanpy (−). Convert each method's cell-by-cluster enrichment score matrix into per-cluster binary cell-to-cluster assignments, then compute accuracy, precision, recall, and F1 per cluster per method. Assemble the metrics into method_benchmark_metrics.csv and check the ranking reproduces the reported order: Giotto (−) > code-scoring > Scanpy (−) > Giotto PAGE > Scanpy-scoring > coexpr-scoring, with negative-subtraction methods outperforming except on recall.

## Related tools

- **cytocipher** (Provides cc.tl.get_markers and cc.tl.code_enrich for marker identification and code scoring in the benchmark) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (Standard workflow preprocessing (log-cpm normalisation, Leiden clustering) and the score_genes baseline scorer)
- **AnnData** (Container format for the processed simulated dataset passed to the scoring functions)
- **Jupyter** (Notebook environment for running the benchmark and inspecting function documentation (e.g. ?cc.tl.get_markers))

## Examples

```
cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden')
```

## Evaluation signals

- method_benchmark_metrics.csv contains all six methods and all four metrics (accuracy, precision, recall, F1) per cluster
- Method ranking matches the reported pattern: Giotto (−), code-scoring, Scanpy (−), Giotto PAGE, Scanpy-scoring, coexpr-scoring from greatest to lowest performance
- Negative-gene-set-subtraction methods (Giotto (−), Scanpy (−)) outperform their standard variants on all metrics except recall
- Input AnnData is log-cpm normalised with high-resolution (over-clustered) Leiden labels, per the expected-input convention
- Every cell receives a score against every cluster for each of the six methods before conversion to binary assignments

## Limitations

- The recall metric is the reported exception where the negative-gene-set-subtraction methods do not outperform the standard variants.
- The benchmark depends on the simulated clusters being defined by unique gene combinations rather than single marker genes; results may not transfer to marker-gene-defined clusters.
- Cytocipher expects high Leiden resolution with deliberate over-clustering; using low-resolution labels changes the marker sets and downstream scores.

## Evidence

- [results] On the simulated-data binary classification task, methods ranked from greatest to lowest performance as Giotto (−), ‘code-scoring’, Scanpy (−), ‘Giotto PAGE’, ‘Scanpy-scoring’, and then ‘coexpr-scoring’: [evidence span withheld — anchor status: not_found]
- [readme] An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow]: "An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow](https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html) to produce log-cpm normalised data with"
- [readme] Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores.: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
- [readme] It's better if the Leiden resolution is high, so that there is alot of over-clustering. Cytocipher merges the non-significantly different clusters.: "It's better if the Leiden resolution is high, so that there is alot of over-clustering."
