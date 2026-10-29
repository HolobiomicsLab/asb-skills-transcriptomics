---
name: single-cell-omics-dataset-acquisition-and-preprocessing
description: Use when a task requires downloading a public multimodal single-cell
  dataset (TEA-seq/CITE-seq RNA + ADT) and preprocessing it into the features-by-cells
  modality format expected by SnapCCESS before embedding generation.
when_to_use_negative:
- The data are already SnapCCESS embeddings or a precomputed feature table — preprocessing/acquisition
  would double-handle the input
- The analysis is single-modality (e.g. scRNA-seq only, no ADT/protein counterpart)
  since this skill targets multimodal CITE-seq/TEA-seq inputs measured across the
  same cells
- 'You need dataset-specific optimal hyperparameters: the tutorial explicitly does
  not recommend best parameters, and these must be taken from the publication'
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: snapccess
  role: Python package to install via pip and into which the preprocessed multimodal
    matrices are fed as a combined PyTorch data loader for embedding generation
  repo: https://github.com/PYangLab/SnapCCESS
- name: Python
  role: Language in which the snapccess package is installed and the preprocessing/training
    data loader is constructed
  repo: https://github.com/PYangLab/SnapCCESS
- name: SnapCCESS
  role: Framework whose tutorial data (tutorials/in/ CITE-seq RNA, ADT, and cell type
    files) and tutorials define the expected input arrangement
  repo: https://github.com/yulijia/SnapCCESS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad382
    title: Ensemble deep learning of embeddings for clustering multimodal single-cell
      omics data
  source_content_hash: sha256:d3c10cead196f6fc3ee31dce0d7cea543e6e7b80cae6bd059eef26e107a13835
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PYangLab/SnapCCESS
  tools:
  - Python
  - SnapCCESS
  - snapccess
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# single-cell-omics-dataset-acquisition-and-preprocessing

## Summary

Acquire a multimodal single-cell omics dataset (e.g. the Swanson TEA-seq data, GEO GSE158013, with RNA and protein/ADT modalities) and arrange each modality as a features-by-cells matrix so SnapCCESS can consume it for embedding generation and consensus clustering.

## When to use

Use when a task requires downloading a public multimodal single-cell dataset (TEA-seq/CITE-seq RNA + ADT) and preprocessing it into the features-by-cells modality format expected by SnapCCESS before embedding generation.

## When NOT to use

- The data are already SnapCCESS embeddings or a precomputed feature table — preprocessing/acquisition would double-handle the input
- The analysis is single-modality (e.g. scRNA-seq only, no ADT/protein counterpart) since this skill targets multimodal CITE-seq/TEA-seq inputs measured across the same cells
- You need dataset-specific optimal hyperparameters: the tutorial explicitly does not recommend best parameters, and these must be taken from the publication

## Inputs

- Public multimodal single-cell omics dataset from GEO accession GSE158013 (TEA-seq), RNA and protein (ADT) modalities measured on the same cells
- CITE-seq RNA and ADT count matrices, each arranged as features-by-cells
- Reference cell-type label file (tutorial data provides small CITE-seq cell type files in tutorials/in/)

## Outputs

- Features-by-cells matrices per modality (RNA, ADT) ready for SnapCCESS preprocessing
- A combined PyTorch data loader for the SnapCCESS training/embedding routine
- Downloaded dataset files organized alongside cell-type labels for downstream clustering and ARI computation

## How to apply

Install the snapccess Python package with 'pip install snapccess --index-url https://pypi.org/simple' (a PyTorch-backed package; GPU is optional depending on the local CUDA/ROCm setup). Download the TEA-seq multimodal dataset from GEO accession GSE158013, covering its RNA and protein (ADT) modalities, which are measured across the same cells. Arrange each modality as a features-by-cells matrix before preprocessing, since SnapCCESS expects a list of modalities across the same cells and the Python API expects the combined PyTorch data loader used by the training routine. If a full dataset is too heavy for a trial run, fall back to the small CITE-seq RNA, ADT, and cell type files provided in the repository's tutorials/in/ directory, and follow the tutorial notebook an_example_of_generate_embedding_using_SnapCCESS_python_version.ipynb. Note that the tutorial does not recommend best parameters for your dataset; for the datasets used in the publication, the parameters are listed in the paper itself.

## Related tools

- **snapccess** (Python package to install via pip and into which the preprocessed multimodal matrices are fed as a combined PyTorch data loader for embedding generation) — https://github.com/PYangLab/SnapCCESS
- **Python** (Language in which the snapccess package is installed and the preprocessing/training data loader is constructed) — https://github.com/PYangLab/SnapCCESS
- **SnapCCESS** (Framework whose tutorial data (tutorials/in/ CITE-seq RNA, ADT, and cell type files) and tutorials define the expected input arrangement) — https://github.com/yulijia/SnapCCESS

## Examples

```
pip install snapccess --index-url https://pypi.org/simple
```

## Evaluation signals

- The downloaded GEO accession GSE158013 contains both RNA and protein (ADT) modalities measured on the same set of cells
- Each modality matrix is arranged features-by-cells before preprocessing, matching SnapCCESS's documented expectation
- The Python API accepts the combined PyTorch data loader without shape errors, confirming cells are aligned across modalities
- A dry run against the small tutorial CITE-seq RNA/ADT/cell-type files in tutorials/in/ completes the embedding workflow end-to-end

## Limitations

- The tutorial only explains how to use the package and does not recommend best parameters for new datasets; parameters used in the publication are listed in the paper itself
- SnapCCESS expects all modalities measured across the same cells, so datasets with unmatched cell sets across modalities require alignment before use
- GPU acceleration is optional and depends on the local PyTorch/CUDA or PyTorch/ROCm installation; CPU-only runs may be slow on full TEA-seq data

## Evidence

- [supplementary] Download the TEA-seq multimodal single-cell omics dataset (GEO accession GSE158013) covering its RNA and protein (ADT) modalities.: [evidence span withheld — anchor status: not_found]
- [readme] SnapCCESS expects a list of modalities measured across the same cells, for example RNA and ADT matrices from CITE-seq. Each modality should be arranged as features by cells before preprocessing in the R wrapper.: [evidence span withheld — anchor status: external_doc]
- [methods] pip install snapccess --index-url https://pypi.org/simple: "pip install snapccess  --index-url https://pypi.org/simple"
- [readme] The tutorial data in tutorials/in/ includes small CITE-seq RNA, ADT, and cell type files that can be used to run the examples.: "The tutorial data in `tutorials/in/` includes small CITE-seq RNA, ADT, and cell
 type files that can be used to run the examples."
- [methods] This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets.: "This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets."
