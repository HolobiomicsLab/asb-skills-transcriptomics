---
name: silhouette-coefficient-computation
description: Use when benchmarking batch-effect correction methods on scRNA-seq data
  with known cell-type labels and you need an unsupervised measure of corrected-data
  separation quality beyond adjusted Rand index.
when_to_use_negative:
- Data have no cluster or cell-type structure to evaluate separation against (silhouette
  needs assignments or labels)
- The goal is batch-effect removal validation on bulk RNA-seq data, where ComBat/SVA-style
  designs rather than single-cell corrected values apply
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: BUSseq
  role: Produces corrected read counts on which silhouette coefficients are computed;
    its median silhouette should exceed that of competing methods
  repo: https://github.com/songfd2018/BUSseq-Rpackage
- name: Seurat
  role: Competing integration method whose corrected representation is scored by silhouette
    for benchmarking
- name: LIGER
  role: Competing integration method whose corrected representation is scored by silhouette
    for benchmarking
- name: Scanorama
  role: Competing integration method whose corrected representation is scored by silhouette
    for benchmarking
- name: scVI
  role: Competing latent-representation method scored by silhouette in the pancreas
    benchmark
- name: ZINB-WaVE
  role: Competing method whose corrected values are scored by silhouette in the pancreas
    benchmark
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:08172b89a652cb327f59f8080add7f59c226d9203b9a82a9388d69a6c9257b11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq-Rpackage
  tools:
  - BUSseq
  - ComBat
  - LIGER
  - MNN
  - RUV
  - SVA
  - Scanorama
  - Seurat
  - ZINB-WaVE
  - scMerge
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 20
---

# silhouette-coefficient-computation

## Summary

Computes silhouette coefficients (median silhouette) on batch-corrected scRNA-seq values to quantify how coherently cells of the same cell type cluster after correction, complementing ARI when benchmarking methods like BUSseq against LIGER, MNN, Scanorama, scVI, Seurat, and ZINB-WaVE.

## When to use

Use when benchmarking batch-effect correction methods on scRNA-seq data with known cell-type labels and you need an unsupervised measure of corrected-data separation quality beyond adjusted Rand index.

## When NOT to use

- Data have no cluster or cell-type structure to evaluate separation against (silhouette needs assignments or labels)
- The goal is batch-effect removal validation on bulk RNA-seq data, where ComBat/SVA-style designs rather than single-cell corrected values apply

## Inputs

- Batch-corrected read count matrices or latent representations from each method (e.g., corrected_read_counts(sce_BUSseqfit))
- Cluster assignments or known cell-type labels (e.g., FACS-sorted labels, pancreas islet/non-islet labels)
- Preprocessed count matrix of shared highly variable genes across batches

## Outputs

- Per-method median silhouette coefficient values
- Comparison table of silhouette (and ARI) values across batch-correction methods, e.g., batch_correction_benchmark.csv

## How to apply

After running each correction method (e.g., BUSseq via BUSseq_MCMC with K selected by BIC over K=3 to 6, or competitors such as Seurat, LIGER, MNN, Scanorama, scVI, ZINB-WaVE) to obtain corrected values or latent representations, compute pairwise silhouette coefficients using the method's cluster assignments or known labels. Report the median silhouette coefficient per method, as the BUSseq benchmarks do for the hematopoietic data (GSE72857, GSE81282-style FACS-labeled cells) and the pancreas data (GSE81076, GSE85241, GSE86473, E-MATB-5061; 7095 cells x 2480 highly variable genes). Compare the median silhouette across methods in the same table as ARI: BUSseq should attain both the highest ARI and the highest median silhouette among the seven methods. Silhouette captures separation of cell types in corrected space, guarding against cases where clustering agreement (ARI) is adequate but corrected values remain poorly separated.

## Related tools

- **BUSseq** (Produces corrected read counts on which silhouette coefficients are computed; its median silhouette should exceed that of competing methods) — https://github.com/songfd2018/BUSseq-Rpackage
- **Seurat** (Competing integration method whose corrected representation is scored by silhouette for benchmarking)
- **LIGER** (Competing integration method whose corrected representation is scored by silhouette for benchmarking)
- **Scanorama** (Competing integration method whose corrected representation is scored by silhouette for benchmarking)
- **scVI** (Competing latent-representation method scored by silhouette in the pancreas benchmark)
- **ZINB-WaVE** (Competing method whose corrected values are scored by silhouette in the pancreas benchmark)

## Examples

```
sil <- silhouette(as.integer(labels), dist(corrected_read_counts(sce_BUSseqfit))); median(sil[, 3])
```

## Evaluation signals

- BUSseq's median silhouette coefficient should be the highest among all benchmarked methods (as reported for both hematopoietic and pancreas data)
- Per-method silhouette values should match the reported benchmark values within tolerance when reproducing the table
- Silhouette is computed on the same preprocessed matrices used for ARI (e.g., 7095 cells x 2480 HVGs for pancreas), so cell/gene dimensions must be consistent across methods
- Silhouette values should fall in [-1, 1], with top-performing corrections approaching the positive range; near-zero or negative values indicate poor separation

## Limitations

- MCMC-based correction (BUSseq) carries heavy computation load, so generating corrected values for silhouette scoring can be expensive
- Multi-stage methods may propagate clustering errors, distorting silhouette scores computed from those assignments
- Silhouette depends on the corrected representation's scale and the labeling used; comparisons are only valid when computed identically across methods

## Evidence

- [methods] Compute the adjusted Rand index of each method's clusters against known labels and the median silhouette coefficient on each method's corrected values.: [evidence span withheld — anchor status: not_found]
- [methods] the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch: "the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch"
- [results] verify that BUSseq attains the highest ARI (0.608) and the highest median silhouette coefficient among all seven methods: [evidence span withheld — anchor status: not_found]
- [methods] corrected_read_counts(sce_BUSseqfit): "corrected_read_counts(sce_BUSseqfit)"
