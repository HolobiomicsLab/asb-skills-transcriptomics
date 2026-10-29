---
name: fusion-caller-performance-evaluation
description: Use when benchmarking a circRNA/fusion caller against a simulated RNA-Seq
  truth set, or when estimating a maximum achievable sensitivity ceiling (e.g. Acfs'
  86.6% after excluding same-strand, same-chromosome gene-pair backsplices).
when_to_use_negative:
- Input is real (non-simulated) RNA-Seq data without a known truth set of backsplices
  — sensitivity/precision cannot be computed without ground truth.
- The caller under evaluation does not impose the same-strand, same-chromosome exclusion
  (the 86.6% ceiling is specific to Acfs' algorithm, not a general benchmark constraint).
- The goal is visualization rather than performance scoring — use INTEGRATE-Vis with
  the fcirc.txt output instead of re-running the benchmark.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: INTEGRATE-Circ
  role: Fusion junction detection tool whose sensitivity, precision, and accuracy
    are benchmarked on simulated RNA-Seq data; its fcirc.txt output is scored against
    the truth set.
  repo: https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC
- name: cmake
  role: Build dependency (version >= 2.8) for compiling INTEGRATE-Circ prior to running
    the benchmark.
- name: g++
  role: Compiler required (version > 12 recommended) to build INTEGRATE-Circ; older
    compilers fail.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad569
    title: 'INTEGRATE-Circ and INTEGRATE-Vis: unbiased detection and visualization
      of fusion-derived circular RNA'
  source_content_hash: sha256:9512fd7f9885d8c5fd86960ff2b181012acac7a2a1831ddfea0c040dabf1b43f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INTEGRATE-Circ
  - cmake
  - g++
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# fusion-caller-performance-evaluation

## Summary

Benchmark fcircRNA/gene-fusion callers (e.g. INTEGRATE-Circ vs. Acfs) on simulated RNA-Seq truth sets by computing sensitivity, precision, and accuracy, including caller-specific sensitivity ceilings such as Acfs' exclusion of same-strand, same-chromosome backsplices.

## When to use

Use when benchmarking a circRNA/fusion caller against a simulated RNA-Seq truth set, or when estimating a maximum achievable sensitivity ceiling (e.g. Acfs' 86.6% after excluding same-strand, same-chromosome gene-pair backsplices).

## When NOT to use

- Input is real (non-simulated) RNA-Seq data without a known truth set of backsplices — sensitivity/precision cannot be computed without ground truth.
- The caller under evaluation does not impose the same-strand, same-chromosome exclusion (the 86.6% ceiling is specific to Acfs' algorithm, not a general benchmark constraint).
- The goal is visualization rather than performance scoring — use INTEGRATE-Vis with the fcirc.txt output instead of re-running the benchmark.

## Inputs

- Simulated RNA-Seq benchmark truth set of backsplices (fusion-derived circRNA events)
- Caller prediction calls over the same simulated reads (e.g. INTEGRATE-Circ fcirc.txt, Acfs calls)
- Gene-level chromosome and strand annotations (e.g. Ensembl annotation file such as example-data/hg19.chr21.annotation.txt)
- Reference genome FASTA (e.g. hg19.chr21.fa) for the BWT required by INTEGRATE-Circ

## Outputs

- Per-caller sensitivity, precision, and accuracy over the simulated truth set
- Caller-specific maximum achievable sensitivity ceiling (computed value in acfs_max_sensitivity.txt; 86.6% for Acfs)
- Reduced detectable event space after excluding same-strand, same-chromosome backsplices

## How to apply

Load the simulated RNA-Seq benchmark truth set of backsplices alongside each caller's predictions (e.g. INTEGRATE-Circ's fcirc.txt output, generated after building a BWT of the reference with `Integrate-Circ mkbwt` and running fusion analysis). For the Acfs ceiling analysis, join each simulated backsplice to gene-level chromosome and strand annotations, exclude backsplices whose two contributing genes lie on the same strand of the same chromosome (four events in the published simulation), and define the reduced detectable event space. Compute maximum achievable sensitivity as the fraction of the reduced truth set any caller could in principle recover — for Acfs this ceiling is 86.6%, confirming it could not have outperformed INTEGRATE-Circ's average sensitivity. Validate the computed ceiling against the reported figure within rounding tolerance and record it (e.g. in acfs_max_sensitivity.txt). To run INTEGRATE-Circ itself, clone ChrisMaherLab/INTEGRATE-CIRC, compile with cmake (>= 2.8) and g++ > 12, add build/bin to $PATH, then run over the simulated reads and score against the truth set.

## Related tools

- **INTEGRATE-Circ** (Fusion junction detection tool whose sensitivity, precision, and accuracy are benchmarked on simulated RNA-Seq data; its fcirc.txt output is scored against the truth set.) — https://www.github.com/ChrisMaherLab/INTEGRATE-CIRC
- **cmake** (Build dependency (version >= 2.8) for compiling INTEGRATE-Circ prior to running the benchmark.)
- **g++** (Compiler required (version > 12 recommended) to build INTEGRATE-Circ; older compilers fail.)

## Examples

```
Integrate-Circ mkbwt /path/to/reference.fa -dir /path/to/bwt_directory
```

## Evaluation signals

- Computed maximum achievable sensitivity for Acfs reproduces the reported 86.6% ceiling on the simulated dataset within rounding tolerance.
- Exactly four simulated backsplices are excluded when filtering same-strand, same-chromosome gene pairs from the truth set.
- The Acfs ceiling (86.6%) is at or below INTEGRATE-Circ's average sensitivity, confirming INTEGRATE-Circ outperformed other tools on simulated data.
- INTEGRATE-Circ compiles cleanly with cmake >= 2.8 and g++ > 12 and produces a fcirc.txt file scorable against the truth set.

## Limitations

- Acfs' same-chromosome/same-strand exclusion rule caps its sensitivity at 86.6% in the simulation regardless of parameter tuning, so apparent underperformance relative to INTEGRATE-Circ may reflect this structural ceiling rather than algorithmic weakness.
- No version number is stated for INTEGRATE-Circ in the paper; results may depend on the cloned commit.
- INTEGRATE-Circ cannot be compiled by some older compilers (recommend g++ > 12), which can block reproduction of the benchmark.

## Evidence

- [abstract] INTEGRATE-Circ was more sensitive, precise and accurate than other tools based on our analysis of simulated RNA-Seq data: "We found that INTEGRATE-Circ was more sensitive, precise and accurate than other tools based on our analysis of simulated RNA-Seq data"
- [intro] INTEGRATE-Circ is a fusion junction detection tool for fcircRNA: "INTEGRATE-Circ is a fusion junction detection tool, capable of identifying gene fusion transcripts and various isoforms that may result from these fusions, including fusion-derived circRNA (fcircRNA)."
- [methods] Compile the tool with cmake >= 2.8: "Next, compile the tool (requires cmake version >= 2.8)"
- [readme] Older compilers cannot build INTEGRATE-Circ: "Note that INTEGRATE-Circ cannot be compiled by some older compilers (recommend version >12 of g++)"
- [readme] BWT of the reference genome must be built before analysis: "Prior to analyzing sequencing data, INTEGRATE-Circ must perform a BWT (Burrows-Wheeler transform) of the reference genome."
