---
name: cell-type-annotation-standardization
description: Use when combining scRNA-seq studies whose free-text cell annotations
  differ, e.g. before rendering a cross-study comparison of expression across aligned
  cell types in Cellenium (H5AD-imported studies with ontologized metadata).
when_to_use_negative:
- The analysis is within a single study only — Cellenium's per-study features (UMAP
  exploration, differential expression on annotated clusters) do not require cross-study
  label alignment.
- Cell annotations need to be created de novo from plot selections rather than harmonized
  — use Cellenium's interactive cell annotation feature for adding user-defined cell
  types instead.
- Input studies already share one ontologized cell-type vocabulary, so remapping would
  be a no-op.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: cellenium
  role: Platform whose backend (PostgreSQL/Python 3/GraphQL) ingests H5AD studies
    and whose cross-study comparisons rely on CellO-standardized cell-type labels
  repo: https://github.com/Bayer-Group/cellenium
- name: CellO
  role: Supplies the standardized cell-type classification resources (resources_v2.0.0.tar.gz)
    used to map per-study annotations onto a shared vocabulary
- name: Python 3
  role: Used in data-import notebooks to normalize studies and prepare annotations/UMAP/differential-expression
    data for Cellenium
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:da2ec8ae63ee39e7c1a252a19478317a2d2a2f496e3a92e441912bd6d905bb94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - CellO
  - Python 3
  - cellenium
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cell-type-annotation-standardization

## Summary

Aligns heterogeneous cell annotations from different scRNA-seq studies to a shared, standardized cell-type vocabulary — the CellO-predicted ontology used by Cellenium — so that corresponding cell types from different studies map to shared labels and can be compared within one visualization.

## When to use

Use when combining scRNA-seq studies whose free-text cell annotations differ, e.g. before rendering a cross-study comparison of expression across aligned cell types in Cellenium (H5AD-imported studies with ontologized metadata).

## When NOT to use

- The analysis is within a single study only — Cellenium's per-study features (UMAP exploration, differential expression on annotated clusters) do not require cross-study label alignment.
- Cell annotations need to be created de novo from plot selections rather than harmonized — use Cellenium's interactive cell annotation feature for adding user-defined cell types instead.
- Input studies already share one ontologized cell-type vocabulary, so remapping would be a no-op.

## Inputs

- scRNA-seq study expression matrices and cell annotations in H5AD format
- CellO classification resources archive (resources_v2.0.0.tar.gz from deweylab resources_v2.0.0)
- per-study author cell-type labels (e.g. from public example studies such as blood_covid.ipynb)

## Outputs

- cell annotations remapped to CellO-standardized cell-type labels shared across studies
- per-cell-type aggregates: mean expression and fraction of expressing cells per gene
- aligned cell-type sets ready for cross-study plots (e.g. a vega-lite dotplot spec .vl.json and rendered PNG)

## How to apply

First stage the CellO classification resources by downloading resources_v2.0.0.tar.gz from deweylab into scratch/cello_resources and extracting it (a documented workaround for deweylab/CellO issue #29). Import each study's expression data and cell annotations in H5AD format into the Cellenium PostgreSQL backend, then align each study's cell annotations to the standardized cell-type vocabulary predicted by CellO so corresponding cell types from different studies resolve to shared labels. Aggregate per aligned cell type by computing mean expression and the fraction of cells expressing each selected gene, following the Seurat/Scanpy dotplot convention. The rationale is that Cellenium's cross-study features — like its use-case-2 dotplot over tissues — only produce interpretable comparisons when cell types are on one ontology-aligned label set rather than per-study author annotations.

## Related tools

- **cellenium** (Platform whose backend (PostgreSQL/Python 3/GraphQL) ingests H5AD studies and whose cross-study comparisons rely on CellO-standardized cell-type labels) — https://github.com/Bayer-Group/cellenium
- **CellO** (Supplies the standardized cell-type classification resources (resources_v2.0.0.tar.gz) used to map per-study annotations onto a shared vocabulary)
- **Python 3** (Used in data-import notebooks to normalize studies and prepare annotations/UMAP/differential-expression data for Cellenium) — https://github.com/Bayer-Group/cellenium

## Examples

```
mkdir -p scratch/cello_resources && curl https://deweylab.biostat.wisc.edu/cell_type_classification/resources_v2.0.0.tar.gz >scratch/cello_resources/resources_v2.0.0.tar.gz && tar -C scratch/cello_resources -zxf scratch/cello_resources/resources_v2.0.0.tar.gz
```

## Evaluation signals

- Corresponding cell types from different studies resolve to identical standardized labels after alignment.
- Per aligned cell type, both aggregates exist: mean expression and expressing-cell fraction per gene (Seurat/Scanpy dotplot convention).
- A rendered cross-study dotplot shows aligned standardized cell types across studies with size bound to expressing fraction and color bound to mean expression.
- The CellO resources_v2.0.0 archive downloads and extracts into scratch/cello_resources without errors, and H5AD studies import through the docker-compose stack.

## Limitations

- CellO resources require a documented workaround (deweylab/CellO issue #29) to stage; without it the annotation standardization path fails.
- Full study imports ('normal_studydata_import') process studies with full amounts of cells and genes and can take a couple of hours to run.
- The toolchain assumes H5AD input with pre-computed annotations; studies lacking cell annotations cannot be aligned without prior annotation work.

## Evidence

- [readme] CellO resources staging workaround: [evidence span withheld — anchor status: external_doc]
- [readme] H5AD ingestion of annotations: [evidence span withheld — anchor status: external_doc]
- [abstract] interactive cell annotation: "An interactive cell annotation feature allows for adding user-defined cell types"
- [intro] ontologized metadata rationale: "organize and semantically find scRNA studies with ontologized metadata for tissues and diseases"
