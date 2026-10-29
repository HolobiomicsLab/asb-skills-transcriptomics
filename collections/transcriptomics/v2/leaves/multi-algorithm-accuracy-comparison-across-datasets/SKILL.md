---
name: multi-algorithm-accuracy-comparison-across-datasets
description: Use when a new clustering strategy or feature-space transformation must
  be validated against a baseline across several algorithms and benchmark datasets,
  with accuracy quantified by NMI at multiple subsampling rates.
when_to_use_negative:
- The task is a single clustering run rather than a framework-level accuracy comparison
  against a baseline strategy
- Ground-truth cell-type labels are unavailable, since NMI against true labels is
  the sole accuracy metric
- Datasets are not scRNA-seq count matrices and the Spearman-correlation feature space
  is not applicable
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3308
tools:
- name: SSCC
  role: Provides the Spearman-correlation feature-space clustering strategy being
    benchmarked against the SCC baseline via the sscClust R package
  repo: https://github.com/Japrin/sscClust
- name: SC3
  role: One of the five clustering algorithms compared under both SCC and SSCC strategies;
    also the accuracy/efficiency reference package (20% improvement, 50-fold acceleration)
- name: SIMLR
  role: One of the five clustering algorithms run under both strategies in the NMI
    benchmark
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: R
  role: Implementation language; sscClust is an R package and optimized BLAS (ATLAS,
    MKL) is recommended for the expensive calculations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:2ee00ddb86d41bdc3e49596e690c6310a957b11adfb76537afad05a4ded9795c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SC3
  - SIMLR
  - SSCC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# multi-algorithm-accuracy-comparison-across-datasets

## Summary

Benchmark a framework-level change (SCC vs SSCC clustering in original gene-expression space vs Spearman-correlation feature space) across multiple clustering algorithms (AP, k-means, k-medoids, SC3, SIMLR) and multiple scRNA-seq datasets at varying subsampling rates, quantifying accuracy as ΔNMI over ten random repeats.

## When to use

Use when a new clustering strategy or feature-space transformation must be validated against a baseline across several algorithms and benchmark datasets, with accuracy quantified by NMI at multiple subsampling rates.

## When NOT to use

- The task is a single clustering run rather than a framework-level accuracy comparison against a baseline strategy
- Ground-truth cell-type labels are unavailable, since NMI against true labels is the sole accuracy metric
- Datasets are not scRNA-seq count matrices and the Spearman-correlation feature space is not applicable

## Inputs

- Five benchmark scRNA-seq count matrices (Kolodziejczyk, Pollen, Usoskin, Zeisel, Zheng) with true cell-type labels
- Subsampling rate grid applied to cells
- Clustering algorithm implementations: AP, k-means, k-medoids, SC3, SIMLR

## Outputs

- nmi_benchmark_results.csv with mean NMI and spread per dataset-algorithm-strategy-subsampling configuration over ten repeats
- Per-dataset ΔNMI (SSCC vs SCC) ranges used to compare accuracy and robustness

## How to apply

Load five benchmark single-cell RNA-seq datasets with their true cell-type labels in R using the sscClust package. For each dataset, run the five algorithms (AP, k-means, k-medoids, SC3, SIMLR) under the SCC strategy — clustering on subsampled cells in the original gene expression space — then repeat identically under the SSCC strategy, where clustering is performed in the Spearman-correlation feature space built from the subsample. Sweep varying subsampling rates per dataset-algorithm-strategy combination, compute NMI between inferred clusters and true labels over ten random repeats, and record mean and spread. Aggregate per dataset and compute ΔNMI (SSCC minus SCC) ranges; expected improvement ranges are 0.12–0.60 (Kolodziejczyk), 0.04–0.19 (Pollen), 0.14–0.37 (Usoskin), 0.02–0.28 (Zeisel), and 0.10–0.28 (Zheng), with SSCC accuracy over k-means reaching up to 3-fold. Write the full NMI benchmark table with per-dataset ΔNMI ranges to nmi_benchmark_results.csv.

## Related tools

- **SSCC** (Provides the Spearman-correlation feature-space clustering strategy being benchmarked against the SCC baseline via the sscClust R package) — https://github.com/Japrin/sscClust
- **SC3** (One of the five clustering algorithms compared under both SCC and SSCC strategies; also the accuracy/efficiency reference package (20% improvement, 50-fold acceleration))
- **SIMLR** (One of the five clustering algorithms run under both strategies in the NMI benchmark) — https://github.com/BatzoglouLabSU/SIMLR
- **R** (Implementation language; sscClust is an R package and optimized BLAS (ATLAS, MKL) is recommended for the expensive calculations)

## Examples

```
R: sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11); sce.all <- ssc.run(sce.Pollen, subsampling=F, k.batch=11)
```

## Evaluation signals

- Reproduced per-dataset ΔNMI ranges fall within the published ranges (e.g., 0.12–0.60 for Kolodziejczyk, 0.04–0.19 for Pollen)
- All NMI values lie in the valid range [0, 1]
- Ten random repeats are recorded for every dataset-algorithm-strategy-subsampling configuration
- SSCC outperforms SCC for all five algorithms on all five datasets (positive ΔNMI in every configuration)
- SSCC accuracy improvement over k-means reaches up to 3-fold, matching the abstract claim

## Limitations

- SCC (and by extension comparisons at low subsampling rates) suffers from biases introduced by subsampling that can decrease clustering accuracy and robustness
- The article's enriched index provides no dataset accessions or identifiers for the five evaluation datasets, so data acquisition must come from the original publications (e.g., Zheng data via 10x Genomics datasets site)
- Large datasets make calculations expensive; the sscClust README strongly recommends R linked to optimized BLAS (ATLAS, MKL) or Microsoft R Open

## Evidence

- [results] SSCC significantly improved NMI versus SCC for all five clustering algorithms on all benchmark datasets: [evidence span withheld — anchor status: not_found]
- [results] NMI computed over ten random repeats per configuration: [evidence span withheld — anchor status: not_found]
- [abstract] SSCC framework based on random projection and feature construction: "Spearman subsampling-clustering-classification (SSCC), a new clustering framework based on random projection and feature construction"
- [abstract] R implementation of SSCC available at sscClust repository: "An R implementation of SSCC is available at https://github.com/Japrin/sscClust"
- [readme] sscClust README describes Spearman-correlation feature space for clustering and subsampling-classification: "projecting data to a feature space using spearman correlation which make visualiaztion and clustering more efficient"
