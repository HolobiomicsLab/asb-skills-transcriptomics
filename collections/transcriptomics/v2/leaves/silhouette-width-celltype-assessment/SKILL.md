---
name: silhouette-width-celltype-assessment
description: Use when evaluating a batch-corrected/embedded Seurat object (e.g. STACAS
  output) to determine whether cell type structure survives integration, or when demonstrating
  that batch-mixing metrics like iLISI reward overcorrected embeddings.
when_to_use_negative:
- Cell type annotations are absent — CiLISI and similar metrics require cell type
  labels
- Cell types are present in only one dataset — CiLISI cannot be calculated for them
- Datasets have highly unequal cell numbers or lopsided cell proportions across batches
  — LISI/CiLISI are suboptimal and robust batch-mixing metrics are still lacking
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: scIntegrationMetrics
  role: Computes iLISI, CiLISI, per-cell-type ASW, and normalized cLISI on the integrated
    embedding; Seurat-compatible
  repo: https://github.com/carmonalab/scIntegrationMetrics
- name: STACAS
  role: Produces the semi-supervised batch-corrected integrated embedding (Run.STACAS)
    that the metrics are applied to
  repo: https://github.com/carmonalab/STACAS
- name: Splatter
  role: Simulates synthetic scRNA-seq count matrices across batch-effect and differential-expression
    scenarios to make metric behavior interpretable
- name: Seurat v4
  role: Hosts the integrated assay/reductions on which metrics are computed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:db17d99a51779279b131c95cfa0bce4b5e68b8d689b0a0a2112347f78f44a8af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/scIntegrationMetrics
  tools:
  - STACAS
  - Seurat v4
  - Splatter
  - scIntegrationMetrics
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Silhouette-width cell type assessment of integrated single-cell embeddings

## Summary

A benchmarking skill that scores integrated scRNA-seq embeddings with cell type-aware metrics — average silhouette width (ASW), CiLISI, and normalized cLISI from the scIntegrationMetrics R package — to test whether an integration (e.g. STACAS) preserves biological variance while correcting batch effects, as opposed to overcorrecting.

## When to use

Use when evaluating a batch-corrected/embedded Seurat object (e.g. STACAS output) to determine whether cell type structure survives integration, or when demonstrating that batch-mixing metrics like iLISI reward overcorrected embeddings.

## When NOT to use

- Cell type annotations are absent — CiLISI and similar metrics require cell type labels
- Cell types are present in only one dataset — CiLISI cannot be calculated for them
- Datasets have highly unequal cell numbers or lopsided cell proportions across batches — LISI/CiLISI are suboptimal and robust batch-mixing metrics are still lacking

## Inputs

- Seurat object with an integrated embedding/reduction (e.g. STACAS output via Run.STACAS/IntegrateData.STACAS)
- Cell type annotation column in the Seurat metadata (required for CiLISI and ASW-by-cell-type)
- Batch/sample label column in the Seurat metadata (required for iLISI/CiLISI)
- Optionally: Splatter-simulated count matrices parameterized by batch.facLoc/batch.facScale and de.facLoc/de.facScale

## Outputs

- Per-scenario metric table with iLISI, CiLISI, cell type ASW, and normalized cLISI values
- Quantitative diagnosis of overcorrection (high iLISI with degraded ASW/CiLISI) vs variance-preserving correction

## How to apply

Integrate the dataset first (in the reference workflow, simulated Splatter data with 1000 cells, 3 batches, 2 cell types are integrated with STACAS via Run.STACAS). Then compute metrics on the integrated embedding using the scIntegrationMetrics package: iLISI for batch mixing, CiLISI (cell type label mixing) and per-cell-type average silhouette width (ASW) for biological variance preservation, plus normalized cLISI for cross-dataset comparability. Assemble all values into a table keyed by scenario and metric. The rationale is that iLISI alone is cell type-agnostic and rises toward the number of batches even when integration destroys cell type structure (≈1.75 in the batch0 scenario vs ≈2.75 in the overcorrected scenario), whereas cell type-aware ASW and CiLISI score the no-batch-effect cases similarly regardless of biological variance, exposing overcorrection.

## Related tools

- **scIntegrationMetrics** (Computes iLISI, CiLISI, per-cell-type ASW, and normalized cLISI on the integrated embedding; Seurat-compatible) — https://github.com/carmonalab/scIntegrationMetrics
- **STACAS** (Produces the semi-supervised batch-corrected integrated embedding (Run.STACAS) that the metrics are applied to) — https://github.com/carmonalab/STACAS
- **Splatter** (Simulates synthetic scRNA-seq count matrices across batch-effect and differential-expression scenarios to make metric behavior interpretable)
- **Seurat v4** (Hosts the integrated assay/reductions on which metrics are computed)

## Examples

```
integrated <- Run.STACAS(seurat_obj)  # then score the integrated embedding with scIntegrationMetrics (iLISI, CiLISI, ASW)
```

## Evaluation signals

- On simulated no-batch-effect data, iLISI rises from ~1.75 (batch0 scenario) toward ~2.75 in the overcorrected scenario (approaching 3 batches), showing batch-mixing inflation
- CiLISI remains comparable across no-batch-effect scenarios regardless of biological variance, while ASW distinguishes variance-preserving from overcorrected embeddings
- All metric values assemble into a table keyed by scenario and metric without missing entries
- Per-cell-type ASW is finite for every annotated cell type present in more than one dataset
- Metric computation succeeds directly on the Seurat reduction produced by Run.STACAS without format conversion

## Limitations

- CiLISI and similar cell type-aware metrics require cell type annotations
- CiLISI cannot be calculated for cell types that are only present in one dataset
- LISI and CiLISI remain suboptimal when datasets have highly unequal numbers of cells; batch mixing metrics robust to lopsided cell proportions are still lacking
- The utility of cell type-agnostic batch mixing metrics is arguably very limited, and only relevant for integration of technical replicates

## Evidence

- [readme] scIntegrationMetrics provides ASW/CiLISI: [evidence span withheld — anchor status: external_doc]
- [discussion] CiLISI requires annotations: "One obvious limitation of CiLISI and similar metrics is that it requires cell type annotations."
- [discussion] CiLISI single-dataset cell types: "Another limitation is that CiLISI cannot be calculated for cell types that are only present in one dataset."
- [methods] STACAS integration pipeline step: "Wrapper function to run the complete anchor-finding algorithm in STACAS."
- [abstract] STACAS rationale for preserving biological variance: "we present STACAS, a batch correction method for scRNA-seq that leverages prior knowledge on cell types to preserve biological variability upon integration."
