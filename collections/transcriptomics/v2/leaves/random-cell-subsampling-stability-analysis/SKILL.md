---
name: random-cell-subsampling-stability-analysis
description: Use when you must quantify how stable an unsupervised cell clustering
  (e.g., SCASL spectral clustering on AS probabilities) is to sparsity, forced dropout,
  or cell sub-sampling, with a comparator clustering on gene expression.
when_to_use_negative:
- The goal is discovering new clusters rather than benchmarking stability of an existing
  clustering — run SCASL normally on the full unperturbed matrix instead
- The clustering input is already a complete, dense feature table without missing
  values — dropout robustness testing adds little information
- Only one perturbation level is feasible and no comparator method (e.g., gene-expression
  clustering) can be run — the skill's core output is the SCASL-vs-expression entropy
  comparison
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SCASL
  role: Spectral clustering on imputed single-cell AS probability matrices under forced-dropout
    and cell sub-sampling perturbations to measure cluster stability
  repo: https://github.com/xryanglab/SCASL
- name: Leafcutter
  role: Junction-read-based AS event identification upstream of SCASL clustering,
    installed via conda (r-leafcutter) or from source
  repo: https://github.com/davidaknowles/leafcutter
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:7b673d97a9b6e7f9e53ad83e4aa414b63c5072e035f3b5b48ff08f683420567d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xryanglab/SCASL
  tools:
  - Leafcutter
  - SCASL
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# random-cell-subsampling-stability-analysis

## Summary

Stress-tests cell cluster definitions by forcing 50% AS-value dropout and randomly retaining 100%, 75%, and 50% of cells, then comparing normalized entropy of re-clustered labels against the reference clustering. Applied to demonstrate that SCASL clusters from single-cell AS profiles are more robust and stable than Seurat-style gene-expression clusters.

## When to use

Use when you must quantify how stable an unsupervised cell clustering (e.g., SCASL spectral clustering on AS probabilities) is to sparsity, forced dropout, or cell sub-sampling, with a comparator clustering on gene expression.

## When NOT to use

- The goal is discovering new clusters rather than benchmarking stability of an existing clustering — run SCASL normally on the full unperturbed matrix instead
- The clustering input is already a complete, dense feature table without missing values — dropout robustness testing adds little information
- Only one perturbation level is feasible and no comparator method (e.g., gene-expression clustering) can be run — the skill's core output is the SCASL-vs-expression entropy comparison

## Inputs

- Junction/intermediate files (TNBC-2 demo) in `data/junction`, e.g. junc_matrix.csv / normalized_matrix (rows = sites, columns = cells)
- Reference label file with run_column (cell IDs matching bam files) and truth_column (known cell labels)
- SCASL YAML configuration (`configs/srr.yaml`) with sites_initial, runs_initial, sites_quality, runs_quality, num_iteration, knn, num_cluster
- Gene expression matrix for the same cells as the comparator clustering input

## Outputs

- Per-condition cluster label files for reference, 50% forced-dropout, and 100%/75%/50% sub-sampled runs (SCASL and expression comparator)
- stability_entropy.csv containing normalized entropy of each perturbed clustering versus reference labels for both SCASL and expression-based methods

## How to apply

Load the demo junction/intermediate files (TNBC-2) from `data/junction` and reference labels via `use_label`/`label_file` in `configs/srr.yaml`, then run SCASL (`python main.py -y configs/srr.yaml`) with `cluster_algorithm: 'spectral'` and weighted KNN imputation (`num_iteration`, `knn`) on the unperturbed matrix to establish reference labels. Generate perturbed AS matrices by masking 50% of AS values (forced dropout) and by randomly retaining 100%, 75%, and 50% of cells (sub-sampling), keeping `num_cluster` and the QC thresholds `sites_initial`, `runs_initial`, `sites_quality`, `runs_quality` fixed across all re-runs. Re-cluster each perturbed and full dataset with a canonical gene-expression clustering pipeline (Seurat-style) on the expression matrix as the comparator. Compute normalized entropy between each perturbed clustering's labels and the reference labels for both methods, and write the per-condition labels plus the SCASL-vs-expression entropy table to stability_entropy.csv. The rationale is that SCASL's iterative weighted KNN imputation plus NA-indicator-aware spectral clustering resolves biased and sparse single-cell AS coverage, so its clusters should degrade less under perturbation than expression-based clusters.

## Related tools

- **SCASL** (Spectral clustering on imputed single-cell AS probability matrices under forced-dropout and cell sub-sampling perturbations to measure cluster stability) — https://github.com/xryanglab/SCASL
- **Leafcutter** (Junction-read-based AS event identification upstream of SCASL clustering, installed via conda (r-leafcutter) or from source) — https://github.com/davidaknowles/leafcutter

## Examples

```
python main.py -y configs/srr.yaml
```

## Evaluation signals

- SCASL's normalized entropy versus reference labels is lower (i.e., more stable) than the gene-expression comparator's entropy across the 50% dropout and 100%/75%/50% sub-sampling conditions
- num_cluster and the four QC thresholds (sites_initial, runs_initial, sites_quality, runs_quality) are held fixed between the reference and all perturbed re-runs, so entropy differences are attributable to the perturbation alone
- stability_entropy.csv contains one entropy row per perturbation condition per method, with matching cell sets for the sub-sampling conditions
- Reference SCASL run on the unperturbed matrix converges reproducibly (allowing for randomness in imputation, dimensionality reduction, and unsupervised clustering noted in the README)
- The demo run completes in under 3 minutes on `data/junction` intermediates with results written to `process_result_demo`

## Limitations

- Single-cell AS profiles suffer greatly from missing data points due to dropouts during single-cell RNA sequencing analysis and AS event identification, so sub-sampling results are stochastic and re-runs may differ
- The GitHub demo files are very few and only used to show operation and speed; a full robustness study requires the complete TNBC-2 or equivalent dataset
- SCASL relies on junction reads from SMART-seq-like or STAR SJ.out.tab inputs; data generated only as a processed expression matrix without junction files cannot support the AS-based arm of the comparison

## Evidence

- [other] Under different cell sub-sampling ratios, the clusters defined by SCASL from single-cell AS profiles are much more robust and stable than those defined by Seurat from gene expression profiles.: [evidence span withheld — anchor status: not_found]
- [abstract] SCASL resolves the issue of biased and sparse data coverage on single-cell RNA splicing and provides a new scheme for classifications of cell identities: "SCASL resolves the issue of biased and sparse data coverage on single-cell RNA splicing and provides a new scheme for classifications of cell identities"
- [discussion] single-cell AS profiles suffer greatly from missing data points due to dropouts during single-cell RNA sequencing analysis and the identification of AS events: "single-cell AS profiles suffer greatly from missing data points due to dropouts during single-cell RNA sequencing analysis and the identification of AS events"
- [methods] sites_initial: quality control of the initial junction matrix, minimum number of expressing cells per site: "sites_initial: quality control of the initial junction matrix, minimum number of expressing cells per site"
- [readme] Due to the randomness of the interpolation, dimensionality reduction and unsupervised clustering processes, there may be some differences between the running results: "Due to the randomness of the interpolation, dimensionality reduction and unsupervised clustering processes, there may be some differences between the running results"
