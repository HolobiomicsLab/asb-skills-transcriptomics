---
name: single-cell-multimodal-data-preprocessing
description: Use when raw single-cell datasets with paired TCR sequences and gene
  expression must be annotated (cell type, tissue, cancer source) and filtered before
  training mvTCR or building a multi-study reference atlas.
when_to_use_negative:
- Input is already a preprocessed, annotated dataset downloaded from the mvTCR reproducibility/Zenodo
  release, ready for evaluation or figure notebooks
- Data lacks paired TCR sequences or specificity/cell-type annotations required by
  the multimodal pipeline (e.g. gene expression only)
- The goal is BCR/somatic-hypermutation analysis, for which no large-scale annotated
  datasets and no in-depth benchmark exist
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: mvTCR
  role: Provides the preprocessing tutorial (01_preprocessing.ipynb) that adds required
    annotations and filters the data into the multimodal format consumed by the model
  repo: https://github.com/SchubertLab/mvTCR
- name: Scanpy
  role: Standard single-cell analysis format into which the mvTCR representation and
    preprocessed data integrate
- name: SchubertLab/mvTCR_reproducibility
  role: Source of the preprocessing notebooks and data links for the paper's datasets,
    including the multi-study TIL reference
  repo: https://github.com/SchubertLab/mvTCR_reproducibility
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:529373d54a52023e3ee977457ceb9d3ceaa7bd60d3cf27b97b518220bb1f095e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - PyTorch
  - Scanpy
  - SchubertLab/mvTCR_reproducibility
  - mvTCR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# single-cell-multimodal-data-preprocessing

## Summary

Preprocess single-cell TCR/gene-expression data into the annotated, filtered format required by mvTCR: harmonize multimodal inputs, add cell type, cancer source, and tissue type annotations, and filter cells so the data is ready for multiview VAE training or reference mapping.

## When to use

Use when raw single-cell datasets with paired TCR sequences and gene expression must be annotated (cell type, tissue, cancer source) and filtered before training mvTCR or building a multi-study reference atlas.

## When NOT to use

- Input is already a preprocessed, annotated dataset downloaded from the mvTCR reproducibility/Zenodo release, ready for evaluation or figure notebooks
- Data lacks paired TCR sequences or specificity/cell-type annotations required by the multimodal pipeline (e.g. gene expression only)
- The goal is BCR/somatic-hypermutation analysis, for which no large-scale annotated datasets and no in-depth benchmark exist

## Inputs

- Single-cell datasets with paired TCR sequences and gene expression (10x filtered gene matrices / Scanpy-compatible objects)
- Publicly available TIL data with paired TCR sequencing (e.g. GEO accessions tracked in ncborcherding/utility)
- Sample/cohort metadata providing study, tissue, and cancer-type labels

## Outputs

- Annotated, filtered AnnData/MuData-style single-cell objects with cell type, clonotype, tissue type, and cancer source columns
- Preprocessed reference and query datasets ready for mvTCR model training and scArches reference mapping

## How to apply

Install mvTCR in an isolated conda environment (mvTCR via pip, plus a matching PyTorch with CUDA; the paper's experiments used mvTCR v0.1.3 with PyTorch v1.8.0). Follow the tutorial tutorials/01_preprocessing.ipynb to load the data, add the required annotations (cell type, clonotype, and for TIL atlas work the cancer source and tissue type labels), and apply the tutorial's filtering criteria to remove cells failing quality or annotation requirements. When assembling a multi-study reference (e.g. the 12-study TIL atlas), preprocess each study consistently so annotation columns align across studies before model training. Data links are referenced in the corresponding publication and the reproducibility repository; preprocessed reference data can be taken from SchubertLab/mvTCR_reproducibility. The preprocessed object in Scanpy-compatible format then feeds directly into model training (tutorials/02_training_analysis.ipynb) or reference-mapping workflows.

## Related tools

- **mvTCR** (Provides the preprocessing tutorial (01_preprocessing.ipynb) that adds required annotations and filters the data into the multimodal format consumed by the model) — https://github.com/SchubertLab/mvTCR
- **Scanpy** (Standard single-cell analysis format into which the mvTCR representation and preprocessed data integrate)
- **SchubertLab/mvTCR_reproducibility** (Source of the preprocessing notebooks and data links for the paper's datasets, including the multi-study TIL reference) — https://github.com/SchubertLab/mvTCR_reproducibility

## Examples

```
conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr && conda install nb_conda_kernels -y && jupyter notebook tutorials/01_preprocessing.ipynb
```

## Evaluation signals

- Preprocessed objects carry the required annotation columns (cell type, clonotype, and for TIL work tissue type and cancer source) for every retained cell
- Cells failing the tutorial's filtering criteria are removed and cell counts are consistent with the reference dataset description
- Preprocessed data trains successfully in tutorial 02_training_analysis.ipynb within the documented resource envelope (e.g. 8GB RAM sufficed for the >60k-cell 10x dataset)
- Downstream kNN label transfer from the embedding yields accuracies matching the paper's reported reference-mapping performance
- Environment matches the paper's versions (mvTCR v0.1.3, PyTorch v1.8.0 for reproducibility runs)

## Limitations

- mvTCR was mainly applied to fully paired data (gene expression plus α- and β-CDR3); handling of partially missing data remains insufficiently addressed
- Preprocessing and annotation choices depend on study-specific labels; the desired contribution of each modality is study-dependent and may require retraining with a hyperparameter search
- For reproducibility, PyTorch/CUDA versions and cluster hardware can cause trained models to diverge from the publication's models

## Evidence

- [methods] mvTCR preprocessing tutorial defines the annotation and filtering step: "tutorials/01_preprocessing.ipynb : adding requiered annotation and filtering on the data."
- [methods] mvTCR install command for the preprocessing environment: "conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr"
- [readme] Reproducibility repo hosts the preprocessing notebooks and data links: [evidence span withheld — anchor status: external_doc]
- [readme] Tutorials run quickly on subsampled data with expected outputs provided: "The notebooks are run on a subsampled dataset and can be executed within 15 minutes each. The expected output is provided in each notebook."
