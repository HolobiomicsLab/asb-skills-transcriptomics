---
name: cross-method-scrnaseq-mapping-comparison
description: Use when several single-cell reference-mapping methods must be compared
  head-to-head on the same reference/query dataset pairs, e.g. to judge whether a
  new mapper (ProjectSVR) matches Seurat, scArches, Symphony, ProjecTILs, iNMF or
  SCALEX.
when_to_use_negative:
- The query contains novel cell states absent from the reference — fixed-model projections
  may be misleading and unsupervised clustering should be compared against mapping
  results instead
- The goal is cross-platform projection (e.g. Smart-seq2 to 10x) — performance declines
  due to differing UMI distributions and sequencing depth (Table S5)
- You only need to run a single mapper on one dataset rather than a controlled multi-method
  comparison on shared pairs
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ProjectSVR
  role: Method under test; maps query cells via AUCell gene-set activity scores and
    SVR-predicted UMAP embeddings (ProjectNewdata/MapQuery)
  repo: https://github.com/JarningGau/ProjectSVR
- name: Seurat
  role: Baseline reference-mapping method (Seurat V4) run on the same reference/query
    pairs
  repo: https://github.com/satijalab/seurat
- name: Symphony
  role: Baseline reference-mapping method for the benchmark comparison
  repo: https://github.com/immunogenomics/symphony
- name: ProjecTILs
  role: Baseline reference-mapping method for the benchmark comparison
  repo: https://github.com/carmonalab/ProjecTILs
- name: scArches
  role: Baseline reference-mapping method for the benchmark comparison
  repo: https://github.com/theislab/scarches
- name: online iNMF
  role: Baseline reference-mapping method implemented in liger
  repo: https://github.com/welch-lab/liger
- name: SCALEX
  role: Baseline reference-mapping method for the benchmark comparison
  repo: https://github.com/jsxlei/SCALEX
- name: AUCell
  role: Computes rank-based gene-set activity scores (0-1) that ProjectSVR uses as
    regression features before label transfer
  repo: https://github.com/JarningGau/ProjectSVR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:b53ea2c8e960639fda7d823bad7ea78121a6371af64ff5bac0de994bf7aae3d5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/jarninggau/ProjectSVR
  repo_url: https://github.com/JarningGau/ProjectSVR
  tools:
  - AUCell
  - ProjecTILs
  - ProjectSVR
  - SCALEX
  - Seurat
  - Symphony
  - online iNMF
  - scArches
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-method-scrnaseq-mapping-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark reference-mapping algorithms (ProjectSVR, Seurat, scArches, Symphony, ProjecTILs, online iNMF, SCALEX) on identical reference/query scRNA-seq pairs and compare them via per-cell-type annotation accuracy and adjusted Rand index after k-NN label transfer.

## When to use

Use when several single-cell reference-mapping methods must be compared head-to-head on the same reference/query dataset pairs, e.g. to judge whether a new mapper (ProjectSVR) matches Seurat, scArches, Symphony, ProjecTILs, iNMF or SCALEX.

## When NOT to use

- The query contains novel cell states absent from the reference — fixed-model projections may be misleading and unsupervised clustering should be compared against mapping results instead
- The goal is cross-platform projection (e.g. Smart-seq2 to 10x) — performance declines due to differing UMI distributions and sequencing depth (Table S5)
- You only need to run a single mapper on one dataset rather than a controlled multi-method comparison on shared pairs

## Inputs

- Pre-trained reference model files (model.disco_pbmc.rds, model.Vento2018.MFI.rds, model.mTCA.rds; Zenodo record 8350732)
- Query Seurat objects in .qs format (query_hPBMCs.seurat.slim.qs, Guo2021.MFI.seurat.slim.qs, query_Ythdc2-KO.seurat.slim.qs; Zenodo records 8350746/8350748)
- Manually curated query cell-type annotations used as ground truth
- Benchmark scripts from github.com/JarningGau/ProjectSVR-benchmark

## Outputs

- table2_benchmark_accuracy_ari.csv with per-cell-type annotation accuracy and ARI for each method/dataset combination
- Summary-metric figures analogous to Fig. 3 and reference/query embedding overlays analogous to FigureS3.png
- Per-method label-transfer assignments on the query cells

## How to apply

