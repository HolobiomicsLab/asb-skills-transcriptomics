---
name: auroc-cluster-recovery-evaluation
description: 'Use when quantifying whether an over-clustered scRNA-seq clustering
  can be recovered by merging: compute AUROC of merged clusters against known cell-type
  labels, e.g. Tabula Sapiens artificial over-clusters (AUROC 0.9974/0.9976).'
when_to_use_negative:
- When cluster labels are already final, biologically validated annotations rather
  than deliberately over-clustered Leiden labels — there is nothing to merge and AUROC
  of merge recovery is undefined
- When no ground-truth cell-type labels exist, since the AUROC of merged clusters
  against known labels cannot be computed
- When the goal is discovering new significantly different cell populations in a dataset
  rather than benchmarking scalability/accuracy of cluster merging
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: cytocipher
  role: Provides cc.tl.get_markers, cc.tl.code_enrich, cc.tl.merge_clusters and cc.pl.sig_cluster_diagnostics
    to score enrichment and merge over-clustered Leiden labels being benchmarked
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: Standard workflow used to preprocess the AnnData to log-cpm normalised values
    and produce the high-resolution Leiden over-clustering
  repo: https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- name: AnnData
  role: Input container format holding the log-cpm normalised expression matrix and
    leiden/leiden_merged label slots
- name: Jupyter
  role: Notebook environment for running the benchmark and inspecting diagnostics
    (e.g. ?cc.tl.get_markers)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:8058c9b9a6e60e243eae858c249bce77c13a612bfc747b1b1a49e39a063d97ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Jupyter
  - cytocipher
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# auroc-cluster-recovery-evaluation

## Summary

Benchmark skill for quantifying cluster-merge recovery: on the 483152-cell Tabula Sapiens Atlas artificial over-clusters, AUROC is computed for merged Cytocipher clusters against known cell-type labels, comparing 'code-scoring' and 'coexpr-scoring' against Scanpy-based scoring, Giotto PAGE, Giotto (−) and Sc-SHC runtimes.

## When to use

Use when quantifying whether an over-clustered scRNA-seq clustering can be recovered by merging: compute AUROC of merged clusters against known cell-type labels, e.g. Tabula Sapiens artificial over-clusters (AUROC 0.9974/0.9976).

## When NOT to use

- When cluster labels are already final, biologically validated annotations rather than deliberately over-clustered Leiden labels — there is nothing to merge and AUROC of merge recovery is undefined
- When no ground-truth cell-type labels exist, since the AUROC of merged clusters against known labels cannot be computed
- When the goal is discovering new significantly different cell populations in a dataset rather than benchmarking scalability/accuracy of cluster merging

## Inputs

- AnnData object processed with the scanpy standard workflow to log-cpm normalised values with high-resolution Leiden labels (deliberately over-clustered)
- 483152-cell Tabula Sapiens artificial over-cluster AnnData
- Downsampled 7385-cell Tabula Sapiens over-cluster dataset
- 500-cell, 12-cell-type subset for runtime benchmarking
- Known cell-type ground-truth labels for AUROC computation

## Outputs

- AUROC values for merged clusters vs known cell-type labels (0.9974 code-scoring, 0.9976 coexpr-scoring full-scale; 0.9930/0.9937/0.9945/0.9940 downsampled)
- auroc_benchmark_results.csv assembling all AUROC values and runtimes
- Runtime comparison of Sc-SHC (<3.5 s) vs Cytocipher merging (178.80 s) on the 500-cell 12-cell-type subset
- Enrichment-score diagnostic violin plots for significant vs non-significant cluster pairs

## How to apply

Process the data with the scanpy standard workflow to log-cpm normalised values with high-resolution Leiden labels so clusters are deliberately over-clustered. Run cc.tl.get_markers(data, 'leiden') followed by cc.tl.code_enrich(data, 'leiden') to obtain cell-by-cluster enrichment scores, then cc.tl.merge_clusters(data, 'leiden') under both code-scoring and coexpr-scoring significance testing. Compute the AUROC of merged clusters against the known cell-type labels and verify the reference values: 0.9974 (code-scoring) and 0.9976 (coexpr-scoring) at full scale on 483152 cells, and 0.9930, 0.9937, 0.9945, 0.9940 for Cytocipher code-scoring, Cytocipher coexpr-scoring, Giotto PAGE and Giotto (−) on the 7385-cell downsample. On a 500-cell, 12-cell-type subset, time Sc-SHC against the Cytocipher merging pipeline and reproduce the runtime contrast (<3.5 s vs 178.80 s). Assemble all AUROC values and runtimes into auroc_benchmark_results.csv and use cc.pl.sig_cluster_diagnostics to inspect enrichment-score distributions of significant vs non-significant cluster pairs as a QC diagnostic.

## Related tools

- **cytocipher** (Provides cc.tl.get_markers, cc.tl.code_enrich, cc.tl.merge_clusters and cc.pl.sig_cluster_diagnostics to score enrichment and merge over-clustered Leiden labels being benchmarked) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (Standard workflow used to preprocess the AnnData to log-cpm normalised values and produce the high-resolution Leiden over-clustering) — https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- **AnnData** (Input container format holding the log-cpm normalised expression matrix and leiden/leiden_merged label slots)
- **Jupyter** (Notebook environment for running the benchmark and inspecting diagnostics (e.g. ?cc.tl.get_markers))

## Examples

```
cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden'); cc.tl.merge_clusters(data, 'leiden')
```

## Evaluation signals

- Full-scale Tabula Sapiens AUROC must match 0.9974 (code-scoring) and 0.9976 (coexpr-scoring)
- Downsampled (7385-cell) AUROCs must match 0.9930 (Cytocipher code-scoring), 0.9937 (coexpr-scoring), 0.9945 (Giotto PAGE) and 0.9940 (Giotto (−))
- Runtime benchmark must reproduce Sc-SHC <3.5 s versus Cytocipher 178.80 s on the 500-cell 12-cell-type subset
- Scanpy-based scoring should perform worse, with AUROC <0.9
- auroc_benchmark_results.csv is produced containing all AUROC values and runtimes

## Limitations

- Cytocipher's merging pipeline is slow: 178.80 s on a 500-cell 12-cell-type subset versus Sc-SHC at <3.5 s, so it does not scale in runtime as favourably as Sc-SHC
- Requires high-resolution (deliberately over-clustered) Leiden labels and ground-truth cell-type labels to be meaningful
- The article's discussion section content is absent and the repository has no changelog, so reproducibility limitations must be probed empirically

## Evidence

- [abstract] we developed Cytocipher, a bioinformatics method and scverse compatible software package that statistically determines significant clusters: "we developed Cytocipher, a bioinformatics method and scverse compatible software package that statistically determines significant clusters"
- [abstract] Cytocipher also scales to large datasets with high-test performance, as shown by application to the Tabula Sapiens Atlas representing >480 000 cells: "Cytocipher also scales to large datasets with high-test performance, as shown by application to the Tabula Sapiens Atlas representing >480 000 cells"
- [methods] An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow]: "An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow]"
- [readme] It's better if the Leiden resolution is high, so that there is alot of over-clustering.: "It's better if the Leiden resolution is high, so that there is alot of over-clustering."
- [readme] Below runs the cluster merging and visualises the heatmap of enrichment scores per cell for each of the new merged clusters.: "Below runs the cluster merging and visualises the heatmap of enrichment scores per cell for each of the new merged clusters."
