---
name: multi-method-clustering-pipeline-execution
description: Use when clustering the 15 Krzak et al. 2019 real scRNA-seq evaluation
  datasets under R 4.0.5 and comparing CASCC against CIDR, RaceID3, SC3, Seurat V4,
  SIMLR, and TSCAN using ARI/AMI/NMI against ground-truth cell-type labels.
when_to_use_negative:
- The input is not labeled scRNA-seq count data with ground-truth cell-type annotations,
  since ARI/AMI/NMI against reference labels cannot be computed
- The goal is a single-method exploratory clustering analysis rather than a comparative
  benchmark of multiple methods
- The R environment cannot be pinned to R 4.0.5 with Seurat 4.3.0 and cafr, so results
  would not be comparable to the paper's reported scores
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CASCC
  role: Primary method under test; run.CASCC performs co-expression-based feature
    selection, cluster-number prediction, and k-means clustering with attractor-modified
    centers
  repo: https://github.com/LingyiC/CASCC
- name: cafr
  role: Dependency providing the adaptive attractor algorithm used for co-expression
    feature identification
  repo: https://github.com/weiyi-bitw/cafr
- name: Seurat 4.3.0
  role: Pinned dependency (installed via remotes::install_version) and baseline clustering
    method (Seurat V4) in the comparison
- name: aricode
  role: Computes ARI, AMI, and NMI between each method's cluster labels and ground-truth
    annotations
- name: devtools
  role: Installs CASCC and cafr from GitHub in the benchmark environment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae283
    title: 'CASCC: a co-expression-assisted single-cell RNA-seq data clustering method'
  source_content_hash: sha256:2df79472ebb7d7529b6878d9b9374e0ac67eb40ff4b06990e94d2a874566109a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LingyiC/CASCC
  tools:
  - CASCC
  - R
  - Seurat 4.3.0
  - aricode
  - cafr
  - devtools
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# multi-method-clustering-pipeline-execution

## Summary

Execute a multi-method scRNA-seq clustering benchmark: run CASCC (adaptive-attractor feature selection, Ward.D2 cluster-number prediction, attractor-modified k-means) alongside six baseline methods (CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN) on datasets with ground-truth labels, then score every method-dataset pair with ARI, AMI, and NMI.

## When to use

Use when clustering the 15 Krzak et al. 2019 real scRNA-seq evaluation datasets under R 4.0.5 and comparing CASCC against CIDR, RaceID3, SC3, Seurat V4, SIMLR, and TSCAN using ARI/AMI/NMI against ground-truth cell-type labels.

## When NOT to use

- The input is not labeled scRNA-seq count data with ground-truth cell-type annotations, since ARI/AMI/NMI against reference labels cannot be computed
- The goal is a single-method exploratory clustering analysis rather than a comparative benchmark of multiple methods
- The R environment cannot be pinned to R 4.0.5 with Seurat 4.3.0 and cafr, so results would not be comparable to the paper's reported scores

## Inputs

- Normalized log-transformed scRNA-seq count matrices for the 15 real scRNA-seq evaluation datasets (Krzak et al. 2019) with ground-truth cell-type labels
- Benchmark environment: R 4.0.5, Seurat 4.3.0, cafr 0.312, CASCC, aricode
- CASCC::run.CASCC cluster label assignments for each dataset
- Cluster label assignments from CIDR, RaceID3, SC3, Seurat V4, SIMLR, and TSCAN for each dataset

## Outputs

- benchmark_ari_ami_nmi_scores.csv — per-dataset, per-method ARI, AMI, and NMI table
- Cluster label assignments for each of the seven methods on each of the 15 datasets
- Mean ARI per method for ranking (e.g. CASCC vs. second-best SC3)

## How to apply

Pin the environment as in the paper's sessionInfo: R 4.0.5, Seurat 4.3.0 installed via remotes::install_version, the cafr dependency from weiyi-bitw/cafr, and CASCC from LingyiC/CASCC. For each of the 15 benchmark datasets, obtain the normalized log-transformed count matrix plus ground-truth cell-type labels from the LingyiC/CASCC_benchmark repository, then run CASCC::run.CASCC, which internally performs CASCC.featureSelection (adaptive attractor scanning with exponent.max=10, exponent.min=2), predictClusterK (Ward.D2 hierarchical clustering scored by silhouette index to predict cluster number), and k-means with attractor-modified centers via kmeansCenterModify. Run the six baseline methods (CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN) on the same datasets to get cluster label assignments. Compare every method's labels to ground truth with the aricode package to compute ARI, AMI, and NMI per dataset-method pair. Aggregate into a per-dataset, per-method table saved as benchmark_ari_ami_nmi_scores.csv.

## Related tools

- **CASCC** (Primary method under test; run.CASCC performs co-expression-based feature selection, cluster-number prediction, and k-means clustering with attractor-modified centers) — https://github.com/LingyiC/CASCC
- **cafr** (Dependency providing the adaptive attractor algorithm used for co-expression feature identification) — https://github.com/weiyi-bitw/cafr
- **Seurat 4.3.0** (Pinned dependency (installed via remotes::install_version) and baseline clustering method (Seurat V4) in the comparison)
- **aricode** (Computes ARI, AMI, and NMI between each method's cluster labels and ground-truth annotations)
- **devtools** (Installs CASCC and cafr from GitHub in the benchmark environment)

## Examples

```
res <- CASCC::run.CASCC(Data_PDAC_peng_2k)
```

## Evaluation signals

- Computed ARI/AMI/NMI values for CASCC and each baseline reproduce the per-dataset clustering scores in Fig. 1b and Supplementary Table S3 of the paper
- Mean ARI for CASCC exceeds the second-best method (SC3) by approximately 34% across the 15 datasets
- Output CSV contains a complete score table with all three metrics (ARI, AMI, NMI) for every dataset-method pair (15 datasets x 7 methods)
- Cluster number predicted by predictClusterK (Ward.D2 with silhouette index) is consistent with CASCC's predicted k used in the final k-means step

## Limitations

- Results depend on pinned versions (R 4.0.5, Seurat 4.3.0, cafr); running under other versions may not reproduce the reported benchmark scores
- The benchmark covers the 15 Krzak et al. 2019 real scRNA-seq datasets; performance on other data types or datasets is not established by this comparison
- CASCC depends on co-expression features identified by an unsupervised adaptive attractor algorithm, so sparse or poorly structured co-expression signal may affect cluster-number prediction and k-means center modification

## Evidence

- [methods] run.CASCC primary invocation: "res <- CASCC::run.CASCC(Data_PDAC_peng_2k)"
- [methods] predictClusterK parameters: "res.predictK <- predictClusterK(data, fs.res, method = "ward.D2", index = "silhouette", min.nc_fix = FALSE)"
- [methods] feature selection parameters: "fs.res <- CASCC.featureSelection(Data_PDAC_peng_2k, inputDataType = "well-normalized", attr.raw = NULL, exponent.max = 10, exponent.min = 2"
- [readme] environment pinning: "remotes::install_version("Seurat", "4.3.0") # dependency"
- [intro] method definition: "CASCC, a clustering method designed to improve biological accuracy using gene co-expression features identified using an unsupervised adaptive attractor algorithm."
- [methods] k-means center modification: "res$mainType.output <- kmeansCenterModify(res)"
