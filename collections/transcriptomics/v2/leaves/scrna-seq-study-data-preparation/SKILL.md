---
name: scrna-seq-study-data-preparation
description: Use when importing new scRNA-seq (RNA-, ATAC-, or CITE-seq) studies into
  Cellenium, i.e. when raw/normalized single-cell data must be converted to annotated
  H5AD files with semantically organized study metadata.
when_to_use_negative:
- If the data are already valid Cellenium-ready .h5ad files with complete .uns metadata
  — they can be imported directly via the make targets rather than re-prepared
- For the huge example study heart_failure_reichart2022*.ipynb before editing its
  beginning to define the download URL as the notebooks instruct
- For interactive dataset exploration while creating new notebooks — use the guided
  jupyter-lab workflow instead of headless execution
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: Python 3
  role: Runs the data-import notebooks and stored procedures that download, normalize,
    and serialize studies to H5AD
  repo: https://github.com/Bayer-Group/cellenium
- name: PostgreSQL
  role: Target database into which prepared H5AD study data are ingested after make
    reset_database
  repo: https://github.com/Bayer-Group/cellenium
- name: GraphQL
  role: API layer through which imported studies are exposed; example queries in documentation/graphql_api_usage
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:9a9074e83a0c2a7b21071a16eb601adbf3f85c1cb32d6817ac286d443454191f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - GraphQL
  - PostgreSQL
  - Python 3
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# scrna-seq-study-data-preparation

## Summary

Prepares single-cell study data for Cellenium by executing the repository's data-import Jupyter notebooks to download public scRNA-Seq studies, compute derived artifacts (DE genes, UMAP projection), and serialize each study as an .h5ad file whose .uns dictionary carries ontologized metadata (title, description, species→NCBI taxonomy, disease→MeSH, tissue→NCIT).

## When to use

Use when importing new scRNA-seq (RNA-, ATAC-, or CITE-seq) studies into Cellenium, i.e. when raw/normalized single-cell data must be converted to annotated H5AD files with semantically organized study metadata.

## When NOT to use

- If the data are already valid Cellenium-ready .h5ad files with complete .uns metadata — they can be imported directly via the make targets rather than re-prepared
- For the huge example study heart_failure_reichart2022*.ipynb before editing its beginning to define the download URL as the notebooks instruct
- For interactive dataset exploration while creating new notebooks — use the guided jupyter-lab workflow instead of headless execution

## Inputs

- Public single-cell study identifiers/download URLs referenced in the repository's jupyter notebooks (e.g. the COVID-19 blood studies of blood_covid.ipynb)
- Raw scRNA-seq count matrices and cell annotations (RNA-, ATAC-, or CITE-seq)
- CellO resources bundle (scratch/cello_resources with resources_v2.0.0.tar.gz, a documented workaround for CellO issue #29)

## Outputs

- Per-study .h5ad files with expression matrix, cell/gene annotations, UMAP projection, and .uns study metadata (title, description, NCBI taxonomy species, MeSH disease, NCIT tissue)
- Computed differential expression results and other study data needed for Cellenium features

## How to apply

Locate the study preparation notebooks (e.g. blood_covid.ipynb) in the Bayer-Group/cellenium checkout and create the cellenium_import conda environment from data_import/environment.yml. Execute the notebooks headlessly (they are normally run by 'make' via targets such as test_studydata_import, normal_studydata_import, atac_studydata_import, and cite_studydata_import) so they download public studies, normalize data if necessary, and calculate differentially expressed genes, a UMAP projection, and other study data Cellenium's features require. Construct an AnnData object per study holding the expression matrix, cell and gene annotations, and derived embeddings, then populate the .uns dictionary with title, description, species (NCBI taxonomy), disease (MeSH), and tissue (NCIT) so Cellenium can semantically organize and find studies. Serialize each prepared study as .h5ad and validate that the file loads with AnnData and contains all required .uns fields before ingestion via make reset_database followed by the appropriate import target.

## Related tools

- **Python 3** (Runs the data-import notebooks and stored procedures that download, normalize, and serialize studies to H5AD) — https://github.com/Bayer-Group/cellenium
- **PostgreSQL** (Target database into which prepared H5AD study data are ingested after make reset_database) — https://github.com/Bayer-Group/cellenium
- **GraphQL** (API layer through which imported studies are exposed; example queries in documentation/graphql_api_usage) — https://github.com/Bayer-Group/cellenium

## Examples

```
cd data_import && PYTHONPATH=$(pwd) jupyter nbconvert --to notebook --execute blood_covid.ipynb
```

## Evaluation signals

- Each produced .h5ad file exists and loads with AnnData without errors
- Each .h5ad file's .uns dictionary contains all required fields: title, description, species (NCBI taxonomy), disease (MeSH), and tissue (NCIT)
- DE genes and UMAP projection were computed per study (required for Cellenium's features such as cluster-based DE and UMAP exploration)
- make reset_database completes and the chosen import target (test/normal/atac/cite_studydata_import) ingests the files without errors
- GraphQL API explorer (http://localhost:5000/postgraphile/graphiql) and webapp (http://localhost:6002/) reflect the imported studies

## Limitations

- Initial setup, including execution of the study data processing notebooks, takes a couple of hours; test_studydata is a small dataset used to cover all features in minutes while normal_studydata runs full-sized studies
- Until 'make reset_database' is run, error messages about the missing 'postgraphile' user pile up; these can be ignored
- The huge heart_failure_reichart2022*.ipynb example study requires manual editing of the notebook's beginning to define the download URL
- CellO data files require a documented workaround (curl + tar of resources_v2.0.0.tar.gz into scratch/cello_resources) for deweylab/CellO issue #29

## Evidence

- [readme] Cellenium imports scRNA expression data and cell annotations in H5AD format.: [evidence span withheld — anchor status: external_doc]
- [readme] notebooks download normalize: [evidence span withheld — anchor status: external_doc]
- [readme] headless make: [evidence span withheld — anchor status: not_found]
- [readme] initially takes hours: [evidence span withheld — anchor status: external_doc]
- [abstract] abstract backend stack: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
- [other] task card uns metadata: [evidence span withheld — anchor status: not_found]
