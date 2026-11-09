---
name: ontology-metadata-mapping-ncbi-mesh-ncit
description: Use when preparing .h5ad study files for import into Cellenium (e.g.
  via the blood_covid.ipynb or heart_failure_reichart2022 notebooks) and study-level
  metadata must be mapped to NCBI taxonomy, MeSH, and NCIT ontologies.
when_to_use_negative:
- The study file already carries ontologized metadata in .uns (verify rather than
  re-map)
- Working with per-cell or per-gene metadata — this skill maps study-level semantic
  metadata only, not cell annotations
- Importing into a non-Cellenium pipeline that does not consume the uns-dictionary
  study metadata schema
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3489
tools:
- name: Python 3
  role: Runs the study-data preparation notebooks (headlessly via make or jupyter
    nbconvert) that build AnnData objects and populate .uns metadata
  repo: https://github.com/Bayer-Group/cellenium
- name: PostgreSQL
  role: Backend database into which the ontologized study metadata is ingested during
    database schema setup and data ingestion
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:ea2a21a63d1d3e304b75a88f4a0602ed02c89f4aa6d48c2612430de32516f4d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - PostgreSQL
  - Python 3
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Ontology metadata mapping for NCBI taxonomy, MeSH, and NCIT

## Summary

Prepares single-cell study import files for Cellenium by populating the AnnData .uns dictionary with ontologized study metadata — title, description, species (NCBI taxonomy), disease (MeSH), and tissue (NCIT) — so studies can be semantically organized and found within and across studies.

## When to use

Use when preparing .h5ad study files for import into Cellenium (e.g. via the blood_covid.ipynb or heart_failure_reichart2022 notebooks) and study-level metadata must be mapped to NCBI taxonomy, MeSH, and NCIT ontologies.

## When NOT to use

- The study file already carries ontologized metadata in .uns (verify rather than re-map)
- Working with per-cell or per-gene metadata — this skill maps study-level semantic metadata only, not cell annotations
- Importing into a non-Cellenium pipeline that does not consume the uns-dictionary study metadata schema

## Inputs

- Jupyter study-data notebook from the Bayer-Group/cellenium repository (e.g. blood_covid.ipynb, heart_failure_reichart2022*.ipynb)
- Public scRNA/scATAC/scCITE study downloads with raw expression matrix, cell annotations, and gene annotations
- Ontology mappings: species to NCBI taxonomy, disease to MeSH, tissue to NCIT

## Outputs

- Serialized .h5ad (H5AD) study file whose .uns dictionary contains title, description, NCBI-taxonomy species, MeSH disease, and NCIT tissue
- AnnData object with expression matrix, cell/gene annotations, and derived embeddings such as UMAP

## How to apply

For each study, construct an AnnData object holding the expression matrix, cell and gene annotations, and derived embeddings (e.g. UMAP) computed by the repository's study-data notebooks, then write the semantic metadata into the unstructured .uns dictionary: 'title', 'description', 'species' mapped to NCBI taxonomy, 'disease' mapped to MeSH, and 'tissue' mapped to NCIT. This ontologized metadata is what Cellenium's PostgreSQL/GraphQL backend uses to organize and semantically find scRNA studies with ontologized metadata for tissues and diseases. Serialize each prepared study as an .h5ad file, then import it via the repository's make targets (e.g. test_studydata_import, normal_studydata_import, atac_studydata_import, cite_studydata_import) after the docker-compose stack and cellenium_import conda environment are up.

## Related tools

- **Python 3** (Runs the study-data preparation notebooks (headlessly via make or jupyter nbconvert) that build AnnData objects and populate .uns metadata) — https://github.com/Bayer-Group/cellenium
- **PostgreSQL** (Backend database into which the ontologized study metadata is ingested during database schema setup and data ingestion) — https://github.com/Bayer-Group/cellenium

## Examples

```
jupyter nbconvert --to notebook --execute blood_covid.ipynb
```

## Evaluation signals

- Each produced .h5ad file exists and loads successfully with AnnData
- adata.uns contains all five required fields: title, description, species (NCBI taxonomy), disease (MeSH), tissue (NCIT)
- Species values resolve to NCBI taxonomy terms, disease values to MeSH terms, tissue values to NCIT terms
- Imported studies are retrievable through the GraphQL API explorer (http://localhost:5000/postgraphile/graphiql) using their semantic metadata
- The expression matrix, cell/gene annotations, and derived UMAP embedding are preserved in the .h5ad alongside the metadata

## Limitations

- CellO data files require a manual workaround (download resources_v2.0.0.tar.gz to scratch/cello_resources) due to a known CellO issue before setup
- Running the full study data processing notebooks and imports 'initially, this will take a couple of hours to run'
- Until 'make reset_database' runs, postgraphile-user error messages pile up and should be ignored
- The huge example study (heart_failure_reichart2022*) requires manually editing the notebook's download URL before processing

## Evidence

- [other] Cellenium expects study metadata attributes to be added to the unstructured annotation field (uns-dictionary) of AnnData files, including title, description, and species (NCBI taxonomy).: [evidence span withheld — anchor status: not_found]
- [intro] organize and semantically find scRNA studies with ontologized metadata for tissues and diseases: "organize and semantically find scRNA studies with ontologized metadata for tissues and diseases"
- [readme] Cellenium imports scRNA expression data and cell annotations in H5AD format.: [evidence span withheld — anchor status: external_doc]
