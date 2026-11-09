---
name: marker-gene-branch-assignment
description: Use when you have an inferred branching trajectory over ~270 single-cell
  qPCR hematopoiesis cells (~170 transcription factors, six sorted populations) and
  need to quantitatively evaluate which branches correspond to marker-expressing lineages.
when_to_use_negative:
- The input data has no sorted population labels or equivalent ground truth, so precision/recall/F1
  cannot be computed against a reference
- The dataset is scATAC-seq chromatin accessibility data rather than transcriptomic/qPCR
  marker expression, where marker-gene expression counts per branch are not directly
  available
- You need a mapping of new cells onto a reference trajectory rather than a benchmark
  of branch-marker correspondence
tools:
- name: STREAM
  role: Primary trajectory inference method whose inferred branches are assigned to
    Gata1/Pax5 marker lineages; written on the anndata class and installed via Bioconda
  repo: https://github.com/pinellolab/STREAM
- name: Monocle2
  role: Comparison trajectory inference method; produces over-branched assignments
    (high precision, poor recall) in the benchmark
- name: ElPiGraph
  role: Elastic principal graph algorithm underlying STREAM's branch learning and
    a comparison method in the benchmark
  repo: sysbio-curie/ElPiGraph.R
- name: anndata
  role: Data structure in which STREAM is written; holds the ~270-cell qPCR dataset
    and inferred branch labels
- name: conda
  role: Environment manager used to install STREAM (stream=1.0, python=3.7) from Bioconda
    before running the benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:6007de50fccd4d162d4d3a63adb664e633471a58920fbf5f1a32206befc58252
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# marker-gene-branch-assignment

## Summary

Assigns inferred trajectory branches to marker genes (e.g. Gata1, Pax5) by selecting, per method and marker, the two branches containing the largest number of cells expressing that marker, then scores the assignment with per-branch precision, recall and F1 against sorted-population ground truth. It underpins the STREAM benchmark in which STREAM attains the highest F1 for both erythroid and lymphoid markers.

## When to use

Use when you have an inferred branching trajectory over ~270 single-cell qPCR hematopoiesis cells (~170 transcription factors, six sorted populations) and need to quantitatively evaluate which branches correspond to marker-expressing lineages.

## When NOT to use

- The input data has no sorted population labels or equivalent ground truth, so precision/recall/F1 cannot be computed against a reference
- The dataset is scATAC-seq chromatin accessibility data rather than transcriptomic/qPCR marker expression, where marker-gene expression counts per branch are not directly available
- You need a mapping of new cells onto a reference trajectory rather than a benchmark of branch-marker correspondence

## Inputs

- Branched trajectories inferred per method (STREAM, Monocle2, ElPiGraph) over the cells
- Single-cell qPCR expression matrix of ~270 hematopoiesis cells x ~170 transcription factors (anndata object)
- Ground-truth sorted population labels for the six sorted populations
- Marker gene list: Gata1 (erythroid) and Pax5 (lymphoid)

## Outputs

- Per-method, per-marker branch assignments (two branches per method-marker pair)
- Per-branch precision, recall and F1 values for each method-marker combination
- F1 ranking table (f1_benchmark_results.csv)

## How to apply

Run each trajectory inference method (STREAM, Monocle2, ElPiGraph) on the dataset loaded into the anndata data structure in which STREAM is written, so that each method yields a branched trajectory over the same cells. For every method and each marker gene (Gata1 for erythroid, Pax5 for lymphoid), select the two branches that contain the largest number of cells expressing that marker — this is the branch assignment step, avoiding arbitrary manual branch labeling. Treat the six sorted population labels as ground truth and compute per-branch precision, recall and F1 for each method-marker combination. Rank methods by F1 and write the results table (e.g. to f1_benchmark_results.csv); the expected outcome is STREAM highest F1 for both markers, with SCUBA lower in recall, Monocle2 over-branching (high precision, poor recall), and Wishbone/DPT having low precision due to misplaced Gata1-expressing cells.

## Related tools

- **STREAM** (Primary trajectory inference method whose inferred branches are assigned to Gata1/Pax5 marker lineages; written on the anndata class and installed via Bioconda) — https://github.com/pinellolab/STREAM
- **Monocle2** (Comparison trajectory inference method; produces over-branched assignments (high precision, poor recall) in the benchmark)
- **ElPiGraph** (Elastic principal graph algorithm underlying STREAM's branch learning and a comparison method in the benchmark) — sysbio-curie/ElPiGraph.R
- **anndata** (Data structure in which STREAM is written; holds the ~270-cell qPCR dataset and inferred branch labels)
- **conda** (Environment manager used to install STREAM (stream=1.0, python=3.7) from Bioconda before running the benchmark)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter && conda activate env_stream && jupyter notebook
```

## Evaluation signals

- STREAM must achieve the highest F1 score for both Gata1 and Pax5 among all methods tested
- For each method-marker pair, exactly two branches are selected and those branches contain the largest number of marker-expressing cells
- Precision, recall and F1 are computed per branch per method using sorted population labels as ground truth, yielding one row per method-marker combination in f1_benchmark_results.csv
- Benchmark diagnostics: SCUBA shows lower recall than STREAM; Monocle2 shows high precision but poor recall (over-branching); Wishbone/DPT show low precision from misplaced Gata1-expressing cells

## Limitations

- Monocle2 over-branches, giving high precision but poor recall, so branch count strongly affects the F1 comparison
- Wishbone and DPT can misplace Gata1-expressing cells, driving precision down independently of branch assignment quality
- ElPiGraph supports learning complex graphs but intuitive visualizations for sophisticated topologies remain underdeveloped; further improvement is also needed to adapt STREAM to cyclic or disconnected graph topologies, which can complicate defining 'two branches' per marker

## Evidence

- [other] branch selection rule from the reproduced benchmark: [evidence span withheld — anchor status: not_found]
- [methods] STREAM data structure: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [discussion] comparison method context: "so far very few trajectory inference methods (Monocle2 and STREAM) have been applied to analyze single-cell chromatin accessibility"
- [discussion] ElPiGraph topology caveat: "Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations"
- [readme] environment setup for the benchmark: "$ conda create -n env_stream python=3.7 stream=1.0 jupyter
$ conda activate env_stream"
