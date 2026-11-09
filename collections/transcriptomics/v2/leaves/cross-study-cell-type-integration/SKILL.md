---
name: cross-study-cell-type-integration
description: Use when multiple scRNA-seq studies (e.g. H5AD inputs like the public
  blood_covid example) must be compared at cell-type level and their heterogeneous
  annotations need to be mapped to a shared standardized Cell Ontology vocabulary.
when_to_use_negative:
- The goal is single-study exploration (UMAP, per-study differential expression, violin
  plots) rather than comparison across aligned studies
- Cell annotations are already standardized to the Cell Ontology vocabulary shared
  by all studies, so re-running CellO would add no semantic alignment
- Only raw, unnormalized scRNA data is available — the studies must first be prepared
  with normalized expression, DE genes and UMAP as in Cellenium's data-import notebooks
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: CellO
  role: Assigns standardized Cell Ontology cell type annotations to each individual
    cell of the integrated study, enabling semantically integratable labels
- name: Python 3
  role: Runs CellO classification, per-cell-type aggregation (mean expression, expressing
    fraction), and vega-lite spec authoring
  repo: https://github.com/Bayer-Group/cellenium
- name: vega-lite
  role: Renders the cross-study dotplot with size bound to expressing fraction and
    color to mean expression
- name: PostgreSQL
  role: Backend database storing imported expression data and annotations in the Cellenium
    stack
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:3b9d544487fe7893505ee1f054648fdfc759bdb503e26fdd310eb445e5d837bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - CellO
  - PostgreSQL
  - Python 3
  - vega-lite
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cross-study-cell-type-integration

## Summary

Standardize cell annotations of imported single-cell studies with CellO (Cell Ontology) and compare gene expression across studies using a Seurat/Scanpy-style dotplot, as implemented in Cellenium's cross-study analysis section.

## When to use

Use when multiple scRNA-seq studies (e.g. H5AD inputs like the public blood_covid example) must be compared at cell-type level and their heterogeneous annotations need to be mapped to a shared standardized Cell Ontology vocabulary.

## When NOT to use

- The goal is single-study exploration (UMAP, per-study differential expression, violin plots) rather than comparison across aligned studies
- Cell annotations are already standardized to the Cell Ontology vocabulary shared by all studies, so re-running CellO would add no semantic alignment
- Only raw, unnormalized scRNA data is available — the studies must first be prepared with normalized expression, DE genes and UMAP as in Cellenium's data-import notebooks

## Inputs

- Prepared AnnData/H5AD study files (e.g. the blood_covid example study) with expression matrix and per-cell metadata
- CellO pretrained Cell Ontology classifier resources (resources_v2.0.0.tar.gz)
- List of selected genes for cross-study expression comparison

## Outputs

- cell_annotations.csv: one row per cell with its standardized Cell Ontology annotation
- use_case_2_dotplot.png: rendered cross-study expression dotplot
- use_case_2_dotplot.vl.json: vega-lite dotplot specification

## How to apply

Load each prepared study as an AnnData/H5AD object and align its gene feature space with the gene set required by CellO's pretrained Cell Ontology classifier (resources_v2.0.0 downloaded from deweylab, per the README workaround for CellO issue #29). Run CellO so every individual cell receives a standardized Cell Ontology type, and attach these labels to the AnnData obs metadata so corresponding cell types from different studies map to shared labels. For the comparison, aggregate per aligned cell type: compute mean expression and the fraction of expressing cells for each selected gene, following the Seurat/Scanpy dotplot convention. Author a vega-lite dotplot (mark type 'dot') encoding size as expressing fraction and color as mean expression, with standardized cell types as rows and genes as columns, mirroring Cellenium's use-case-2 visualization. The Cellenium stack this reproduces runs a PostgreSQL backend accessed via a GraphQL API, with server-side plots generated with plotlyjs, seaborn, vega-lite, and nivo.rocks.

## Related tools

- **CellO** (Assigns standardized Cell Ontology cell type annotations to each individual cell of the integrated study, enabling semantically integratable labels)
- **Python 3** (Runs CellO classification, per-cell-type aggregation (mean expression, expressing fraction), and vega-lite spec authoring) — https://github.com/Bayer-Group/cellenium
- **vega-lite** (Renders the cross-study dotplot with size bound to expressing fraction and color to mean expression)
- **PostgreSQL** (Backend database storing imported expression data and annotations in the Cellenium stack)

## Examples

```
curl https://deweylab.biostat.wisc.edu/cell_type_classification/resources_v2.0.0.tar.gz >scratch/cello_resources/resources_v2.0.0.tar.gz && tar -C scratch/cello_resources -zxf scratch/cello_resources/resources_v2.0.0.tar.gz
```

## Evaluation signals

- cell_annotations.csv contains an annotation row for every cell in the prepared study and every label resolves to a valid Cell Ontology term with no missing annotations
- The rendered dotplot shows aligned standardized cell types across studies, with dot size bound to expressing fraction and color to mean expression
- Per-cell-type aggregates follow the Seurat/Scanpy dotplot convention: size encodes fraction of expressing cells, color encodes mean expression
- Corresponding cell types from different studies collapse onto shared CellO labels rather than study-specific annotation strings

## Limitations

- CellO data files require a workaround download (CellO issue #29) before study preparation notebooks can run
- Full data preparation and import via the docker-compose stack can take a couple of hours, especially for real-life studies with full amounts of cells and genes
- Annotations must resolve to valid Cell Ontology terms; user-defined cell types added interactively in Cellenium are not part of this standardized cross-study path

## Evidence

- [results] CellO standardization: [evidence span withheld — anchor status: not_found]
- [results] dotplot: [evidence span withheld — anchor status: not_found]
- [abstract] plotting stack: "plots are generated using plotlyjs, seaborn, vega-lite, and nivo.rocks"
- [readme] H5AD import: [evidence span withheld — anchor status: external_doc]
- [readme] CellO resources workaround: [evidence span withheld — anchor status: external_doc]
