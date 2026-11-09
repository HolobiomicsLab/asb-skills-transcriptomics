---
name: rna-seq-expression-matrix-loading
description: Use when you have a raw RNA-seq expression matrix (genes or transcripts
  x samples, e.g. ENCODE matrices with ~100 samples) that must be formatted, cleaned,
  and loaded into PyWGCNA before co-expression module identification.
when_to_use_negative:
- Input is already a PyWGCNA object or a previously processed network — use the comparison
  tutorials instead of re-loading
- The goal is differential expression or per-gene statistics rather than co-expression
  module identification across many samples
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: PyWGCNA
  role: Python package that loads, cleans, and formats the RNA-seq expression matrix
    and performs downstream WGCNA module identification
  repo: https://github.com/mortazavilab/PyWGCNA
- name: Python 3
  role: Runtime environment required to install and run PyWGCNA (Python 3.10+ per
    README)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:96f2d3e929cdc535eb912940f7f0282b60942b69ab22b3dc9d397bfa30819979
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - PyWGCNA
  - Python
  - Python 3
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# rna-seq-expression-matrix-loading

## Summary

Loads and formats gene- or transcript-level RNA-seq count/TPM expression matrices into PyWGCNA so that co-expression modules can be identified. Correct data formatting is the prerequisite step before WGCNA module detection, and PyWGCNA provides a dedicated data-input/cleaning tutorial for it.

## When to use

Use when you have a raw RNA-seq expression matrix (genes or transcripts x samples, e.g. ENCODE matrices with ~100 samples) that must be formatted, cleaned, and loaded into PyWGCNA before co-expression module identification.

## When NOT to use

- Input is already a PyWGCNA object or a previously processed network — use the comparison tutorials instead of re-loading
- The goal is differential expression or per-gene statistics rather than co-expression module identification across many samples

## Inputs

- Gene-level or transcript-level RNA-seq expression matrix (counts/TPM) with samples as columns and genes/transcripts as rows
- Sample metadata / trait table for downstream module-trait analysis
- Feature-count subset definitions (e.g. sampled gene lists up to 16,000 genes) when benchmarking scaling

## Outputs

- A cleaned, PyWGCNA-compatible expression matrix object ready for module identification
- A PyWGCNA object from which co-expression modules, eigengenes, and module membership measures can be computed

## How to apply

Install PyWGCNA from PyPi (requires Python 3.10 or greater per the README) or clone the mortazavilab/PyWGCNA repository and run 'pip install .'. Follow the 'Data input, cleaning and pre-processing' tutorial to format the expression matrix so sample and gene metadata align with PyWGCNA's expected input structure, then use the Quick Start workflow to load the data. For benchmarking contexts, generate reduced-feature subsets by sampling genes at increasing feature counts (e.g. up to 16,000 genes) or use the full transcript-level matrix (e.g. 96,000 transcripts) before loading, since PyWGCNA scales to large feature counts where the R WGCNA implementation exhausts memory. Once loaded, proceed directly to module identification and downstream analyses such as eigengene-based module-trait relationships.

## Related tools

- **PyWGCNA** (Python package that loads, cleans, and formats the RNA-seq expression matrix and performs downstream WGCNA module identification) — https://github.com/mortazavilab/PyWGCNA
- **Python 3** (Runtime environment required to install and run PyWGCNA (Python 3.10+ per README))

## Examples

```
pip install PyWGCNA
```

## Evaluation signals

- The loaded matrix matches the documented PyWGCNA data-format expectations from tutorials/Data_format.md (consistent gene/transcript and sample identifiers)
- Feature count of the loaded object matches the intended subset (e.g. 16,000 genes or 96,000 transcripts) before module identification
- PyWGCNA runs under Python 3.10+ installed via 'pip install PyWGCNA' without import errors
- Downstream Quick Start (find modules, analyze them) completes on the loaded data, indicating the input was accepted
- For benchmark subsets, the same matrix loads successfully for both the reduced-feature and full 96,000-transcript cases

## Limitations

- Python version matters: the article states Python 3.9 or greater is required while the current README requires Python 3.10 or greater, so environment pinning affects reproducibility
- PyWGCNA's speed/memory advantage is dataset-size dependent: performance was similar to R WGCNA up to 16,000 genes, with roughly 2x speedup only on larger datasets
- No discussion-section text was available in the provided document, so failure modes beyond installation and memory scaling are not documented here

## Evidence

- [readme] How to format, clean and preprocess your input data for PyWGCNA: [evidence span withheld — anchor status: external_doc]
- [methods] Quick Start: load data, find modules, analyze them: "`Quick Start <https://github.com/mortazavilab/PyWGCNA/tree/main/tutorials/Quick_Start.ipynb>`_: How to load data into PyWGCNA, find modules, and analyze them"
- [readme] Python version requirement for installation: [evidence span withheld — anchor status: external_doc]
- [abstract] PyWGCNA designed for large RNA-seq datasets: "We introduce the PyWGCNA Python package, which is designed to identify co-expression modules from large RNA-seq datasets."
