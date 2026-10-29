---
name: jupyter-notebook-headless-execution
description: Use when preparing AnnData import files for Cellenium from its example
  notebooks, or when study notebooks must be executed non-interactively (e.g. inside
  make targets or CI) to generate .h5ad study data.
when_to_use_negative:
- The goal is exploring or annotating data in the already-running Cellenium webapp/GraphQL
  API rather than preparing importable .h5ad study files.
- A notebook requires interactive setup before it can run headlessly — e.g. heart_failure_reichart2022*.ipynb,
  where the download URL must be edited in the notebook beforehand.
- The data is not scRNA study data for Cellenium (e.g. generic notebook development
  or exploratory analysis, which the README suggests doing via `(cd data_import &&
  PYTHONPATH=$(pwd) jupyter-lab)` instead).
tools:
- name: Python 3
  role: Runtime for headless execution of the study-preparation notebooks (server
    backend and import pipeline are Python 3).
- name: Jupyter notebooks
  role: Provided notebooks download public scRNA studies, normalize data, compute
    DEGs/UMAP, and write H5AD files; run headlessly via make.
  repo: https://github.com/Bayer-Group/cellenium
- name: conda
  role: Creates/activates the cellenium_import environment from data_import/environment.yml
    required to execute the notebooks.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:5957320fa6fdc34cc340195474247c8f60c7392a3aa62574d78bdd4c9a429e11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - Jupyter notebooks
  - Python 3
  - conda
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# jupyter-notebook-headless-execution

## Summary

Executes Cellenium's study-data-preparation Jupyter notebooks (e.g. blood_covid.ipynb) headlessly with Python 3 to download public scRNA studies and produce validated .h5ad files ready for Cellenium import.

## When to use

Use when preparing AnnData import files for Cellenium from its example notebooks, or when study notebooks must be executed non-interactively (e.g. inside make targets or CI) to generate .h5ad study data.

## When NOT to use

- The goal is exploring or annotating data in the already-running Cellenium webapp/GraphQL API rather than preparing importable .h5ad study files.
- A notebook requires interactive setup before it can run headlessly — e.g. heart_failure_reichart2022*.ipynb, where the download URL must be edited in the notebook beforehand.
- The data is not scRNA study data for Cellenium (e.g. generic notebook development or exploratory analysis, which the README suggests doing via `(cd data_import && PYTHONPATH=$(pwd) jupyter-lab)` instead).

## Inputs

- Cellenium example study-preparation Jupyter notebooks (e.g. blood_covid.ipynb, heart_failure_reichart2022*.ipynb) from the Bayer-Group/cellenium checkout
- data_import/environment.yml (conda environment spec for the cellenium_import env)
- Makefile targets (test_studydata_import, normal_studydata_import, atac_studydata_import, cite_studydata_import)
- Publicly available scRNA study downloads (expression matrices, cell/gene annotations)

## Outputs

- Serialized .h5ad (H5AD) AnnData files per study with expression matrix, cell and gene annotations
- Derived study data computed by the notebooks: differentially expressed genes, UMAP projection, other embeddings
- Populated AnnData .uns metadata: title, description, species (NCBI taxonomy), disease (MeSH), tissue (NCIT)

## How to apply

First create and activate the dedicated import environment with `conda env create -f data_import/environment.yml` and `conda activate cellenium_import`. Execute the study preparation notebooks headlessly — the README states the notebooks are run in headless mode by make, so prefer the provided targets (`make reset_database test_studydata_import` for a small feature-covering set, then `normal_studydata_import`, `atac_studydata_import`, or `cite_studydata_import` for full studies), or run notebooks directly with Python 3 (e.g. `jupyter nbconvert --to notebook --execute`). Each notebook downloads a public scRNA study, normalizes if needed, computes differentially expressed genes and a UMAP projection, and constructs an AnnData object whose .uns dictionary must carry study metadata: title, description, species mapped to NCBI taxonomy, disease mapped to MeSH, and tissue mapped to NCIT, since Cellenium semantically organizes studies by these fields. Serialize each study as an .h5ad file; before running the huge heart_failure_reichart2022* notebooks, edit the beginning of the notebook to define the download URL. Validate by loading each .h5ad with AnnData and checking that .uns contains all required metadata fields.

## Related tools

- **Python 3** (Runtime for headless execution of the study-preparation notebooks (server backend and import pipeline are Python 3).)
- **Jupyter notebooks** (Provided notebooks download public scRNA studies, normalize data, compute DEGs/UMAP, and write H5AD files; run headlessly via make.) — https://github.com/Bayer-Group/cellenium
- **conda** (Creates/activates the cellenium_import environment from data_import/environment.yml required to execute the notebooks.)

## Examples

```
conda env create -f data_import/environment.yml && conda activate cellenium_import && make reset_database test_studydata_import
```

## Evaluation signals

- Each produced .h5ad file exists and loads with the AnnData reader without errors.
- Each .h5ad's .uns dictionary contains all required metadata fields: title, description, species (NCBI taxonomy), disease (MeSH), tissue (NCIT).
- Notebook execution completes without error and derived outputs (DEGs, UMAP projection) are present in the serialized study.
- The small test_studydata set covers all application features yet imports 'in a few minutes', serving as a fast end-to-end sanity check before full imports.
- Imported studies appear queryable in the PostgreSQL/GraphQL backend and render in the webapp (e.g. blood_covid UMAP projection plot).

## Limitations

- Initial Cellenium setup, including execution of the study data processing notebooks, takes a couple of hours to run.
- The huge example study notebooks (heart_failure_reichart2022*.ipynb) require manually editing the beginning of the notebook to define the download URL before execution.
- Separate make targets exist for RNA, ATAC, and CITE study data — a single run does not cover all modalities.
- A CellO resource workaround is needed beforehand (fetching resources_v2.0.0.tar.gz due to deweylab/CellO issue #29) before setup can succeed.

## Evidence

- [readme] Cellenium imports scRNA expression data and cell annotations in H5AD format.: [evidence span withheld — anchor status: external_doc]
- [readme] The notebooks are run in headless mode by make; jupyter-lab is for interactive exploration.: [evidence span withheld — anchor status: external_doc]
- [readme] Setup includes executing study data processing notebooks and can take a couple of hours.: [evidence span withheld — anchor status: external_doc]
- [abstract] The backend runtime for headless notebook execution is Python 3.: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
