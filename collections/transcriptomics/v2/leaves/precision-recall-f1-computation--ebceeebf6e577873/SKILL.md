---
name: precision-recall-f1-computation
description: Use when comparing trajectory inference methods (e.g. STREAM, Monocle2,
  ElPiGraph) on a branching single-cell dataset where sorted population labels serve
  as ground truth, to rank methods by per-marker precision/recall/F1 on lineage-specific
  branches.
when_to_use_negative:
- When evaluating cyclic or disconnected graph topologies — the article notes further
  improvement is necessary to adapt STREAM to these more general structures, so branch-based
  F1 scoring against a tree/bifurcation ground truth is not valid
- When no ground-truth labels (e.g. sorted populations) are available — precision/recall
  against known classes cannot be computed, only qualitative branch agreement
- When mapping new cells onto a reference trajectory with totally different fates
  — the mapping procedure may not be optimal, so downstream branch assignment metrics
  would be misleading
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: STREAM
  role: Trajectory inference method being benchmarked; expected to attain the highest
    F1 for both Gata1 and Pax5
  repo: https://github.com/pinellolab/STREAM
- name: Monocle2
  role: Comparison trajectory inference method; expected to over-branch, showing high
    precision but poor recall
- name: ElPiGraph
  role: Core algorithm behind STREAM and an additional comparison method in the F1
    benchmark
  repo: https://github.com/Albluca/ElPiGraph.R
- name: anndata
  role: Data structure in which STREAM is written; used to load the qPCR dataset for
    benchmarking
  repo: http://anndata.rtfd.io
- name: conda
  role: Environment manager for installing stream from Bioconda with Python 3.7
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:a2f1ce93d44095e858dd4430aef525deec3c2fd280bc3b09fb7848fb9f9759d2
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
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# precision-recall-f1-computation

## Summary

Benchmark protocol for ranking trajectory inference methods by per-branch precision, recall, and F1 against sorted-population ground truth, as used to show STREAM attains the highest F1 for Gata1 and Pax5 lineage separation on the single-cell qPCR hematopoiesis dataset.

## When to use

Use when comparing trajectory inference methods (e.g. STREAM, Monocle2, ElPiGraph) on a branching single-cell dataset where sorted population labels serve as ground truth, to rank methods by per-marker precision/recall/F1 on lineage-specific branches.

## When NOT to use

- When evaluating cyclic or disconnected graph topologies — the article notes further improvement is necessary to adapt STREAM to these more general structures, so branch-based F1 scoring against a tree/bifurcation ground truth is not valid
- When no ground-truth labels (e.g. sorted populations) are available — precision/recall against known classes cannot be computed, only qualitative branch agreement
- When mapping new cells onto a reference trajectory with totally different fates — the mapping procedure may not be optimal, so downstream branch assignment metrics would be misleading

## Inputs

- Single-cell qPCR hematopoiesis matrix of ~270 cells profiled for ~170 transcription factors across six sorted populations (anndata object)
- Sorted population labels used as ground truth for per-branch precision/recall
- Lineage marker genes to evaluate (Gata1, Pax5)
- Inferred branch assignments from each trajectory inference method (STREAM, Monocle2, ElPiGraph)

## Outputs

- f1_benchmark_results.csv with per-method, per-marker branch precision, recall, and F1
- Ranking of trajectory inference methods by F1 for each marker

## How to apply

Load the ~270-cell single-cell qPCR hematopoiesis dataset (~170 transcription-factor features, six sorted populations) into the `anndata` structure in which STREAM is written, after creating the Bioconda environment (`conda create -n env_stream python=3.7 stream=1.0 jupyter`). Run each trajectory inference method (STREAM, Monocle2, ElPiGraph) to infer branching trajectories, then for each method and each lineage marker (Gata1 for erythroid, Pax5 for lymphoid) select the two branches containing the largest number of cells expressing that marker. Treat the sorted population labels as ground truth and compute per-branch precision, recall, and F1 for every method-marker combination. Rank methods by F1 and write the results to f1_benchmark_results.csv; a correct run reproduces STREAM achieving the highest F1 for both markers, SCUBA lower in recall, Monocle2 over-branching (high precision, poor recall), and Wishbone/DPT with low precision from misplaced Gata1-expressing cells.

## Related tools

- **STREAM** (Trajectory inference method being benchmarked; expected to attain the highest F1 for both Gata1 and Pax5) — https://github.com/pinellolab/STREAM
- **Monocle2** (Comparison trajectory inference method; expected to over-branch, showing high precision but poor recall)
- **ElPiGraph** (Core algorithm behind STREAM and an additional comparison method in the F1 benchmark) — https://github.com/Albluca/ElPiGraph.R
- **anndata** (Data structure in which STREAM is written; used to load the qPCR dataset for benchmarking) — http://anndata.rtfd.io
- **conda** (Environment manager for installing stream from Bioconda with Python 3.7)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter; conda activate env_stream; jupyter notebook
```

## Evaluation signals

- f1_benchmark_results.csv contains precision, recall, and F1 for every method-marker combination (each method × {Gata1, Pax5})
- STREAM attains the highest F1 score for both the erythroid marker Gata1 and the lymphoid marker Pax5 while balancing precision and recall
- Monocle2 exhibits high precision with poor recall (over-branching), and Wishbone/DPT exhibit low precision due to misplaced Gata1-expressing cells
- SCUBA shows lower recall than STREAM, matching the expected recall deficit pattern
- Branch selection step, per marker, picks exactly the two branches with the largest number of cells expressing that marker

## Limitations

- The mapping procedure may not be optimal if new cells have totally different fates compared to those in the reference structure, which would invalidate branch-level evaluation for such cells
- Further improvement is necessary to adapt STREAM to cyclic or disconnected graph topologies, restricting the benchmark to tree/bifurcation trajectories
- Many current trajectory inference methods, including STREAM, do not scale to datasets profiling millions of cells, limiting benchmark scope to small datasets like the ~270-cell qPCR data

## Evidence

- [discussion] STREAM has been extensively tested using several published datasets from different organisms (zebrafish, mouse, human) and single-cell technologies (qPCR, scRNA-seq, scATAC-seq): "STREAM has been extensively tested using several published datasets from different organisms (zebrafish, mouse, human) and single-cell technologies (qPCR, scRNA-seq, scATAC-seq)"
- [methods] STREAM is written using the class `anndata`: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [discussion] the mapping procedure may not be optimal if the new cells have totally different fates: "the mapping procedure may not be optimal if the new cells have totally different fates compared to the ones present in the reference structure"
- [readme] conda create -n env_stream python=3.7 stream=1.0 jupyter: "$ conda create -n env_stream python=3.7 stream=1.0 jupyter"
