---
name: per-cell-annotation-table-export
description: Use when an imported single-cell study needs standardized, ontology-resolved
  cell type labels per individual cell so that cross-study expression comparisons
  in Cellenium are semantically integratable.
when_to_use_negative:
- When the study already carries standardized Cell Ontology cell type labels in obs
  and only needs to be imported — Cellenium already accepts cell annotations in H5AD
  format without re-running CellO
- When the goal is adding user-defined (non-ontology) cell annotations interactively
  via Cellenium's plot-selection feature rather than automated standardized classification
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CellO
  role: Pretrained Cell Ontology classifier applied to the prepared study so every
    individual cell receives a standardized cell type assignment
- name: Python 3
  role: Runtime for loading the AnnData study with the anndata API, running CellO,
    and writing cell_annotations.csv
- name: Cellenium
  role: Downstream application that consumes the H5AD study with obs-attached standardized
    annotations for semantically integrated cross-study comparisons
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:32e5998c2b44dd980fdb18447e977a551903d2e5d64328acc43d565244dc7758
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - CellO
  - Cellenium
  - Python 3
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# per-cell-annotation-table-export

## Summary

Reconstructs Cellenium's standardized cell typing step by running CellO's pretrained Cell Ontology classifier on a prepared AnnData study (e.g. the blood_covid demo study) and exporting one standardized annotation per cell to cell_annotations.csv.

## When to use

Use when an imported single-cell study needs standardized, ontology-resolved cell type labels per individual cell so that cross-study expression comparisons in Cellenium are semantically integratable.

## When NOT to use

- When the study already carries standardized Cell Ontology cell type labels in obs and only needs to be imported — Cellenium already accepts cell annotations in H5AD format without re-running CellO
- When the goal is adding user-defined (non-ontology) cell annotations interactively via Cellenium's plot-selection feature rather than automated standardized classification

## Inputs

- Prepared single-cell study AnnData file in H5AD format (e.g. blood_covid produced by the demo-preparation task)
- CellO pretrained Cell Ontology classifier resources (resources_v2.0.0.tar.gz from deweylab)
- Python 3 environment with the anndata and CellO packages installed

## Outputs

- cell_annotations.csv with one row per cell containing its standardized Cell Ontology cell type label
- AnnData object with standardized cell type labels attached to per-cell metadata (obs)

## How to apply

Load the prepared study (the demo-preparation task's blood_covid AnnData/H5AD file) with the anndata API in Python 3. Align the study's gene feature space with the gene set required by CellO's pretrained Cell Ontology classifier, downloading CellO's resources_v2.0.0 bundle if the classifier resources are missing (the Cellenium README documents this as a workaround for CellO issue #29). Run CellO so that every individual cell receives a standardized Cell Ontology cell type assignment, then attach these labels to the per-cell metadata (obs) of the AnnData object. Export exactly one row per cell with its standardized annotation to cell_annotations.csv; Cellenium ingests expression data and cell annotations in H5AD format, so the obs-attached labels must stay consistent with that object.

## Related tools

- **CellO** (Pretrained Cell Ontology classifier applied to the prepared study so every individual cell receives a standardized cell type assignment)
- **Python 3** (Runtime for loading the AnnData study with the anndata API, running CellO, and writing cell_annotations.csv)
- **Cellenium** (Downstream application that consumes the H5AD study with obs-attached standardized annotations for semantically integrated cross-study comparisons) — https://github.com/Bayer-Group/cellenium

## Examples

```
curl https://deweylab.biostat.wisc.edu/cell_type_classification/resources_v2.0.0.tar.gz >scratch/cello_resources/resources_v2.0.0.tar.gz; tar -C scratch/cello_resources -zxf scratch/cello_resources/resources_v2.0.0.tar.gz
```

## Evaluation signals

- cell_annotations.csv contains an annotation row for every cell present in the prepared study (row count matches the AnnData object's number of cells)
- No missing annotations: every exported label resolves to a valid Cell Ontology term
- Standardized labels are attached to the AnnData obs metadata so they remain consistent with the H5AD that Cellenium imports
- The gene feature space of the study was aligned with the gene set required by CellO's classifier before classification (no gene-mismatch failures)
- Exported labels are usable as semantically integratable annotations for cross-study expression comparisons in Cellenium

## Limitations

- CellO resource download requires a workaround for CellO issue #29: the resources_v2.0.0 bundle must be fetched manually and extracted into scratch/cello_resources before setup
- The annotation step depends on the upstream demo-preparation task having produced a correctly normalized, aligned AnnData study; initial study data processing via the notebooks takes a couple of hours to run

## Evidence

- [other] CellO-based standardized annotations feed Cellenium: [evidence span withheld — anchor status: not_found]
- [readme] H5AD ingestion format: [evidence span withheld — anchor status: external_doc]
- [readme] CellO resource workaround: [evidence span withheld — anchor status: external_doc]
- [abstract] Backend stack: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
