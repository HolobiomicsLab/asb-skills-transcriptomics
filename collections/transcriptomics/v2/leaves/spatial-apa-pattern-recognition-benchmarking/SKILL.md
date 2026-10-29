---
name: spatial-apa-pattern-recognition-benchmarking
description: Use when you must stress-test a KNN imputation of spatial APA usage (RUD)
  matrices against artificial dropout (10%–90% spots masked or subsampled) and quantify
  sensitivity of spatial pattern recovery and Moran's I before vs. after imputation.
when_to_use_negative:
- Input is not a spatially barcoded ST-derived APA (RUD) matrix with matching spot
  coordinates and gene counts — the benchmark corrupts spot rows, so non-spatial or
  single-cell data lacks the spatial structure being tested
- You need to identify APA sites from raw sequencing data first; this skill benchmarks
  imputation downstream and assumes APA sites were already identified (e.g., with
  scAPAtrap) and quantified
- Data are from imaging-based platforms like MERFISH/seqFISH — the article notes APA
  sites can only be identified from spatially barcoded ST data with existing scRNA-seq
  tools
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
tools:
- name: stAPAminer
  role: Provides computeAPAIndex, imputeAPAIndex, optimalKvalue, and findSpatialPattern
    used to compute, corrupt, impute, and evaluate RUD matrices across dropout levels
  repo: https://github.com/BMILAB/stAPAminer
- name: movAPA
  role: Data structure (PACdataset) holding the APA-spot count matrix loaded from
    APA.RDA
- name: Seurat
  role: Required dependency for clustering/spatial pattern infrastructure used by
    stAPAminer functions
- name: fdm2id
  role: Required R dependency supporting the KNN-based imputation model
- name: ClusterR
  role: Required R dependency for clustering used in optimal-k selection
- name: cluster
  role: Required R dependency for clustering evaluation in optimalKvalue
- name: clusterSim
  role: Required R dependency for cluster-indices used in optimal-k determination
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:74794a18c1b73da5f049d22d5c935fe112f70bcecd79b9980ad5e937faab4164
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/stAPAminer
  tools:
  - ClusterR
  - Seurat
  - cluster
  - clusterSim
  - fdm2id
  - ggplot2
  - movAPA
  - stAPAminer
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# spatial-apa-pattern-recognition-benchmarking

## Summary

Benchmark the robustness and sensitivity of the stAPAminer KNN-based APA imputation by corrupting the ST-MOB RUD matrix at controlled dropout rates and spot-subsample sizes, then testing whether imputation recovers spatial pattern recognition and Moran's I. It answers whether the imputation model is overfitted and how optimal k scales with sparsity.

## When to use

Use when you must stress-test a KNN imputation of spatial APA usage (RUD) matrices against artificial dropout (10%–90% spots masked or subsampled) and quantify sensitivity of spatial pattern recovery and Moran's I before vs. after imputation.

## When NOT to use

- Input is not a spatially barcoded ST-derived APA (RUD) matrix with matching spot coordinates and gene counts — the benchmark corrupts spot rows, so non-spatial or single-cell data lacks the spatial structure being tested
- You need to identify APA sites from raw sequencing data first; this skill benchmarks imputation downstream and assumes APA sites were already identified (e.g., with scAPAtrap) and quantified
- Data are from imaging-based platforms like MERFISH/seqFISH — the article notes APA sites can only be identified from spatially barcoded ST data with existing scRNA-seq tools

## Inputs

- APA.RDA — APA-spot count matrix stored as a movAPA PACdataset (system.file('extdata','APA.RDA',package='stAPAminer'))
- count.csv — gene-spot expression count matrix
- position.txt — spot barcodes with spatial coordinates
- Raw RUD matrix from computeAPAIndex(APA, rownames(position))

## Outputs

- Per-dropout-level table of optimal k values (from optimalKvalue, ncluster=5, k candidates seq(4,14,2))
- Pattern-recognition sensitivity (findSpatialPattern) for raw vs. imputed matrices at each dropout level
- Moran's I spatial autocorrelation values before and after imputation across 10%–30% artificial dropout
- figure8_sensitivity_results.csv aggregating optimal k, sensitivity, and Moran's I per dropout level

