---
name: background-corrected-permutation-pvalue-estimation
description: Use when per-cell TRS or module scores (e.g., Seurat AddModuleScore output)
  need statistically calibrated significance, because raw module scores lack a null
  model and cannot be thresholded (e.g., q<0.05) for calling trait-relevant cells.
when_to_use_negative:
- Input is already a calibrated per-cell p-value or q-value vector from scDRS (which
  uses its own MC null via control gene sets) — re-running this correction would double-correct
- The gene set is not trait-relevant (no gPAS-derived scGet_PCC prioritization was
  performed), so background correction would only calibrate an uninterpretable module
  score
- Cell-type-level (not single-cell) association testing is the goal — scPagwas uses
  block bootstrap (200 iterations) over pathway-based genome blocks for that instead
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_2269
tools:
- name: scPagwas
  role: Implements the MC background-correction via Get_CorrectBg_p; runs the full
    polygenic regression pipeline (gPAS estimation, gene prioritization, TRS scoring)
    whose output the correction calibrates
  repo: https://github.com/dengchunyu/scPagwas
- name: Seurat AddModuleScore function
  role: Computes the per-cell TRS and the expression-bin-matched control gene sets
    that define the empirical null distribution
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:25fae2afdf27d376f6f9624c8f63726987aae273cd541b917897cfcd9f546683
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dengchunyu/scPagwas
  repo_url: https://github.com/dengchunyu/scPagwas
  tools:
  - MAGMA
  - Seurat AddModuleScore function
  - scPagwas
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# background-corrected-permutation-pvalue-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A Monte Carlo significance-assessment step in scPagwas that converts raw per-cell trait relevance scores (TRS) into background-corrected p-values by comparing each cell's TRS to an empirical null distribution of control TRSs, generated from matched control gene sets over 100 iterations.

## When to use

Use when per-cell TRS or module scores (e.g., Seurat AddModuleScore output) need statistically calibrated significance, because raw module scores lack a null model and cannot be thresholded (e.g., q<0.05) for calling trait-relevant cells.

## When NOT to use

- Input is already a calibrated per-cell p-value or q-value vector from scDRS (which uses its own MC null via control gene sets) — re-running this correction would double-correct
- The gene set is not trait-relevant (no gPAS-derived scGet_PCC prioritization was performed), so background correction would only calibrate an uninterpretable module score
- Cell-type-level (not single-cell) association testing is the goal — scPagwas uses block bootstrap (200 iterations) over pathway-based genome blocks for that instead

## Inputs

- Per-cell TRS computed with Seurat AddModuleScore on the scPagwas-prioritized top 1,000 trait-relevant genes (e.g., from monocyte count GWAS, IEU ID ieu-b-31)
- Control gene sets matched to the target gene set by Seurat expression bins (generated automatically by AddModuleScore)
- Seurat scRNA-seq object with normalized RNA assay (e.g., BMMC scRNA-seq with monocyte, T, B, DC, NK cells)
- Iteration count parameter (iters_singlecell = 100 by default in the BMMC benchmark)

## Outputs

- Background-corrected per-cell p-values for the TRS (Get_CorrectBg_p output over 100 iterations)
- Empirical null distribution of control TRSs per cell
- Binary/vector call of trait-relevant cells at the q<0.05 threshold, used to compute retrieval precision against ground-truth labels

## How to apply

First compute the per-cell trait relevance score (TRS) using Seurat's AddModuleScore with the top 1,000 trait-relevant genes prioritized by Pearson correlation of gene expression with summed gPAS scores (scGet_PCC). Then construct control TRSs: for each of iters_singlecell=100 iterations, score the same cells with control gene sets matched to the target set by expression bins, yielding an empirical null distribution per cell. Apply a Monte Carlo (MC) test comparing the observed TRS to this empirical distribution of control TRSs to obtain per-cell p-values, implemented via Get_CorrectBg_p, and optionally convert to q-values (positive cells defined at q<0.05). This background correction is essential because sparse, noisy single-cell expression data shift module scores across cells with different expression baselines; the matched control sets remove that baseline so significance reflects trait-gene enrichment rather than expression level.

## Related tools

- **scPagwas** (Implements the MC background-correction via Get_CorrectBg_p; runs the full polygenic regression pipeline (gPAS estimation, gene prioritization, TRS scoring) whose output the correction calibrates) — https://github.com/dengchunyu/scPagwas
- **Seurat AddModuleScore function** (Computes the per-cell TRS and the expression-bin-matched control gene sets that define the empirical null distribution) — https://github.com/satijalab/seurat

## Examples

```
Pagwas_data <- scPagwas_main2(Pagwas = NULL, gwas_data = system.file("extdata", "GWAS_summ_example.txt", package = "scPagwas"), Single_data = system.file("extdata", "scRNAexample.rds", package = "scPagwas"), singlecell = TRUE, iters_singlecell = 100)
```

## Evaluation signals

- On the real BMMC ground-truth dataset, TRS-based precision of monocyte count-relevant cells should reproduce approximately 98.2% (Figure 3F) after applying q<0.05
- Running Get_CorrectBg_p with fewer iterations (e.g., iters_singlecell=10 in the quick-start) should yield p-values directionally consistent with the 100-iteration default, with coarser resolution (minimum p ~ 1/n_iter)
- Per-cell p-values should be approximately uniform for random/control gene sets and skewed toward significance for the scPagwas top-1,000 gene set
- Corrected p-values should remove the expression-level confound: cells with high baseline expression should no longer be spuriously called significant

## Limitations

- The default TRS relies on Seurat's AddModuleScore purely for Seurat compatibility, so the null distribution inherits that method's expression-bin matching behavior
- Significance is statistical association, not causality: identification of a statistical association between traits and individual cells does not imply causality
- Monte Carlo p-value resolution is bounded by the iteration count (iters_singlecell); fewer iterations coarsen the minimum attainable p-value

## Evidence

- [discussion] MC null against control TRSs: "we employ a MC method22 to determine the statistical significance of individual cells by comparing the scPagwas TRS to the empirical distribution of control TRSs"
- [discussion] TRS computed via Seurat AddModuleScore with matched control sets: "The AddModuleScore cell-scoring method in Seurat44 is employed to calculate the TRS with default parameters."
- [intro] Per-cell inference motivation (sparsity/noise): "The top genetic association signals at specific loci may be absent from most cells because of the extensive sparsity and technical noise in single-cell data."
- [readme] Concrete pipeline invocation with iteration parameter: "iters_singlecell = 10,
                     chrom_ld = chrom_ld,# The LD data is provided by package."
