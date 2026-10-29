---
name: jive-joint-individual-variation-decomposition-scrna
description: Use when multi-source or multi-batch scRNA-seq count matrices need batch-effect
  correction with preservation of cell-type signal, or when benchmarking JIVE against
  Seurat v5, Harmony, LIGER, and ComBat-seq under balanced versus unbalanced batch
  conditions.
when_to_use_negative:
- Data is a single batch/source with no multi-source structure to decompose — the
  joint/individual decomposition has no batch effect to correct
- The analysis targets bulk multi-omics integration where original r.jive is appropriate;
  the article notes original JIVE 'is computationally infeasible for large-scale single-cell
  sequencing datasets' without the scJIVE enhancements
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: JIVE (scJIVE)
  role: 'Primary method: joint/individual variation decomposition for scRNA-seq batch
    correction; installed from GitHub'
  repo: https://github.com/oconnell-statistics-lab/scJIVE
- name: Seurat v5
  role: Benchmark integration method producing a comparable corrected representation
- name: Harmony
  role: Benchmark method run with sigma=2, lambda=0.1
- name: LIGER
  role: Benchmark integration method on the same simulated inputs
- name: ComBat-seq
  role: Benchmark batch-correction method on the same simulated inputs
- name: r.jive
  role: Original JIVE R package providing the jive, jive.predict, bic.jive, and jive.perm
    functions underlying the decomposition workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:e2f70cc58d8a355d1e32b20c99c8e389015b789dfa370744d79cae18ed2a614e
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
  - r.jive
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# jive-joint-individual-variation-decomposition-scrna

## Summary

Apply scJIVE (enhanced JIVE for single-cell data) to decompose multi-source scRNA-seq datasets into a joint structure capturing shared biological variability and individual structures capturing batch-level technical variability, and benchmark the corrected embeddings against Seurat v5, Harmony, LIGER, and ComBat-seq using PVCA, kBET, ASW, and LISI metrics.

## When to use

Use when multi-source or multi-batch scRNA-seq count matrices need batch-effect correction with preservation of cell-type signal, or when benchmarking JIVE against Seurat v5, Harmony, LIGER, and ComBat-seq under balanced versus unbalanced batch conditions.

## When NOT to use

- Data is a single batch/source with no multi-source structure to decompose — the joint/individual decomposition has no batch effect to correct
- The analysis targets bulk multi-omics integration where original r.jive is appropriate; the article notes original JIVE 'is computationally infeasible for large-scale single-cell sequencing datasets' without the scJIVE enhancements

## Inputs

- Simulated or real scRNA-seq count matrices split by batch (balanced and unbalanced batch conditions, chained from the data-generation task)
- Per-cell batch labels and cell-type labels
- Optional multi-source matrices in the r.jive format (e.g., BRCA_data with gene expression, methylation, and miRNA blocks)

## Outputs

- Batch-corrected joint embeddings from scJIVE and each benchmark method
- Estimated joint and individual rank structure (e.g., final joint rank: 2, individual ranks: 27/26/25 on BRCA data)
- CSV table of scaled kBET acceptance rates, ASW (batch and cell type), and LISI values organized by method and batch-balance condition, mirroring Tables 4-6

## How to apply

Chain the simulated (or real) count matrices and batch/cell-type labels into the scJIVE pipeline, running scJIVE with a fixed joint rank of 10 and individual ranks of 20 to obtain batch-corrected joint embeddings; select ranks via the permutation-based (jive.perm) or BIC-motivated (bic.jive) approach when data-driven estimation is preferred. Run the four benchmarks on identical inputs — Seurat v5 integration, Harmony (sigma=2, lambda=0.1), LIGER, and ComBat-seq — to produce comparable corrected representations. Quantify each output with PVCA, kBET (scaled as an acceptance rate per method and condition), silhouette width (ASW computed on batch and on cell type), and LISI, then assemble the metrics into one CSV organized by method and batch-balance condition mirroring Tables 4-6 of Section 3.2. The rationale is that scJIVE's decomposition assigns true biological variability to the joint structure and technical variability to per-batch individual structures, which the article shows preserves cell-type effects best when batch sizes are balanced.

## Related tools

- **JIVE (scJIVE)** (Primary method: joint/individual variation decomposition for scRNA-seq batch correction; installed from GitHub) — https://github.com/oconnell-statistics-lab/scJIVE
- **Seurat v5** (Benchmark integration method producing a comparable corrected representation)
- **Harmony** (Benchmark method run with sigma=2, lambda=0.1)
- **LIGER** (Benchmark integration method on the same simulated inputs)
- **ComBat-seq** (Benchmark batch-correction method on the same simulated inputs)
- **r.jive** (Original JIVE R package providing the jive, jive.predict, bic.jive, and jive.perm functions underlying the decomposition workflow)

## Examples

```
devtools::install_github("oconnell-statistics-lab/scJIVE"); library(scJIVE); result <- jive(data, rank.j = 10, rank.a = c(20, 20))
```

## Evaluation signals

- The CSV output contains a kBET, ASW, and LISI column for every method-by-condition row
- JIVE achieves the best cell-type ASW among all methods under the balanced-batch condition, matching the paper's reported ordering
- PVCA and scaled kBET acceptance rates reproduce the relative method ordering reported in Tables 4-6 of Section 3.2
- The joint and individual estimates satisfy row-orthogonality between joint and individual structures and between each individual estimate
- Data-driven rank estimates (permutation-based via jive.perm or BIC-based via bic.jive) are consistent with the fixed ranks (joint=10, individual=20) used in the benchmark

## Limitations

- JIVE's original implementation was designed for bulk sequencing data and is computationally infeasible for large-scale single-cell datasets without the scJIVE efficiency enhancements
- JIVE performed best in scenarios with balanced batch sizes; unbalanced batch conditions are a relative weakness compared with the balanced case

## Evidence

- [abstract] We benchmarked the results against four popular tools, Seurat v5, Harmony, LIGER, and Combat-seq: "We benchmarked the results against four popular tools, Seurat v5, Harmony, LIGER, and Combat-seq, which were developed for this purpose."
- [abstract] JIVE preserves cell-type effects: "JIVE performed best in terms of preserving cell-type effects and in scenarios in which the batch sizes are balanced."
- [readme] scJIVE decomposes scRNA-seq data: "scJIVE aims to decompose scRNA-seq datasets into a joint structure capturing the true biological variability and individual structures which capture technical variability within each batch."
- [methods] Rank selection approaches: "One is a permutation-based approach described in @Lock, the other is a BIC-motivated approach."
- [methods] BRCA estimated ranks: "Final joint rank: 2 , final individual ranks: 27 26 25"
