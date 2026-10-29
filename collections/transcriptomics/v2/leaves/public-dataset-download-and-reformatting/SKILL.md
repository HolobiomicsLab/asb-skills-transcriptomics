---
name: public-dataset-download-and-reformatting
description: Use when importing public scRNA-seq studies into Cellenium or preparing
  .h5ad study files whose .uns dictionary must contain ontologized metadata (title,
  description, NCBI taxonomy species, MeSH disease, NCIT tissue).
when_to_use_negative:
- Input data is already a validated .h5ad with complete .uns metadata fields and derived
  embeddings — it can be imported directly without re-running the notebooks
- The goal is developing or exploring new notebooks interactively rather than batch
  production — use (cd data_import && PYTHONPATH=$(pwd) jupyter-lab) instead of headless
  execution
- Processing the huge example study (heart_failure_reichart2022) before editing the
  download URL defined at the beginning of the notebook
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Python 3
  role: Runs the study preparation notebooks and Python stored procedures that download,
    normalize, and compute DEGs/UMAP for the public studies
  repo: https://github.com/Bayer-Group/cellenium
- name: PostgreSQL
  role: Receives the prepared .h5ad study data via make import targets as the Cellenium
    backend database
  repo: https://github.com/Bayer-Group/cellenium
- name: AnnData
  role: 'Target format: each study is serialized as an .h5ad AnnData object with expression
    matrix, annotations, embeddings, and .uns metadata'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:11252525cdfcfe62c8e6bf32b4f58780be6084f5e0e60b89d7d56f5d441e7f8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - AnnData
  - PostgreSQL
  - Python 3
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# public-dataset-download-and-reformatting

## Summary

Download publicly available single-cell RNA-seq studies via the Cellenium example Jupyter notebooks (e.g. blood_covid.ipynb) and reformat them into AnnData .h5ad files carrying semantically organized study metadata, ready for Cellenium import.

## When to use

Use when importing public scRNA-seq studies into Cellenium or preparing .h5ad study files whose .uns dictionary must contain ontologized metadata (title, description, NCBI taxonomy species, MeSH disease, NCIT tissue).

## When NOT to use

- Input data is already a validated .h5ad with complete .uns metadata fields and derived embeddings — it can be imported directly without re-running the notebooks
- The goal is developing or exploring new notebooks interactively rather than batch production — use (cd data_import && PYTHONPATH=$(pwd) jupyter-lab) instead of headless execution
- Processing the huge example study (heart_failure_reichart2022) before editing the download URL defined at the beginning of the notebook

## Inputs

- Publicly available scRNA-seq studies (download URLs defined at the top of example notebooks such as blood_covid.ipynb and heart_failure_reichart2022*.ipynb)
- Jupyter study preparation notebooks from the data_import folder of Bayer-Group/cellenium

## Outputs

- AnnData .h5ad files with expression matrix, cell/gene annotations, UMAP embedding, and .uns study metadata (title, description, NCBI taxonomy species, MeSH disease, NCIT tissue)
- Database-imported study data via make studydata_import targets

## How to apply

Locate the study preparation notebooks shipped in the Bayer-Group/cellenium repository (e.g. blood_covid.ipynb), which download public studies, normalize the data if necessary, and calculate differentially expressed genes, a UMAP projection and other study data. Execute the notebook headlessly (the notebooks are run in headless mode by make) after activating the cellenium_import conda environment from data_import/environment.yml. Construct an AnnData object per study holding the expression matrix, cell and gene annotations, and derived embeddings, then populate .uns with title, description, species (NCBI taxonomy), disease (MeSH), and tissue (NCIT). Serialize each study as an .h5ad file and import via the make targets (test_studydata_import, normal_studydata_import, atac_studydata_import, cite_studydata_import).

## Related tools

- **Python 3** (Runs the study preparation notebooks and Python stored procedures that download, normalize, and compute DEGs/UMAP for the public studies) — https://github.com/Bayer-Group/cellenium
- **PostgreSQL** (Receives the prepared .h5ad study data via make import targets as the Cellenium backend database) — https://github.com/Bayer-Group/cellenium
- **AnnData** (Target format: each study is serialized as an .h5ad AnnData object with expression matrix, annotations, embeddings, and .uns metadata)

## Examples

```
conda activate cellenium_import && make reset_database test_studydata_import
```

## Evaluation signals

- Each produced .h5ad file exists and loads with AnnData
- The .uns dictionary of every .h5ad contains all required fields: title, description, species (NCBI taxonomy), disease (MeSH), tissue (NCIT)
- Derived study data (differentially expressed genes, UMAP projection) is present so Cellenium's features work
- make reset_database followed by test_studydata_import completes, covering all application features with a small dataset in a few minutes
- Imported studies are findable via the GraphQL API explorer at http://localhost:5000/postgraphile/graphiql

## Limitations

- Initial full setup including execution of study data processing notebooks takes a couple of hours to run
- Until the make reset_database step is run, error messages about the missing 'postgraphile' user pile up and can be ignored
- Preparation of CellO data files requires a workaround for deweylab/CellO issue #29 (manual download and extraction of resources_v2.0.0.tar.gz)
- The huge example study requires editing the beginning of heart_failure_reichart2022*.ipynb to define the download URL before processing

## Evidence

- [readme] Cellenium imports scRNA expression data and cell annotations in H5AD format. We provide jupyter notebooks for downloading some publicly available scRNA studies, normalize the data if necessary, and calculate differentially expressed genes, a UMAP projection and other study data: [evidence span withheld — anchor status: external_doc]
- [intro] the public study example blood_covid.ipynb: "the public study example blood_covid.ipynb"
- [readme] The setup steps below automate the download and creation of appropriate H5AD files, docker image build, database schema setup and data ingestion.: [evidence span withheld — anchor status: external_doc]
- [readme] The notebooks are run in headless mode by make. To create new notebooks and explore datasets: (cd data_import && PYTHONPATH=$(pwd) jupyter-lab): [evidence span withheld — anchor status: not_found]
- [readme] Before you process and import the huge example study (there are two additional make targets for that), edit the beginning of heart_failure_reichart2022*.ipynb and define the download URL as described in the notebooks.: [evidence span withheld — anchor status: external_doc]
