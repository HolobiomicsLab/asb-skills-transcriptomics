---
name: clustering-accuracy-nmi-evaluation
description: Use when predicted cluster assignments from an scRNA-seq clustering run
  (e.g., SC3, Seurat, SIMLR, k-means under SCC/SSCC) must be scored against known
  cell-type labels, or when consistency between two subsampling rates must be measured.
when_to_use_negative:
- Input is raw scRNA-seq count data with no clustering performed yet — this skill
  evaluates cluster assignments, it does not produce them (use the SSCC/sscClust clustering
  pipeline first)
- The goal is differential-expression or marker-gene analysis rather than judging
  clustering quality — NMI/ARI against cell-type labels is not meaningful there
- You need an external reference-free quality measure (e.g., silhouette) alone; note
  the paper pairs silhouette analysis with NMI for the Macosko consistency benchmark,
  so a silhouette-only check would miss the label-agreement signal
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SSCC
  role: Framework whose cluster assignments are the subject of the NMI evaluation;
    Spearman subsampling-clustering-classification with feature construction and random
    projection
  repo: https://github.com/Japrin/sscClust
- name: sscClust
  role: R package providing the clustering pipeline (ssc.run) and the evaluation utilities
    used to compute NMI/ARI
  repo: https://github.com/Japrin/sscClust
- name: SC3
  role: Baseline clustering method whose default-parameter NMI (0.48 on PBMC 68k)
    is compared against SSCC-improved SC3 (0.59)
- name: SIMLR
  role: One of the clustering algorithms whose 10%-vs-50% subsample consistency is
    scored by NMI
  repo: BatzoglouLabSU/SIMLR
- name: dropClust
  role: Comparator clustering method included in the PBMC 68k NMI/runtime/memory benchmark
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_004
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:1b8c621668137917e58bf1c827e6112d8c10068744b8d41bb586770269927410
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - CIDR
  - R
  - SC3
  - SIMLR
  - SSCC
  - Seurat
  - ZIFA
  - dropClust
  - sscClust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# clustering-accuracy-nmi-evaluation

## Summary

Quantifies single-cell RNA-seq clustering quality by computing normalized mutual information (NMI) — and complementary ARI — between predicted cluster assignments and reference cell-type labels, as used to benchmark SSCC against SC3, k-means, and other algorithms on datasets like Pollen and PBMC 68k.

## When to use

Use when predicted cluster assignments from an scRNA-seq clustering run (e.g., SC3, Seurat, SIMLR, k-means under SCC/SSCC) must be scored against known cell-type labels, or when consistency between two subsampling rates must be measured.

## When NOT to use

- Input is raw scRNA-seq count data with no clustering performed yet — this skill evaluates cluster assignments, it does not produce them (use the SSCC/sscClust clustering pipeline first)
- The goal is differential-expression or marker-gene analysis rather than judging clustering quality — NMI/ARI against cell-type labels is not meaningful there
- You need an external reference-free quality measure (e.g., silhouette) alone; note the paper pairs silhouette analysis with NMI for the Macosko consistency benchmark, so a silhouette-only check would miss the label-agreement signal

## Inputs

- Predicted cluster label vector from an scRNA-seq clustering run (e.g., k-means with k=11 on Pollen, SC3 on PBMC 68k)
- Reference/true cell-type label vector (e.g., Pollen dataset cell-type labels bundled with sscClust)
- Second predicted label set when evaluating subsampling consistency (e.g., 10% vs 50% subsample clusterings)

## Outputs

- NMI value (and ARI where labels are available) per clustering run, e.g., written to pollen_clustering_metrics.csv
- Per-method benchmark table of NMI, runtime, and peak memory, e.g., pbmc68k_benchmark_results.csv
- NMI-consistency table across subsampling rates, e.g., nmi_consistency_table.csv

## How to apply

