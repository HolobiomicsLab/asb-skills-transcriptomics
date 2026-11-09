---
name: single-cell-reference-mapping
description: Use when a query scRNA-seq dataset (Seurat object, e.g. .qs/.rds) must
  be placed into a published reference atlas space for cell-type label transfer, cross-protocol
  batch-effect correction, pseudotime projection, or group composition comparison.
when_to_use_negative:
- Query cells contain novel states absent from the reference — fixed-model projections
  may be misleading; compare unsupervised clustering to mapping results first.
- Mapping across dissimilar platforms (e.g. Smart-seq2 to 10x), where performance
  declines due to differing UMI distributions and sequencing depth.
- The reference atlas was built with a data-tailored integration strategy incompatible
  with integration-dependent mappers — but note ProjectSVR is specifically designed
  to avoid this dependency; do not substitute Seurat/scArches/Symphony/ProjecTILs
  for the continuous-state pseudotime use case (regression, not classification).
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ProjectSVR
  role: 'Core framework: ComputeModuleScore, ProjectNewdata/MapQuery, MapQCPlot, LabelTransfer,
    MajorityVote, AbundanceTest for reference mapping'
  repo: https://github.com/jarninggau/ProjectSVR
- name: Seurat
  role: Holds reference and query objects; provides the Seurat object structure and
    subsetting used throughout the mapping workflow
  repo: https://github.com/satijalab/seurat
- name: AUCell
  role: Computes gene activity scores (0–1) from rank expression; the gene-set scoring
    backend feeding the SVR projection
- name: UCell
  role: Alternative signature-score calculation package supported by ProjectSVR (v1.3
    for R < 4.3)
  repo: https://github.com/carmonalab/UCell
- name: cNMF
  role: Derives cell-type gene sets for continuous cell states when building reference
    models (via the RunCNMF wrapper)
  repo: https://github.com/dylkot/cNMF
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_002
  - task_007
  - task_006
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:b1940d5824956686fdf963c06687764e7ab0db37d1cb6216a21587f918d30e79
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/jarninggau/ProjectSVR
  repo_url: https://github.com/jarninggau/ProjectSVR
  tools:
  - AUCell
  - ProjecTILs
  - ProjectSVR
  - Seurat
  - Symphony
  - UCell
  - cNMF
  - scArches
  - tidyverse
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# single-cell-reference-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Project query single-cell RNA-seq data onto a pre-built reference atlas using ProjectSVR, which trains ensemble support vector regression models to predict reference UMAP embeddings from AUCell/UCell gene set activity scores, decoupling mapping from any specific integration method.

## When to use

Use when a query scRNA-seq dataset (Seurat object, e.g. .qs/.rds) must be placed into a published reference atlas space for cell-type label transfer, cross-protocol batch-effect correction, pseudotime projection, or group composition comparison.

## When NOT to use

- Query cells contain novel states absent from the reference — fixed-model projections may be misleading; compare unsupervised clustering to mapping results first.
- Mapping across dissimilar platforms (e.g. Smart-seq2 to 10x), where performance declines due to differing UMI distributions and sequencing depth.
- The reference atlas was built with a data-tailored integration strategy incompatible with integration-dependent mappers — but note ProjectSVR is specifically designed to avoid this dependency; do not substitute Seurat/scArches/Symphony/ProjecTILs for the continuous-state pseudotime use case (regression, not classification).

## Inputs

- Pre-trained ProjectSVR reference model (.rds, e.g. model.disco_pbmc.rds, model.Vento2018.MFI.rds, model.mTCA.rds, model.Zheng2021.CD8Tcell.rds)
- Reference atlas Seurat object (.qs, e.g. DISCO_hPBMCs.seurat.slim.qs, mTCA.seurat.slim.qs)
- Query dataset Seurat object (.qs/.rds, e.g. query_hPBMCs.seurat.slim.qs, Guo2021.MFI.seurat.slim.qs, ICB_BRCA.1864-counts_cd8tcell_cohort1.seurat.rds)
- Reference gene.sets and gss.method (AUCell) stored in the model

## Outputs

- Query Seurat object with a 'ref.umap' dimension reduction and mapping-quality metrics (mean.knn.dist, mapQ.p.adj)
- Transferred cell-type labels (knn.pred.celltype) via LabelTransfer
- Projection plots (PlotProjection, AlluviaPlot, MapQCPlot, GroupPreferencePlot, VolcanoPlot)
- Group composition/abundance tables (e.g. dnk_proportions.csv, lisi_comparison.csv)

