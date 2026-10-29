---
name: cross-tissue-qval-variance-ranking
description: Use when you have a multi-tissue single-cell atlas (e.g. blood, bone
  marrow, lymph node, lung) with a shared cell-type annotation and want to find pathway-level
  features unique to one tissue site rather than testing tissue pairs in isolation.
when_to_use_negative:
- Your dataset contains only one tissue site, so variance across tissue-vs-blood comparisons
  is undefined — use a two-population compare_pathways analysis instead.
- You need per-gene differential expression rather than pathway-level distribution
  changes; use DE tools rather than qval ranking.
- Your object is not a Seurat/SCE single-cell object with metadata you can split by
  tissue (e.g. it is already an aggregated feature table) — seurat_extract cannot
  subset it.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SCPA
  role: 'Runs the pathway perturbation statistics: seurat_extract pulls tissue/cell-type
    expression matrices and compare_pathways computes qvals for each tissue-vs-blood
    comparison'
  repo: https://github.com/jackbibby1/SCPA
- name: Seurat
  role: Holds the multi-tissue T cell object; DimPlot for QC visualization and SplitObject
    to split by tissue before extraction
- name: msigdbr
  role: Source of canonical pathway gene sets (KEGG, Reactome, BioCarta, Wiki, PID,
    HALLMARK) filtered to ~3,000 sets for comparison
- name: tidyverse/dplyr
  role: grepl-based gene-set filtering, joining qval tables with Reduce(full_join,
    ...), and computing per-pathway variance
- name: ggplot2
  role: Ranks pathways by variance (royalblue2 points, blank panel background) and
    renders DEFA1/DEFA3 VlnPlots by tissue
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:b1172ba34cf69601aa9cd858d808bb67ffdfd18d91bfe664c0bce9fb317cac61
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jackbibby1/SCPA
  tools:
  - R
  - SCPA
  - Seurat
  - dplyr
  - ggplot2
  - msigdbr
  - tidyverse
  - tidyverse/dplyr
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Cross-tissue qval variance ranking

## Summary

A systematic SCPA-based workflow that ranks pathway perturbations (qvals) by their variance across many tissue-vs-blood comparisons to expose tissue-specific pathway features, e.g. alpha-defensin expression in bone-marrow T cells.

## When to use

Use when you have a multi-tissue single-cell atlas (e.g. blood, bone marrow, lymph node, lung) with a shared cell-type annotation and want to find pathway-level features unique to one tissue site rather than testing tissue pairs in isolation.

## When NOT to use

- Your dataset contains only one tissue site, so variance across tissue-vs-blood comparisons is undefined — use a two-population compare_pathways analysis instead.
- You need per-gene differential expression rather than pathway-level distribution changes; use DE tools rather than qval ranking.
- Your object is not a Seurat/SCE single-cell object with metadata you can split by tissue (e.g. it is already an aggregated feature table) — seurat_extract cannot subset it.

## Inputs

- Seurat object with multi-tissue samples and cell-type metadata (e.g. szabo_t_cell.rds with a 'fine' cell-type column and 'tissue'/'stimulation' metadata)
- msigdbr Homo sapiens output (KEGG, Reactome, BioCarta, Wiki, PID subcategories plus HALLMARK gene sets)
- Metadata columns naming tissue site, stimulation state, and cell type

## Outputs

- Per-cell-type, per-tissue qval table of pathway perturbations vs blood (Pathway + qval columns, tissue/cell-type labeled)
- Full-join qval matrix across all tissue/cell-type comparisons
- Per-pathway variance ranking plot (e.g. tissue_variance.png)
- VlnPlots of candidate genes (DEFA1, DEFA3) grouped by tissue confirming tissue-restricted expression

## How to apply

