---
name: count-matrix-preprocessing-and-filtering
description: Use when a raw gene-by-sample count matrix is the input to PyWGCNA and
  lowly expressed genes must be removed per the Data_format tutorial so network construction
  (adjacency, TOM, dynamic tree cut) is not driven by noise.
when_to_use_negative:
- The matrix is already filtered/cleaned per the Data_format tutorial and ready for
  adjacency computation — re-filtering would drop genes needed for module recovery
- The goal is differential expression testing or single-cell analysis rather than
  co-expression module detection from bulk RNA-seq samples
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PyWGCNA
  role: Python package whose Data_format tutorial defines the input cleaning and low-count
    gene filtering applied before module detection
  repo: github.com/mortazavilab/PyWGCNA
- name: Python
  role: 'runtime for PyWGCNA; python version 3.9 or greater required (README: 3.10+)'
  repo: https://github.com/mortazavilab/PyWGCNA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:706a193e55e12c414cd8ef99214770a9451ce6f3bae8a9b2039308bda3cb6b0d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyWGCNA
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# count-matrix-preprocessing-and-filtering

## Summary

Format, clean, and filter a bulk RNA-seq count matrix (e.g., the 192-sample 5xFAD/C57BL/6J cortex and hippocampus dataset) so that only well-expressed genes are retained before building a WGCNA co-expression network with PyWGCNA.

## When to use

Use when a raw gene-by-sample count matrix is the input to PyWGCNA and lowly expressed genes must be removed per the Data_format tutorial so network construction (adjacency, TOM, dynamic tree cut) is not driven by noise.

## When NOT to use

- The matrix is already filtered/cleaned per the Data_format tutorial and ready for adjacency computation — re-filtering would drop genes needed for module recovery
- The goal is differential expression testing or single-cell analysis rather than co-expression module detection from bulk RNA-seq samples

## Inputs

- bulk RNA-seq gene-by-sample count matrix (e.g., 192-sample 5xFAD/C57BL/6J cortex and hippocampus dataset)
- sample trait table (age, genotype, tissue, sex) keyed to the same samples
- PyWGCNA object loaded per the Data_format tutorial

## Outputs

- filtered count matrix containing only well-expressed genes, ready for pickSoftThreshold and TOM construction
- sample-aligned trait table for downstream module-trait correlation

## How to apply

Download the count matrix and sample trait table (e.g., from github.com/mortazavilab/PyWGCNA/tutorials/5xFAD_paper), then follow the Data_format tutorial to format, clean, and preprocess the data inside a PyWGCNA object. Filter genes with low counts so only well-expressed genes remain for network construction, keeping the sample trait table aligned to the filtered gene-by-sample matrix. Python 3.9 or greater is required per the article (the README specifies 3.10+). After filtering, proceed to soft-thresholding power selection with pickSoftThreshold for scale-free topology fit, then adjacency, TOM, and dynamic tree cut module identification. The filtering decision directly conditions downstream results: on the 5xFAD dataset this path recovers 17 modules (e.g., coral with 1335 genes, white with 435 genes) associated with age, genotype, tissue, and sex.

## Related tools

- **PyWGCNA** (Python package whose Data_format tutorial defines the input cleaning and low-count gene filtering applied before module detection) — github.com/mortazavilab/PyWGCNA
- **Python** (runtime for PyWGCNA; python version 3.9 or greater required (README: 3.10+)) — https://github.com/mortazavilab/PyWGCNA

## Evaluation signals

- Filtered matrix contains only well-expressed genes; low-count genes are absent relative to the raw matrix
- Matrix and trait table remain sample-aligned after preprocessing (same 192 samples in the 5xFAD case)
- pickSoftThreshold yields a candidate power with acceptable scale-free topology fit on the filtered data
- Downstream dynamic tree cut produces a plausible module set (reference result: 17 modules on the 5xFAD dataset, coral 1335 genes, white 435 genes)

## Limitations

- Filtering thresholds are not enumerated numerically in the provided context — they follow the Data_format tutorial, so results depend on the tutorial's exact cutoffs
- Installation requirements differ between the article (Python 3.9+) and the README (Python 3.10+); use the README's requirement for current releases

## Evidence

- [methods] Data input, cleaning and pre-processing: "`Data input, cleaning and pre-processing <https://github.com/mortazavilab/PyWGCNA/tree/main/tutorials/Data_format.md>`_"
- [abstract] identify co-expression modules from large RNA-seq datasets: "We introduce the PyWGCNA Python package, which is designed to identify co-expression modules from large RNA-seq datasets."
- [readme] format, clean and preprocess input data: [evidence span withheld — anchor status: not_found]