## How to apply

Download a pre-trained reference model (.rds, e.g. model.disco_pbmc.rds, model.Vento2018.MFI.rds, model.mTCA.rds from Zenodo record 8350732) and the matching reference atlas and query Seurat objects. Run ProjectSVR::MapQuery with add.map.qual=TRUE and ncores=10, which computes gene-set activity scores internally per the model's stored gss.method (AUCell gene set scoring, top 25 genes by default) and stores embeddings as a 'ref.umap' reduction. Filter poorly mapped cells using MapQCPlot cutoffs — dataset-specific thresholds such as mean.knn.dist < 1.6 (ICB BRCA CD8 T cells), < 2 (RPL decidual immune), < 2.4 (Zfp541-KO mTCA), or mapQ.p.adj < 0.1 (NRRA in vitro meiosis) — before downstream analyses. Transfer reference labels with ProjectSVR::LabelTransfer (ref.label.col, e.g. 'cell_type' or 'cluster.name'), optionally refining with MajorityVote (k=100, min.prop=0.3), and visualize with PlotProjection/AlluviaPlot. The rationale is that ProjectSVR learns the relationship between gene signature score matrices and low-dimensional reference embeddings via ensemble SVR (sub-sampled to 4000–10,000 reference cells, median-aggregated sub-models), making it platform-agnostic and integration-independent.

## Related tools

- **ProjectSVR** (Core framework: ComputeModuleScore, ProjectNewdata/MapQuery, MapQCPlot, LabelTransfer, MajorityVote, AbundanceTest for reference mapping) — https://github.com/jarninggau/ProjectSVR
- **Seurat** (Holds reference and query objects; provides the Seurat object structure and subsetting used throughout the mapping workflow) — https://github.com/satijalab/seurat
- **AUCell** (Computes gene activity scores (0–1) from rank expression; the gene-set scoring backend feeding the SVR projection)
- **UCell** (Alternative signature-score calculation package supported by ProjectSVR (v1.3 for R < 4.3)) — https://github.com/carmonalab/UCell
- **cNMF** (Derives cell-type gene sets for continuous cell states when building reference models (via the RunCNMF wrapper)) — https://github.com/dylkot/cNMF

## Examples

```
seu.q <- ProjectSVR::MapQuery(seu.q, reference, add.map.qual = TRUE, ncores = 10); seu.q <- subset(seu.q, mean.knn.dist < 1.6); seu.q <- ProjectSVR::LabelTransfer(seu.q, reference, ref.label.col = "cell_type")
```

## Evaluation signals

- LISI-based batch mixing: post-mapping mean LISI (~2.30) should exceed the uncorrected baseline (~1.06) for the 10x 3'v1/3'v2/5' PBMC cross-protocol task
- Mapping-quality filtering must precede downstream analysis: retain only cells passing the dataset-specific cutoff (mean.knn.dist < 1.6/2/2.4 or mapQ.p.adj < 0.1)
- Label-transfer accuracy/ARI vs manually curated annotations: ProjectSVR should reach accuracy and ARI above 0.92 on mTCA, comparable to Seurat, scArches, Symphony, ProjecTILs
- Pseudotime validation: Pearson correlation between predicted and published Pseudotime_dm should be approximately r = 0.973, and no Zfp541-KO cells should be assigned to diplotene
- Biological reproduction checks: RPL dNK shift toward dNK3 (dNK1 decreased), and CD8.c11 (GZMK+ Tex)/CD8.c12 (terminal Tex) enriched in ICB responders (positive log2(R/NR))

## Limitations

- Independent SVR models per UMAP dimension neglect correlations among embedding dimensions, which may limit the model's ability to learn joint structures.
- Reference building time is ~O(Nc^2); for scalability the reference is subsampled to 4000–10,000 cells with ensemble sub-models using median outputs.
- Projections may be misleading for query cells containing novel states absent in the reference; users must compare unsupervised clustering to mapping results.
- Performance declines when projecting across platforms (e.g. Smart-seq2 to 10x) due to differing UMI distributions and sequencing depth.
- Current implementation trains independent SVR models per dimension; multi-output SVR and deep regression backends (MLPs, Transformers) are future work only.

## Evidence

- [readme] README install instructions for UCell/AUCell dependencies: [evidence span withheld — anchor status: external_doc]
- [results] Benchmark comparison context: [evidence span withheld — anchor status: not_found]
