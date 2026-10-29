---
name: noisy-celltype-label-perturbation
description: Use when benchmarking supervised/semi-supervised single-cell integration
  methods that consume cell type labels, and you must quantify degradation under incomplete
  or imprecise annotations rather than only testing on curated ground truth.
when_to_use_negative:
- When the integration methods under test do not consume cell type labels (e.g., unsupervised
  Harmony, FastMNN, Seurat v4, scVI, Scanorama baselines) — label perturbation has
  no effect on their behavior.
- When cell type annotations are missing entirely or too incomplete for CiLISI, since
  the evaluation metric requires cell type annotations and cannot be computed for
  cell types present in only one dataset.
- When the goal is final atlas construction with high-quality curated labels (e.g.,
  the CD8 TIL atlas) rather than robustness benchmarking — do not corrupt the labels
  used for semi-supervised integration.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STACAS
  role: Semi-supervised integration method under test; run via the anchor-finding
    pipeline with semisupervised mode enabled in IntegrateData.STACAS on each corrupted
    label variant
  repo: github.com/carmonalab/STACAS
- name: scGen
  role: Supervised integration method under test; baseline expected to drop sharply
    in performance with 10–20% shuffled labels
- name: scANVI
  role: Supervised integration method under test; robustness comparison target for
    ssSTACAS on celltype ASW
- name: scib-pipeline
  role: Snakemake benchmarking pipeline (based on theislab/scib) in which the corrupted-label
    variants are run and 50-dim latent embeddings are extracted
  repo: carmonalab/scib-pipeline
- name: scIntegrationMetrics
  role: R package used to compute the evaluation metrics CiLISI (cell-type-aware batch
    mixing) and celltype_ASW against the TRUE uncorrupted labels
  repo: carmonalab/scIntegrationMetrics
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:91606aa785adfec42ff7d1ab27c55df45b8977ce94571e4eb524d09054bfa357
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastMNN
  - Harmony
  - STACAS
  - Scanorama
  - Seurat v4
  - scANVI
  - scGen
  - scIntegrationMetrics
  - scVI
  - scib-pipeline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# noisy-celltype-label-perturbation

## Summary

A benchmarking procedure from Andreatta et al. (Nat Commun 2024) that deliberately corrupts input cell type annotations — masking a fraction as 'unknown' and shuffling a fraction of the rest — to measure how robust semi-supervised integration methods (ssSTACAS, scANVI, scGen) are to noisy labels.

## When to use

Use when benchmarking supervised/semi-supervised single-cell integration methods that consume cell type labels, and you must quantify degradation under incomplete or imprecise annotations rather than only testing on curated ground truth.

## When NOT to use

- When the integration methods under test do not consume cell type labels (e.g., unsupervised Harmony, FastMNN, Seurat v4, scVI, Scanorama baselines) — label perturbation has no effect on their behavior.
- When cell type annotations are missing entirely or too incomplete for CiLISI, since the evaluation metric requires cell type annotations and cannot be computed for cell types present in only one dataset.
- When the goal is final atlas construction with high-quality curated labels (e.g., the CD8 TIL atlas) rather than robustness benchmarking — do not corrupt the labels used for semi-supervised integration.

## Inputs

- scRNA-seq collections with true cell type annotations (Pancreas, Immune, Lung, mouse T cell) as Seurat objects
- Corruption specifications: fraction of labels set to 'unknown' (e.g., 15%) and fraction of labels shuffled (e.g., 20%), swept 0–100%
- scib-pipeline (snakemake) configuration for the 11 integration methods

## Outputs

- Corrupted label variants of each collection (Seurat objects with perturbed metadata columns)
- Per-method celltype_ASW and CiLISI scores per dataset collection and corruption level
- Metric-versus-corruption-level robustness curves per method (Fig. 4-style)
- label_corruption_benchmark_scores.csv and, for the 4-task ranking, integration_benchmark_ranking.csv

## How to apply

