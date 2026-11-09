---
name: cross-language-runtime-benchmarking
description: 'Use when evaluating whether a Python reimplementation is faster or more
  memory-robust than its R reference: you have matched expression datasets, both packages
  installed, and need replicate-timed scaling runs across feature-count subsets.'
when_to_use_negative:
- The analysis goal is biological (e.g. identifying co-expression modules or functional
  enrichment) rather than measuring implementation performance
- Only one implementation of the algorithm is available or only one language runtime
  is installed, so no cross-language comparison is possible
- Datasets are small enough that runtime differences are dominated by I/O overhead
  rather than the co-expression network computation
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PyWGCNA
  role: Python package whose co-expression module identification runtime is benchmarked
    against the R reference implementation
  repo: https://github.com/mortazavilab/PyWGCNA
- name: WGCNA
  role: R reference implementation of weighted gene co-expression network analysis
    used as the timing baseline
- name: Python 3
  role: 'Runtime environment for PyWGCNA benchmark replicates (paper: version 3.9+;
    README: 3.10+)'
  repo: https://github.com/mortazavilab/PyWGCNA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:57e2165a7dc66af64b66305daba3f6e33875c298fd1d90a319aa367307823833
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - PyWGCNA
  - Python
  - Python 3
  - WGCNA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-language-runtime-benchmarking

## Summary

A benchmarking protocol for comparing the wall-clock runtime of two implementations of the same algorithm in different languages (Python vs R), exemplified by PyWGCNA vs R WGCNA co-expression module detection on ENCODE expression matrices with increasing feature counts.

## When to use

Use when evaluating whether a Python reimplementation is faster or more memory-robust than its R reference: you have matched expression datasets, both packages installed, and need replicate-timed scaling runs across feature-count subsets.

## When NOT to use

- The analysis goal is biological (e.g. identifying co-expression modules or functional enrichment) rather than measuring implementation performance
- Only one implementation of the algorithm is available or only one language runtime is installed, so no cross-language comparison is possible
- Datasets are small enough that runtime differences are dominated by I/O overhead rather than the co-expression network computation

## Inputs

- ENCODE gene-level expression matrix (100 samples)
- ENCODE transcript-level expression matrix (100 samples, ~96,000 transcripts)
- Reduced-feature subsets sampled at increasing gene/transcript counts (up to 16,000 genes)
- Python 3.9+ environment with PyWGCNA and R environment with WGCNA

## Outputs

- Per-replicate wall-clock runtime records per package per subset size
- runtime_comparison.csv with columns for feature count, package, and mean runtime
- Record of out-of-memory failures for the R implementation on large feature counts

## How to apply

Download the reference expression matrices (here, ENCODE gene-level and transcript-level matrices over 100 samples) and generate reduced-feature subsets by sampling genes or transcripts at increasing feature counts, up to 16,000 genes and the full 96,000-transcript matrix. Run the Python package (PyWGCNA, installed with pip; requires Python 3.9+ per the paper, 3.10+ per the README) and the R WGCNA package on each subset under identical settings, with 3 replicates per package per subset size, recording wall-clock runtime per replicate and any out-of-memory failures. Average the three replicate timings per package per subset size and compile the results into a runtime-comparison table (e.g. runtime_comparison.csv with columns for feature count, package, and mean runtime). Rationale: the paper claims PyWGCNA matches R WGCNA up to 16,000 genes, is roughly 2x faster above that threshold, and completes module identification on the 96,000-transcript matrix where R WGCNA fails on memory, so the benchmark design must span those feature-count regimes to expose the scaling and memory differences.

## Related tools

- **PyWGCNA** (Python package whose co-expression module identification runtime is benchmarked against the R reference implementation) — https://github.com/mortazavilab/PyWGCNA
- **WGCNA** (R reference implementation of weighted gene co-expression network analysis used as the timing baseline)
- **Python 3** (Runtime environment for PyWGCNA benchmark replicates (paper: version 3.9+; README: 3.10+)) — https://github.com/mortazavilab/PyWGCNA

## Examples

```
pip install PyWGCNA
```

## Evaluation signals

- runtime_comparison.csv exists with the required columns: feature count, package, and mean runtime
- Mean runtimes for PyWGCNA and R WGCNA are comparable up to 16,000 genes
- Above 16,000 genes, PyWGCNA's mean runtime is roughly 2x faster than R WGCNA's
- PyWGCNA successfully identifies modules on the full 96,000-transcript matrix while R WGCNA records an out-of-memory failure on the same input
- Each package/subset-size cell is backed by 3 replicate timings that were averaged, not a single run

## Limitations

- The R implementation cannot compute a co-expression network on very large feature sets (~96,000 transcripts) due to memory constraints, so it cannot be timed at the top end of the scaling curve
- Runtime advantage of PyWGCNA only emerges above ~16,000 genes; at smaller feature counts the two packages perform similarly
- The paper's environment used Python 3.9 or greater, but the current README requires Python 3.10+, so environment versions affect reproducibility

## Evidence

- [abstract] Python package designed for large RNA-seq co-expression modules: "We introduce the PyWGCNA Python package, which is designed to identify co-expression modules from large RNA-seq datasets."
- [abstract] R WGCNA is slow: "Weighted gene co-expression network analysis (WGCNA) is frequently used to identify modules of genes that are co-expressed across many RNA-seq samples. However, the current R implementation is slow"
- [readme] Install PyWGCNA from PyPi: "### Install from PyPi (recommended)
To install the most recent release, run

`pip install PyWGCNA`"
- [methods] Python version requirement: "To install PyWGCNA, python version 3.9 or greater is required."
- [readme] Python 3.10+ per README: [evidence span withheld — anchor status: external_doc]
