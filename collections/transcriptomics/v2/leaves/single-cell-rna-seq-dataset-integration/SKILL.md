---
name: single-cell-rna-seq-dataset-integration
description: Use when two raw scRNA-seq count matrices (e.g. 10X PBMC 8k and Pan T
  cells) need batch integration in gene space, with quality judged by mean iLISI and
  mean 50-RNN metrics.
when_to_use_negative:
- More than two datasets need integrating — the current BATMAN implementation works
  only with two datasets (multi-dataset integration is only possible 'in principle')
- Extra-large datasets where exact blossom-based matching is too slow is fine (use
  -greedy), but the skill does not cover dimensionality-reduction-based integration
  (e.g. ZIFA) or simulation-based workflows (Splatter)
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BATMAN
  role: 'Primary integration method: minimum-weight matching of anchors, local shifting
    vectors, and batch correction in high-dimensional gene space'
  repo: https://github.com/mandricigor/batman
- name: Seurat
  role: V3.0 pre-processing to obtain log-counts for each dataset, and comparison
    integrator in the benchmark
- name: MNNcorrect
  role: Comparison integrator (mutual nearest neighbor paradigm) run on the same log-count
    matrices
- name: Scanorama
  role: Comparison integrator run on the same log-count matrices
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:1dcea13851e93724af8e73e6c379dfe93c837a943225026979ae8bec91067fd2
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
  - MNNcorrect
  - Scanorama
  - Seurat
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# single-cell-rna-seq-dataset-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Batch-correct and integrate two single-cell RNA-seq datasets using BATMAN, which builds a bipartite graph over representative anchor cells, finds a minimum-weight one-to-one matching (exact blossom-based or greedy ratio-2 approximation), and applies local shifting vectors in high-dimensional gene space. Rationale: it removes batch effects while preserving local cell-to-cell structure, outperforming MNN-based methods.

## When to use

Use when two raw scRNA-seq count matrices (e.g. 10X PBMC 8k and Pan T cells) need batch integration in gene space, with quality judged by mean iLISI and mean 50-RNN metrics.

## When NOT to use

- More than two datasets need integrating — the current BATMAN implementation works only with two datasets (multi-dataset integration is only possible 'in principle')
- Extra-large datasets where exact blossom-based matching is too slow is fine (use -greedy), but the skill does not cover dimensionality-reduction-based integration (e.g. ZIFA) or simulation-based workflows (Splatter)

## Inputs

- Raw scRNA-seq count matrices for two datasets (e.g. 10X Genomics PBMC 8k, 8,381 cells; Pan T cells, 3,555 cells)
- Log-count matrices produced via Seurat V3.0 pre-processing (comma-separated file paths accepted by -data)
- Number of anchor points per dataset and optional Pearson correlation filter threshold

## Outputs

- Batch-corrected integrated dataset written to the path given by -output
- Benchmark CSV with one row per method (Original, BATMAN, Seurat V3.0, MNNcorrect, Scanorama) reporting mean iLISI, mean 50-RNN, and wall-clock runtime

## How to apply

Pre-process each dataset with Seurat V3.0 steps to obtain log-counts, then run BATMAN: it identifies a user-specified number of anchor points (-anchors) per dataset, builds a weighted bipartite graph with partitions for each dataset and edges weighted by Euclidean distance, finds the minimum-weight matching (blossom-based exact algorithm by default, or greedy approximation with ratio 2 via -greedy for extra-large datasets), determines local shifting vectors, and performs batch correction. Optionally filter anchors with a Pearson correlation threshold (-filter-cor) between anchors across datasets. Benchmark against Seurat V3.0, MNNcorrect, and Scanorama on the same log-count matrices, computing mean iLISI and mean 50-RNN. Success is indicated by high iLISI (mixing) together with high 50-RNN (structure preservation) and low runtime.

## Related tools

- **BATMAN** (Primary integration method: minimum-weight matching of anchors, local shifting vectors, and batch correction in high-dimensional gene space) — https://github.com/mandricigor/batman
- **Seurat** (V3.0 pre-processing to obtain log-counts for each dataset, and comparison integrator in the benchmark)
- **MNNcorrect** (Comparison integrator (mutual nearest neighbor paradigm) run on the same log-count matrices)
- **Scanorama** (Comparison integrator run on the same log-count matrices)

## Examples

```
python3.7 batman.py -data pbmc8k_logcounts.csv,panT_logcounts.csv -output corrected.csv -anchors 100 -greedy
```

## Evaluation signals

- LB-scenario benchmark reproduces mean iLISI 1.84 and mean 50-RNN 0.92 for BATMAN, versus ≤1.01 iLISI (1.00 Seurat V3.0, 1.00 MNNcorrect, 1.01 Scanorama) and ≤0.58 50-RNN for the other methods
- Runtime ordering holds: BATMAN 20.1 s < Scanorama 31.0 s < Seurat V3.0 211.2 s < MNNcorrect 1,099.5 s
- Output CSV contains exactly 5 rows (Original, BATMAN, Seurat V3.0, MNNcorrect, Scanorama) including a BATMAN row
- Integrated result shows high iLISI (mixing) without collapse of 50-RNN, confirming local cell-to-cell relationships are retained
- Original-data baseline metrics check out (mean iLISI 1.82, mean 50-RNN 1.00) before correction

## Limitations

- The current BATMAN implementation supports integration of only two datasets; multi-dataset integration is not implemented
- The greedy matching is an approximation with ratio 2 (recommended for extra large datasets); the exact blossom-based algorithm may be slower
- Anchor quality depends on the number of anchors (-anchors) and the Pearson correlation filter threshold (-filter-cor); MNN-based integration in general can fail to both correct batch effects and maintain local dataset structure

## Evidence

- [readme] Building a weighted bipartite graph where each partition corresponds to each of the datasets and the edges are weighted by the Euclidean distance between the points: [evidence span withheld — anchor status: external_doc]
