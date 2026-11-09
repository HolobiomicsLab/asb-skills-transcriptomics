---
name: negative-gene-set-subtraction-scoring
description: Use when benchmarking gene-set enrichment scoring methods on scRNA-seq
  data where clusters are defined by unique gene combinations rather than single marker
  genes, and standard scoring underperforms on precision/accuracy.
when_to_use_negative:
- When the analysis goal is statistical significance testing or merging of clusters
  (cc.tl.merge_clusters / sig_cluster_diagnostics) rather than benchmarking enrichment
  scoring approaches
- When the input is not a log-cpm normalised AnnData with tentative cluster labels
  — marker derivation and code scoring expect the scanpy standard workflow output
tools:
- name: cytocipher
  role: Provides get_markers and code_enrich for marker gene identification and cell-by-cluster
    enrichment scoring; the baseline 'code-scoring' method in the six-method benchmark
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: Supplies the standard workflow preprocessing (log-cpm normalisation, Leiden
    clustering) and the score_genes-based 'Scanpy (−)' negative gene set subtraction
    variant
  repo: https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- name: AnnData
  role: Input object format processed per the scanpy standard workflow
- name: Jupyter
  role: Environment for running the scoring and benchmarking (documentation accessed
    via ?cc.tl.get_markers)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:e670d1558245c258f583238aa7c0208865ade801f2d4eeaded293ced6755e4a0
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# negative-gene-set-subtraction-scoring

## Summary

A cell-scoring strategy from the Cytocipher enrichment-scoring benchmark in which a cell's enrichment for one cluster's marker gene set is computed while subtracting the negative (counter-) evidence from other clusters' marker sets, implemented as the 'Scanpy (−)' and 'Giotto (−)' variants. These subtraction approaches were the top performers on the simulated-data per-cluster binary classification task.

## When to use

Use when benchmarking gene-set enrichment scoring methods on scRNA-seq data where clusters are defined by unique gene combinations rather than single marker genes, and standard scoring underperforms on precision/accuracy.

## When NOT to use

- When the analysis goal is statistical significance testing or merging of clusters (cc.tl.merge_clusters / sig_cluster_diagnostics) rather than benchmarking enrichment scoring approaches
- When the input is not a log-cpm normalised AnnData with tentative cluster labels — marker derivation and code scoring expect the scanpy standard workflow output

## Inputs

- AnnData object with log-cpm normalised scRNA-seq counts and tentative high-resolution Leiden cluster labels (e.g. column 'leiden')
- Per-cluster marker gene sets derived via cc.tl.get_markers(data, 'leiden')

## Outputs

- Cell-by-cluster enrichment score matrix per scoring method (Scanpy (−), Giotto (−))
- Per-cluster binary cell-to-cluster assignments per method
- Benchmark metrics table (accuracy, precision, recall, F1 per cluster per method, e.g. method_benchmark_metrics.csv)

## How to apply

Start from an AnnData object processed per the scanpy standard workflow to log-cpm normalisation, with tentative Leiden cluster labels at high resolution so there is over-clustering. Derive per-cluster marker gene sets with cc.tl.get_markers(data, 'leiden'), then compute cell-by-cluster enrichment scores with each variant: the subtraction variants Scanpy (−) and Giotto (−) (negative gene set subtraction), alongside cc.tl.code_enrich (code-scoring), Giotto PAGE, standard Scanpy score_genes, and coexpr-scoring. Convert each method's cell-by-cluster score matrix into per-cluster binary cell-to-cluster assignments and compute accuracy, precision, recall, and F1 per cluster. The expected outcome (Fig. 2D benchmark) is that negative-subtraction methods rank highest — Giotto (−), 'code-scoring', Scanpy (−), 'Giotto PAGE', 'Scanpy-scoring', then 'coexpr-scoring' — with the subtraction methods outperforming standard variants except on recall.

## Related tools

- **cytocipher** (Provides get_markers and code_enrich for marker gene identification and cell-by-cluster enrichment scoring; the baseline 'code-scoring' method in the six-method benchmark) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (Supplies the standard workflow preprocessing (log-cpm normalisation, Leiden clustering) and the score_genes-based 'Scanpy (−)' negative gene set subtraction variant) — https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- **AnnData** (Input object format processed per the scanpy standard workflow)
- **Jupyter** (Environment for running the scoring and benchmarking (documentation accessed via ?cc.tl.get_markers))

## Examples

```
cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden')
```

## Evaluation signals

- The assembled method_benchmark_metrics.csv contains all six methods × four metrics (accuracy, precision, recall, F1) per cluster
- The method ranking reproduces the reported pattern: Giotto (−) > 'code-scoring' > Scanpy (−) > 'Giotto PAGE' > 'Scanpy-scoring' > 'coexpr-scoring'
- Negative-subtraction methods outperform the standard scoring variants on accuracy/precision but NOT on the recall metric
- Input AnnData is log-cpm normalised with Leiden labels and marker sets were obtained from cc.tl.get_markers(data, 'leiden')

## Limitations

- The superiority of negative gene set subtraction methods does not extend to recall, where standard variants are not outperformed
- Subtraction variants were validated on a simulated 3000-cell dataset where clusters are defined by unique gene combinations rather than single marker genes; ranking may not generalise to marker-defined clusters
- The paper's discussion/limitations content was absent from the extracted text, so further caveats could not be traced

## Evidence

- [readme] An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow](https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html): "An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow]"
- [methods] to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering): "to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering)"
- [readme] Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores.: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
- [readme] It's better if the Leiden resolution is high, so that there is alot of over-clustering.: "It's better if the Leiden resolution is high, so that there is alot of over-clustering."