Load scRNA-seq collections with true cell type annotations (e.g., Pancreas, Immune, Lung, and mouse T cell sets from figshare DOIs 10.6084/m9.figshare.12420968.v8 and 10.6084/m9.figshare.12478571) into the carmonalab/scib-pipeline (snakemake) environment. Generate corrupted-input variants by setting a fraction of labels to 'unknown' (15% in the default noisy-label setting) and shuffling 20% of the remaining labels, sweeping both corruption fractions from 0% to 100%. Run each supervised method (scGen, scANVI, and STACAS with semisupervised=TRUE in IntegrateData.STACAS) on every corrupted variant, extracting a 50-dimensional latent embedding. Evaluate each integration against the TRUE uncorrupted labels using celltype_ASW (biological variance preservation) and CiLISI (cell-type-aware batch mixing) from carmonalab/scIntegrationMetrics. The rationale is that labels in real-life tasks are partial and imprecise, so benchmarks of label-consuming methods must account for this; the paper's key decision point is checking for a sharp performance drop (scGen degrades with as little as 10–20% shuffled labels) versus robustness (ssSTACAS and scANVI).

## Related tools

- **STACAS** (Semi-supervised integration method under test; run via the anchor-finding pipeline with semisupervised mode enabled in IntegrateData.STACAS on each corrupted label variant) — github.com/carmonalab/STACAS
- **scGen** (Supervised integration method under test; baseline expected to drop sharply in performance with 10–20% shuffled labels)
- **scANVI** (Supervised integration method under test; robustness comparison target for ssSTACAS on celltype ASW)
- **scib-pipeline** (Snakemake benchmarking pipeline (based on theislab/scib) in which the corrupted-label variants are run and 50-dim latent embeddings are extracted) — carmonalab/scib-pipeline
- **scIntegrationMetrics** (R package used to compute the evaluation metrics CiLISI (cell-type-aware batch mixing) and celltype_ASW against the TRUE uncorrupted labels) — carmonalab/scIntegrationMetrics

## Examples

```
IntegrateData.STACAS(anchorset = anchors, new.assay.name = "STACASintegrated", dims = 1:50, semisupervised = TRUE)
```

## Evaluation signals

- Verify the corruption is applied as specified: 15% of labels set to 'unknown' and 20% of remaining labels shuffled in the default setting (fractions swept 0–100% in the robustness sweep).
- Confirm evaluation is performed against the TRUE (uncorrupted) annotations, not the perturbed input labels, using celltype_ASW and CiLISI on the 50-dimensional latent embedding.
- Reproduced curves should show scGen dropping sharply in performance with as little as 10–20% shuffled labels.
- ssSTACAS should attain higher celltype ASW than scANVI across all four tasks; ssSTACAS and scANVI curves should remain robust to increasing label shuffle.
- Scores table should parse as label_corruption_benchmark_scores.csv with per-method, per-task, per-corruption-level rows for both metrics.

## Limitations

- CiLISI and similar cell type-aware metrics require cell type annotations, so they cannot be computed when labels are absent.
- CiLISI cannot be calculated for cell types that are only present in one dataset.
- Like LISI, CiLISI remains suboptimal when datasets have highly unequal numbers of cells; batch mixing metrics robust to lopsided cell proportions are still lacking.
- Even partial and imperfect annotations can still be beneficial towards preserving biological variance while correcting for technical batch effects, so corrupted labels should not be treated as purely harmful input.

## Evidence

- [abstract] STACAS scales well to large datasets and is robust to incomplete and imprecise input cell type labels, which are commonly encountered in real-life integration tasks.: "STACAS scales well to large datasets and is robust to incomplete and imprecise input cell type labels, which are commonly encountered in real-life integration tasks."
- [discussion] Benchmarks of supervised methods should account for noisy labels: "We argue that benchmarks that include supervised integration methods should account for noisy and partial cell-type information"
- [discussion] Partial/imperfect annotations are beneficial: "we showed that even partial and imperfect annotations can be beneficial towards preserving biological variance while correcting for technical batch effects"
- [intro] Label-consuming methods scored highest on biological variance preservation: "in recent benchmarks of single-cell data integration tools, methods that take cell type labels as input showed the highest performance in terms of preservation of biological variance"
- [methods] IntegrateData.STACAS has a semisupervised flag: "`IntegrateData.STACAS(anchorset, new.assay.name, features.to.integrate, dims, k.weight, sample.tree, hclust.method, semisupervised, verbose)`"
- [readme] scib-pipeline focuses on (semi)supervised tools with STACAS and CiLISI: [evidence span withheld — anchor status: external_doc]
- [readme] STACAS README on label robustness: "STACAS scales well to large datasets and is robust to incomplete and imprecise input cell type labels, which are commonly encountered in real-life integration tasks."
