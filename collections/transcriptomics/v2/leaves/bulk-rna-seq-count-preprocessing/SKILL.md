---
name: bulk-rna-seq-count-preprocessing
description: Use when a raw bulk RNA-seq count matrix (e.g., 38 3xTg-AD/WT female
  hippocampus samples across three ages) must be formatted and cleaned—filtering low-expression
  genes—before module detection in PyWGCNA.
when_to_use_negative:
- Input is already a filtered expression/feature table rather than raw counts—the
  Data_format cleaning step would remove signal that was already removed
- The goal is single-cell co-expression analysis—PyWGCNA is designed for bulk RNA-seq
  datasets with many samples
- Only descriptive expression statistics are needed with no co-expression network
  or module output required
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PyWGCNA
  role: Installs via pip, loads the count matrix into a PyWGCNA object, performs data
    cleaning/pre-processing (low-expression gene filtering), and computes modules,
    eigengenes, and module membership
  repo: https://github.com/mortazavilab/PyWGCNA
- name: Python 3
  role: Runtime required to install and run PyWGCNA (Python 3.9+ per the article;
    3.10+ per the README)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:71e4b484f509bf8658b0055e879095c887b8954a5435e3e2de962751e304aa4d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - PyWGCNA
  - Python
  - Python 3
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bulk-rna-seq-count-preprocessing

## Summary

Format, clean, and pre-filter a bulk RNA-seq count matrix so it can be loaded into a PyWGCNA object for weighted co-expression network construction, following the PyWGCNA Data_format and Quick_Start tutorials.

## When to use

Use when a raw bulk RNA-seq count matrix (e.g., 38 3xTg-AD/WT female hippocampus samples across three ages) must be formatted and cleaned—filtering low-expression genes—before module detection in PyWGCNA.

## When NOT to use

- Input is already a filtered expression/feature table rather than raw counts—the Data_format cleaning step would remove signal that was already removed
- The goal is single-cell co-expression analysis—PyWGCNA is designed for bulk RNA-seq datasets with many samples
- Only descriptive expression statistics are needed with no co-expression network or module output required

## Inputs

- bulk RNA-seq count matrix (gene x sample), e.g., female hippocampus 3xTg-AD/WT samples from the PyWGCNA Quick Start tutorial data
- sample trait table (e.g., age and genotype) for downstream module–trait correlation

## Outputs

- cleaned, low-expression-filtered count matrix loaded into a PyWGCNA object
- module_assignments.csv gene-to-module assignment table
- module eigengenes and module membership measures

## How to apply

Install PyWGCNA from PyPi (`pip install PyWGCNA`), noting the article requires Python 3.9 or greater while the current README states 3.10 or greater. Download the count matrix and load it into a PyWGCNA object, then perform data cleaning and pre-processing exactly as described in the Data_format tutorial (tutorials/Data_format.md), whose purpose is 'How to format, clean and preprocess your input data for PyWGCNA'—the key decision point is filtering low-expression genes so weakly expressed genes do not corrupt the weighted correlation network. With the cleaned matrix, select the soft-thresholding power and run module detection, computing module eigengenes and module membership measures as in the Quick_Start notebook. Validate against the 3xTg-AD tutorial dataset, where the 38-sample female hippocampus matrix yields 17 modules with a dark gray module of exactly 380 genes.

## Related tools

- **PyWGCNA** (Installs via pip, loads the count matrix into a PyWGCNA object, performs data cleaning/pre-processing (low-expression gene filtering), and computes modules, eigengenes, and module membership) — https://github.com/mortazavilab/PyWGCNA
- **Python 3** (Runtime required to install and run PyWGCNA (Python 3.9+ per the article; 3.10+ per the README))

## Examples

```
pip install PyWGCNA
```

## Evaluation signals

- For the 38-sample female hippocampus 3xTg-AD/WT tutorial matrix, the pipeline must yield exactly 17 modules
- The dark gray module must contain exactly 380 genes and be strongly correlated with the 3xTg-AD genotype and the 18-month time point
- module_assignments.csv must exist with a gene-to-module assignment for every gene passing the low-expression filter
- The PyWGCNA object must carry computed module eigengenes and module membership measures before module analysis proceeds
- Results should reproduce when re-run against the Quick_Start.ipynb workflow on the same downloaded count matrix

## Limitations

- Python version requirement is inconsistent across sources: the article methods state 'python version 3.9 or greater is required' while the README states 'Python version 3.10 or greater'
- The article references the R WGCNA implementation as background; results depend on the same soft-thresholding and module-detection parameters, so changing filtering or power choices can change the number and size of modules
- PyWGCNA is designed for large RNA-seq datasets with many samples; very small or single-cell inputs are outside its demonstrated scope

## Evidence

- [methods] Data input, cleaning and pre-processing <https://github.com/mortazavilab/PyWGCNA/tree/main/tutorials/Data_format.md>: "Data input, cleaning and pre-processing <https://github.com/mortazavilab/PyWGCNA/tree/main/tutorials/Data_format.md>"
- [abstract] We introduce the PyWGCNA Python package, which is designed to identify co-expression modules from large RNA-seq datasets.: "We introduce the PyWGCNA Python package, which is designed to identify co-expression modules from large RNA-seq datasets."
- [readme] How to format, clean and preprocess your input data for PyWGCNA: [evidence span withheld — anchor status: external_doc]
- [readme] To install PyWGCNA, Python version 3.10 or greater is required.: [evidence span withheld — anchor status: external_doc]
- [methods] Quick Start (load data, find modules, analyze them): "`Quick Start <https://github.com/mortazavilab/PyWGCNA/tree/main/tutorials/Quick_Start.ipynb>`_: How to load data into PyWGCNA, find modules, and analyze them"
