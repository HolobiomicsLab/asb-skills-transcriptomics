---
name: tsne-embedding-visualization
description: Use when comparing batch-correction methods on multi-batch scRNA-seq
  matrices and a visual QC of batch mixing versus cell-type separation is needed to
  accompany quantitative metrics.
when_to_use_negative:
- When only quantitative integration metrics are required and no visual QC panels
  are part of the deliverable
- When the input is already a low-dimensional embedding (e.g. a PCA or JIVE joint
  estimate) with no raw multi-batch count matrix to reprocess
- When the dataset is too large for the JIVE pipeline, since the article notes JIVE's
  original bulk-oriented implementation is 'computationally infeasible for large-scale
  single-cell sequencing datasets'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: JIVE (scJIVE)
  role: Provides the joint low-rank integrated representation whose t-SNE panels visualize
    batch mixing vs. cell-type preservation
  repo: https://github.com/oconnell-statistics-lab/scJIVE
- name: Seurat v5
  role: Comparator integration method whose corrected representation is embedded and
    visualized alongside JIVE
- name: Harmony
  role: Comparator alignment method whose corrected representation is embedded and
    visualized alongside JIVE
- name: R
  role: Language in which the t-SNE embeddings, panels, and benchmarking figure are
    produced
- name: gplots
  role: Plotting dependency used for rendering the JIVE estimates and figure panels
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:97e360bd553811861f0706e009b8e46f530d45d4597776f56e33424dccf56f6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oconnell-statistics-lab/scJIVE
  tools:
  - Harmony
  - JIVE (scJIVE)
  - R
  - Seurat v5
  - gplots
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# tsne-embedding-visualization

## Summary

Generate t-SNE embeddings of raw and batch-corrected (JIVE/scJIVE, Seurat v5, Harmony) scRNA-seq representations, rendered as panels colored by batch and by cell type/cluster, to qualitatively assess batch mixing versus cell-type preservation alongside kBET, ASW, and LISI metrics.

## When to use

Use when comparing batch-correction methods on multi-batch scRNA-seq matrices and a visual QC of batch mixing versus cell-type separation is needed to accompany quantitative metrics.

## When NOT to use

- When only quantitative integration metrics are required and no visual QC panels are part of the deliverable
- When the input is already a low-dimensional embedding (e.g. a PCA or JIVE joint estimate) with no raw multi-batch count matrix to reprocess
- When the dataset is too large for the JIVE pipeline, since the article notes JIVE's original bulk-oriented implementation is 'computationally infeasible for large-scale single-cell sequencing datasets'

## Inputs

- Log-normalized scRNA-seq expression matrix (top 2000 variable genes, e.g. 2000 genes x 1867 cells Bacher CD4+ T-cell; 2000 genes x 2533 cells Zilionis mouse lung)
- Batch labels per cell (e.g. two of the fifteen Bacher batches; two Zilionis batches)
- Cell-type/cluster annotations per cell (e.g. B-cell and T-cell clusters)
- Low-dimensional integrated representations from jive() (joint structure), Seurat v5 integration, and Harmony alignment

## Outputs

- t-SNE panels colored by batch for raw, JIVE, Seurat v5, and Harmony embeddings
- t-SNE panels colored by cell type/cluster for the same four embeddings
- Figure 3- and Figure 5-style benchmarking PDF combining the panels
- Embeddings feeding quantitative metrics (kBET, ASW, LISI) compiled into metrics_table.csv

## How to apply

After preprocessing (log-normalized counts, top 2000 variable genes — e.g. 2000 genes x 1867 cells for the Bacher CD4+ T-cell data or 2000 genes x 2533 cells for the Zilionis mouse lung data), compute t-SNE embeddings for four representations: the raw (uncorrected) matrix as baseline, the JIVE/scJIVE joint low-rank structure, the Seurat v5 integrated representation, and the Harmony-aligned representation. Render t-SNE panels for each method colored once by batch label and once by cell-type/cluster annotation, in the style of Figures 3 and 5 of the article, using R with gplots for supporting plotting. Interpret panels jointly with the metric table (kBET acceptance rate, ASW, LISI, PVCA shares): on the Bacher data expect JIVE panels to show the strongest batch mixing (best kBET acceptance rate, Seurat v5 close behind, Harmony barely above raw), while on the Zilionis data expect JIVE and Seurat v5 panels to show cells of the same type separated despite good batch-mixing LISI. Treat the raw-data panels as the reference against which mixing gains and cell-type losses are judged.

## Related tools

- **JIVE (scJIVE)** (Provides the joint low-rank integrated representation whose t-SNE panels visualize batch mixing vs. cell-type preservation) — https://github.com/oconnell-statistics-lab/scJIVE
- **Seurat v5** (Comparator integration method whose corrected representation is embedded and visualized alongside JIVE)
- **Harmony** (Comparator alignment method whose corrected representation is embedded and visualized alongside JIVE)
- **R** (Language in which the t-SNE embeddings, panels, and benchmarking figure are produced)
- **gplots** (Plotting dependency used for rendering the JIVE estimates and figure panels)

## Examples

```
devtools::install_github("oconnell-statistics-lab/scJIVE"); library(scJIVE); res <- jive(list(batch1 = X1, batch2 = X2)); showPCA(res, n = 2)
```

## Evaluation signals

- t-SNE panels exist for all four representations (raw, JIVE, Seurat v5, Harmony), each colored by both batch and cell type/cluster
- kBET acceptance rates computed on the embeddings fall within 0-1 and JIVE's kBET on the Bacher data exceeds Seurat v5's, with Harmony only slightly above raw
- On the Zilionis data, JIVE and Seurat v5 show cell-type LISI worse than raw data, matching the expected cell-type preservation failure visible in the panels
- For the Bacher workflow, PVCA on the corrected output yields a batch share near ~4.9% (accept ~4-6%) and cluster share near ~9.3% before panels are finalized
- The benchmarking figure file (Figure 5-style PDF) is produced and the metrics_table.csv with kBET/ASW/LISI columns is populated from the same embeddings shown in the panels

## Limitations

- t-SNE panels are qualitative; on the Zilionis data good batch-mixing appearance (best-in-class batch LISI for JIVE and Harmony) coexisted with failed cell-type preservation, so panels must be read with LISI/ASW/kBET values
- JIVE's original implementation is computationally infeasible for large-scale single-cell datasets; only the enhanced scJIVE implementation should be used to generate embeddings for large data
- JIVE performed best with balanced batch sizes, so t-SNE mixing impressions may be unreliable for strongly unbalanced batches

## Evidence

- [methods] we consider the "point cloud" view provided by the `showPCA` function: "we consider the "point cloud" view provided by the `showPCA` function"
- [methods] We can display the actual JIVE estimates, in the form of low-rank matrix approximations, as heatmaps using the `showHeatmaps` function: "We can display the actual JIVE estimates, in the form of low-rank matrix approximations, as heatmaps using the `showHeatmaps` function"
- [abstract] scJIVE aims to decompose scRNA-seq datasets into a joint structure capturing the true biological variability and individual structures which capture technical variability: "scJIVE aims to decompose scRNA-seq datasets into a joint structure capturing the true biological variability and individual structures, which capture technical variability"
- [readme] devtools::install_github("oconnell-statistics-lab/scJIVE"): [evidence span withheld — anchor status: external_doc]
