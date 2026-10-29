---
name: over-clustered-annata-merging-benchmark
description: Use when a high-resolution Leiden-clustered scRNA-seq AnnData is intentionally
  over-clustered and you must quantify how well a merging method discriminates significant
  from non-significant artificial over-cluster pairs via AUROC.
when_to_use_negative:
- The input clustering is not over-clustered — Cytocipher assumes high-resolution
  tentative Leiden labels so that non-significantly different clusters exist to merge
- The task is de novo clustering rather than testing/merging of existing tentative
  cluster labels
- The dataset lacks known cell-type labels, so AUROC of merged clusters against ground
  truth cannot be computed
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: cytocipher
  role: Provides the significance-tested cluster merging (cc.tl.get_markers, cc.tl.code_enrich,
    cc.tl.merge_clusters) and diagnostics being benchmarked
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: Used for the standard preprocessing workflow (log-cpm normalisation) and high-resolution
    Leiden clustering that generates the over-clustered input
- name: AnnData
  role: Container format for the processed single-cell matrix with 'leiden' cluster
    labels consumed by Cytocipher
- name: Jupyter
  role: Notebook environment in which the benchmark workflow and Cytocipher documentation
    are used
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:a4370d980ef29aca9f4d0abcf6c04c5bf4a5569713696f3a7e45b2111585c295
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Jupyter
  - cytocipher
  - scanpy
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# over-clustered-annata-merging-benchmark

## Summary

Benchmark cluster-merging of deliberately over-clustered single-cell AnnData (e.g. the 483152-cell Tabula Sapiens Atlas with high-resolution Leiden labels) using Cytocipher's significance testing, scoring performance as AUROC against known cell-type labels, and comparing against Sc-SHC and Giotto-based scoring.

## When to use

Use when a high-resolution Leiden-clustered scRNA-seq AnnData is intentionally over-clustered and you must quantify how well a merging method discriminates significant from non-significant artificial over-cluster pairs via AUROC.

## When NOT to use

- The input clustering is not over-clustered — Cytocipher assumes high-resolution tentative Leiden labels so that non-significantly different clusters exist to merge
- The task is de novo clustering rather than testing/merging of existing tentative cluster labels
- The dataset lacks known cell-type labels, so AUROC of merged clusters against ground truth cannot be computed

## Inputs

- AnnData object (.h5ad) of 483152-cell Tabula Sapiens artificial over-clusters, log-cpm normalised via the scanpy standard workflow with high-resolution Leiden labels
- Known cell-type labels for AUROC computation against merged clusters
- Downsampled 7385-cell subset of the same data
- 500-cell, 12-cell-type subset for runtime benchmarking against Sc-SHC

## Outputs

- auroc_benchmark_results.csv with AUROC values per method/scoring variant and runtime comparisons
- Merged cluster labels ('leiden_merged') on the AnnData
- Diagnostic violin plots of enrichment-score distributions for significant vs non-significant cluster pairs (cc.pl.sig_cluster_diagnostics)

## How to apply

Start from log-cpm normalised AnnData produced with the scanpy standard workflow, using deliberately high Leiden resolution so clusters are over-clustered. Run cc.tl.get_markers(data, 'leiden') then cc.tl.code_enrich(data, 'leiden') to compute cell-by-cluster enrichment scores, followed by cc.tl.merge_clusters(data, 'leiden') under both 'code-scoring' and 'coexpr-scoring' significance testing. Compute AUROC of merged clusters against known cell-type labels: expect ~0.9974 (code-scoring) and ~0.9976 (coexpr-scoring) on the full 483152-cell Tabula Sapiens data, and ~0.9930/0.9937 after downsampling to 7385 cells, versus Giotto PAGE 0.9945 and Giotto (−) 0.9940. For the runtime contrast, time Sc-SHC against Cytocipher's merging on a 500-cell, 12-cell-type subset (reported <3.5 s vs 178.80 s). Use cc.pl.sig_cluster_diagnostics to inspect enrichment-score distributions for significant versus non-significant pairs as a QC diagnostic, and assemble AUROCs and runtimes into auroc_benchmark_results.csv.

## Related tools

- **cytocipher** (Provides the significance-tested cluster merging (cc.tl.get_markers, cc.tl.code_enrich, cc.tl.merge_clusters) and diagnostics being benchmarked) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (Used for the standard preprocessing workflow (log-cpm normalisation) and high-resolution Leiden clustering that generates the over-clustered input)
- **AnnData** (Container format for the processed single-cell matrix with 'leiden' cluster labels consumed by Cytocipher)
- **Jupyter** (Notebook environment in which the benchmark workflow and Cytocipher documentation are used)

## Examples

```
cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden'); cc.tl.merge_clusters(data, 'leiden'); cc.pl.sig_cluster_diagnostics(data, 'leiden', plot_pair=('3', '9'))
```

## Evaluation signals

- Full-scale Tabula Sapiens AUROC reproduces 0.9974 with 'code-scoring' and 0.9976 with 'coexpr-scoring'
- Downsampled (7385-cell) AUROCs match 0.9930 (code-scoring), 0.9937 (coexpr-scoring), 0.9945 (Giotto PAGE), and 0.9940 (Giotto (−))
- Runtime benchmark on the 500-cell 12-cell-type subset reproduces Sc-SHC <3.5 s versus Cytocipher 178.80 s
- Scanpy-based scoring should perform worse (AUROC <0.9), serving as a negative control
- auroc_benchmark_results.csv contains all AUROC values and runtimes, and cc.pl.sig_cluster_diagnostics shows separable enrichment-score distributions for significant vs non-significant pairs

## Limitations

- Cytocipher's merging pipeline is slow: 178.80 s on only a 500-cell, 12-cell-type subset, versus <3.5 s for Sc-SHC
- Competing Scanpy-based scoring approaches performed worse (AUROC <0.9), so scoring-variant choice materially affects results
- No changelog for the repository was found and discussion-section limitations/reproducibility gaps were absent from the extracted content

## Evidence

- [abstract] Cytocipher also scales to large datasets with high-test performance, as shown by application to the Tabula Sapiens Atlas representing >480 000 cells: "Cytocipher also scales to large datasets with high-test performance, as shown by application to the Tabula Sapiens Atlas representing >480 000 cells"
- [readme] An AnnData object, *data*, that has been processed similarly to the scanpy standard workflow: "An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow]"
- [readme] It's better if the Leiden resolution is high, so that there is alot of over-clustering. Cytocipher merges the non-significantly different clusters.: "It's better if the Leiden resolution is high,
 so that there is alot of over-clustering. 
 Cytocipher merges the non-significantly different clusters."
- [methods] Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores.: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
- [methods] to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering): "to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering)"
