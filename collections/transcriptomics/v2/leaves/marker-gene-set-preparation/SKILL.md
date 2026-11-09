---
name: marker-gene-set-preparation
description: Use when you need cell-type marker gene sets (positive and negative)
  from the ScTypeDB database restricted to a specific tissue type (e.g. 'Immune system',
  'Liver', 'Eye', 'Brain') before running sctype_score or comparable marker-based
  annotation methods.
when_to_use_negative:
- The expression dataset's tissue has no matching tissueType entry in the ScTypeDB
  (fewer than 5 cell types were excluded from the database), so no candidate gene
  sets can be prepared
- Gene identifiers in the expression matrix are not approved gene symbols and cannot
  be converted, since the database was standardized to approved gene symbols
- You need de novo marker discovery without prior marker knowledge — ScType requires
  a marker database as background information
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ScType
  role: Provides the gene_sets_prepare() function and the ScTypeDB marker database
    used to build positive/negative gene sets
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: R
  role: Language in which gene_sets_prepare is sourced and executed
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: openxlsx
  role: R package used to read the ScTypeDB XLSX marker database
  repo: https://github.com/IanevskiAleksandr/sc-type
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_006
  - task_003
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:6fb9a0b46e0d584107ec06c1be5696a2da33401af51d181fb0f75a954f9a4228
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - SCINA
  - ScType
  - dplyr
  - openxlsx
  - scCATCH
  - scSorter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# marker-gene-set-preparation

## Summary

Prepares tissue-restricted positive and negative marker gene sets for ScType cell-type annotation by reading the ScTypeDB XLSX marker database (tissueType, cellName, geneSymbolmore1, geneSymbolmore2 columns) through the gene_sets_prepare() function, producing gs_positive and gs_negative lists per cell type.

## When to use

Use when you need cell-type marker gene sets (positive and negative) from the ScTypeDB database restricted to a specific tissue type (e.g. 'Immune system', 'Liver', 'Eye', 'Brain') before running sctype_score or comparable marker-based annotation methods.

## When NOT to use

- The expression dataset's tissue has no matching tissueType entry in the ScTypeDB (fewer than 5 cell types were excluded from the database), so no candidate gene sets can be prepared
- Gene identifiers in the expression matrix are not approved gene symbols and cannot be converted, since the database was standardized to approved gene symbols
- You need de novo marker discovery without prior marker knowledge — ScType requires a marker database as background information

## Inputs

- ScTypeDB XLSX marker database (columns: tissueType, cellName, geneSymbolmore1 positive markers, geneSymbolmore2 negative markers), e.g. ScTypeDB_full.xlsx or ScTypeDB_short.xlsx
- Tissue type string matching a database entry (e.g. 'Immune system', 'Liver', 'Pancreas', 'Kidney', 'Eye', 'Brain')
- Optionally an edited/customized ScTypeDB-format XLSX with specific cell-type marker rows removed or added

## Outputs

- gs_list$gs_positive — named list of positive marker gene vectors per candidate cell type
- gs_list$gs_negative — named list of negative marker gene vectors per candidate cell type

## How to apply

Source the gene_sets_prepare.R function from the sc-type GitHub repository and call it with the path to the ScTypeDB XLSX file (ScTypeDB_full.xlsx or ScTypeDB_short.xlsx) and the target tissue type string; the function returns a list with gs_positive and gs_negative gene sets per candidate cell type. Restrict to exactly one tissue type so that marker specificity scores S_i^t are computed within a biologically plausible candidate set — e.g. 'Immune system' for PBMC 3k, 'Liver' for GSE124395, retina-appropriate tissue for GSE63473. The database itself was compiled by integrating CellMarker and PanglaoDB after converting non-uniform gene IDs to approved gene symbols, removing low-evidence CellMarker genes (those with only one supporting reference), dropping genes absent across all PanglaoDB datasets, removing genes appearing in fewer than five clusters of a cell type, and excluding tissues with fewer than 5 cell types. For perturbation experiments (leave-one-cell-type-out), edit the ScTypeDB-format XLSX directly so the marker signatures of one cell type are removed at a time, then rerun gene_sets_prepare. The resulting gs and gs2 lists are passed to sctype_score(scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative); genes must match approved gene symbols in the expression matrix.

## Related tools

- **ScType** (Provides the gene_sets_prepare() function and the ScTypeDB marker database used to build positive/negative gene sets) — https://github.com/IanevskiAleksandr/sc-type
- **R** (Language in which gene_sets_prepare is sourced and executed) — https://github.com/IanevskiAleksandr/sc-type
- **openxlsx** (R package used to read the ScTypeDB XLSX marker database) — https://github.com/IanevskiAleksandr/sc-type

## Examples

```
source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/gene_sets_prepare.R"); gs_list <- gene_sets_prepare("ScTypeDB_full.xlsx", "Immune system")
```

## Evaluation signals

- gs_list contains both a gs_positive and gs_negative component, with one entry per cell type of the requested tissueType only
- Gene set members are approved gene symbols present in the scaled expression matrix (e.g. Seurat scale.data row names)
- Tissue restriction is respected — no cell types from other tissues appear in the returned gene sets
- For leave-one-cell-type-out perturbations, the edited database yields gene sets missing exactly the perturbed cell type while all others remain intact
- Downstream sctype_score runs without error and, on PBMC 3k with the Immune-system gene sets, reproduces expected cluster-level annotations including negative-marker-driven naïve vs effector T-cell separation

## Limitations

- Negative markers remain relatively few compared to positive markers, limiting annotation power
- Low-evidence CellMarker genes (single supporting reference) were removed during database construction, so the database is not exhaustive
- It could not be confirmed whether the CellMarker portion of the database incorporated markers from benchmark datasets (reproducibility gap)
- Drop-out events in scRNA-seq data can obscure true negative markers and are not corrected in this version

## Evidence

- [methods] gene_sets_prepare loads tissue-restricted positive/negative sets from the ScTypeDB: "source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/gene_sets_prepare.R")"
- [readme] README quick-start call with tissue restriction: "gs_list <- gene_sets_prepare("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/ScTypeDB_short.xlsx", "Immune system")"
- [discussion] Database was standardized to approved gene symbols: "we firstly converted the non-uniform gene IDs to approved gene symbols within and between the databases."
- [discussion] Low-evidence markers were removed during database curation: [evidence span withheld — anchor status: not_found]