Load the multi-tissue Seurat object (e.g. szabo_t_cell.rds) with SCPA, Seurat, and tidyverse and inspect it with DimPlot split by tissue/stimulation. Build ~3,000 gene sets from msigdbr Homo sapiens output by grepl-filtering gs_subcat for 'kegg', 'reactome', 'biocarta', 'wiki', 'pid' plus all HALLMARK sets, then format with format_pathways(). SplitObject by tissue and loop over all cell types in the 'fine' metadata column, pulling blood and tissue expression matrices with seurat_extract(meta1='fine', meta2='stimulation', value_meta2='none'). For each cell type run compare_pathways(list(blood, tissue), pathways) for each tissue site, keep only Pathway and qval columns labeled with tissue and cell type, and combine all tables with Reduce(full_join, ...). Compute per-pathway variance with apply(all_data, 1, var) and rank pathways by variance in ggplot2; the most variable pathways (e.g. antimicrobial peptide production, prostaglandin synthesis) are candidates for tissue-specific biology, which you then confirm with VlnPlots of the underlying genes (DEFA1/DEFA3) grouped by tissue. The rationale is that pathways whose qval is highly variable across tissue-vs-blood comparisons — not simply significant in one comparison — point to site-specific pathway activity; use qval as the primary statistic per SCPA guidance.

## Related tools

- **SCPA** (Runs the pathway perturbation statistics: seurat_extract pulls tissue/cell-type expression matrices and compare_pathways computes qvals for each tissue-vs-blood comparison) — https://github.com/jackbibby1/SCPA
- **Seurat** (Holds the multi-tissue T cell object; DimPlot for QC visualization and SplitObject to split by tissue before extraction)
- **msigdbr** (Source of canonical pathway gene sets (KEGG, Reactome, BioCarta, Wiki, PID, HALLMARK) filtered to ~3,000 sets for comparison)
- **tidyverse/dplyr** (grepl-based gene-set filtering, joining qval tables with Reduce(full_join, ...), and computing per-pathway variance)
- **ggplot2** (Ranks pathways by variance (royalblue2 points, blank panel background) and renders DEFA1/DEFA3 VlnPlots by tissue)

## Examples

```
scpa_out <- compare_pathways(list(blood, bm), pathways); all_qval <- Reduce(full_join, tissue_tables); variance <- apply(all_data, 1, var)
```

## Evaluation signals

- The variance-ranked plot places antimicrobial peptide production and prostaglandin synthesis among the top variable pathways.
- DEFA1 and DEFA3 VlnPlots show expression restricted to bone-marrow-derived T cells and absent from blood, lymph node, and lung T cells.
- The joined qval table has one row per pathway with columns for every tissue/cell-type comparison (blood vs bm, ln, lung), with no dropped pathways from the full_join.
- The gene-set input contains ~3,000 sets after filtering msigdbr gs_subcat for kegg/reactome/biocarta/wiki/pid plus all HALLMARK sets.
- qval, not fold change, is used as the ranking statistic per SCPA interpretation guidance.

## Limitations

- Paths can show large qvals without any fold change/enrichment (e.g. arachidonic acid metabolism), so ranking by qval alone can surface pathways with no directional shift.
- qval should be the primary statistic and pathway fold change/enrichment only a secondary informative value; interpreting fold change alone risks missing real perturbations.
- Findings are cell-type and tissue-specific to the atlas analyzed (Szabo T cell dataset: blood, bone marrow, lymph node, lung); generalization to other tissues requires repeating the loop within the new atlas.

## Evidence

- [methods] qval as primary statistic: "we suggest that the qval should be used as the primary statistic when judging biological relevance, and the pathway fold change/enrichment should only be a secondary informative value"
- [methods] loop SCPA over all cell types: "we just need to loop SCPA over all the cell types in the dataset"
- [methods] extract expression matrices: "we can use the `seurat_extract` function from the SCPA package, which takes a Seurat object and subsets the data based on the metadata"
- [abstract] alpha-defensin finding: "identifies alpha-defensin expression as a hallmark of bone-marrow-derived T cells"
- [methods] large qval without fold change: "we highlight arachidonic acid as one particular pathway that shows this pattern"
