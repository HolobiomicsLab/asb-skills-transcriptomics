---
name: knn-label-transfer-annotation-accuracy-evaluation
description: Use when comparing reference mapping methods (e.g. ProjectSVR, Seurat,
  scArches, Symphony, ProjecTILs) on reference/query pairs where query cells have
  manually curated labels and you need accuracy/ARI benchmark numbers.
when_to_use_negative:
- Query cells lack manually curated ground-truth annotations, so accuracy and ARI
  against curated labels cannot be computed
- Query cells contain novel cell states absent from the reference — projections may
  be misleading and label-transfer accuracy would be artificially low
- The goal is reference-model construction or gene-set scoring rather than post-mapping
  evaluation of label transfer
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ProjectSVR
  role: Maps query cells into the reference UMAP space before label transfer; provides
    LabelTransfer for k-NN annotation transfer
  repo: https://github.com/JarningGau/ProjectSVR
- name: Seurat
  role: Reference-mapping baseline whose transferred labels are scored for accuracy/ARI
- name: scArches
  role: Reference-mapping baseline scored in the same accuracy/ARI benchmark
- name: Symphony
  role: Reference-mapping baseline scored in the same accuracy/ARI benchmark
- name: ProjecTILs
  role: Reference-mapping baseline scored in the same accuracy/ARI benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:3cbcfa1a461d40f13f8dfccb945e88c5f56c58c3a6f590722b1831f0595d582c
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
  - ProjecTILs
  - ProjectSVR
  - Seurat
  - Symphony
  - scArches
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# knn-label-transfer-annotation-accuracy-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates reference-mapping methods by transferring cell-type labels from the reference to mapped query cells via k-NN (k=10) and scoring per-cell-type annotation accuracy and adjusted Rand index (ARI) against manually curated query annotations, as done in the ProjectSVR benchmark (Table 2, Fig. 3).

## When to use

Use when comparing reference mapping methods (e.g. ProjectSVR, Seurat, scArches, Symphony, ProjecTILs) on reference/query pairs where query cells have manually curated labels and you need accuracy/ARI benchmark numbers.

## When NOT to use

- Query cells lack manually curated ground-truth annotations, so accuracy and ARI against curated labels cannot be computed
- Query cells contain novel cell states absent from the reference — projections may be misleading and label-transfer accuracy would be artificially low
- The goal is reference-model construction or gene-set scoring rather than post-mapping evaluation of label transfer

## Inputs

- Mapped query Seurat objects with projected reference-space embeddings for each method (ProjectSVR, Seurat, scArches, Symphony, ProjecTILs)
- Reference atlas objects with a cell-type label column (e.g. model.disco_pbmc.rds, model.Vento2018.MFI.rds, model.mTCA.rds)
- Query datasets with manually curated ground-truth annotations (query_hPBMCs.seurat.slim.qs, Guo2021.MFI.seurat.slim.qs, query_Ythdc2-KO.seurat.slim.qs)

## Outputs

- Per-cell-type annotation accuracy per method/dataset combination
- Adjusted Rand index (ARI) per method/dataset combination
- Benchmark comparison table (e.g. table2_benchmark_accuracy_ari.csv) and Fig. 3-style visualizations

## How to apply

After each method (ProjectSVR via ComputeModuleScore + ProjectNewdata/MapQuery, or baselines like Seurat, scArches, Symphony, ProjecTILs) projects query cells into the reference embedding, run k-NN label transfer with k=10 from reference cell labels (e.g. ref.label.col = 'cell_type') onto every query cell. Before scoring, optionally drop low-quality projections with the map-quality filters (mean.knn.dist < 2, or mapQ.p.adj < 0.1) so unreliable mappings do not penalize accuracy. For each method/dataset combination, compute per-cell-type annotation accuracy and adjusted Rand index against the manually curated query annotations across the DISCO PBMC, MFI, and mTCA benchmark pairs. Assemble the values into a comparison table (e.g. table2_benchmark_accuracy_ari.csv) and validate that ProjectSVR achieves comparable or superior accuracy/ARI to the baselines, matching the reported Table 2 / Fig. 3 results (accuracy and ARI above 0.92 on mTCA).

## Related tools

- **ProjectSVR** (Maps query cells into the reference UMAP space before label transfer; provides LabelTransfer for k-NN annotation transfer) — https://github.com/JarningGau/ProjectSVR
- **Seurat** (Reference-mapping baseline whose transferred labels are scored for accuracy/ARI)
- **scArches** (Reference-mapping baseline scored in the same accuracy/ARI benchmark)
- **Symphony** (Reference-mapping baseline scored in the same accuracy/ARI benchmark)
- **ProjecTILs** (Reference-mapping baseline scored in the same accuracy/ARI benchmark)

## Examples

```
seu.q <- ProjectSVR::LabelTransfer(seu.q, reference, ref.label.col = "cell_type")
```

## Evaluation signals

- Assembled table shows ProjectSVR achieving accuracy and ARI above 0.92 on the mTCA benchmark, close to the de novo integration gold standard
- ProjectSVR accuracy/ARI are comparable or superior to Seurat, scArches, Symphony, and ProjecTILs across DISCO PBMC, MFI, and mTCA pairs, consistent with reported Table 2 / Fig. 3
- Same reference/query pairs and k=10 k-NN settings are applied uniformly to every baseline method
- Low-quality mapped cells are excluded consistently (e.g. mean.knn.dist < 2 filter) before computing metrics
- Metrics are computed against manually curated query annotations, not transferred or predicted labels

## Limitations

- Performance declines when projecting across platforms (e.g. Smart-seq2 to 10x) due to differing UMI distributions and sequencing depth, which would depress label-transfer accuracy
- If query cells contain novel states absent in the reference, fixed-model projections may be misleading; compare unsupervised clustering to mapping results to detect inconsistencies
- Metrics depend on the quality of manually curated query annotations and on upstream map-quality filtering thresholds (e.g. mean.knn.dist < 1.6 vs < 2.4 in different tutorials)

## Evidence

- [other] workflow-k10-grounding: [evidence span withheld — anchor status: not_found]
- [other] mtca-threshold-grounding: [evidence span withheld — anchor status: not_found]
- [readme] readme-benchmark-grounding: [evidence span withheld — anchor status: external_doc]
