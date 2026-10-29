---
name: seurat-expression-matrix-extraction
description: Use when you have a Seurat object with cell-level metadata (e.g. 'milestone'
  bins along a pseudotime trajectory, stimulation time points, or cell-type labels)
  and need per-population expression matrices to feed into SCPA multisample pathway
  comparison.
when_to_use_negative:
- Input is a SingleCellExperiment object rather than a Seurat object — use `sce_extract()`
  / `compare_sce()` instead
- You already have manually subsetted expression matrices; SCPA accepts these directly
  in compare_pathways() without seurat_extract
- You want to run the comparison directly on the full Seurat object without pre-extracting
  matrices — use `compare_seurat()` instead
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCPA
  role: Provides seurat_extract() to subset Seurat object expression data by metadata;
    also supplies compare_pathways() that consumes the extracted matrices
  repo: https://github.com/jackbibby1/SCPA
- name: Seurat
  role: Loads and holds the single-cell object (e.g. naive_cd4.rds) whose metadata
    columns drive the seurat_extract subsetting; v5 layer system is supported
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:7b7e1dd57f17cff6c9c9b4a31adeafe2b4e984a4928c4d3d2b93de497a28dd79
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jackbibby1/SCPA
  tools:
  - ComplexHeatmap
  - SCPA
  - Seurat
  - dyno
  - tidyverse
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# seurat-expression-matrix-extraction

## Summary

Extract condition- or population-specific expression matrices from a Seurat object using SCPA's `seurat_extract()`, which subsets cells by metadata columns (e.g. timepoint, milestone, cell type) and returns raw or pseudocount-adjusted matrices ready for downstream `compare_pathways()` analysis.

## When to use

Use when you have a Seurat object with cell-level metadata (e.g. 'milestone' bins along a pseudotime trajectory, stimulation time points, or cell-type labels) and need per-population expression matrices to feed into SCPA multisample pathway comparison.

## When NOT to use

- Input is a SingleCellExperiment object rather than a Seurat object — use `sce_extract()` / `compare_sce()` instead
- You already have manually subsetted expression matrices; SCPA accepts these directly in compare_pathways() without seurat_extract
- You want to run the comparison directly on the full Seurat object without pre-extracting matrices — use `compare_seurat()` instead

## Inputs

- Seurat object (v3/v5; version auto-detected by SCPA v1.6+), e.g. naive_cd4.rds
- Metadata column(s) defining population/condition splits (e.g. 'milestone', timepoint, cell type, Status)
- Assay name (e.g. 'RNA') with counts or expression layer
- Optional pseudocount value

## Outputs

- Per-population expression matrices (genes x cells) subsetting the Seurat object by the specified metadata
- A list of expression matrices, one per milestone/condition sample, suitable as the `samples` argument of SCPA's compare_pathways()

## How to apply

Load the Seurat object (e.g. naive_cd4.rds) and first remove unwanted populations such as Treg cells, keeping only the naive/activated populations of interest. Call `seurat_extract(seu_obj, assay, meta1, value_meta1, meta2, value_meta2, pseudocount)`, specifying the metadata column (e.g. 'milestone' after `group_onto_nearest_milestones` binning, or a timepoint/status column) and the value(s) identifying each target population. For trajectory analyses, extract one matrix per milestone by looping `seurat_extract` over the 'milestone' metadata column, producing a list of samples that is passed directly to `compare_pathways()` against gene sets like combined_metabolic_pathways.csv. SCPA v1.6+ automatically detects Seurat object version, so Seurat v5 layer-based objects are handled transparently. A pseudocount is added by the function, so raw counts from the assay's counts layer can be supplied without manual log transformation.

## Related tools

- **SCPA** (Provides seurat_extract() to subset Seurat object expression data by metadata; also supplies compare_pathways() that consumes the extracted matrices) — https://github.com/jackbibby1/SCPA
- **Seurat** (Loads and holds the single-cell object (e.g. naive_cd4.rds) whose metadata columns drive the seurat_extract subsetting; v5 layer system is supported)

## Examples

```
scpa_out <- seurat_extract(seu_obj, meta1 = "milestone", value_meta1 = "1")
```

## Evaluation signals

- Each extracted matrix has columns equal to the number of cells matching the metadata filter (e.g. cells assigned to a given 'milestone' bin) and rows equal to genes in the assay
- Extracted cell counts sum to the total number of retained cells after Treg removal, confirming no cells are lost or duplicated across milestone samples
- SCPA's Seurat version auto-detection succeeds — no layer-related errors on Seurat v5 objects
- Downstream compare_pathways() runs without error on the list of extracted matrices, returning per-pathway qvals for every milestone sample
- Internal control: multisample pseudotime qvals can be cross-checked against a two-sample comparison of 0 h resting vs 12 h intermediate and 24 h activated naive CD4 T cells

## Limitations

- SCPA is R-based and Seurat/SCE-oriented; extraction functions assume the assay and metadata columns exist in the object
- Extraction depends on correct upstream metadata annotation — e.g. milestone bins must first be assigned via dyno's group_onto_nearest_milestones before looping seurat_extract
- Installation requires specific dependency versions (crossmatch 1.3.1, multicross 2.1.0), which can cause install errors if dependencies are missing

## Evidence

- [readme] SCPA input flexibility: [evidence span withheld — anchor status: external_doc]
