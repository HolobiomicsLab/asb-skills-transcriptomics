---
name: memory-failure-diagnostics
description: Use when a large RNA-seq expression matrix (e.g. tens of thousands of
  transcripts) makes a co-expression tool slow or prone to out-of-memory failure,
  and you need to characterize the scaling limit empirically.
when_to_use_negative:
- The goal is biological interpretation of identified modules (functional enrichment
  via GO/KEGG/REACTOME or PPI via STRING) rather than tool performance or memory scaling.
- The expression matrix is small enough that both implementations complete without
  memory pressure, so no scaling crossover or OOM boundary can be observed.
- Input is not a gene/transcript co-expression matrix amenable to WGCNA-style module
  identification.
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: PyWGCNA
  role: Python implementation of weighted correlation network analysis run on each
    feature subset to time module identification and confirm it scales to 96,000 transcripts
  repo: https://github.com/mortazavilab/PyWGCNA
- name: WGCNA
  role: Reference R implementation run on the same subsets under identical settings
    to record runtimes and out-of-memory failures
- name: Python
  role: Runtime environment (version 3.9+ required per the paper) for installing and
    executing PyWGCNA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:b8b5bbcade4c971d32e2569c2cb7f030752ec049a0dadbff3cf779da5440cc47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - PyWGCNA
  - Python
  - WGCNA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# memory-failure-diagnostics

## Summary

A benchmarking-and-diagnostics skill for probing where co-expression network implementations hit memory walls: it runs PyWGCNA and R WGCNA on expression matrices with increasing feature counts (up to 16,000 genes and the full 96,000-transcript ENCODE matrix) to expose out-of-memory failures and quantify the runtime scaling crossover between the two implementations.

## When to use

Use when a large RNA-seq expression matrix (e.g. tens of thousands of transcripts) makes a co-expression tool slow or prone to out-of-memory failure, and you need to characterize the scaling limit empirically.

## When NOT to use

- The goal is biological interpretation of identified modules (functional enrichment via GO/KEGG/REACTOME or PPI via STRING) rather than tool performance or memory scaling.
- The expression matrix is small enough that both implementations complete without memory pressure, so no scaling crossover or OOM boundary can be observed.
- Input is not a gene/transcript co-expression matrix amenable to WGCNA-style module identification.

## Inputs

- ENCODE gene-level expression matrix (100 samples) as used in the PyWGCNA paper
- ENCODE transcript-level expression matrix (96,000 transcripts, 100 samples)
- Reduced-feature subsets sampled at increasing gene/transcript counts (up to 16,000 genes)
- PyWGCNA Python package (Python 3.9+ per the paper; README requires 3.10+) and the R WGCNA package

## Outputs

- runtime_comparison.csv with columns for feature count, package, and mean runtime
- Per-replicate wall-clock runtime records (three replicates per package per subset size)
- Out-of-memory failure log for the R WGCNA runs on large feature sets

## How to apply

Download the ENCODE gene-level and transcript-level expression matrices (100 samples) and generate reduced-feature subsets by sampling genes/transcripts at increasing counts up to 16,000 genes plus the full 96,000-transcript matrix. Run PyWGCNA co-expression module identification on each subset three times, recording wall-clock runtime per replicate, then run the R WGCNA package on identical subsets under identical settings, logging both runtimes and any out-of-memory failures. Average the three replicate timings per package per subset size and compile runtime_comparison.csv with columns for feature count, package, and mean runtime. The rationale comes from the PyWGCNA paper: the two implementations perform comparably up to 16,000 genes, but PyWGCNA is roughly twice as fast above that threshold and identifies modules for 96,000 transcripts where R WGCNA fails on memory — so the crossover point and the OOM boundary are exactly the diagnostics this skill extracts.

## Related tools

- **PyWGCNA** (Python implementation of weighted correlation network analysis run on each feature subset to time module identification and confirm it scales to 96,000 transcripts) — https://github.com/mortazavilab/PyWGCNA
- **WGCNA** (Reference R implementation run on the same subsets under identical settings to record runtimes and out-of-memory failures)
- **Python** (Runtime environment (version 3.9+ required per the paper) for installing and executing PyWGCNA)

## Examples

```
pip install PyWGCNA
```

## Evaluation signals

- runtime_comparison.csv exists with the required schema: feature count, package, and mean runtime columns.
- Averaged timings (three replicates per package per subset) show comparable runtimes up to 16,000 genes.
- Mean runtimes above the 16,000-gene threshold show roughly a 2x speedup for PyWGCNA over R WGCNA.
- R WGCNA fails with an out-of-memory error on the 96,000-transcript matrix while PyWGCNA still identifies modules on the same dataset.
- Each package's timing per subset size is an average over exactly three replicate runs, not a single measurement.

## Limitations

- The R WGCNA implementation cannot compute a co-expression network on the 96,000-transcript dataset due to memory constraints, so no timing above that point is obtainable for R.
- Scaling behavior is benchmarked on ENCODE matrices with 100 samples; results may differ for other sample counts or datasets.
- The provided article text lacks a discussion section ('No changelog found.' placeholder), so no author-stated caveats beyond the abstract/methods are available.

## Evidence

- [abstract] We introduce the PyWGCNA Python package, which is designed to identify co-expression modules from large RNA-seq datasets.: "We introduce the PyWGCNA Python package, which is designed to identify co-expression modules from large RNA-seq datasets."
- [abstract] R implementation is slow: "However, the current R implementation is slow"
- [abstract] PyWGCNA faster than R WGCNA: "PyWGCNA has a faster implementation than the R version of WGCNA and several additional downstream analysis modules"
- [methods] Python version requirement: "To install PyWGCNA, python version 3.9 or greater is required."
- [readme] README Python version and install: [evidence span withheld — anchor status: external_doc]
