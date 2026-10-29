---
name: downsampled-dataset-reproducibility
description: Use when validating that a cluster-significance/merging method retains
  test performance (AUROC) and acceptable runtime at reduced cell counts, e.g. reproducing
  Cytocipher's Tabula Sapiens 483152-cell benchmark and its 7385-cell downsample.
when_to_use_negative:
- The dataset lacks known cell-type labels, so AUROC of merged clusters against ground
  truth cannot be computed and the benchmark cannot be validated.
- The analysis goal is novel biological discovery (e.g. identifying new subpopulations)
  rather than benchmarking scalability/reproducibility of a published result.
- Cluster labels come from low Leiden resolution with no deliberate over-clustering,
  leaving nothing for Cytocipher to merge and making the AUROC benchmark ill-posed.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cytocipher
  role: Performs marker gene identification (cc.tl.get_markers), code enrichment scoring
    (cc.tl.code_enrich) and cluster merging (cc.tl.merge_clusters) under code-scoring
    and coexpr-scoring significance testing whose AUROCs are benchmarked.
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: Standard workflow used to preprocess the Tabula Sapiens AnnData into log-cpm
    normalised values with Leiden cluster labels before benchmarking.
  repo: https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- name: AnnData
  role: Container format of the input data object (data) consumed by the Cytocipher
    functions in the benchmark.
- name: Sc-SHC
  role: Competing significant-cluster-detection method timed against Cytocipher's
    merging pipeline on the 500-cell 12-cell-type subset (reported <3.5 s vs 178.80
    s).
- name: Giotto PAGE
  role: Competing method included in the 7385-cell downsampled AUROC comparison (reported
    0.9945).
- name: Giotto (−)
  role: Competing method included in the 7385-cell downsampled AUROC comparison (reported
    0.9940).
- name: Jupyter
  role: Notebook environment in which the Cytocipher functions and their documentation
    are run.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:22abcce2b78ee8f6046fc1c1acca1704910cf47ae49c692ed72fe113297ddc09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Cytocipher
  - Giotto (−)
  - Giotto PAGE
  - Jupyter
  - Sc-SHC
  - cytocipher
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# downsampled-dataset-reproducibility

## Summary

Reproduces the Cytocipher Tabula Sapiens scalability benchmark by running cluster merging (code-scoring and coexpr-scoring) on the 483152-cell artificial over-cluster AnnData and on a 7385-cell downsample, computing AUROCs of merged clusters against known cell-type labels, and timing Sc-SHC versus Cytocipher on a 500-cell 12-cell-type subset.

## When to use

Use when validating that a cluster-significance/merging method retains test performance (AUROC) and acceptable runtime at reduced cell counts, e.g. reproducing Cytocipher's Tabula Sapiens 483152-cell benchmark and its 7385-cell downsample.

## When NOT to use

- The dataset lacks known cell-type labels, so AUROC of merged clusters against ground truth cannot be computed and the benchmark cannot be validated.
- The analysis goal is novel biological discovery (e.g. identifying new subpopulations) rather than benchmarking scalability/reproducibility of a published result.
- Cluster labels come from low Leiden resolution with no deliberate over-clustering, leaving nothing for Cytocipher to merge and making the AUROC benchmark ill-posed.

## Inputs

- 483152-cell Tabula Sapiens artificial over-cluster AnnData (log-cpm normalised, high-resolution Leiden labels)
- Known ground-truth cell-type labels for AUROC computation
- 7385-cell downsampled subset of the same dataset
- 500-cell, 12-cell-type subset for runtime benchmarking

## Outputs

- auroc_benchmark_results.csv with AUROC values per scoring method and comparator (Giotto PAGE, Giotto (−)) at full and downsampled scale
- Runtime comparison of Sc-SHC vs Cytocipher merging on the 500-cell 12-cell-type subset
- cc.pl.sig_cluster_diagnostics violin plots of enrichment scores for significant vs non-significant cluster pairs (QC)

## How to apply

