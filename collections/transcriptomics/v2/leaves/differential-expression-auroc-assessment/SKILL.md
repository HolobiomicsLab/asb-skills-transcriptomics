---
name: differential-expression-auroc-assessment
description: Use when benchmarking a feature selection algorithm (e.g., DUBStepR vs.
  HVGDisp, HVGVST, trendVar, devianceFS, HLG, M3Drop/DANB, GiniClust) on scRNA-seq
  data and you need a DE-detection AUROC against independently ascertained cell type
  annotations.
when_to_use_negative:
- Input is already a filtered feature set or DE gene table with no ranked gene list
  — AUROC needs the full method ranking to score overlap with top-500 positives vs.
  bottom-500 negatives
- Datasets lack independent cell type annotations, so the top-500 DE / bottom-500
  non-DE gene ground truth cannot be defined
- The goal is clustering-quality assessment rather than DE-gene detection — the paper
  pairs this AUROC with the mean scaled silhouette index, which should be computed
  separately
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DUBStepR
  role: Feature selection method under test; produces the ranked gene lists whose
    DE-detection AUROC is computed
  repo: github.com/prabhakarlab/DUBStepR
- name: M3Drop
  role: Baseline feature selection method (DANB) benchmarked with the same AUROC protocol
- name: GiniClust
  role: Baseline feature selection method benchmarked with the same AUROC protocol
- name: Seurat
  role: Used for data loading and LogNormalize normalization of count matrices prior
    to feature ranking
- name: R
  role: Implementation environment; DUBStepR requires R >= 3.5.0
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:7549aa759e46f58a365928bc8e7a1e0f3b5b97d3dba32e7441bca62844ba97f2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - GiniClust
  - M3Drop
  - R
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# differential-expression-auroc-assessment

## Summary

Assesses a scRNA-seq feature selection method by how well its ranked gene list separates known differentially expressed (DE) genes from non-DE genes, quantified as AUROC against independently annotated cell types. This mirrors the paper's objective criterion that a good feature selection algorithm selects cell-type-specific DE genes as features and rejects the remaining genes.

## When to use

Use when benchmarking a feature selection algorithm (e.g., DUBStepR vs. HVGDisp, HVGVST, trendVar, devianceFS, HLG, M3Drop/DANB, GiniClust) on scRNA-seq data and you need a DE-detection AUROC against independently ascertained cell type annotations.

## When NOT to use

- Input is already a filtered feature set or DE gene table with no ranked gene list — AUROC needs the full method ranking to score overlap with top-500 positives vs. bottom-500 negatives
- Datasets lack independent cell type annotations, so the top-500 DE / bottom-500 non-DE gene ground truth cannot be defined
- The goal is clustering-quality assessment rather than DE-gene detection — the paper pairs this AUROC with the mean scaled silhouette index, which should be computed separately

## Inputs

- Raw scRNA-seq count matrices from the 7 benchmark datasets (e.g., 10X Genomics pbmc_1k_v2 PBMC)
- Independent cell type annotations per cell for each dataset
- Per-method ranked feature/gene lists (from DUBStepR, HVGDisp, HVGVST, trendVar, devianceFS, HLG, M3Drop/DANB, GiniClust)

## Outputs

- DE-detection AUROC per method per dataset (values reported in Fig 4c)
- Positives/negatives definition: top-500 DE genes and bottom-500 non-DE genes per dataset
- CSV table of AUROC values suitable for comparison against the published benchmark results

## How to apply

Start from the seven public benchmark scRNA-seq count matrices (including the 10X Genomics pbmc_1k_v2 PBMC dataset) with independent cell type annotations. Filter low-quality genes (e.g., mitochondrial, ribosomal, spike-in, pseudogenes; DUBStepR's getFilteredData with min.cells = 0.05*ncol), log-normalize each dataset (logNormalize or Seurat's LogNormalize), and run the feature selection method to obtain a ranked gene list. Define ground truth by ranking all genes by differential expression across the annotated cell types, taking the top-500 DE genes as positives and the bottom-500 genes as negatives. Score the overlap with the method's ranked feature list and report the DE-detection AUROC. The paper's validation bar is AUROC > 0.97 on all 7 datasets, with DUBStepR meeting this while outperforming all seven baselines.

## Related tools

- **DUBStepR** (Feature selection method under test; produces the ranked gene lists whose DE-detection AUROC is computed) — github.com/prabhakarlab/DUBStepR
- **M3Drop** (Baseline feature selection method (DANB) benchmarked with the same AUROC protocol)
- **GiniClust** (Baseline feature selection method benchmarked with the same AUROC protocol)
- **Seurat** (Used for data loading and LogNormalize normalization of count matrices prior to feature ranking)
- **R** (Implementation environment; DUBStepR requires R >= 3.5.0)

## Examples

```
res <- DUBStepR(input.data, min.cells = 0.05*ncol(input.data), optimise.features = TRUE, k = 10, num.pcs = 20, error = 0)
```

## Evaluation signals

- DUBStepR attains AUROC > 0.97 on all 7 benchmark datasets, matching the reported Fig 4c results
- Every method's AUROC is computed against the identical positives/negatives definition (top-500 DE genes vs. bottom-500 non-DE genes), so AUROCs are directly comparable across methods
- Assembled AUROC CSV table reproduces the ranking where DUBStepR substantially outperforms HVGDisp, HVGVST, trendVar, devianceFS, HLG, M3Drop/DANB, and GiniClust
- Ground-truth gene labels derive from independently ascertained cell type annotations, not from the same clustering being evaluated

## Limitations

- Existing feature selection methods perform inconsistently across datasets, occasionally even resulting in poorer clustering accuracy than without feature selection — AUROC from one dataset may not generalize
- Only a handful of systematic benchmarking studies of scRNA-seq feature selection methods exist, so ground-truth DE labels depend on the stringently defined collection of datasets with independently ascertained cell type annotations
- DUBStepR approaches its maximum silhouette index value at 200–500 feature genes (Supp. Fig. S3); full quantitative details are deferred to supplementary notes not included in the main text

## Evidence

- [intro] A good feature selection algorithm is one that selects cell-type-specific (DE) genes as features, and rejects the remaining genes.: "A good feature selection algorithm is one that selects cell-type-specific (DE) genes as features, and rejects the remaining genes."
- [abstract] DUBStepR outperformed existing methods across benchmarks: "Despite selecting a relatively small number of genes, DUBStepR substantially outperformed existing single-cell feature selection methods across diverse clustering benchmarks."
- [readme] DUBStepR filters genes by correlation range score before ranking: "After filtering genes based on a correlation range score, DUBStepR exploits structure in the gene-gene correlation matrix to prioritize genes as features for clustering."
- [discussion] Benchmark collection with independent annotations identified only by reference number: "we used a stringently defined collection of single-cell datasets for which cell type annotations could be independently ascertained28."
- [methods] pbmc_1k_v2 is a benchmark input dataset: "we use a publicly available PBMC dataset generated by 10X Genomics. Here's a [link](https://support.10xgenomics.com/single-cell-gene-expression/datasets/3.0.0/pbmc_1k_v2)"
