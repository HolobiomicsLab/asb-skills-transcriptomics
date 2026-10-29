---
name: minimum-weight-matching-batch-correction
description: Use when integrating exactly two scRNA-seq datasets (e.g., 10X PBMC 8k
  vs Pan T cells, or Splatter-simulated pairs) in gene or PCA space, prioritizing
  both batch mixing (iLISI) and local structure retention (RNN) over batch-effect
  removal alone.
when_to_use_negative:
- You need to integrate more than two datasets — the current BATMAN implementation
  works only with two datasets (multi-dataset integration is possible only 'in principle')
- Input is raw counts that have not gone through Seurat V3.0 pre-processing to log-counts,
  or datasets lacking a comparable feature/PCA space across batches
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BATMAN
  role: 'Core method: builds the weighted bipartite graph over anchors, finds the
    minimum-weight matching (exact blossom-based or greedy ratio-2), derives local
    shifting vectors, and applies batch correction'
  repo: https://github.com/mandricigor/batman
- name: Seurat
  role: Provides the pre-processing steps used to obtain log-counts for each dataset,
    and serves as a comparison integration method (Seurat V3.0)
- name: Splatter
  role: Simulates the scRNA-seq dataset pairs (1,000 genes; 100 datasets per scenario)
    used to benchmark the matching-based correction
- name: Scanorama
  role: Comparison integrator run on the same preprocessed data to benchmark iLISI/50-RNN
    and runtime
- name: MNNcorrect
  role: Comparison MNN-based integrator run on the same log-count matrices
- name: Harmony
  role: PCA-space comparison method when the correction is performed in a shared PCA
    embedding of the two 10X PBMC datasets
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:05c412f474423b73cc6ff748db159d90257e88d8d11aebebc6e21d303b6c90d1
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
  - Seurat
  - Splatter
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# minimum-weight-matching-batch-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

BATMAN-style batch correction for two single-cell RNA-seq datasets: after Seurat V3.0 preprocessing to log-counts, a minimum-weight matching of anchor cells between datasets is used to derive local shifting vectors and apply the minimal total correction needed to align the query to the reference while preserving local structure.

## When to use

Use when integrating exactly two scRNA-seq datasets (e.g., 10X PBMC 8k vs Pan T cells, or Splatter-simulated pairs) in gene or PCA space, prioritizing both batch mixing (iLISI) and local structure retention (RNN) over batch-effect removal alone.

## When NOT to use

- You need to integrate more than two datasets — the current BATMAN implementation works only with two datasets (multi-dataset integration is possible only 'in principle')
- Input is raw counts that have not gone through Seurat V3.0 pre-processing to log-counts, or datasets lacking a comparable feature/PCA space across batches

## Inputs

- Two count matrices (e.g., 10X Genomics PBMC 8k with 8,381 cells and Pan T cells with 3,555 T cells) or Splatter-simulated dataset pairs (1,000 genes per simulated dataset)
- Seurat V3.0 preprocessed log-counts for each dataset, or shared PCA embeddings of both datasets

## Outputs

- Batch-corrected expression matrix (query dataset aligned to the reference)
- Integration quality metrics: mean iLISI (batch mixing) and mean 50-RNN / 10-RNN (local structure retention), plus wall-clock runtime per method

## How to apply

First apply Seurat V3.0 pre-processing steps to obtain log-counts for each dataset, or project both datasets into a shared PCA space when comparing against PCA-space tools like Harmony. Identify a set of anchor cells in each dataset (number controlled by the -anchors parameter, with a Pearson-correlation filtering threshold -filter-cor on anchors across datasets), then build a weighted bipartite graph whose partitions correspond to the two datasets with edges weighted by the Euclidean distance between cells in N-dimensional space. Find the minimum-weight matching using either the exact blossom-based algorithm or the greedy approximation with ratio 2 (recommended via the -greedy flag for extra large datasets), determine local shifting vectors from the matched pairs, and apply batch correction to align the query dataset to the reference. Evaluate the result with mean iLISI for batch mixing and mean 50-RNN (or 10-RNN in PCA space) for local structure preservation, benchmarking against Seurat V3.0, MNNcorrect, and Scanorama, and keeping the uncorrected concatenation as the Original baseline.

## Related tools

- **BATMAN** (Core method: builds the weighted bipartite graph over anchors, finds the minimum-weight matching (exact blossom-based or greedy ratio-2), derives local shifting vectors, and applies batch correction) — https://github.com/mandricigor/batman
- **Seurat** (Provides the pre-processing steps used to obtain log-counts for each dataset, and serves as a comparison integration method (Seurat V3.0))
- **Splatter** (Simulates the scRNA-seq dataset pairs (1,000 genes; 100 datasets per scenario) used to benchmark the matching-based correction)
- **Scanorama** (Comparison integrator run on the same preprocessed data to benchmark iLISI/50-RNN and runtime)
- **MNNcorrect** (Comparison MNN-based integrator run on the same log-count matrices)
- **Harmony** (PCA-space comparison method when the correction is performed in a shared PCA embedding of the two 10X PBMC datasets)

## Examples

```
python3.7 batman.py -data pbmc8k_logcounts.csv,pantcells_logcounts.csv -output batman_corrected.csv -anchors 200 -filter-cor 0.7
```

## Evaluation signals

- LB simulation benchmark reproduces mean iLISI of 1.82/1.84/1.00/1.00/1.01 and mean 50-RNN of 1.00/0.92/0.58/0.51/0.14 for Original/BATMAN/Seurat V3.0/MNNcorrect/Scanorama, with BATMAN preserving the highest 50-RNN among corrected methods
- On 10X PBMC 8k vs Pan T cells in gene space, BATMAN achieves the highest iLISI of the four gene-space methods with mean iLISI ~1.84 and mean 50-RNN ~0.92, versus ≤1.01 iLISI for the other methods
- Runtime ordering on the PBMC benchmark holds: BATMAN 20.1 s < Scanorama 31.0 s < Seurat V3.0 211.2 s < MNNcorrect 1,099.5 s
- Metrics CSV contains exactly one row per method (Original, BATMAN, Seurat V3.0, MNNcorrect, Scanorama) including a BATMAN row
- In PCA space, runtimes for BATMAN and Harmony are comparable (~14 s each) and Harmony scores slightly better iLISI and 10-RNN than BATMAN

## Limitations

- The current BATMAN implementation works only with two datasets; multi-dataset integration is not implemented (only possible 'in principle')
- The greedy minimum-weight matching approximation has an approximation ratio of 2, so it is a heuristic trade-off recommended for extra large datasets rather than the exact solution
- Anchor quality depends on the -anchors count and the -filter-cor Pearson correlation filtering threshold between anchors across datasets

## Evidence

- [readme] Building a weighted bipartite graph where each partition corresponds to each of the datasets and the edges are weighted by the Euclidean distance between the points: [evidence span withheld — anchor status: external_doc]
