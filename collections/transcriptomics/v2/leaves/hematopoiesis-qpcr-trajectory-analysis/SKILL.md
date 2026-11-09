---
name: hematopoiesis-qpcr-trajectory-analysis
description: Use when comparing branching trajectory inference methods on a small
  single-cell qPCR dataset where sorted population labels can serve as ground truth
  for lineage-specific marker genes such as Gata1 and Pax5.
when_to_use_negative:
- Input is scATAC-seq data with peaks/k-mers/motifs as features — use the stream_atac
  conda variant instead of the standard RNA-seq environment
- The dataset has cyclic or disconnected topology — the article notes STREAM's adaptation
  to these more general structures requires further improvement
- Cells may have totally different fates than the reference structure — the mapping
  procedure is not guaranteed optimal in that case
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STREAM
  role: Primary trajectory inference method benchmarked; reconstructs branching trajectories
    and pseudotime from the qPCR dataset using anndata
  repo: https://github.com/pinellolab/STREAM
- name: Monocle2
  role: Competing trajectory inference method evaluated for over-branching behavior
    (high precision, poor recall) on the hematopoiesis qPCR data
- name: ElPiGraph
  role: Core elastic principal-graph algorithm behind STREAM and an additional benchmarked
    method; R implementation available
  repo: https://github.com/Albluca/ElPiGraph.R
- name: anndata
  role: Data structure in which STREAM is written; holds the ~270-cell qPCR dataset
    with transcription-factor features and sorted population labels
- name: conda
  role: Environment manager used to install STREAM from Bioconda with Python 3.7
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:a0e6b4d3a5a1b890af667b4cdc7f49e27f48a711f55df591d65b5fa1dd7005fb
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# hematopoiesis-qpcr-trajectory-analysis

## Summary

Benchmark trajectory inference methods (STREAM, Monocle2, ElPiGraph) on the ~270-cell single-cell qPCR hematopoiesis dataset (~170 transcription factors, six sorted populations) by scoring branch assignments for Gata1 and Pax5 lineages with precision, recall, and F1 against sorted-population ground truth.

## When to use

Use when comparing branching trajectory inference methods on a small single-cell qPCR dataset where sorted population labels can serve as ground truth for lineage-specific marker genes such as Gata1 and Pax5.

## When NOT to use

- Input is scATAC-seq data with peaks/k-mers/motifs as features — use the stream_atac conda variant instead of the standard RNA-seq environment
- The dataset has cyclic or disconnected topology — the article notes STREAM's adaptation to these more general structures requires further improvement
- Cells may have totally different fates than the reference structure — the mapping procedure is not guaranteed optimal in that case

## Inputs

- ~270-cell single-cell qPCR hematopoiesis dataset with ~170 transcription-factor features across six sorted populations (anndata-compatible)
- Marker gene labels (Gata1, Pax5) per cell
- Sorted population labels used as ground truth

## Outputs

- Inferred branching trajectories with branch assignments per cell for each method (STREAM, Monocle2, ElPiGraph)
- Per-method, per-marker precision, recall, and F1 table (f1_benchmark_results.csv)
- Ranked comparison of trajectory inference methods by F1 for Gata1 and Pax5

## How to apply

Install STREAM via Bioconda into a Python 3.7 conda environment (`conda create -n env_stream python=3.7 stream=1.0 jupyter`), since STREAM is written using the `anndata` class. Load the ~270-cell qPCR hematopoiesis dataset with ~170 transcription-factor features and six sorted populations into anndata. Run each trajectory inference method (STREAM, Monocle2, ElPiGraph) to infer branching trajectories; for each method and each marker (Gata1, Pax5), select the two branches containing the largest number of cells expressing that marker. Treat the sorted population labels as ground truth and compute per-branch precision, recall, and F1 for every method-marker combination, then rank methods by F1. The rationale is that STREAM uses ElPiGraph as its core elastic principal-graph algorithm and does not require prior knowledge such as time points, start cell, or the number of branching events, which affects how each method balances precision against recall on this bifurcating hematopoiesis system.

## Related tools

- **STREAM** (Primary trajectory inference method benchmarked; reconstructs branching trajectories and pseudotime from the qPCR dataset using anndata) — https://github.com/pinellolab/STREAM
- **Monocle2** (Competing trajectory inference method evaluated for over-branching behavior (high precision, poor recall) on the hematopoiesis qPCR data)
- **ElPiGraph** (Core elastic principal-graph algorithm behind STREAM and an additional benchmarked method; R implementation available) — https://github.com/Albluca/ElPiGraph.R
- **anndata** (Data structure in which STREAM is written; holds the ~270-cell qPCR dataset with transcription-factor features and sorted population labels)
- **conda** (Environment manager used to install STREAM from Bioconda with Python 3.7)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter && conda activate env_stream && jupyter notebook
```

## Evaluation signals

- STREAM attains the highest F1 score for both Gata1 and Pax5 among all tested methods in f1_benchmark_results.csv
- Monocle2 exhibits high precision but poor recall (over-branching) on branch assignments
- Wishbone/DPT show low precision due to misplaced Gata1-expressing cells
- SCUBA shows lower recall than STREAM on the branch assignments
- Per-branch precision/recall computed against sorted population labels is reproducible when the two largest-marker-expressing branches are selected per method-marker combination

## Limitations

- ElPiGraph supports learning complex graphs but intuitive visualizations for sophisticated topologies require more work
- Many trajectory inference methods, including STREAM, do not scale to datasets profiling millions of cells
- The ~270-cell qPCR dataset is small, so conclusions are specific to this hematopoiesis benchmark
- Existing methods may require fitting a new model for new cells since the fitting procedure is not deterministic or the mapping feature is absent

## Evidence

- [results] STREAM achieves the highest F1 score for both the erythroid marker Gata1 and the lymphoid marker Pax5 among all trajectory inference methods tested: [evidence span withheld — anchor status: not_found]
- [methods] conda environment creation command from methods: "$ conda create -n env_stream python=3.7 stream=1.0 jupyter"
- [methods] anndata data structure statement: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [discussion] Monocle2 and STREAM applied to branching trajectory analysis: "so far very few trajectory inference methods (Monocle2 and STREAM) have been applied to analyze single-cell chromatin accessibility"
- [readme] STREAM installation with Bioconda from README: "Create an environment named `env_stream` , install **stream**, **jupyter**, and activate it with the following commands"
- [discussion] STREAM does not require prior branching knowledge: "Our method does not require prior knowledge such as time points, start cell, or the number of branching events to reconstruct trajectories"