## How to apply

Load the packaged ST-MOB inputs (APA.RDA movAPA object, count.csv, position.txt) from the stAPAminer extdata directory and compute the raw RUD index with computeAPAIndex(APA, rownames(position)). Apply controlled corruption by masking 10%–90% of spots and separately subsampling 20%–90% of spots from the RUD matrix. For each degraded matrix, call optimalKvalue(data, count, position, ncluster=5, orders=c(seq(4,14,2))) to record the optimal k, then impute with imputeAPAIndex using that k. Quantify sensitivity of spatial pattern recognition via findSpatialPattern on identified APA gene sets, plus Moran's I spatial autocorrelation, on raw versus imputed matrices, and aggregate results into one table (figure8_sensitivity_results.csv).

## Related tools

- **stAPAminer** (Provides computeAPAIndex, imputeAPAIndex, optimalKvalue, and findSpatialPattern used to compute, corrupt, impute, and evaluate RUD matrices across dropout levels) — https://github.com/BMILAB/stAPAminer
- **movAPA** (Data structure (PACdataset) holding the APA-spot count matrix loaded from APA.RDA)
- **Seurat** (Required dependency for clustering/spatial pattern infrastructure used by stAPAminer functions)
- **fdm2id** (Required R dependency supporting the KNN-based imputation model)
- **ClusterR** (Required R dependency for clustering used in optimal-k selection)
- **cluster** (Required R dependency for clustering evaluation in optimalKvalue)
- **clusterSim** (Required R dependency for cluster-indices used in optimal-k determination)

## Examples

```
RUD_RAW <- computeAPAIndex(APA, rownames(position)); optimalK <- optimalKvalue(RUD_RAW, count, position, ncluster=5, orders=c(seq(4,14,2))); RUD <- imputeAPAIndex(RUD_RAW, count, k=10)
```

## Evaluation signals

- Optimal k increases monotonically with dropout rate (larger dropout → larger optimal k), reproducing the Figure 8A trend
- Post-imputation sensitivity of spatial pattern recognition on imputed data is considerably higher than before imputation, regardless of sparsity (Figure 8C trend)
- Moran's I on imputed RUD matrices exceeds Moran's I on raw matrices across the 10%–30% artificial dropout range
- Results table exports cleanly as figure8_sensitivity_results.csv with one row per dropout level containing optimal k, sensitivity, and pre/post Moran's I
- Optimal k values fall within the evaluated candidate range seq(4,14,2) for every degradation level

## Limitations

- Entries still missing after the full imputation iteration are arbitrarily set to 0, which can bias the imputed matrix at extreme dropout rates
- No method exists to determine whether an APA ratio matrix entry is truly missing before imputation, so artificial masking may not reflect biological missingness
- Benchmarking is confined to spatially barcoded ST data; APA sites cannot currently be identified from MERFISH/seqFISH data, limiting generalization

## Evidence

- [full_text] Regardless of the sparsity of the data, the sensitivity of pattern recognition on the imputed data was considerably higher than that before imputation, and the larger the dropout rate, the larger the optimal k value.: "Regardless of the sparsity of the data, the sensitivity of pattern recognition on the imputed data was considerably higher than that before imputation"
- [intro] An imputation model based on KNN borrows spatial gene expression information to recover APA signals.: "an imputation model based on k-nearest neighbors (KNN) was designed to recover APA signals obtained from the ST data by borrowing information from the spatial gene expression profile"
- [readme] README shows the concrete imputation call with k=10 on the packaged ST-MOB extdata inputs.: "RUD <- imputeAPAIndex(RUD_RAW,count,k=10)"
- [methods] Methods quote the optimal-k selection call used in the sensitivity analysis.: "optimalK <- optimalKvalue(RUD_RAW,count,position,ncluster=5,orders = c(seq(4,14,2)))"
- [discussion] Discussion notes missing entries remaining after imputation are set to 0, a caveat for extreme dropout levels.: "If there are still missing entries after the entire iteration process, we can directly set their values to 0."
