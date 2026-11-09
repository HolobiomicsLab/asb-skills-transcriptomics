---
name: marker-file-preparation-panglaodb
description: Use when running marker-aware cell selection or active-learning initialization
  on scRNA-seq/snRNA-seq/CyTOF data and a marker gene set per expected cell type must
  be sourced, formatted, or validated.
when_to_use_negative:
- No prior knowledge of expected cell types exists and no marker resource covers the
  tissue — cluster-based leader mode or purely random initialization is the appropriate
  choice instead
- The downstream tool does not accept the YAML marker structure (positive/negative
  lists) and reformatted marker tables are not available
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: leader
  role: Consumes the prepared marker file via get_markers() for marker-aware adaptive
    reweighting (mode='marker_based')
  repo: https://github.com/camlab-bioml/leader
- name: scran
  role: findMarkers function derives data-driven marker files via differential expression
    between a cell line and all others
- name: PanglaoDB
  role: Database source of the 20 most sensitive markers per cell type for the Tabula
    datasets
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:07f1085f719eb6dbd503f62e171b2079a442ecd87b8daa364cb9d66919497717
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - PanglaoDB
  - leader
  - leader (R package)
  - scran
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# marker-file-preparation-panglaodb

## Summary

Prepare cell-type marker gene files (YAML for the leader package or marker tables) from author-provided knowledge, differential expression, or the 20 most sensitive PanglaoDB markers, so that marker-ranked cell selection can be applied to single-cell expression data.

## When to use

Use when running marker-aware cell selection or active-learning initialization on scRNA-seq/snRNA-seq/CyTOF data and a marker gene set per expected cell type must be sourced, formatted, or validated.

## When NOT to use

- No prior knowledge of expected cell types exists and no marker resource covers the tissue — cluster-based leader mode or purely random initialization is the appropriate choice instead
- The downstream tool does not accept the YAML marker structure (positive/negative lists) and reformatted marker tables are not available

## Inputs

- List of expected cell types for the dataset
- PanglaoDB database entries (to extract the 20 most sensitive markers per cell type)
- Processed log-normalized Seurat object for data-derived markers (scran findMarkers)
- Author-provided marker knowledge (e.g., personal communication for the snRNA-Seq PDAC dataset)

## Outputs

- Marker YAML file with cell_types, positive, and optional negative marker gene lists
- Marker files paired with each deposited single-cell dataset (Zenodo 10.5281/zenodo.10403475)

## How to apply

For each dataset, obtain a marker file from one of three sources used in the paper: (1) author-provided knowledge (e.g., the snRNA-Seq PDAC marker file adapted from personal communication with authors), (2) data-derived markers via differential expression using scran's findMarkers between a cell line and all others, or (3) the 20 most sensitive markers per cell type from PanglaoDB (used for the Tabula datasets). Format markers as a YAML file with a top-level 'cell_types' key listing each expected cell type with required 'positive' markers (e.g., CD3, CD8 for CD8 T cells) and optional 'negative' markers (e.g., B220). Load the file with leader's get_markers() and pass it to adaptive_reweighting with mode='marker_based'. For marker corruption tests, replace 10–100% of markers with random highly expressed genes not in the original file to quantify sensitivity to marker quality.

## Related tools

- **leader** (Consumes the prepared marker file via get_markers() for marker-aware adaptive reweighting (mode='marker_based')) — https://github.com/camlab-bioml/leader
- **scran** (findMarkers function derives data-driven marker files via differential expression between a cell line and all others)
- **PanglaoDB** (Database source of the 20 most sensitive markers per cell type for the Tabula datasets)

## Examples

```
markers <- get_markers(system.file("extdata", "markers.yml", package = "leader")); adaptive_reweighting(seu, markers, n_requested_cells = 40, mode = "marker_based")
```

## Evaluation signals

- YAML file parses with leader's get_markers() and the first row/key contains the word 'cell_types'
- Each expected cell type has at least one positive marker; negative markers are optional but correctly nested
- Marker genes are present in the rownames of the expression matrix of the target dataset (no mismatched gene identifiers, e.g., human vs mouse symbols)
- For PanglaoDB-derived files, exactly the 20 most sensitive markers per cell type were extracted
- In the marker corruption test, replacing 10–100% of markers should degrade annotation accuracy monotonically with corruption level; intact marker files should yield positive F1 improvement over random initialization

## Limitations

- Circularity: marker files for cell line data derived from differential expression between the same cell lines confound the evaluation
- A marker file for the snRNA-Seq dataset was adapted from personal communication with authors rather than a published, citable resource
- Dropout at the single-cell level can complicate annotation if specific markers are not expressed in individual cells

## Evidence

- [discussion] PanglaoDB marker source for Tabula datasets: "the marker files for the tabula datasets were created based on the 20 most sensitive markers from the PanglaoDB database45"
- [readme] YAML marker structure with positive and negative markers: "The marker yaml file must follow the following structure:

    cell_types:
      CD8 T cells:
        positive:
          - CD3
          - CD8
        negative:
          - B220"
- [readme] Loading and running marker-based leader selection: "markers <- get_markers(system.file("extdata", "markers.yml", package = "leader"))
adaptive_reweighting(seu, markers, n_requested_cells = 40, mode = "marker_based")"
- [discussion] Marker corruption test: "randomly corrupted 10, 25, 50, 75 and 100% of markers. Each of these randomly selected markers was replaced with a random gene not in the original marker file"
- [discussion] Circularity of marker definitions: "There is also circularity in the way some of our cell type markers are defined, especially for the cell line data, as these are derived from a differential expression between the cell lines"
