---
name: cross-tool-runtime-benchmarking
description: Use when comparing batch-integration methods on the same datasets and
  you must report both integration quality (iLISI/cLISI/RNN) and wall-clock runtime
  per tool in a single comparison CSV.
when_to_use_negative:
- You need to integrate more than two datasets — BATMAN's current implementation only
  supports two datasets, so multi-dataset benchmarks require different tooling
- You only need a single tool's integration result without a runtime or cross-method
  accuracy comparison
- The datasets are extremely large and exact blossom matching is required — use the
  -greedy option instead of benchmarking the exact algorithm
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BATMAN
  role: Fastest integrator under benchmark; minimum-weight matching batch correction
    whose runtime (9–20 s) is recorded against competitors
  repo: https://github.com/mandricigor/batman
- name: Seurat V3.0
  role: MNN-based comparison integrator run on the same log-counts; runtime (22.3–211.2
    s) and iLISI/50-RNN recorded
- name: MNNcorrect
  role: MNN-based comparison integrator; slowest tool in both benchmarks (228.6 s
    pancreas, 1,099.5 s 10X)
- name: Scanorama
  role: MNN-based comparison integrator; runtime (14–31 s) and metrics recorded on
    identical inputs
- name: Harmony
  role: PCA-space integration baseline compared against BATMAN on the 10X PBMC datasets
    (~14 s per tool)
provenance:
  source_task_ids:
  - task_006
  - task_003
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:de3ba9ecdd4c7bf312eba6ad108ec37d2921e81d54963425bf8df5407950fb86
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mandricigor/batman
  repo_url: https://github.com/mandricigor/batman
  tools:
  - BATMAN
  - Harmony
  - MNNcorrect
  - Scanorama
  - Seurat V3.0
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# cross-tool-runtime-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks multiple scRNA-seq integration tools (BATMAN, Seurat V3.0, MNNcorrect, Scanorama, Harmony) on identical inputs, recording mean integration metrics (iLISI, cLISI, RNN) alongside wall-clock runtimes so accuracy and speed can be compared in one table.

## When to use

Use when comparing batch-integration methods on the same datasets and you must report both integration quality (iLISI/cLISI/RNN) and wall-clock runtime per tool in a single comparison CSV.

## When NOT to use

- You need to integrate more than two datasets — BATMAN's current implementation only supports two datasets, so multi-dataset benchmarks require different tooling
- You only need a single tool's integration result without a runtime or cross-method accuracy comparison
- The datasets are extremely large and exact blossom matching is required — use the -greedy option instead of benchmarking the exact algorithm

## Inputs

- Raw count matrices for two single-cell datasets (e.g. Muraro CEL-Seq2 2,285 cells; Segerstolpe Smart-Seq2 2,394 cells; 10X PBMC 8k 8,381 cells; 10X Pan T cells 3,555 cells)
- Seurat V3.0-style log-count matrices shared across all tools
- Optionally, shared PCA embeddings for PCA-space comparisons (BATMAN vs Harmony)

## Outputs

- CSV comparison table with one row per method reporting mean iLISI, mean cLISI or k-RNN, and wall-clock runtime (e.g. integration_benchmark.csv, batman_harmony_pbmc_comparison.csv)
- Per-tool integrated data representations for metric computation

## How to apply

Apply Seurat V3.0 pre-processing to obtain log-counts for each dataset, then run every integrator on the identical log-count matrices (or shared PCA embeddings, as in the BATMAN-vs-Harmony 10X PBMC comparison). For BATMAN, build the bipartite graph with edges weighted by Euclidean distance, solve minimum-weight matching (exact blossom-based or greedy with ratio 2, the latter recommended for extra large datasets), derive local shifting vectors, and apply batch correction. Compute mean iLISI plus cLISI or k-RNN (50-RNN in gene space, 10-RNN in PCA space) on each integrated representation and record the wall-clock runtime of each tool. Write one CSV row per method (Original, BATMAN, Seurat V3.0, MNNcorrect, Scanorama, optionally Harmony) with the metrics and runtime, e.g. pancreas all-cell-types: BATMAN iLISI 1.55/9 s, Seurat V3.0 1.35/22.3 s, MNNcorrect 1.18/228.6 s, Scanorama 1.22/14 s; 10X LB scenario: BATMAN 20.1 s < Scanorama 31.0 s < Seurat V3.0 211.2 s < MNNcorrect 1,099.5 s.

## Related tools

- **BATMAN** (Fastest integrator under benchmark; minimum-weight matching batch correction whose runtime (9–20 s) is recorded against competitors) — https://github.com/mandricigor/batman
- **Seurat V3.0** (MNN-based comparison integrator run on the same log-counts; runtime (22.3–211.2 s) and iLISI/50-RNN recorded)
- **MNNcorrect** (MNN-based comparison integrator; slowest tool in both benchmarks (228.6 s pancreas, 1,099.5 s 10X))
- **Scanorama** (MNN-based comparison integrator; runtime (14–31 s) and metrics recorded on identical inputs)
- **Harmony** (PCA-space integration baseline compared against BATMAN on the 10X PBMC datasets (~14 s per tool))

## Examples

```
time python3.7 batman.py -data pbmc8k_logcounts.txt,panT_logcounts.txt -output corrected.csv -greedy
```

## Evaluation signals

- The comparison CSV contains exactly one row per method plus an Original row (5 rows for the 10X gene-space benchmark) with a BATMAN row present
- Runtime ordering reproduces the reported values (10X LB: BATMAN 20.1 s < Scanorama 31.0 s < Seurat V3.0 211.2 s < MNNcorrect 1,099.5 s; pancreas: BATMAN 9 s < Scanorama 14 s < Seurat V3.0 22.3 s < MNNcorrect 228.6 s)
- Metric values match reported outcomes: BATMAN mean iLISI 1.84 and mean 50-RNN 0.92 in the LB scenario; BATMAN iLISI 1.55 best on the pancreas all-cell-types benchmark
- PCA-space 10X comparison shows Harmony slightly better than BATMAN on both iLISI and 10-RNN with both tools completing in ~14 s
- All tools were run on the identical log-count (or PCA embedding) inputs, so metric differences are attributable to the integrators, not preprocessing

## Limitations

- BATMAN's current implementation works only with two datasets; multi-dataset integration is not implemented
- Runtimes are wall-clock measurements on specific datasets (thousands of cells), so absolute times will not transfer to larger or different data
- The greedy matching option trades exactness for speed (approximation ratio 2) and is recommended only for extra large datasets

## Evidence

- [results] In the LB scenario, mean iLISI was 1.84 for BATMAN, 1.00 for Seurat V3.0, 1.00 for MNNcorrect, and 1.01 for Scanorama, versus 1.82 for the original data: [evidence span withheld — anchor status: not_found]
