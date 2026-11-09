---
name: pathway-activity-score-computation-benchmarking
description: Use when comparing computational efficiency of PAS scoring methods (fast_ucell,
  fast_score_genes vs UCell, AUCell, score_genes) across datasets scaling from 10,000
  cells to over 1.2 million cells.
when_to_use_negative:
- The goal is to produce PAS for biological interpretation (e.g. multicellular pathway
  modules via MOFA) rather than to measure computational performance
- The dataset has few cells (<10^3) or few pathways (<10^3), where runtime differences
  between methods are negligible and benchmarking adds no decision value
- Input is not a cell x gene expression matrix with a pathway gene-set dictionary,
  so no PAS scoring comparison can be constructed
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPAFA
  role: Provides fast_ucell and fast_score_genes, the computationally efficient PAS
    implementations being benchmarked (run with 10 cores; 20 cores on the 14,300-pathway
    C5 expansion)
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: UCell
  role: Baseline PAS method run with 10 cores; reference for verifying that fast_ucell
    returns identical PAS
- name: AUCell
  role: Baseline PAS method run with 10 cores; related to fast_ucell PAS when the
    same max rank threshold is set
- name: AddModuleScore
  role: Original module-scoring method whose Scanpy equivalent 'score_genes' (run
    with 1 core) serves as the slow baseline that fast_score_genes accelerates
- name: Scanpy
  role: Supplies the score_genes implementation used as the AddModuleScore baseline
    in the benchmark
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:e3592bd03b689f0fcb329c1e92cd751b2ae260f8ae01b93f8ea935458b836d73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuoliHuang/scPAFA
  tools:
  - AUCell
  - AddModuleScore
  - Python
  - Scanpy
  - UCell
  - scPAFA
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# pathway-activity-score-computation-benchmarking

## Summary

Benchmarks the runtime and peak-memory cost of computing pathway activity scores (PAS) with scPAFA's fast_ucell and fast_score_genes against UCell, AUCell and Scanpy score_genes on large single-cell datasets with >10^3 pathways.

## When to use

Use when comparing computational efficiency of PAS scoring methods (fast_ucell, fast_score_genes vs UCell, AUCell, score_genes) across datasets scaling from 10,000 cells to over 1.2 million cells.

## When NOT to use

- The goal is to produce PAS for biological interpretation (e.g. multicellular pathway modules via MOFA) rather than to measure computational performance
- The dataset has few cells (<10^3) or few pathways (<10^3), where runtime differences between methods are negligible and benchmarking adds no decision value
- Input is not a cell x gene expression matrix with a pathway gene-set dictionary, so no PAS scoring comparison can be constructed

## Inputs

- Single-cell gene expression matrices for the CRC and lupus benchmark datasets (up to 1,263,676 cells)
- Pathway dictionaries from MsigDB (1629 CRC pathways; C5 collection with 14,300 pathways) and NCATS BioPlanet (1383 lupus pathways)
- Downsampled cell-count series from 10,000 to 1,263,676 cells

## Outputs

- Per-combination runtime and peak-memory measurements for every method-dataset-cell-count combination
- Aggregated benchmark table written to runtime_memory_benchmark.csv

## How to apply

First construct a pathway dictionary (Python dict: pathway name -> gene list) from MsigDB or NCATS BioPlanet, matching the article's scale (1629 pathways for CRC, 1383 for the lupus atlas). Downsample the single-cell expression matrices at cell counts spanning 10k to 1,263,676 cells, then compute PAS with fast_ucell and fast_score_genes using 10 cores. Run the baselines under matched resources: UCell and AUCell with 10 cores, Scanpy score_genes with 1 core. To probe scaling with pathway number, rerun fast_ucell and fast_score_genes with 20 cores on an expanded C5 (MsigDB) collection of 14,300 pathways. Record elapsed runtime and peak memory for every method-dataset-cell-count combination and aggregate into runtime_memory_benchmark.csv. Judge success by the paper's reported envelope: fast_ucell ~27.05 min and fast_score_genes ~29.9 min on the full lupus dataset (1,263,676 cells, 1383 pathways), versus UCell 21.4 h, score_genes 9.3 h and AUCell 5.1 h.

## Related tools

- **scPAFA** (Provides fast_ucell and fast_score_genes, the computationally efficient PAS implementations being benchmarked (run with 10 cores; 20 cores on the 14,300-pathway C5 expansion)) — https://github.com/ZhuoliHuang/scPAFA
- **UCell** (Baseline PAS method run with 10 cores; reference for verifying that fast_ucell returns identical PAS)
- **AUCell** (Baseline PAS method run with 10 cores; related to fast_ucell PAS when the same max rank threshold is set)
- **AddModuleScore** (Original module-scoring method whose Scanpy equivalent 'score_genes' (run with 1 core) serves as the slow baseline that fast_score_genes accelerates)
- **Scanpy** (Supplies the score_genes implementation used as the AddModuleScore baseline in the benchmark)

## Examples

```
conda activate scPAFA_env; pip install scPAFA
```

## Evaluation signals

- On the complete lupus dataset (1,263,676 cells, 1383 pathways) with 10 cores, fast_ucell completes in ~27 min and fast_score_genes in ~30 min, consistent with the paper's 30-minute claim
- Observed speedups fall in the reported ranges: ~3.8-47.4x versus the original UCell/score_genes implementations and ~4.4-11.4x versus AUCell
- fast_ucell output must reproduce UCell's PAS values exactly (same max rank threshold), not merely be fast
- runtime_memory_benchmark.csv contains one row for every method-dataset-cell-count combination across the 10k-1.26M cell downsampling series
- The 14,300-pathway C5 scaling run (fast_ucell and fast_score_genes, 20 cores) completes and is included in the aggregated table

## Limitations

- PAS itself is not the end product here; downstream multicellular module identification via MOFA requires at least 15 samples (not cells)
- fast_ucell's ranking-based PAS is insensitive to dataset composition, so runtime benchmarking does not assess biological sensitivity differences between rank-based and non-rank-based scorers
- Using a redundant or biologically misaligned pathway collection changes both the result complexity and the measured computational cost, so pathway-set choice confounds the benchmark

## Evidence

- [abstract] scPAFA can achieve over 40-fold reductions in the runtime of PAS computation: "scPAFA can achieve over 40-fold reductions in the runtime of PAS computation"
- [intro] PAS for 1383 pathways in million-cell data within 30 min: "which can compute PAS for 1383 pathways in million-cell-level scRNA-seq data within 30 min"
- [discussion] fast_ucell provides the same PAS of UCell at the same max rank threshold: "The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold."
- [readme] fast_Ucell and fast_score_genes are efficient reimplementations of UCell and score_genes: "These functions are more computationally efficient implementation of UCell and AddModuleScore (also known as ‘score_genes’ in Scanpy)"
- [readme] Pathway dictionaries come from MsigDB or NCATS BioPlanet: "Pathway collection can be downloaded from [MsigDB](https://www.gsea-msigdb.org/gsea/msigdb/human/collections.jsp#H) ('JSON bundle' is recommended)"
