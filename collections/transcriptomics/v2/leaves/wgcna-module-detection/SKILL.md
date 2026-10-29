---
name: wgcna-module-detection
description: Use when you have a bulk RNA-seq count matrix across many samples (e.g.
  38 female hippocampus 3xTg-AD/WT samples) and need to find co-expressed gene modules
  and correlate them with external traits.
when_to_use_negative:
- You need single-cell co-expression or differential expression rather than bulk-sample
  co-expression modules — PyWGCNA is designed for bulk RNA-seq datasets with many
  samples
- You only need pairwise gene correlations or a small pre-selected gene list analysis
  rather than unsupervised module discovery across the whole transcriptome
- Input is not a gene-by-sample count/normalized expression matrix (e.g. already-aggregated
  module statistics or a feature table) — the Data_format preprocessing step expects
  raw count-level input
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
tools:
- name: PyWGCNA
  role: 'Primary library: builds the weighted co-expression network, picks the soft-thresholding
    power, detects modules, computes eigengenes, module membership, and module-trait
    correlations'
  repo: https://github.com/mortazavilab/PyWGCNA
- name: WGCNA (R reference)
  role: Reference implementation/methodology; the paper recommends reading the original
    WGCNA publication if unfamiliar with the method
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:cd450f41697202d9fc5e1d9a2775152bcefef7355038d2a26dfea21e2bc7ecda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - PyWGCNA
  - Python
  - WGCNA (R reference)
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# wgcna-module-detection

## Summary

Detect co-expression modules from bulk RNA-seq count matrices using PyWGCNA, the Python implementation of weighted gene co-expression network analysis (WGCNA). It identifies clusters of highly correlated genes, summarizes them as module eigengenes, and relates modules to sample traits such as genotype and age.

## When to use

Use when you have a bulk RNA-seq count matrix across many samples (e.g. 38 female hippocampus 3xTg-AD/WT samples) and need to find co-expressed gene modules and correlate them with external traits.

## When NOT to use

- You need single-cell co-expression or differential expression rather than bulk-sample co-expression modules — PyWGCNA is designed for bulk RNA-seq datasets with many samples
- You only need pairwise gene correlations or a small pre-selected gene list analysis rather than unsupervised module discovery across the whole transcriptome
- Input is not a gene-by-sample count/normalized expression matrix (e.g. already-aggregated module statistics or a feature table) — the Data_format preprocessing step expects raw count-level input

## Inputs

- Bulk RNA-seq raw or normalized count matrix (genes x samples), e.g. the 38-sample female hippocampus 3xTg-AD/WT matrix from the PyWGCNA tutorial data
- Sample trait/annotation table (e.g. genotype, age/time point) for module-trait correlation

## Outputs

- Gene-to-module assignment table (e.g. module_assignments.csv) with module colors and sizes
- Module eigengenes and module membership measures
- Module-trait correlation results (e.g. modules correlated with 3xTg-AD genotype or the 18-month time point)

## How to apply

Install PyWGCNA from PyPi (pip install PyWGCNA), which requires Python 3.9 or greater per the paper (the README states 3.10 or greater for the latest release). Load the count matrix into a PyWGCNA object following the Data_format tutorial for input, cleaning, and pre-processing (e.g. filtering low-expression genes). Select the soft-thresholding power to build the weighted network, then run module detection to find clusters of highly correlated genes, computing module eigengenes and module membership measures. Relate modules to external sample traits (e.g. genotype, age/time point) using eigengene network methodology or the module-trait relationships heatmap tutorial. Export gene-to-module assignments (e.g. module_assignments.csv) and validate module counts and sizes against known tutorial results, such as 17 modules with a dark gray module of 380 genes in the 3xTg-AD Quick Start dataset.

## Related tools

- **PyWGCNA** (Primary library: builds the weighted co-expression network, picks the soft-thresholding power, detects modules, computes eigengenes, module membership, and module-trait correlations) — https://github.com/mortazavilab/PyWGCNA
- **WGCNA (R reference)** (Reference implementation/methodology; the paper recommends reading the original WGCNA publication if unfamiliar with the method)

## Examples

```
pip install PyWGCNA
import PyWGCNA; pywgcna = PyWGCNA.WGCNA(name='3xTg-AD', species='mouse', networkType='unsigned', TOMType='unsigned', savePath='results'); pywgcna.addSampleInfo(path='3xTg-AD_colData.csv'); pywgcna.preprocess(filter=True); pywgcna.analyzeWGCNA(); pywgcna.saveWGCNA()
```

## Evaluation signals

- On the Quick Start 3xTg-AD female hippocampus dataset (38 samples, three ages), the output must contain 17 modules
- The dark gray module must contain exactly 380 genes and be strongly correlated with the 3xTg-AD genotype and the 18-month time point
- Gene-to-module assignment table exports with one row per gene and a valid module color label for each gene (no unassigned genes unless filtered out in preprocessing)
- Module eigengene values and module membership (kME) values are computed for every detected module
- Soft-thresholding power selection is documented/justified before network construction, matching the PyWGCNA workflow

## Limitations

- Python version requirement differs between sources: the paper states Python 3.9 or greater is required, while the README for recent releases states 3.10 or greater — check the release you install
- PyWGCNA's tutorials compare datasets via dedicated notebooks (two objects, multiple objects, gene marker lists); cross-dataset module comparison requires these separate workflows rather than a single run
- Results depend on soft-thresholding power choice and preprocessing (filtering low-expression genes); deviating from the tutorial preprocessing may change module counts and sizes relative to published results
- The paper applies PyWGCNA to brain bulk RNA-seq from MODEL-AD; performance and module biology are characterized for bulk datasets, not single-cell data

## Evidence

- [abstract] We introduce the PyWGCNA Python package, which is designed to identify co-expression modules from large RNA-seq datasets.: "We introduce the PyWGCNA Python package, which is designed to identify co-expression modules from large RNA-seq datasets."
- [methods] To install PyWGCNA, python version 3.9 or greater is required.: "To install PyWGCNA, python version 3.9 or greater is required."
- [readme] Quick Start: How to load data into PyWGCNA, find modules, and analyze them: [evidence span withheld — anchor status: external_doc]
- [intro] It can be used for finding clusters (modules) of highly correlated genes, for summarizing such clusters using the module eigengene: "It can be used for finding clusters (modules) of highly correlated genes, for summarizing such clusters using the module eigengene"
- [intro] for relating modules to one another and to external sample traits (using eigengene network methodology): "for relating modules to one another and to external sample traits (using eigengene network methodology)"
