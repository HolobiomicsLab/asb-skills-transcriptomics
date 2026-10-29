---
name: cpp-command-line-tool-execution
description: Use when a workflow step requires building a scientific tool from source
  with a C++11 compiler and invoking its CLI binary on CSV inputs (e.g., running Sphetcher
  on an expression matrix to obtain a sketch indicator vector).
when_to_use_negative:
- The input is not a cell-by-feature expression matrix (rows must be samples/cells,
  columns features/genes/PCs) — Sphetcher's CLI expects exactly this CSV layout.
- The target tool provides prebuilt binaries or a non-C++ pipeline; this skill specifically
  covers cloning and compiling C++11 source via make before CLI invocation.
- The analysis requires uniform random subsampling or box-based geometric sketching
  rather than sphere-based representative selection covering the transcriptomic space.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: Sphetcher
  role: C++11 CLI binary compiled from source via make; invoked as 'sphetcher expression_matrix.csv
    sketch_size sketch_indicator_output.csv' to produce a spherical sketch of single-cell
    transcriptomic data
  repo: https://github.com/canzarlab/Sphetcher
- name: git
  role: Used to clone the canzarlab/Sphetcher repository before compilation
  repo: https://github.com/canzarlab/Sphetcher
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:2dd0b3d228c3ebc5187c1786e3bf1f4d39b29f0915dab366e39b9dfefc10fa7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Sphetcher
  - git
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cpp-command-line-tool-execution

## Summary

Clone, compile, and run a C++11 command-line tool (Sphetcher) from a GitHub repository to execute a scientific pipeline step — spherical thresholding greedy set-cover sketching of single-cell RNA-seq data — via its documented CLI interface.

## When to use

Use when a workflow step requires building a scientific tool from source with a C++11 compiler and invoking its CLI binary on CSV inputs (e.g., running Sphetcher on an expression matrix to obtain a sketch indicator vector).

## When NOT to use

- The input is not a cell-by-feature expression matrix (rows must be samples/cells, columns features/genes/PCs) — Sphetcher's CLI expects exactly this CSV layout.
- The target tool provides prebuilt binaries or a non-C++ pipeline; this skill specifically covers cloning and compiling C++11 source via make before CLI invocation.
- The analysis requires uniform random subsampling or box-based geometric sketching rather than sphere-based representative selection covering the transcriptomic space.

## Inputs

- expression_matrix.csv — CSV matrix with rows as cells and columns as features/genes/PCs (e.g., /data/zeisel_pca.csv)
- sketch_size — integer number of samples to obtain (e.g., 1000, or 20296 for 1% of the MOCA 2,026,641-cell dataset)
- class_labels.csv (optional) — column vector of integer class labels between 1 and K encoding prior information such as cell label or collection time point
- l_min (optional) — minimum number of representatives to sample from each class

## Outputs

- sketch_indicator_output.csv — binary indicator vector of length n (number of input samples), with exactly sketch_size ones marking selected sketch cells and zeros elsewhere

## How to apply

Clone the tool's GitHub repository (git clone https://github.com/canzarlab/Sphetcher), enter the source directory (cd src), and build with make, ensuring the compiler supports C++11. Prepare the input as a CSV expression matrix where rows are samples (cells) and columns are features (genes, transcripts, or principal components), e.g. the example /data/zeisel_pca.csv. Invoke the binary following the documented usage: 'sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv', or the four-argument variant with class_labels.csv and l_min to preserve a minimum number of representatives per prior class. Decide the sketch_size parameter (e.g., 1000 for the example, or 1% of cells such as 20296 for the 2,026,641-cell MOCA dataset), then parse the output indicator vector and verify it matches the requested sketch size.

## Related tools

- **Sphetcher** (C++11 CLI binary compiled from source via make; invoked as 'sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv' to produce a spherical sketch of single-cell transcriptomic data) — https://github.com/canzarlab/Sphetcher
- **git** (Used to clone the canzarlab/Sphetcher repository before compilation) — https://github.com/canzarlab/Sphetcher

## Examples

```
git clone https://github.com/canzarlab/Sphetcher && cd Sphetcher/src && make && ./sphetcher zeisel_pca.csv 1000 sketch_indicator_output.csv
```

## Evaluation signals

- Compilation succeeds with a C++11-supporting compiler via 'make' in the src directory without errors.
- The output sketch_indicator_output.csv is a binary indicator vector over all n input samples.
- The indicator vector contains exactly sketch_size ones (e.g., exactly 1000 for the zeisel_pca.csv example with sketch size 1000).
- The number of ones equals the requested fraction of the dataset, e.g., 20296 cells for 1% of the 2,026,641-cell MOCA matrix.
- Runtime for large datasets is on the expected order, e.g., Sphetcher-H completes on the two-million-cell Cao dataset in roughly 16 minutes (grid sampling ~10 s, distances ~600 s, set cover ~400 s).

## Limitations

- A compiler that supports C++11 is required to build sphetcher; older toolchains may fail.
- Sphetcher uses a specific distance metric (most experiments in the study used Pearson correlation), and its underlying model might be less compatible with assumptions made by other downstream algorithms; the interplay of sketching with a given distance metric and subsequent analysis was not systematically addressed.
- Benefits of leveraging prior categorical knowledge (e.g., cell types) in sketch selection are only demonstrated at proof-of-principle level; additional experiments are required.

## Evidence

- [methods] A compiler that supports C++11 is needed to build sphetcher.: "A compiler that supports C++11 is needed to build sphetcher."
- [readme] git clone and make: "git clone  https://github.com/canzarlab/Sphetcher
cd src
make"
- [readme] Usage invocation: "sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv"
- [methods] Input matrix format: "First you need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs)."
- [readme] Output indicator format: "sketch_indicator_output.csv : an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise"
- [intro] Scalability finding: "Sphetcher requires only 16 minutes to compute a sketch for a mouse embryonic dataset comprising two million cells."
