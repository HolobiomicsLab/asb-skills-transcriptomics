---
name: gene-co-expression-network-construction
description: Use when you have a bulk RNA-seq count matrix (e.g., 38 3xTg-AD/WT female
  hippocampus samples across three ages) and need to identify co-expression modules
  correlated with traits such as genotype or age.
when_to_use_negative:
- The goal is single-sample or single-cell-style marker scoring rather than module
  detection across many samples — WGCNA-type co-expression analysis needs a sample-rich
  count matrix
- You only need functional enrichment of a pre-existing gene list; PyWGCNA's enrichment
  tools exist, but the core skill of network construction is unnecessary
- You are unfamiliar with the R WGCNA reference implementation and need conceptual
  background first — the authors recommend reading the original WGCNA publication
  in that case
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
tools:
- name: PyWGCNA
  role: Primary library for data cleaning, soft-thresholding power selection, weighted
    network construction, module detection, eigengene computation, and module membership
    calculation
  repo: https://github.com/mortazavilab/PyWGCNA
- name: Python
  role: Runtime environment; PyWGCNA requires Python 3.9 or greater (article) / 3.10
    or greater (README)
  repo: https://github.com/mortazavilab/PyWGCNA
- name: WGCNA
  role: R reference implementation and conceptual basis; recommended reading if unfamiliar
    with WGCNA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:69e8e490933471e10f994d79501981e93fe0302fbdf222ef0c45bdb54a5a90a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - PyWGCNA
  - Python
  - WGCNA
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gene-co-expression-network-construction

## Summary

Builds a weighted gene co-expression network from bulk RNA-seq count matrices using PyWGCNA, detecting clusters (modules) of highly correlated genes and summarizing them via module eigengenes with module membership measures.

## When to use

Use when you have a bulk RNA-seq count matrix (e.g., 38 3xTg-AD/WT female hippocampus samples across three ages) and need to identify co-expression modules correlated with traits such as genotype or age.

## When NOT to use

- The goal is single-sample or single-cell-style marker scoring rather than module detection across many samples — WGCNA-type co-expression analysis needs a sample-rich count matrix
- You only need functional enrichment of a pre-existing gene list; PyWGCNA's enrichment tools exist, but the core skill of network construction is unnecessary
- You are unfamiliar with the R WGCNA reference implementation and need conceptual background first — the authors recommend reading the original WGCNA publication in that case

## Inputs

- Bulk RNA-seq gene count matrix (genes x samples), e.g., the 38-sample female hippocampus 3xTg-AD/WT matrix from the PyWGCNA Quick Start tutorial data
- Sample trait/phenotype table (e.g., genotype, age/time point) for module-trait correlation

## Outputs

- Gene-to-module assignment table (e.g., module_assignments.csv) with module colors such as 'dark gray'
- Module eigengene values and module-trait correlations (e.g., against 3xTg-AD genotype and 18-month time point)
- Module membership measures per gene

## How to apply

Install PyWGCNA from PyPi (requires Python 3.9 or greater per the paper; the README now states 3.10+). Load the count matrix into a PyWGCNA object and perform data input, cleaning and pre-processing (formatting and filtering low-expression genes) following the Data_format tutorial. Select the soft-thresholding power to build the weighted network, then run module detection to find clusters of highly correlated genes. Compute module eigengenes and module membership measures, and relate modules to external sample traits using eigengene network methodology. Export the gene-to-module assignments (e.g., module_assignments.csv) and validate against reference results such as the 3xTg-AD tutorial (17 modules; dark gray module with 380 genes correlated with 3xTg-AD genotype and the 18-month time point).

## Related tools

- **PyWGCNA** (Primary library for data cleaning, soft-thresholding power selection, weighted network construction, module detection, eigengene computation, and module membership calculation) — https://github.com/mortazavilab/PyWGCNA
- **Python** (Runtime environment; PyWGCNA requires Python 3.9 or greater (article) / 3.10 or greater (README)) — https://github.com/mortazavilab/PyWGCNA
- **WGCNA** (R reference implementation and conceptual basis; recommended reading if unfamiliar with WGCNA)

## Examples

```
pip install PyWGCNA
```

## Evaluation signals

- On the 3xTg-AD tutorial dataset, the output must contain exactly 17 modules
- On the same dataset, the dark gray module must contain exactly 380 genes
- The dark gray module must be strongly correlated with the 3xTg-AD genotype and the 18-month time point
- Module eigengenes and module membership measures are present in the PyWGCNA object after module detection
- The gene-to-module assignment table covers all genes surviving the low-expression filtering step from the Data_format pre-processing

## Limitations

- Python version gate differs across sources: the paper states Python 3.9 or greater is required, while the README states 3.10 or greater
- Results depend on the soft-thresholding power choice and the low-expression gene filtering performed during data cleaning/pre-processing
- The PyWGCNA paper and tutorials use brain bulk RNA-seq (MODEL-AD, 3xTg-AD/5xFAD) datasets; replication on other tissues or sample sizes may not reproduce the 17-module / 380-gene reference values

## Evidence

- [intro] PyWGCNA is a Python library designed to do weighted correlation network analysis (WGCNA).: "PyWGCNA is a Python library designed to do weighted correlation network analysis (WGCNA)."
- [methods] To install PyWGCNA, python version 3.9 or greater is required.: "To install PyWGCNA, python version 3.9 or greater is required."
- [methods] Data input, cleaning and pre-processing tutorial link: "`Data input, cleaning and pre-processing <https://github.com/mortazavilab/PyWGCNA/tree/main/tutorials/Data_format.md>`_"
- [methods] Quick Start tutorial link: "`Quick Start <https://github.com/mortazavilab/PyWGCNA/tree/main/tutorials/Quick_Start.ipynb>`_: How to load data into PyWGCNA, find modules, and analyze them"
- [readme] for finding clusters (modules) of highly correlated genes: "It can be used for finding clusters (modules) of highly correlated genes, for summarizing such clusters using the module eigengene,"
- [readme] Python version requirement in README: [evidence span withheld — anchor status: external_doc]