Run the clustering (e.g., ssc.run on the Pollen dataset with k.batch=11, with or without subsampling at sub.frac like 0.8 or 0.2), then compute NMI and ARI between predicted cluster labels and the true cell-type labels using the sscClust evaluation utilities. For subsampling-consistency evaluation, compute NMI between the label sets obtained at two sampling fractions (e.g., 10% vs 50%, or 5% vs 10%) instead of against a reference. For benchmark comparisons, evaluate each method (SSCC-wrapped SC3 vs SC3 default vs dropClust vs plain k-means) on the same subsample and record NMI alongside wall-clock runtime and peak memory. Interpret NMI on an absolute scale grounded in the paper's reported values (e.g., 0.59 vs 0.48 NMI on PBMC 68k) rather than expecting a universal threshold.

## Related tools

- **SSCC** (Framework whose cluster assignments are the subject of the NMI evaluation; Spearman subsampling-clustering-classification with feature construction and random projection) — https://github.com/Japrin/sscClust
- **sscClust** (R package providing the clustering pipeline (ssc.run) and the evaluation utilities used to compute NMI/ARI) — https://github.com/Japrin/sscClust
- **SC3** (Baseline clustering method whose default-parameter NMI (0.48 on PBMC 68k) is compared against SSCC-improved SC3 (0.59))
- **SIMLR** (One of the clustering algorithms whose 10%-vs-50% subsample consistency is scored by NMI) — BatzoglouLabSU/SIMLR
- **dropClust** (Comparator clustering method included in the PBMC 68k NMI/runtime/memory benchmark)

## Examples

```
data("sce.Pollen"); sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)
```

## Evaluation signals

- On PBMC 68k, SC3+SSCC should reach ~0.59 average NMI vs ~0.48 for SC3 default (a ~20–21% improvement) with runtime ~2.2 min
- On Kolodziejczyk, NMI consistency between 10% and 50% subsample clusterings should exceed 0.5 for all five algorithms (SC3, Seurat, SIMLR, ZIFA, CIDR) under SSCC
- On Pollen with 20% subsampling and k-means k=11, the Spearman (iCor)-projected data should retain high NMI while original and PCA-projected data show degraded NMI/ARI
- On the Macosko retina dataset, NMI between 5% and 10% subsampling schemes should rise from ~0.60 (SCC) to ~0.69 (SSCC)
- Output CSVs must contain one NMI (and optionally ARI) value per method × strategy × dataset × subsampling-fraction combination, with no missing entries for the stated grid

## Limitations

- NMI against reference labels requires known cell-type annotations; on datasets without labels, only subsampling-consistency NMI (clustering vs clustering) is computable
- The plain SCC (subsampling-clustering-classification) strategy suffers from subsampling biases that can greatly decrease clustering accuracy and robustness, so NMI measured on plain SCC runs may be unstable across sampling fractions
- Large-dataset clustering that feeds the evaluation is computationally expensive; the sscClust README recommends an R linked to optimized BLAS (ATLAS, MKL; Microsoft R Open on Windows) or timings/memory figures will be inflated

## Evidence

- [abstract] SSCC achieved 20% improvement for clustering accuracy and 50-fold acceleration, but only consumed 66% memory usage, compared to the widelyused software package SC3: "SSCC achieved 20% improvement for clustering accuracy and 50-fold acceleration, but only consumed 66% memory usage, compared to the widelyused software package SC3"
- [abstract] Compared to k-means, the accuracy improvement of SSCC can reach 3-fold.: "Compared to k-means, the accuracy improvement of SSCC can reach 3-fold."
- [discussion] SSCC significantly improves clustering accuracy and consistency for many clustering algorithms: [evidence span withheld — anchor status: not_found]
- [readme] sscClust provides basic scRNA-seq analysis procedures including clustering on reduced data: "a package implement mutilple functionalities which are basic procedures in single cell RNAseq data analysis, including variable genes identification, dimension reduction, clustering on reduced data"
- [readme] README shows the concrete clustering invocation whose outputs are then evaluated by NMI/ARI: [evidence span withheld — anchor status: external_doc]