Download the pre-trained reference models (model.disco_pbmc.rds, model.Vento2018.MFI.rds, model.mTCA.rds from Zenodo record 8350732) and the matched query objects (query_hPBMCs.seurat.slim.qs, Guo2021.MFI.seurat.slim.qs, query_Ythdc2-KO.seurat.slim.qs from Zenodo records 8350746/8350748), which cover the DISCO PBMC, maternal-fetal interface (MFI) and mouse testicular cell atlas (mTCA) benchmarks. For each pair, run ProjectSVR mapping (AUCell gene set scoring, then ProjectNewdata/MapQuery into the reference UMAP space) and the competing baselines using the scripts in github.com/JarningGau/ProjectSVR-benchmark. Apply the same post-processing to every method: optionally filter poor projections with the map-quality thresholds (e.g. mean.knn.dist < 2) and perform k-NN label transfer with k=10 from the reference cell-type labels. Compute per-cell-type annotation accuracy and adjusted Rand index (ARI) against the manually curated query annotations for each method/dataset combination. Assemble the values into a comparison table (analogous to Table 2, saved as table2_benchmark_accuracy_ari.csv) and reproduce the Fig. 3 summary-metric visualizations.

## Related tools

- **ProjectSVR** (Method under test; maps query cells via AUCell gene-set activity scores and SVR-predicted UMAP embeddings (ProjectNewdata/MapQuery)) — https://github.com/JarningGau/ProjectSVR
- **Seurat** (Baseline reference-mapping method (Seurat V4) run on the same reference/query pairs) — https://github.com/satijalab/seurat
- **Symphony** (Baseline reference-mapping method for the benchmark comparison) — https://github.com/immunogenomics/symphony
- **ProjecTILs** (Baseline reference-mapping method for the benchmark comparison) — https://github.com/carmonalab/ProjecTILs
- **scArches** (Baseline reference-mapping method for the benchmark comparison) — https://github.com/theislab/scarches
- **online iNMF** (Baseline reference-mapping method implemented in liger) — https://github.com/welch-lab/liger
- **SCALEX** (Baseline reference-mapping method for the benchmark comparison) — https://github.com/jsxlei/SCALEX
- **AUCell** (Computes rank-based gene-set activity scores (0-1) that ProjectSVR uses as regression features before label transfer) — https://github.com/JarningGau/ProjectSVR

## Examples

```
seu.q <- ProjectSVR::LabelTransfer(seu.q, reference, ref.label.col = "cell_type")
```

## Evaluation signals

- table2_benchmark_accuracy_ari.csv contains accuracy and ARI rows for every method/dataset combination (ProjectSVR, Seurat, scArches, Symphony, ProjecTILs, iNMF, SCALEX x DISCO PBMC, MFI, mTCA)
- ProjectSVR attains accuracy and ARI above 0.92 on mTCA, close to the de novo integration gold standard
- All methods use identical post-processing: k-NN label transfer with k=10 from reference cell-type labels, so differences are attributable to mapping quality
- Fig. 3-style summary metrics and FigureS3-style reference/query embeddings regenerate from the benchmark scripts
- Label transfer succeeds only after map-quality filtering (e.g. mean.knn.dist < 2), consistent with the methods code

## Limitations

- Runtime and memory differ between methods (e.g. scArches and Seurat are less favorable), so wall-clock comparisons must be run on identical hardware
- Benchmark results depend on the specific reference/query pairs (DISCO PBMC, MFI, mTCA); ProjectSVR performance declines when projecting across platforms such as Smart-seq2 to 10x
- Independent SVR models per UMAP dimension neglect correlations among embedding dimensions; ProjectSVR reference building is ~O(Nc^2), mitigated by subsampling to 4000-10,000 cells

## Evidence

- [discussion] Our results (Fig. 3, Table 2) show that ProjectSVR achieves comparable or superior accuracy and ARI across multiple datasets: "Our results (Fig. 3, Table 2) show that ProjectSVR achieves comparable or superior accuracy and ARI across multiple datasets, while also exhibiting favorable runtime and memory profiles."
- [methods] seu.q <- ProjectSVR::LabelTransfer(seu.q, reference, ref.label.col = "cell_type"): "seu.q <- subset(seu.q, mean.knn.dist < 2)
seu.q <- ProjectSVR::LabelTransfer(seu.q, reference, ref.label.col = "cell_type")"
- [readme] Benchmark results of ProjectSVR and other reference mapping algorithms were listed at: [evidence span withheld — anchor status: external_doc]
- [methods] map quality filter mean.knn.dist < 1.6: "seu.q <- subset(seu.q, mean.knn.dist < 1.6)"
