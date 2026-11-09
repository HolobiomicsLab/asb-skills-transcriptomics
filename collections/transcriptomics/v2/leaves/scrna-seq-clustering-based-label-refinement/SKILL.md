---
name: scrna-seq-clustering-based-label-refinement
description: Use when partial cell type annotations (e.g., scGate multi-classifier
  output) cover only a subset of cells in an integrated TIL map and complete labels
  are needed before a second semi-supervised STACAS integration round.
when_to_use_negative:
- When cell type annotations are already complete and trustworthy for every cell —
  re-clustering and transfer can inject noise rather than refine
- When no intermediate integrated embedding exists (labels transferred in uncorrected
  space will follow batch structure, not biology)
- When the cell type of interest is present in only one dataset, where CiLISI-type
  evaluation and cross-sample label transfer are not meaningful
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STACAS
  role: Provides annotate.by.neighbors for neighbor-based label transfer in the integrated
    embedding, and the round-1 semi-supervised integration space being clustered
  repo: https://github.com/carmonalab/STACAS
- name: scGate_models
  role: Supplies the partial CD8_TIL gating-model labels that serve as the seed annotations
    for refinement
  repo: https://github.com/carmonalab/scGate_models
- name: Seurat v4
  role: Hosts the Seurat object, clustering of the round-1 integrated embedding, and
    metadata column manipulation for refined labels
- name: scIntegrationMetrics
  role: Computes celltype ASW and CiLISI to verify that refined labels and round-2
    integration improve biological variance preservation
  repo: https://github.com/carmonalab/scIntegrationMetrics
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:31e0f5d22c3e9789f18bad2088ad3b9a640cf5491ed3613c3e0a02cd20850dda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/STACAS
  tools:
  - Harmony
  - STACAS
  - Seurat v4
  - scGate_models
  - scIntegrationMetrics
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# scrna-seq-clustering-based-label-refinement

## Summary

Refine partial scGate-derived cell type labels by clustering an intermediate semi-supervised STACAS integrated embedding and propagating labels to unannotated cells with annotate.by.neighbors, producing the refined labels used as prior information for the second ssSTACAS round.

## When to use

Use when partial cell type annotations (e.g., scGate multi-classifier output) cover only a subset of cells in an integrated TIL map and complete labels are needed before a second semi-supervised STACAS integration round.

## When NOT to use

- When cell type annotations are already complete and trustworthy for every cell — re-clustering and transfer can inject noise rather than refine
- When no intermediate integrated embedding exists (labels transferred in uncorrected space will follow batch structure, not biology)
- When the cell type of interest is present in only one dataset, where CiLISI-type evaluation and cross-sample label transfer are not meaningful

## Inputs

- Seurat object with round-1 semi-supervised STACAS integrated embedding (dims=50, 800 HVGs)
- Partial cell type annotations from scGate CD8_TIL gating models (e.g., scGate_multi labels)
- Set of reference cells carrying confident labels for neighbor-based transfer

## Outputs

- Seurat object metadata column with refined, near-complete cell type labels (labels.col)
- Label-refined Seurat object ready for the second ssSTACAS integration round
- Input to celltype_ASW and CiLISI quality assessment via scIntegrationMetrics

## How to apply

After the first semi-supervised STACAS round (Run.STACAS/FindAnchors.STACAS with semisupervised=TRUE using scGate CD8_TIL labels), cluster cells in the round-1 integrated embedding and treat clusters as neighborhoods from which labels can be transferred. Use the STACAS annotate.by.neighbors function with a reference set of labeled cells, the integrated reduction (e.g., the STACAS embedding), a chosen number of dimensions and k neighbors, and a background pseudocount to smooth label probabilities; write refined labels into a metadata column (labels.col). The rationale is grounded in the paper's demonstration that even partial and imperfect annotations preserve biological variance while correcting batch effects — so label transfer from confident cells through the integrated space extends coverage without new manual curation. Feed the refined labels into the second ssSTACAS round on 800 highly variable genes and 50 principal components, then judge success by improved celltype ASW (and CiLISI) relative to unsupervised STACAS as computed with the scib-pipeline/scIntegrationMetrics.

## Related tools

- **STACAS** (Provides annotate.by.neighbors for neighbor-based label transfer in the integrated embedding, and the round-1 semi-supervised integration space being clustered) — https://github.com/carmonalab/STACAS
- **scGate_models** (Supplies the partial CD8_TIL gating-model labels that serve as the seed annotations for refinement) — https://github.com/carmonalab/scGate_models
- **Seurat v4** (Hosts the Seurat object, clustering of the round-1 integrated embedding, and metadata column manipulation for refined labels)
- **scIntegrationMetrics** (Computes celltype ASW and CiLISI to verify that refined labels and round-2 integration improve biological variance preservation) — https://github.com/carmonalab/scIntegrationMetrics

## Examples

```
obj <- annotate.by.neighbors(obj, ref.cells = labeled.cells, reduction = "STACAS", ndim = 50, labels.col = "refined_labels")
```

## Evaluation signals

- Round-2 ssSTACAS using the refined labels achieves higher celltype ASW than unsupervised STACAS without degrading CiLISI batch mixing
- Coverage of the labels.col metadata column increases substantially after annotate.by.neighbors (most of the 11,021 CD8 T cells receive a label)
- Refined labels are coherent with cluster structure of the round-1 integrated embedding (clusters are near-homogeneous in transferred label)
- scGate seed labels are preserved: cells already annotated by scGate largely retain their original label after transfer
- Expected benchmark pattern holds: Harmony shows high batch mixing but low celltype ASW relative to ssSTACAS

## Limitations

- CiLISI and similar cell type-aware metrics require cell type annotations, so refinement quality directly bounds evaluation quality
- CiLISI cannot be calculated for cell types present in only one dataset, and remains suboptimal when datasets have highly unequal numbers of cells
- Partial and imperfect seed annotations propagate through refinement; the method is robust to such labels but noisy priors can bias transferred labels

## Evidence

- [methods] annotate.by.neighbors signature: "annotate.by.neighbors(obj, ref.cells, reduction, ndim, k, ncores, bg.pseudocount, labels.col)"
- [readme] scGate labels as prior knowledge in atlas construction: [evidence span withheld — anchor status: not_found]
- [discussion] partial/imperfect annotations beneficial: "even partial and imperfect annotations can be beneficial towards preserving biological variance while correcting for technical batch effects"
