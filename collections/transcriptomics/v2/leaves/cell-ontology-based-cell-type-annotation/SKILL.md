---
name: cell-ontology-based-cell-type-annotation
description: Use when a prepared H5AD study (e.g. the blood_covid AnnData file) needs
  standardized, ontology-resolved cell type labels per cell — e.g. before cross-study
  expression comparison in Cellenium — rather than ad hoc cluster labels.
when_to_use_negative:
- The study has not yet been normalized and processed by Cellenium's study data preparation
  notebooks (differential genes, UMAP, etc.), so the H5AD is not ready for annotation
  import
- The goal is to add user-defined, free-form cell types via Cellenium's interactive
  annotation feature rather than standardized ontology terms
- The input lacks an expression matrix (metadata-only), since CellO classifies cells
  from the gene feature space
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellO
  role: Pretrained Cell Ontology classifier applied to the prepared study to assign
    standardized cell type annotations to each individual cell
  repo: https://github.com/deweylab/CellO/issues/29
- name: Python 3
  role: Runtime for loading the AnnData study, aligning the gene feature space, running
    CellO, and writing cell_annotations.csv
- name: anndata
  role: API used to load the prepared H5AD file and attach standardized labels to
    per-cell obs metadata
- name: Cellenium
  role: Downstream visual analytics app that consumes the CellO-based standardized
    annotations for cross-study expression comparison
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:62e3b0669e6ae3657a7c6c652b1bdaffda42ddec594cd965251da78994775049
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellO
  - Cellenium
  - Python 3
  - anndata
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cell-ontology-based-cell-type-annotation

## Summary

Assigns standardized Cell Ontology cell type labels to every individual cell of a prepared single-cell study using CellO's pretrained classifier, as done in Cellenium so that annotations become semantically integratable for cross-study expression comparisons.

## When to use

Use when a prepared H5AD study (e.g. the blood_covid AnnData file) needs standardized, ontology-resolved cell type labels per cell — e.g. before cross-study expression comparison in Cellenium — rather than ad hoc cluster labels.

## When NOT to use

- The study has not yet been normalized and processed by Cellenium's study data preparation notebooks (differential genes, UMAP, etc.), so the H5AD is not ready for annotation import
- The goal is to add user-defined, free-form cell types via Cellenium's interactive annotation feature rather than standardized ontology terms
- The input lacks an expression matrix (metadata-only), since CellO classifies cells from the gene feature space

## Inputs

- Prepared single-cell study in H5AD/AnnData format (e.g. blood_covid study produced by the demo-preparation task)
- CellO pretrained classifier resources (resources_v2.0.0.tar.gz from deweylab.biostat.wisc.edu)

## Outputs

- AnnData object whose obs metadata carries a standardized Cell Ontology cell type label for every cell
- cell_annotations.csv with one row per cell containing its standardized annotation

## How to apply

First stage CellO's resources by downloading resources_v2.0.0.tar.gz into scratch/cello_resources and extracting it (a documented workaround for deweylab/CellO issue #29). Load the prepared blood_covid AnnData file with the anndata API in Python 3 and align its gene feature space with the gene set required by CellO's pretrained Cell Ontology classifier — feature alignment is the critical precondition, since the classifier scores only on genes it was trained on. Run CellO so that every individual cell receives a standardized Cell Ontology assignment, then attach these labels to the per-cell metadata (obs) of the AnnData object. Export one row per cell with its standardized annotation to cell_annotations.csv. The rationale is that Cellenium's cross-study comparisons rely on CellO-predicted standardized types, not on free-text or cluster-based labels.

## Related tools

- **CellO** (Pretrained Cell Ontology classifier applied to the prepared study to assign standardized cell type annotations to each individual cell) — https://github.com/deweylab/CellO/issues/29
- **Python 3** (Runtime for loading the AnnData study, aligning the gene feature space, running CellO, and writing cell_annotations.csv)
- **anndata** (API used to load the prepared H5AD file and attach standardized labels to per-cell obs metadata)
- **Cellenium** (Downstream visual analytics app that consumes the CellO-based standardized annotations for cross-study expression comparison) — https://github.com/Bayer-Group/cellenium

## Examples

```
curl https://deweylab.biostat.wisc.edu/cell_type_classification/resources_v2.0.0.tar.gz >scratch/cello_resources/resources_v2.0.0.tar.gz; tar -C scratch/cello_resources -zxf scratch/cello_resources/resources_v2.0.0.tar.gz
```

## Evaluation signals

- cell_annotations.csv contains exactly one annotation row for every cell in the prepared blood_covid study, with no missing annotations
- Every assigned label resolves to a valid Cell Ontology term
- The gene feature space of the AnnData matches the gene set required by CellO's pretrained classifier before scoring
- Standardized labels are present in the per-cell metadata (obs) of the AnnData object rather than only in the exported CSV
- CellO resources are staged from the v2.0.0 resource bundle (resources_v2.0.0.tar.gz) as required by the documented workaround

## Limitations

- CellO resource download requires a documented workaround (deweylab/CellO issue #29) — resources must be fetched from resources_v2.0.0.tar.gz and extracted into scratch/cello_resources
- CellO is a pretrained classifier, so labels are constrained to the Cell Ontology terms its model supports; user-defined cell types from Cellenium's interactive annotation feature are a separate mechanism
- Full study data processing in the Cellenium stack initially takes a couple of hours, so annotation of very large studies is not instantaneous

## Evidence

- [other] Cellenium applies CellO (Bernstein 2021) to assign standardized cell type annotations to each individual cell of an integrated study: [evidence span withheld — anchor status: not_found]
- [readme] README: CellO resources preparation workaround: [evidence span withheld — anchor status: external_doc]
- [readme] README: input format: [evidence span withheld — anchor status: external_doc]
- [readme] README: resource download: [evidence span withheld — anchor status: external_doc]
- [intro] Article: purpose of standardized annotations: "organize and semantically find scRNA studies with ontologized metadata for tissues and diseases"
