---
name: annadata-metadata-annotation
description: Use when preparing single-cell RNA-, ATAC-, or CITE-seq studies as .h5ad
  files for Cellenium import, and study-level metadata must be ontologized so studies
  can be semantically found and organized across the app.
when_to_use_negative:
- Data is not destined for Cellenium import and a plain AnnData/.h5ad representation
  without ontologized .uns metadata suffices
- Input is not single-cell expression data (e.g. bulk RNA-seq or non-omics tabular
  data) — Cellenium targets scRNA-, ATAC-, and CITE-sequencing studies
- The .h5ad file already exists and only needs database ingestion — use the make import
  targets (test/normal/atac/cite_studydata_import) instead of re-annotating metadata
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: Python 3
  role: Runs the study data preparation notebooks and constructs/serializes the AnnData
    objects with populated .uns metadata
  repo: https://github.com/Bayer-Group/cellenium
- name: PostgreSQL
  role: Cellenium's server backend into which the prepared .h5ad study files are ingested
    after metadata annotation
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:136e1e0ed5731ddd0e4df800f2f367ceceb6efa8fce0b181914a3212b5bfb41c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - PostgreSQL
  - Python 3
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# annadata-metadata-annotation

## Summary

Prepares AnnData (.h5ad) import files for Cellenium by populating the unstructured .uns annotation dictionary with ontologized study metadata (title, description, species as NCBI taxonomy, disease as MeSH, tissue as NCIT), enabling semantic organization and retrieval of single-cell studies.

## When to use

Use when preparing single-cell RNA-, ATAC-, or CITE-seq studies as .h5ad files for Cellenium import, and study-level metadata must be ontologized so studies can be semantically found and organized across the app.

## When NOT to use

- Data is not destined for Cellenium import and a plain AnnData/.h5ad representation without ontologized .uns metadata suffices
- Input is not single-cell expression data (e.g. bulk RNA-seq or non-omics tabular data) — Cellenium targets scRNA-, ATAC-, and CITE-sequencing studies
- The .h5ad file already exists and only needs database ingestion — use the make import targets (test/normal/atac/cite_studydata_import) instead of re-annotating metadata

## Inputs

- Single-cell expression data and cell annotations (raw or normalized from public studies)
- Study data preparation Jupyter notebooks from Bayer-Group/cellenium (e.g. blood_covid.ipynb)
- Derived study data required by Cellenium features: differentially expressed genes, UMAP projection

## Outputs

- .h5ad AnnData files per study with .uns populated with title, description, NCBI-taxonomy species, MeSH disease, and NCIT tissue
- AnnData objects containing expression matrix, cell and gene annotations, and embeddings (e.g. UMAP)

## How to apply

Start from the example data-preparation notebooks in the Bayer-Group/cellenium repository (e.g. blood_covid.ipynb) and execute them headlessly with Python 3 (jupyter nbconvert --to notebook --execute) so they download public studies, normalize data if necessary, and compute differentially expressed genes and a UMAP projection. Construct one AnnData object per study holding the expression matrix, cell annotations, gene annotations, and derived embeddings. Populate the .uns dictionary with Cellenium's required semantic fields: title, description, species mapped to NCBI taxonomy, disease mapped to MeSH, and tissue mapped to NCIT — this is what enables the app's semantic study organization. Serialize each study as an .h5ad file, sized according to the intended make target (test_studydata is small and imports in minutes; normal_studydata carries full cells and genes).

## Related tools

- **Python 3** (Runs the study data preparation notebooks and constructs/serializes the AnnData objects with populated .uns metadata) — https://github.com/Bayer-Group/cellenium
- **PostgreSQL** (Cellenium's server backend into which the prepared .h5ad study files are ingested after metadata annotation) — https://github.com/Bayer-Group/cellenium

## Examples

```
(cd data_import && PYTHONPATH=$(pwd) jupyter-lab)
```

## Evaluation signals

- Each produced .h5ad file exists on disk and loads successfully with AnnData (e.g. anndata.read_h5ad)
- Each file's .uns dictionary contains all required fields: title, description, species (NCBI taxonomy), disease (MeSH), tissue (NCIT)
- The .h5ad imports successfully via the relevant make target (test_studydata_import, normal_studydata_import, atac_studydata_import, or cite_studydata_import) without schema errors
- The imported study is visible and semantically findable in the Cellenium webapp with ontologized tissue/disease metadata
- Derived features (differentially expressed genes, UMAP projection) are present so Cellenium's annotation and plotting features work

## Limitations

- Initial study data processing and import takes a couple of hours when run via 'docker compose up' and the make targets
- The huge heart_failure_reichart2022 example requires manually editing the download URL at the beginning of the notebook before processing
- CellO data files need a documented workaround (downloading resources_v2.0.0.tar.gz to scratch/cello_resources) due to deweylab/CellO issue #29
- Missing postgraphile-user error messages may pile up during setup until 'make reset_database' is run; these can be ignored

## Evidence

- [readme] Cellenium imports scRNA expression data and cell annotations in H5AD format. We provide jupyter notebooks for downloading some publicly available scRNA studies, normalize the data if necessary: [evidence span withheld — anchor status: external_doc]
- [intro] organize and semantically find scRNA studies with ontologized metadata for tissues and diseases: "organize and semantically find scRNA studies with ontologized metadata for tissues and diseases"
- [other] Populate the AnnData .uns dictionary with the study metadata fields: title, description, species mapped to NCBI taxonomy, disease mapped to MeSH, and tissue mapped to NCIT: [evidence span withheld — anchor status: not_found]
- [readme] The notebooks are run in headless mode by make. To create new notebooks and explore datasets:: [evidence span withheld — anchor status: external_doc]
