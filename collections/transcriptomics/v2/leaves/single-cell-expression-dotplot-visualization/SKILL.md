---
name: single-cell-expression-dotplot-visualization
description: Use when comparing expression of selected genes across multiple scRNA-seq
  studies whose cell annotations must be aligned to a shared standardized cell-type
  vocabulary (CellO) and a concise cross-study overview plot is required.
when_to_use_negative:
- The goal is within-study exploration (single-gene UMAP overlay or grouped violin
  plots), which Cellenium provides through other plot types rather than the cross-study
  dotplot
- Cell annotations have not been standardized to a shared vocabulary (CellO), so cell
  types from different studies cannot be aligned row-wise
- A quantitative statistical comparison of differential expression across studies
  is needed rather than a qualitative expression overview
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: vega-lite
  role: Declarative plot library used to author and render the dotplot specification
    (dot mark, size = expressing fraction, color = mean expression)
  repo: https://github.com/Bayer-Group/cellenium
- name: Python 3
  role: Used for loading expression matrices, aligning CellO-standardized cell annotations,
    and aggregating mean expression and expressing-fraction per cell type
  repo: https://github.com/Bayer-Group/cellenium
- name: PostgreSQL
  role: Backend database storing imported scRNA study data queried via GraphQL for
    cross-study analysis
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:80cdc0fac2e3c3c468ae1fe541f4ec819c467753b8903112fbc69b86c2f0a11d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - PostgreSQL
  - Python 3
  - vega-lite
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# single-cell-expression-dotplot-visualization

## Summary

Renders a vega-lite dotplot that qualitatively compares gene expression across single-cell studies, with cell types standardized to the CellO vocabulary as rows, genes as columns, dot size encoding the fraction of expressing cells and dot color encoding mean expression (the Seurat/Scanpy dotplot convention used in Cellenium's use case 2).

## When to use

Use when comparing expression of selected genes across multiple scRNA-seq studies whose cell annotations must be aligned to a shared standardized cell-type vocabulary (CellO) and a concise cross-study overview plot is required.

## When NOT to use

- The goal is within-study exploration (single-gene UMAP overlay or grouped violin plots), which Cellenium provides through other plot types rather than the cross-study dotplot
- Cell annotations have not been standardized to a shared vocabulary (CellO), so cell types from different studies cannot be aligned row-wise
- A quantitative statistical comparison of differential expression across studies is needed rather than a qualitative expression overview

## Inputs

- Normalized scRNA-seq expression matrices in H5AD format for the studies being compared (e.g. from cellenium's data_import notebooks)
- Cell annotations / CellO-predicted standardized cell-type labels per study
- List of selected genes (e.g. marker genes or differentially expressed genes) to compare

## Outputs

- Rendered dotplot image, e.g. use_case_2_dotplot.png
- Vega-lite specification file, e.g. use_case_2_dotplot.vl.json

## How to apply

Load the expression matrices for the integrated studies (e.g. the public blood_covid example study distributed via the cellenium data_import notebooks) in Python 3. Align cell annotations to the standardized cell-type vocabulary predicted by CellO so corresponding cell types from different studies map to shared labels; if preparing CellO resources, fetch resources_v2.0.0.tar.gz as described in the cellenium README (workaround for deweylab/CellO issue 29). Aggregate per cell type following the Seurat/Scanpy dotplot convention: compute mean expression and the fraction of cells expressing each selected gene. Author a vega-lite specification using mark type 'dot' (or 'circle') with cell types as rows and genes as columns, size bound to expressing-fraction and color bound to mean expression. Render the specification to PNG (e.g. use_case_2_dotplot.png) and save the spec as a .vl.json file.

## Related tools

- **vega-lite** (Declarative plot library used to author and render the dotplot specification (dot mark, size = expressing fraction, color = mean expression)) — https://github.com/Bayer-Group/cellenium
- **Python 3** (Used for loading expression matrices, aligning CellO-standardized cell annotations, and aggregating mean expression and expressing-fraction per cell type) — https://github.com/Bayer-Group/cellenium
- **PostgreSQL** (Backend database storing imported scRNA study data queried via GraphQL for cross-study analysis) — https://github.com/Bayer-Group/cellenium

## Examples

```
{"mark": "dot", "encoding": {"y": {"field": "cell_type"}, "x": {"field": "gene"}, "size": {"field": "fraction_expressing"}, "color": {"field": "mean_expression"}}}
```

## Evaluation signals

- The rendered dotplot displays the aligned standardized (CellO) cell types across studies as rows with genes as columns
- The size channel is correctly bound to the fraction of cells expressing each gene and the color channel to mean expression
- Output files exist: a rendered PNG (use_case_2_dotplot.png) and the saved vega-lite spec (use_case_2_dotplot.vl.json)
- The plot reproduces the cross-study qualitative expression comparison shown in the paper's use case 2

## Limitations

- CellO data preparation requires a documented workaround (fetching resources_v2.0.0.tar.gz) due to deweylab/CellO issue 29
- The workflow depends on studies already imported into the cellenium stack (H5AD ingestion, PostgreSQL schema setup), which can take a couple of hours to run for real-life studies with full cells and genes

## Evidence

- [abstract] plots are generated using plotlyjs, seaborn, vega-lite, and nivo.rocks: "plots are generated using plotlyjs, seaborn, vega-lite, and nivo.rocks"
- [readme] find marker genes in all imported studies and qualitatively compare gene expression across studies: "find marker genes in all imported studies and qualitatively compare gene expression across studies"
- [intro] the public study example blood_covid.ipynb: "the public study example blood_covid.ipynb"
- [readme] Cellenium imports scRNA expression data and cell annotations in H5AD format: [evidence span withheld — anchor status: external_doc]
