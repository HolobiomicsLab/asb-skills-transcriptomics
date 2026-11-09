---
name: forced-dropout-robustness-testing
description: Use when you need to quantify the robustness and stability of cell clusters
  from SCASL AS profiles (or any clustering labels) under forced dropout or random
  cell sub-sampling, with a gene-expression comparator.
when_to_use_negative:
- If the goal is differential splicing quantification with known cell labels (e.g.,
  BRIE18, Expedition19, rMATS) rather than unsupervised clustering robustness — those
  tools require known labels and serve a different question
- If input is already a clustering-stability summary (e.g., a completed entropy table)
  rather than raw junction or expression matrices needing perturbation
- If the underlying data are bulk RNA-seq rather than single-cell SMART-seq junction
  reads, since sub-sampling individual cells is not defined
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SCASL
  role: 'Primary clustering method: identifies AS events from junction reads, performs
    iterative weighted KNN imputation, and spectral clusters cells on the imputed
    AS probability matrix under perturbed and unperturbed conditions'
  repo: https://github.com/xryanglab/SCASL
- name: Leafcutter
  role: Junction-based AS event identification strategy that SCASL mirrors (de novo
    AS event identification from junction reads); installed as a SCASL dependency
    via conda or from source
  repo: https://github.com/davidaknowles/leafcutter
- name: Seurat
  role: Gene-expression clustering comparator whose cluster labels are used as the
    baseline for the normalized entropy robustness comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:a00717106ef66476220b5d123e2644c7225abcd012e1f126a44c231c32006c1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xryanglab/SCASL
  tools:
  - Leafcutter
  - SCASL
  - Seurat
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# forced-dropout-robustness-testing

## Summary

A stability benchmarking skill that stress-tests single-cell clustering results by forcing 50% dropout on AS values and randomly sub-sampling cells (100%, 75%, 50%), then comparing normalized entropy of re-derived clusters against reference labels. It was used to show SCASL AS-based clusters are more robust than Seurat gene-expression clusters.

## When to use

Use when you need to quantify the robustness and stability of cell clusters from SCASL AS profiles (or any clustering labels) under forced dropout or random cell sub-sampling, with a gene-expression comparator.

## When NOT to use

- If the goal is differential splicing quantification with known cell labels (e.g., BRIE18, Expedition19, rMATS) rather than unsupervised clustering robustness — those tools require known labels and serve a different question
- If input is already a clustering-stability summary (e.g., a completed entropy table) rather than raw junction or expression matrices needing perturbation
- If the underlying data are bulk RNA-seq rather than single-cell SMART-seq junction reads, since sub-sampling individual cells is not defined

## Inputs

- SCASL demo junction/intermediate files from TNBC-2 in data/junction
- Reference cell labels via use_label/label_file and truth_column in configs/srr.yaml
- AS probability (normalized_matrix) and gene expression matrix for the comparator pipeline

## Outputs

- Per-condition cluster label files for each perturbed dataset (50% masked AS values; 100%/75%/50% sub-sampled cells)
- stability_entropy.csv with normalized entropy values and the SCASL-vs-gene-expression comparison table

## How to apply

First establish reference labels by running SCASL spectral clustering with weighted KNN imputation (num_iteration and knn set in configs/srr.yaml) on the unperturbed TNBC-2 junction/intermediate matrix loaded from data/junction, fixing num_cluster and the QC thresholds sites_initial, runs_initial, sites_quality, and runs_quality. Then generate perturbed AS matrices by masking 50% of AS values (forced dropout) and by randomly retaining 100%, 75%, and 50% of cells, re-running SCASL with the same fixed parameters on each. In parallel, re-cluster each perturbed and full dataset with a canonical gene-expression clustering pipeline (Seurat) on the expression matrix as the comparator. Finally, compute normalized entropy between each perturbed clustering's labels and the reference labels for both methods, writing per-condition labels and the SCASL-vs-expression entropy table to stability_entropy.csv.

## Related tools

- **SCASL** (Primary clustering method: identifies AS events from junction reads, performs iterative weighted KNN imputation, and spectral clusters cells on the imputed AS probability matrix under perturbed and unperturbed conditions) — https://github.com/xryanglab/SCASL
- **Leafcutter** (Junction-based AS event identification strategy that SCASL mirrors (de novo AS event identification from junction reads); installed as a SCASL dependency via conda or from source) — https://github.com/davidaknowles/leafcutter
- **Seurat** (Gene-expression clustering comparator whose cluster labels are used as the baseline for the normalized entropy robustness comparison)

## Examples

```
python main.py -y configs/srr.yaml
```

## Evaluation signals

- SCASL's normalized entropy relative to reference labels is lower (more stable) than the gene-expression comparator's across the 50% dropout and all sub-sampling ratios
- Reference SCASL run on the unperturbed matrix completes with the fixed num_cluster and unchanged QC parameters (sites_initial, runs_initial, sites_quality, runs_quality)
- Each perturbed re-run uses identical num_iteration and knn settings for weighted KNN imputation so entropy differences are attributable to perturbation only
- stability_entropy.csv contains per-condition labels plus the SCASL-vs-expression entropy table for the 50%/75%/100% sub-sampling and 50% dropout conditions

## Limitations

- Single-cell AS profiles suffer greatly from missing data points due to dropouts during scRNA-seq analysis and AS event identification, so imputed values under extreme dropout carry uncertainty
- Due to the randomness of imputation, dimensionality reduction and unsupervised clustering, results may differ between runs even on identical input
- The GitHub demo files from TNBC-2 are very few and only demonstrate operation and speed, not full-scale robustness conclusions

## Evidence

- [results] Under different cell sub-sampling ratios, the clusters defined by SCASL from single-cell AS profiles are much more robust and stable than those defined by Seurat from gene expression profiles.: [evidence span withheld — anchor status: not_found]
- [discussion] SCASL employs an iterative, weighted KNN process for imputation of the missing values: "SCASL employs an iterative, weighted KNN process for imputation of the missing values"
- [readme] The following parameters can be adjusted directly in `configs/srr.yaml` to use scasl: "The following parameters can be adjusted directly in `configs/srr.yaml` to use scasl"
- [methods] you can also choose to use the demo of the intermediate files (from TNBC-2) in `data/junction` for testing: "you can also choose to use the demo of the intermediate files (from TNBC-2) in `data/junction` for testing"
