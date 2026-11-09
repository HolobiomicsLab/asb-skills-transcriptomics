---
name: balanced-unbalanced-simulation-benchmark-tabulation
description: Use when comparing batch-effect correction methods on simulated scRNA-seq
  data with balanced and unbalanced batch conditions, and you need per-method kBET,
  ASW, and LISI metrics assembled into a benchmark table.
when_to_use_negative:
- Input data has only a single batch or no batch structure, so batch-correction benchmarking
  and batch-ASW/kBET metrics are meaningless
- Cell-type labels are unavailable, making cell-type ASW and the key JIVE-preserves-cell-type-effects
  comparison impossible
- The goal is rank selection or fitting scJIVE itself (permutation- or BIC-based)
  rather than benchmarking multiple correction methods head-to-head
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: JIVE (scJIVE)
  role: Primary method under benchmark; decomposes each simulated dataset into joint
    and individual structures to obtain batch-corrected joint embeddings
  repo: https://github.com/oconnell-statistics-lab/scJIVE
- name: Seurat v5
  role: Benchmark integration method run on the same simulated inputs to produce comparable
    corrected representations
- name: Harmony
  role: Benchmark correction method run with sigma=2 and lambda=0.1 on the same simulated
    inputs
- name: LIGER
  role: Benchmark correction method run on the same simulated inputs
- name: ComBat-seq
  role: Benchmark correction method run on the same simulated inputs
- name: R
  role: Analysis environment for scJIVE, the benchmark methods, and the metric/tabulation
    workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:3d9961b1f130a81b6abf2006c73ee169db5844704880229da6c089778d50f06f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oconnell-statistics-lab/scJIVE
  tools:
  - ComBat-seq
  - Harmony
  - JIVE (scJIVE)
  - LIGER
  - R
  - Seurat v5
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# balanced-unbalanced-simulation-benchmark-tabulation

## Summary

Benchmarks JIVE (scJIVE) against Seurat v5, Harmony, LIGER, and ComBat-seq on simulated scRNA-seq data under balanced versus unbalanced batch conditions, scoring each corrected output with PVCA, kBET, ASW (batch and cell type), and LISI, and tabulating results into a CSV mirroring the paper's Tables 4-6.

## When to use

Use when comparing batch-effect correction methods on simulated scRNA-seq data with balanced and unbalanced batch conditions, and you need per-method kBET, ASW, and LISI metrics assembled into a benchmark table.

## When NOT to use

- Input data has only a single batch or no batch structure, so batch-correction benchmarking and batch-ASW/kBET metrics are meaningless
- Cell-type labels are unavailable, making cell-type ASW and the key JIVE-preserves-cell-type-effects comparison impossible
- The goal is rank selection or fitting scJIVE itself (permutation- or BIC-based) rather than benchmarking multiple correction methods head-to-head

## Inputs

- Simulated scRNA-seq count matrices for the balanced-batch condition (chained from the simulation task)
- Simulated scRNA-seq count matrices for the unbalanced-batch condition
- Batch labels and cell-type labels for each simulated cell
- Method parameter settings (scJIVE joint rank 10 / individual ranks 20; Harmony sigma=2, lambda=0.1)

## Outputs

- One CSV table of benchmark metrics organized by method and batch-balance condition, mirroring the paper's Tables 4-6 of Section 3.2
- Per-method-and-condition values of scaled kBET acceptance rate, ASW on batch, ASW on cell type, and LISI
- Corrected joint embeddings / representations from scJIVE, Seurat v5, Harmony, LIGER, and ComBat-seq

## How to apply

Load the simulated count matrices and batch/cell-type labels for both the balanced-batch and unbalanced-batch conditions, then run scJIVE on each dataset with a fixed joint rank of 10 and individual ranks of 20 to obtain batch-corrected joint embeddings. Run the four benchmark methods on the same simulated inputs — Seurat v5 integration, Harmony with sigma=2 and lambda=0.1, LIGER, and ComBat-seq — so every method produces comparable corrected representations. Quantify each method's output with PVCA and the kBET, silhouette-width (ASW, computed separately on batch and on cell type), and LISI metrics, scaling kBET as an acceptance rate per method and condition. Assemble the scaled kBET acceptance rates, ASW batch/cell-type, and LISI values into one CSV organized by method and batch-balance condition, mirroring the paper's Tables 4-6 of Section 3.2. The rationale is the paper's finding that JIVE performed best at preserving cell-type effects and in scenarios with balanced batch sizes, so the benchmark must separate balanced from unbalanced conditions to expose this differential performance.

## Related tools

- **JIVE (scJIVE)** (Primary method under benchmark; decomposes each simulated dataset into joint and individual structures to obtain batch-corrected joint embeddings) — https://github.com/oconnell-statistics-lab/scJIVE
- **Seurat v5** (Benchmark integration method run on the same simulated inputs to produce comparable corrected representations)
- **Harmony** (Benchmark correction method run with sigma=2 and lambda=0.1 on the same simulated inputs)
- **LIGER** (Benchmark correction method run on the same simulated inputs)
- **ComBat-seq** (Benchmark correction method run on the same simulated inputs)
- **R** (Analysis environment for scJIVE, the benchmark methods, and the metric/tabulation workflow)

## Examples

```
devtools::install_github("oconnell-statistics-lab/scJIVE")
```

## Evaluation signals

- Output CSV exists and contains kBET, ASW, and LISI columns for every method-by-condition row
- Both batch-balance conditions (balanced and unbalanced) are represented for each of the five methods
- kBET values are reported as scaled acceptance rates per method and condition (in [0,1])
- Qualitative ordering matches the reported benchmark: JIVE achieves the best cell-type ASW under balanced batches
- All five methods were run on the same simulated count matrices and labels, yielding comparable corrected representations

## Limitations

- JIVE's original implementation was designed for bulk sequencing data and is computationally infeasible for large-scale single-cell datasets; the enhanced scJIVE implementation is required
- Performance is batch-balance dependent: JIVE performed best at preserving cell-type effects and in scenarios with balanced batch sizes, so results may differ under unbalanced conditions
- Benchmark uses fixed ranks (joint rank 10, individual ranks 20) rather than data-estimated ranks, which may not match the true simulated structure

## Evidence

- [abstract] We benchmarked the results against four popular tools, Seurat v5, Harmony, LIGER, and Combat-seq, which were developed for this purpose.: "We benchmarked the results against four popular tools, Seurat v5, Harmony, LIGER, and Combat-seq, which were developed for this purpose."
- [abstract] JIVE performed best in terms of preserving cell-type effects and in scenarios in which the batch sizes are balanced.: "JIVE performed best in terms of preserving cell-type effects and in scenarios in which the batch sizes are balanced."
- [methods] The default function to estimate the JIVE decomposition is `jive`: "The default function to estimate the JIVE decomposition is `jive`"
- [readme] scJIVE aims to decompose scRNA-seq datasets into a joint structure capturing the true biological variability and individual structures which capture technical variability within each batch.: "scJIVE aims to decompose scRNA-seq datasets into a joint structure capturing the true biological variability and individual structures which capture technical variability within each batch."
