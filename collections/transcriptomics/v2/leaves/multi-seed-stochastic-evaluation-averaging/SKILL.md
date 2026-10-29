---
name: multi-seed-stochastic-evaluation-averaging
description: Use when benchmarking stochastic batch-correction/integration methods
  whose embeddings or scores vary run-to-run, and you need seed-averaged metrics (e.g.
  over-correction scores) comparable to a reference figure such as Fig. 3c.
when_to_use_negative:
- When the method being benchmarked is deterministic (e.g. fully reproducible runs
  give identical embeddings), seed averaging adds no information.
- When the evaluation metric is computed on a fixed pre-computed embedding rather
  than on stochastic integration runs you execute yourself.
- When comparing methods you have tuned per-method rather than run with default parameters
  — seed averaging will not fix a comparability bias from unequal tuning.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scCobra
  role: Primary stochastic integration method evaluated over 5 seeds alongside benchmarks
  repo: https://github.com/mcgilldinglab/scCobra
- name: Seurat
  role: Benchmarking method run with default parameters across the 5 seeds
- name: Harmony
  role: Benchmarking method run with default parameters across the 5 seeds
- name: Scanorama
  role: Benchmarking method run with default parameters across the 5 seeds
- name: scVI
  role: Benchmarking method (scvi-tools v0.11) run with default parameters across
    the 5 seeds
- name: scDML
  role: Benchmarking method run with default parameters across the 5 seeds
- name: scDREAMER
  role: Benchmarking method run with default parameters across the 5 seeds
- name: Scanpy
  role: Preprocessing and UMAP embedding generation used for score computation
- name: scIB
  role: Evaluation pipeline used for downstream clustering-based assessment across
    resolutions 0.1–2.0
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:f22705523a3d42ff32684ea6c1c4152b03fea6b6ea4bcf48c8db03570c0ff248
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scCobra
  tools:
  - Harmony
  - Scanorama
  - Scanpy
  - Seurat
  - scCobra
  - scDML
  - scDREAMER
  - scIB
  - scVI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# multi-seed-stochastic-evaluation-averaging

## Summary

Protocol for benchmarking stochastic single-cell integration methods (scCobra, Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) by repeating each run over 5 random seeds, computing over-correction scores per seed from 2D UMAP embeddings, then normalizing and averaging across seeds to obtain stable scores comparable to published values (e.g. scCobra 0.043 vs scVI 0.086 on the perturbed GSE149614 immune dataset).

## When to use

Use when benchmarking stochastic batch-correction/integration methods whose embeddings or scores vary run-to-run, and you need seed-averaged metrics (e.g. over-correction scores) comparable to a reference figure such as Fig. 3c.

## When NOT to use

- When the method being benchmarked is deterministic (e.g. fully reproducible runs give identical embeddings), seed averaging adds no information.
- When the evaluation metric is computed on a fixed pre-computed embedding rather than on stochastic integration runs you execute yourself.
- When comparing methods you have tuned per-method rather than run with default parameters — seed averaging will not fix a comparability bias from unequal tuning.

## Inputs

- Preprocessed scRNA-seq dataset (e.g. GSE149614 immune dataset, 33506 cells, 16 cell types, 10 batches; optionally perturbed by +1 expression of 'Defense response to Virus' GOBP genes in CD4+ T cells of the '10X' batch)
- Per-seed 2D UMAP embeddings from each integrated output (scCobra, Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER)
- Seed list (5 random seeds per method)

## Outputs

- overcorrection_scores.csv with normalized, seed-averaged over-correction scores per method (all cell types and CD4+ T cell subset)
- Per-seed over-correction scores before averaging (for variance inspection)

## How to apply

Preprocess the dataset per the paper (filter cells with <200 expressed genes, genes in <3 cells, mitochondrial fraction >5%; normalize to 10,000 counts/cell with log1p; select 2000 highly variable genes via Scanpy). Run each method — scCobra plus the six benchmarking methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) with default parameters — independently over 5 random seeds on identical input. For each seed, compute the over-correction score from the two-dimensional UMAP embedding of the integrated output following the SCALEX methodology, both for all cell types and for the biologically relevant subset (here CD4+ T cells carrying the implanted 'Defense response to Virus' perturbation). Normalize the scores across methods, then average across the 5 seeds and report the mean per method. Rationale: single stochastic runs can inflate or deflate over-correction scores, so seed averaging plus subset-level reporting distinguishes genuine signal retention (low over-correction on the perturbed CD4+ T cell subset) from batch-effect removal artifacts.

## Related tools

- **scCobra** (Primary stochastic integration method evaluated over 5 seeds alongside benchmarks) — https://github.com/mcgilldinglab/scCobra
- **Seurat** (Benchmarking method run with default parameters across the 5 seeds)
- **Harmony** (Benchmarking method run with default parameters across the 5 seeds)
- **Scanorama** (Benchmarking method run with default parameters across the 5 seeds)
- **scVI** (Benchmarking method (scvi-tools v0.11) run with default parameters across the 5 seeds)
- **scDML** (Benchmarking method run with default parameters across the 5 seeds)
- **scDREAMER** (Benchmarking method run with default parameters across the 5 seeds)
- **Scanpy** (Preprocessing and UMAP embedding generation used for score computation)
- **scIB** (Evaluation pipeline used for downstream clustering-based assessment across resolutions 0.1–2.0)

## Examples

```
avg = pd.concat(seed_runs).groupby(['method','celltype']).score.mean(); avg.to_csv('overcorrection_scores.csv')
```

## Evaluation signals

- Seed-averaged normalized over-correction scores match Fig. 3c reference values within tolerance: scCobra ≈ 0.043 and Scanorama ≈ 0.012 (all cell types); scCobra ≈ 0.038, scVI ≈ 0.158, scDML ≈ 0.278 (CD4+ T cell subset).
- Every method contributes exactly 5 per-seed runs before averaging; no seed is dropped silently.
- Scores are computed from two-dimensional UMAP embeddings following the SCALEX methodology, for both all cell types and the CD4+ T cell subset.
- scCobra exhibits the lowest over-correction among the seven methods on the perturbed dataset, confirming successful reproduction of the paper's claim.
- The output table overcorrection_scores.csv contains one normalized score per method per cell-type scope, averaged over the 5 seeds.

## Limitations

- Over-correction scores are derived from two-dimensional UMAP embeddings, introducing dependence on the UMAP projection and the SCALEX methodology.
- Benchmarking methods were run with default parameters, which may not reflect their tuned performance.
- Per-seed stochastic variation means individual runs may deviate from the reported seed-averaged values; conclusions should rely on the averaged table.

## Evidence

- [other] Run scCobra on the perturbed data and the six benchmark methods with default parameters, repeating each method over 5 random seeds.: [evidence span withheld — anchor status: not_found]
- [discussion] we used Seurat, Harmony, Scanorama, scVI, scDML, and scDREAMER as benchmarking methods with default parameters: "we used Seurat, Harmony, Scanorama, scVI, scDML, and scDREAMER as benchmarking methods with default parameters"
- [discussion] The scores were derived from two-dimensional UMAP embeddings, following the methodology described in the original SCALEX paper: "The scores were derived from two-dimensional UMAP embeddings, following the methodology described in the original SCALEX26 paper"
- [other] Normalize the scores and average across the 5 seeds, then write the results table with the expected values: [evidence span withheld — anchor status: not_found]
