---
name: adjusted-rand-index-clustering-comparison
description: Use when comparing cell-type label predictions from multiple reference-mapping
  or clustering methods against manually curated ground-truth annotations, and you
  need per-cell-type accuracy plus ARI summary metrics to build a comparison table.
when_to_use_negative:
- Query cells contain novel cell states absent from the reference atlas — projections
  and transferred labels may be misleading, so unsupervised clustering should be compared
  to mapping results instead of relying on accuracy/ARI alone
- Evaluating a single mapping without competing methods or ground-truth annotations
  — there is nothing to compute ARI against
- Cross-platform projection scenarios (e.g. Smart-seq2 to 10x) where the paper reports
  performance declines due to differing UMI distributions and sequencing depth — metrics
  may not reflect intrinsic method quality
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: ProjectSVR
  role: Reference-mapping method under evaluation; projects query cells into the reference
    UMAP space via AUCell gene set scores and ComputeModuleScore/ProjectNewdata/MapQuery
    before label transfer
  repo: https://github.com/JarningGau/ProjectSVR
- name: Seurat
  role: Baseline reference-mapping method (Seurat V4) compared in the benchmark
  repo: https://github.com/satijalab/seurat
- name: scArches
  role: Baseline deep-learning reference-mapping method compared in the benchmark
  repo: https://github.com/theislab/scarches
- name: Symphony
  role: Baseline reference-mapping method compared in the benchmark
  repo: https://github.com/immunogenomics/symphony
- name: ProjecTILs
  role: Baseline reference-mapping method compared in the benchmark
  repo: https://github.com/carmonalab/ProjecTILs
- name: AUCell
  role: Computes gene activity scores (0–1) from rank expression used as ProjectSVR's
    input features prior to mapping and label transfer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:4f5f40ae11581c36cae21d86cc8c2ee598180644151eee813fa6c2b37cc22fcb
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
  - Seurat
  - Symphony
  - scArches
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# adjusted-rand-index-clustering-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for quantifying agreement between predicted cell-type labels and reference annotations using accuracy and adjusted Rand index (ARI), as used to compare ProjectSVR against Seurat, scArches, Symphony, ProjecTILs, iNMF, and SCALEX on the DISCO PBMC, MFI, and mTCA reference-mapping tasks.

## When to use

Use when comparing cell-type label predictions from multiple reference-mapping or clustering methods against manually curated ground-truth annotations, and you need per-cell-type accuracy plus ARI summary metrics to build a comparison table.

## When NOT to use

- Query cells contain novel cell states absent from the reference atlas — projections and transferred labels may be misleading, so unsupervised clustering should be compared to mapping results instead of relying on accuracy/ARI alone
- Evaluating a single mapping without competing methods or ground-truth annotations — there is nothing to compute ARI against
- Cross-platform projection scenarios (e.g. Smart-seq2 to 10x) where the paper reports performance declines due to differing UMI distributions and sequencing depth — metrics may not reflect intrinsic method quality

## Inputs

- Predicted per-cell labels for each method (e.g. from k-NN label transfer, k=10, onto reference cell_type labels)
- Manually curated ground-truth query annotations (e.g. cell_type column of query_hPBMCs.seurat.slim.qs, Guo2021.MFI.seurat.slim.qs, query_Ythdc2-KO.seurat.slim.qs)
- Reference/query embeddings from each mapping method (ProjectSVR, Seurat V4, scArches, Symphony, ProjecTILs, online iNMF, SCALEX)

## Outputs

- Per-cell-type annotation accuracy for each method/dataset combination
- Adjusted Rand index (ARI) for each method/dataset combination
- Benchmark comparison table (table2_benchmark_accuracy_ari.csv, matching the paper's Table 2)
- Summary-metric and embedding visualizations (Fig. 3, FigureS3)

## How to apply

For each reference/query pair (e.g. model.disco_pbmc.rds vs query_hPBMCs.seurat.slim.qs, model.Vento2018.MFI.rds vs Guo2021.MFI.seurat.slim.qs, model.mTCA.rds vs query_Ythdc2-KO.seurat.slim.qs), first obtain predicted labels by running k-NN label transfer with k=10 from the reference cell labels onto the mapped query embeddings for every method. Then compute per-cell-type annotation accuracy and adjusted Rand index against the manually curated query annotations. Assemble the accuracy and ARI values into a comparison table (the paper's Table 2, saved as table2_benchmark_accuracy_ari.csv) and generate accompanying visualizations (Fig. 3 / FigureS3 embeddings). Validation: the assembled table should show ProjectSVR achieving accuracy and ARI above 0.92 on mTCA—close to the de novo integration gold standard—and comparable or superior values to Seurat, scArches, Symphony, and ProjecTILs, consistent with the published Table 2 / Fig. 3.

## Related tools

- **ProjectSVR** (Reference-mapping method under evaluation; projects query cells into the reference UMAP space via AUCell gene set scores and ComputeModuleScore/ProjectNewdata/MapQuery before label transfer) — https://github.com/JarningGau/ProjectSVR
- **Seurat** (Baseline reference-mapping method (Seurat V4) compared in the benchmark) — https://github.com/satijalab/seurat
- **scArches** (Baseline deep-learning reference-mapping method compared in the benchmark) — https://github.com/theislab/scarches
- **Symphony** (Baseline reference-mapping method compared in the benchmark) — https://github.com/immunogenomics/symphony
- **ProjecTILs** (Baseline reference-mapping method compared in the benchmark) — https://github.com/carmonalab/ProjecTILs
- **AUCell** (Computes gene activity scores (0–1) from rank expression used as ProjectSVR's input features prior to mapping and label transfer)

## Examples

```
seu.q <- subset(seu.q, mean.knn.dist < 2)
seu.q <- ProjectSVR::LabelTransfer(seu.q, reference, ref.label.col = "cell_type")
```

## Evaluation signals

- ProjectSVR achieves accuracy and ARI above 0.92 on the mTCA benchmark, close to the de novo integration gold standard
- Assembled table (table2_benchmark_accuracy_ari.csv) shows ProjectSVR comparable or superior to Seurat, scArches, Symphony, and ProjecTILs across the three datasets, matching published Table 2 / Fig. 3
- Every method/dataset combination has both an accuracy and an ARI value computed against the same manually curated annotations
- k-NN label transfer uses k=10 from reference cell labels for every method identically, so metrics are comparable across methods
- Reference and query embedding visualizations (FigureS3) qualitatively agree with the summary metrics in Fig. 3

## Limitations

- Fixed-model projections may be misleading if query cells contain novel states absent in the reference; users should compare unsupervised clustering to mapping results to detect inconsistencies
- Performance declines when projecting across platforms (e.g. Smart-seq2 to 10x) due to differing UMI distributions and sequencing depth (Table S5)
- ProjectSVR trains independent SVR models per UMAP dimension, neglecting correlations among embedding dimensions, which may limit learning of joint structures

## Evidence

- [other] k-NN label transfer (k=10) from the reference cell labels for every method on every pair: [evidence span withheld — anchor status: not_found]
- [other] accuracy and ARI computation against curated annotations: [evidence span withheld — anchor status: not_found]
- [discussion] comparable or superior accuracy and ARI finding: [evidence span withheld — anchor status: not_found]
- [discussion] novel states limitation: "If query cells contain novel states absent in the reference, fixed-model projections may be misleading. Users should compare unsupervised clustering to mapping results to detect inconsistencies."
- [readme] benchmark methods and datasets: [evidence span withheld — anchor status: external_doc]
