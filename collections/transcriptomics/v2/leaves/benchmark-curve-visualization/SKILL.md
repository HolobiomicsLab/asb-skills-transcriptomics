---
name: benchmark-curve-visualization
description: 'Use when reproducing label-noise robustness curves for (semi)supervised
  integration methods: you have integration scores at 0–100% shuffled or 0–100% ''unknown''
  label corruption and need method-comparison curves per task.'
when_to_use_negative:
- The cell type annotations required by CiLISI are unavailable, since CiLISI and similar
  cell type-aware metrics require cell type annotations and cannot be calculated for
  cell types present in only one dataset
- The goal is to benchmark unsupervised methods under noise-free labels — this skill
  specifically targets robustness to incomplete and shuffled input cell type labels
- Datasets have highly unequal numbers of cells, where CiLISI remains suboptimal as
  a batch-mixing readout
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: STACAS
  role: Semi-supervised integration method whose label-corruption robustness is plotted;
    run via the anchor-finding pipeline with semisupervised mode in IntegrateData.STACAS
  repo: https://github.com/carmonalab/STACAS
- name: scANVI
  role: Supervised integration baseline whose metric-versus-corruption curves are
    compared against ssSTACAS (celltype ASW)
- name: scGen
  role: Supervised integration baseline expected to drop sharply in performance with
    as low as 10–20% shuffled labels
- name: scib-pipeline
  role: Modified snakemake benchmarking pipeline environment in which the four collections
    are loaded, corrupted variants integrated, and metrics computed
  repo: https://github.com/carmonalab/scib-pipeline
- name: scIntegrationMetrics
  role: R package providing CiLISI and per-cell type ASW used to score each integration
    against the true labels
  repo: https://github.com/carmonalab/scIntegrationMetrics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:16f0f55321fd058d129a1d841bf5d5ec5177d752b0533bd8de11e0335a29148d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/STACAS
  tools:
  - STACAS
  - scANVI
  - scGen
  - scIntegrationMetrics
  - scib-pipeline
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# benchmark-curve-visualization

## Summary

Plot metric-versus-corruption-level curves (Fig. 4 style) showing how semi-supervised single-cell integration methods (ssSTACAS, scANVI, scGen) degrade as input cell type labels are increasingly shuffled or masked to 'unknown', quantified with celltype_ASW and CiLISI across four benchmark tasks (Pancreas, Immune, Lung, T cells).

## When to use

Use when reproducing label-noise robustness curves for (semi)supervised integration methods: you have integration scores at 0–100% shuffled or 0–100% 'unknown' label corruption and need method-comparison curves per task.

## When NOT to use

- The cell type annotations required by CiLISI are unavailable, since CiLISI and similar cell type-aware metrics require cell type annotations and cannot be calculated for cell types present in only one dataset
- The goal is to benchmark unsupervised methods under noise-free labels — this skill specifically targets robustness to incomplete and shuffled input cell type labels
- Datasets have highly unequal numbers of cells, where CiLISI remains suboptimal as a batch-mixing readout

## Inputs

- Four benchmark scRNA-seq collections with true cell type annotations (Pancreas, Immune, Lung, T cells)
- Corrupted-label variants of each collection (0–100% shuffled labels; 0–100% labels set to 'unknown')
- Integrated embeddings produced by scGen, scANVI, and semi-supervised STACAS on each variant

## Outputs

- Metric-versus-corruption-level curves (celltype_ASW and CiLISI per method per task), reproducing Fig. 4
- label_corruption_benchmark_scores.csv with all integration scores

## How to apply

Generate corrupted-input variants of each benchmark scRNA-seq collection by shuffling a fraction of true cell type labels from 0% to 100% and, separately, setting a fraction of labels to 'unknown' from 0% to 100%. Run scGen, scANVI, and semi-supervised STACAS (anchor-finding pipeline with semisupervised mode enabled in IntegrateData.STACAS) on each variant, then evaluate every integration against the TRUE (uncorrupted) labels using the scib metrics celltype_ASW (biological variance preservation) and CiLISI (cell-type-aware batch mixing, computed via carmonalab/scIntegrationMetrics). Plot metric-versus-corruption-level curves per method per task and write all scores to label_corruption_benchmark_scores.csv. Correct reproduction is confirmed by the sharp scGen performance drop at as low as 10–20% shuffled labels and by ssSTACAS attaining higher celltype ASW than scANVI across all tasks.

## Related tools

- **STACAS** (Semi-supervised integration method whose label-corruption robustness is plotted; run via the anchor-finding pipeline with semisupervised mode in IntegrateData.STACAS) — https://github.com/carmonalab/STACAS
- **scANVI** (Supervised integration baseline whose metric-versus-corruption curves are compared against ssSTACAS (celltype ASW))
- **scGen** (Supervised integration baseline expected to drop sharply in performance with as low as 10–20% shuffled labels)
- **scib-pipeline** (Modified snakemake benchmarking pipeline environment in which the four collections are loaded, corrupted variants integrated, and metrics computed) — https://github.com/carmonalab/scib-pipeline
- **scIntegrationMetrics** (R package providing CiLISI and per-cell type ASW used to score each integration against the true labels) — https://github.com/carmonalab/scIntegrationMetrics

## Examples

```
IntegrateData.STACAS(anchorset, new.assay.name = "STACASintegrated", semisupervised = TRUE)
```

## Evaluation signals

- scGen's celltype_ASW (and/or CiLISI) curve shows a sharp performance drop already at 10–20% shuffled input labels
- ssSTACAS attains higher celltype ASW than scANVI across all four tasks (Pancreas, Immune, Lung, T cells)
- Every integration is scored against the TRUE (uncorrupted) labels, not against the corrupted inputs
- All scores are written to label_corruption_benchmark_scores.csv with one row per (method, task, corruption type, corruption level, metric)
- Curves cover the full 0–100% range for both corruption modes (shuffling and 'unknown' masking) per method and task

## Limitations

- CiLISI and similar metrics require cell type annotations and cannot be calculated for cell types that are only present in one dataset
- CiLISI remains suboptimal when datasets have highly unequal numbers of cells, and batch mixing metrics robust to lopsided cell proportions are still lacking
- Benchmarks including supervised integration methods should account for noisy and partial cell-type information; conclusions may differ under clean labels

## Evidence

- [abstract] STACAS robustness curves: "STACAS scales well to large datasets and is robust to incomplete and imprecise input cell type labels, which are commonly encountered in real-life integration tasks."
- [discussion] CiLISI annotation requirement: "One obvious limitation of CiLISI and similar metrics is that it requires cell type annotations."
- [readme] Benchmark pipeline and CiLISI metric: [evidence span withheld — anchor status: not_found]
- [methods] Semi-supervised STACAS integration call: "`IntegrateData.STACAS(anchorset, new.assay.name, features.to.integrate, dims, k.weight, sample.tree, hclust.method, semisupervised, verbose)`"
- [other] Corruption-curve workflow and expected verification: [evidence span withheld — anchor status: not_found]
