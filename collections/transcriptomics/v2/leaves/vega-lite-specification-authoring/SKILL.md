---
name: vega-lite-specification-authoring
description: Use when rendering a gene-by-cell-type expression comparison across studies,
  or when a task requires a declarative vega-lite spec (e.g., dotplot) with size/color
  channels bound to aggregate statistics derived from an scRNA-seq expression matrix.
when_to_use_negative:
- When the goal is interactive in-app exploration in Cellenium rather than reproducing
  a static figure — use the webapp's UMAP/violin plot views instead
- When plotting single-gene expression in UMAP space or grouped violin plots, which
  Cellenium renders with plotlyjs/seaborn rather than vega-lite
- When cell annotations are not yet harmonized to a shared vocabulary, since cross-study
  cell types will not map to shared rows
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3170
tools:
- name: vega-lite
  role: Declarative visualization grammar used to author the dotplot spec (dot mark;
    size=expressing fraction, color=mean expression); one of the plot-rendering libraries
    Cellenium uses
- name: Python 3
  role: Used to load the expression matrix, align CellO-standardized cell-type annotations,
    aggregate per-cell-type statistics, and render the spec to PNG
  repo: https://github.com/Bayer-Group/cellenium
- name: PostgreSQL
  role: Backend database of the cellenium deployment from which study expression data
    is accessed
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:47e7b9771d5a377d97a5d99c9ad1053c1ef998032cc16cd666c8d3bd4a73f517
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# vega-lite-specification-authoring

## Summary

Author a vega-lite specification to reproduce Cellenium-style scientific plots, specifically the use-case-2 cross-study dotplot where dot size encodes the fraction of expressing cells and dot color encodes mean expression, following the Seurat/Scanpy dotplot convention.

## When to use

Use when rendering a gene-by-cell-type expression comparison across studies, or when a task requires a declarative vega-lite spec (e.g., dotplot) with size/color channels bound to aggregate statistics derived from an scRNA-seq expression matrix.

## When NOT to use

- When the goal is interactive in-app exploration in Cellenium rather than reproducing a static figure — use the webapp's UMAP/violin plot views instead
- When plotting single-gene expression in UMAP space or grouped violin plots, which Cellenium renders with plotlyjs/seaborn rather than vega-lite
- When cell annotations are not yet harmonized to a shared vocabulary, since cross-study cell types will not map to shared rows

## Inputs

- Expression matrix (H5AD-derived) for one or more integrated scRNA-seq studies, e.g., the blood_covid example study
- Cell annotations aligned to the CellO-standardized cell-type vocabulary
- List of selected genes to display as dotplot columns

## Outputs

- use_case_2_dotplot.vl.json — the vega-lite dotplot specification
- use_case_2_dotplot.png — rendered dotplot image
- Per-cell-type aggregate table of mean expression and expressing fraction per gene (intermediate)

## How to apply

Load the expression matrix for the integrated studies (e.g., the public blood_covid example study) from the cellenium deployment or its data_import notebooks, then align cell annotations to the standardized CellO-predicted cell-type vocabulary so corresponding cell types from different studies share labels. Aggregate per cell type: compute mean expression per gene and the fraction of cells expressing each gene, mirroring the Seurat/Scanpy dotplot convention. Author a vega-lite spec with mark type 'dot' (or 'point'/'circle'), cell types encoded as rows and genes as columns, the size channel bound to expressing fraction, and the color channel bound to mean expression. Render the specification to a PNG image and persist the spec as a .vl.json file; this mirrors how Cellenium itself renders plots server-side via vega-lite among plotlyjs, seaborn, and nivo.rocks.

## Related tools

- **vega-lite** (Declarative visualization grammar used to author the dotplot spec (dot mark; size=expressing fraction, color=mean expression); one of the plot-rendering libraries Cellenium uses)
- **Python 3** (Used to load the expression matrix, align CellO-standardized cell-type annotations, aggregate per-cell-type statistics, and render the spec to PNG) — https://github.com/Bayer-Group/cellenium
- **PostgreSQL** (Backend database of the cellenium deployment from which study expression data is accessed) — https://github.com/Bayer-Group/cellenium

## Examples

```
make reset_database test_studydata_import; conda activate cellenium_import && jupyter nbconvert --to notebook --execute blood_covid.ipynb
```

## Evaluation signals

- The rendered PNG displays cell types as rows and genes as columns across all aligned studies
- Dot size channel is bound to the fraction of cells expressing each gene (values in [0,1])
- Dot color channel is bound to mean expression per cell type
- Cell-type labels correspond to the CellO-standardized vocabulary so homologous types from different studies share rows
- The saved use_case_2_dotplot.vl.json is a valid vega-lite spec with mark type dot and re-renders to the same figure

## Limitations

- Requires cell annotations to be standardized (CellO) beforehand; unaligned studies will not produce comparable rows
- The cellenium stack setup (docker compose, conda env cellenium_import, make reset_database/test_studydata_import) can take hours for normal_studydata imports, limiting quick reproduction
- CellO resource files need a workaround (deweylab issue #29) when fetching resources_v2.0.0.tar.gz

## Evidence

- [other] Cellenium's use-case-2 cross-study expression comparison is rendered as a vega-lite dotplot, a visualization type known from Seurat and Scanpy: [evidence span withheld — anchor status: not_found]
- [abstract] plots are generated using plotlyjs, seaborn, vega-lite, and nivo.rocks: "plots are generated using plotlyjs, seaborn, vega-lite, and nivo.rocks"
- [readme] find marker genes in all imported studies and qualitatively compare gene expression across studies: "* find marker genes in all imported studies and qualitatively compare gene expression across studies"
- [readme] Cellenium imports scRNA expression data and cell annotations in H5AD format.: "Cellenium imports scRNA expression data and cell annotations in H5AD format. We provide jupyter notebooks for
downloading some publicly available scRNA studies, normalize the data if necessary"
- [intro] the public study example blood_covid.ipynb: "the public study example blood_covid.ipynb"