Load the 483152-cell Tabula Sapiens artificial over-cluster AnnData processed with the scanpy standard workflow to log-cpm normalised values with high-resolution Leiden labels (deliberate over-clustering is intentional so Cytocipher can merge non-significantly different clusters). Run cc.tl.get_markers(data, 'leiden') then cc.tl.code_enrich(data, 'leiden') to obtain cell-by-cluster enrichment scores, then cc.tl.merge_clusters(data, 'leiden') under both code-scoring and coexpr-scoring significance testing; compute AUROC of the merged clusters against the known cell-type labels and check 0.9974 (code-scoring) and 0.9976 (coexpr-scoring) at full scale. Downsample to 7385 cells and repeat under each scoring variant plus the competing methods Giotto PAGE and Giotto (−), expecting AUROCs 0.9930, 0.9937, 0.9945 and 0.9940 respectively. On a 500-cell, 12-cell-type subset, time Sc-SHC against Cytocipher's merging pipeline to verify the reported runtime contrast (<3.5 s vs 178.80 s). Assemble the AUROCs and runtimes into auroc_benchmark_results.csv and use cc.pl.sig_cluster_diagnostics to inspect enrichment-score distributions for significant versus non-significant cluster pairs as a QC diagnostic.

## Related tools

- **Cytocipher** (Performs marker gene identification (cc.tl.get_markers), code enrichment scoring (cc.tl.code_enrich) and cluster merging (cc.tl.merge_clusters) under code-scoring and coexpr-scoring significance testing whose AUROCs are benchmarked.) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (Standard workflow used to preprocess the Tabula Sapiens AnnData into log-cpm normalised values with Leiden cluster labels before benchmarking.) — https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- **AnnData** (Container format of the input data object (data) consumed by the Cytocipher functions in the benchmark.)
- **Sc-SHC** (Competing significant-cluster-detection method timed against Cytocipher's merging pipeline on the 500-cell 12-cell-type subset (reported <3.5 s vs 178.80 s).)
- **Giotto PAGE** (Competing method included in the 7385-cell downsampled AUROC comparison (reported 0.9945).)
- **Giotto (−)** (Competing method included in the 7385-cell downsampled AUROC comparison (reported 0.9940).)
- **Jupyter** (Notebook environment in which the Cytocipher functions and their documentation are run.)

## Examples

```
cc.tl.get_markers(data, 'leiden'); cc.tl.code_enrich(data, 'leiden'); cc.tl.merge_clusters(data, 'leiden')
```

## Evaluation signals

- Full-scale AUROC of merged clusters matches 0.9974 for code-scoring and 0.9976 for coexpr-scoring on the 483152-cell Tabula Sapiens over-clusters.
- Downsampled (7385-cell) AUROCs match 0.9930 (Cytocipher code-scoring), 0.9937 (coexpr-scoring), 0.9945 (Giotto PAGE) and 0.9940 (Giotto (−)).
- Runtime benchmark reproduces Sc-SHC <3.5 s versus Cytocipher 178.80 s on the 500-cell 12-cell-type subset.
- auroc_benchmark_results.csv is assembled containing all computed AUROC values and runtimes.
- cc.pl.sig_cluster_diagnostics shows visibly different enrichment-score distributions for significant versus non-significant cluster pairs.

## Limitations

- Cytocipher's merging pipeline is much slower than Sc-SHC (178.80 s vs <3.5 s on only 500 cells), so full-scale merging is computationally expensive.
- Scanpy-based scoring performed worse than Cytocipher's scoring on the Tabula Sapiens benchmark (AUROC <0.9), so scoring variant choice matters.
- The repository has no changelog and no discussion of limitations, so reproducibility caveats are under-documented.

## Evidence

- [abstract] Cytocipher also scales to large datasets with high-test performance, as shown by application to the Tabula Sapiens Atlas representing >480 000 cells: "Cytocipher also scales to large datasets with high-test performance, as shown by application to the Tabula Sapiens Atlas representing >480 000 cells"
- [methods] An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow]: "An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow]"
- [readme] to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering): "to produce log-cpm normalised data with tentative cluster labels (e.g. from Leiden clustering)"
- [readme] It's better if the Leiden resolution is high, so that there is alot of over-clustering.: "It's better if the Leiden resolution is high, so that there is alot of over-clustering. Cytocipher merges the non-significantly different clusters."
- [methods] Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores.: "Functions below run the marker gene identification, code scoring, & subsequent visualisation of the resulting cell by cluster enrichment scores."
