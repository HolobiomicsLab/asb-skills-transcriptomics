---
name: trajectory-inference-method-benchmarking
description: Decision support for choosing a trajectory inference method when a branching
  single-cell dataset has trusted ground-truth population labels and lineage marker
  genes (e.g. Gata1, Pax5) to score branch assignments against.
when_to_use_negative:
- The dataset lacks reliable ground-truth population labels, so precision/recall against
  sorted populations cannot be computed
- The data is scATAC-seq chromatin accessibility rather than transcriptomic/qPCR data
  and the methods being ranked do not support accessibility data
- The trajectory structure is cyclic or disconnected — STREAM's authors note further
  improvement is necessary to properly describe such general structures, so F1 rankings
  from these tools would not transfer
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STREAM
  role: Trajectory inference method under evaluation; reconstructs branching trajectories
    with pseudotime from transcriptomic and epigenomic data, and was the top-scoring
    method by F1
  repo: https://github.com/pinellolab/STREAM
- name: Monocle2
  role: Competing trajectory inference method included in the F1 comparison; over-branches,
    yielding high precision but poor recall
- name: ElPiGraph
  role: Core principal-graph algorithm behind STREAM and a competing method in the
    benchmark
  repo: sysbio-curie/ElPiGraph.R
- name: anndata
  role: Data structure in which STREAM is written; used to load the qPCR cell-by-TF
    matrix
- name: conda
  role: Environment manager for reproducible per-method installation from Bioconda
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:d48ca0c5d2f6b8030b6c31f9607331761141d3e40db654346fc834f55e9d14a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pinellolab/STREAM
  tools:
  - ElPiGraph
  - Monocle2
  - Python 3
  - STREAM
  - anndata
  - conda
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# trajectory-inference-method-benchmarking

## Summary

A head-to-head benchmarking protocol that evaluates trajectory inference methods (STREAM, Monocle2, ElPiGraph) by computing per-branch precision, recall and F1 for lineage marker genes against sorted-population ground-truth labels, then ranking methods by F1.

## When to use

Decision support for choosing a trajectory inference method when a branching single-cell dataset has trusted ground-truth population labels and lineage marker genes (e.g. Gata1, Pax5) to score branch assignments against.

## When NOT to use

- The dataset lacks reliable ground-truth population labels, so precision/recall against sorted populations cannot be computed
- The data is scATAC-seq chromatin accessibility rather than transcriptomic/qPCR data and the methods being ranked do not support accessibility data
- The trajectory structure is cyclic or disconnected — STREAM's authors note further improvement is necessary to properly describe such general structures, so F1 rankings from these tools would not transfer

## Inputs

- Single-cell expression matrix (~270 cells × ~170 transcription factors) in anndata format, from single-cell qPCR hematopoiesis data with six sorted populations
- Sorted population labels used as ground truth
- Lineage marker genes of interest (Gata1, Pax5)
- Trajectory graphs inferred by each method (STREAM, Monocle2, ElPiGraph)

## Outputs

- Per-branch precision, recall and F1 for every method-marker combination
- Ranked method comparison table written to f1_benchmark_results.csv
- Reconstructed branching trajectories per method

## How to apply

Install each competitor method in an isolated environment (e.g. STREAM via Bioconda: `conda create -n env_stream python=3.7 stream=1.0 jupyter` using the anndata data structure) and run all methods on the same input dataset — here the ~270-cell single-cell qPCR hematopoiesis dataset with ~170 transcription-factor features across six sorted populations. For each method and each marker gene, select the two reconstructed branches containing the largest number of cells expressing that marker, treat the sorted-population labels as ground truth, and compute per-branch precision, recall and F1 for every method-marker combination. Rank methods by F1 and write the results table (f1_benchmark_results.csv); the expected outcome from the source work is that STREAM attains the highest F1 for both Gata1 and Pax5 by balancing precision and recall, while SCUBA shows lower recall, Monocle2 over-branches (high precision, poor recall), and Wishbone/DPT lose precision due to misplaced Gata1-expressing cells.

## Related tools

- **STREAM** (Trajectory inference method under evaluation; reconstructs branching trajectories with pseudotime from transcriptomic and epigenomic data, and was the top-scoring method by F1) — https://github.com/pinellolab/STREAM
- **Monocle2** (Competing trajectory inference method included in the F1 comparison; over-branches, yielding high precision but poor recall)
- **ElPiGraph** (Core principal-graph algorithm behind STREAM and a competing method in the benchmark) — sysbio-curie/ElPiGraph.R
- **anndata** (Data structure in which STREAM is written; used to load the qPCR cell-by-TF matrix)
- **conda** (Environment manager for reproducible per-method installation from Bioconda)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter; conda activate env_stream; jupyter notebook
```

## Evaluation signals

- STREAM attains the highest F1 score for both Gata1 and Pax5 among all evaluated methods
- Per-branch precision and recall are reported for every method-marker combination, not only aggregate scores
- SCUBA shows lower recall, Monocle2 shows high precision with poor recall, and Wishbone/DPT show low precision due to misplaced Gata1-expressing cells — the pattern-check for the comparison table
- The same ~270-cell anndata input is fed to every method, so score differences reflect inference quality, not input differences
- f1_benchmark_results.csv is written and ranks all methods consistently with the per-branch metrics

## Limitations

- STREAM does not yet adapt to cyclic or disconnected graph topologies; further improvement is necessary to describe these more general structures
- Many current trajectory inference methods, including STREAM, have not been shown to scale to assays profiling millions of cells
- Only branches containing the largest number of marker-expressing cells are scored; branches with sparse marker expression may be underrepresented in the F1 comparison
- Ground truth is limited to the six sorted populations, so F1 reflects agreement with sorting labels rather than absolute developmental truth

## Evidence

- [readme] Install STREAM via Bioconda into a Python 3 conda environment: "Create an environment named `env_stream` , install **stream**, **jupyter**, and activate it"
- [methods] Load qPCR dataset with anndata data structure: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [discussion] STREAM tested across organisms and technologies including qPCR: "STREAM has been extensively tested using several published datasets from different organisms (zebrafish, mouse, human) and single-cell technologies (qPCR, scRNA-seq, scATAC-seq)"
- [discussion] STREAM does not require prior knowledge of branchings: "Our method does not require prior knowledge such as time points, start cell, or the number of branching events to reconstruct trajectories"
- [readme] Reproduce STREAM install command in example invocation: "$ conda create -n env_stream python=3.7 stream=1.0 jupyter
$ conda activate env_stream"
